# 09 — Performance

## What it's for

The subject asks for **all test prompts in under 5 minutes** on standard hardware, with 90%+
correct answers and 100% valid JSON. A correct but slow solution fails; so does a fast but sloppy one.

## Think about

- Where does the time go? Make a list of the expensive operations per prompt, and per generated
  token. Which one do you suspect dominates?
- Inspect the supplied scoring method and its public caching capabilities. Measure how
  sequence length affects cost. What can you do about instruction length and generated length?
- How many tokens do you actually need the model to generate for one answer? Are there stretches
  where there's only one valid continuation? What would you save by not asking the model there —
  and what would you risk? (Remember: the function must still be *chosen by the LLM*.)
- Computing the mask: the vocabulary has well over a hundred thousand entries. If you loop over
  all of them in plain Python at every step, what happens? What could you precompute **once**
  (when the program starts, or when the schema is known) instead of repeating it?
- Loading the model and vocabulary takes time too. How many times should that happen per run?
- Could something you worked out for one prompt be reused for the next? What can't be reused?
- Array operations can reduce Python overhead for score handling. Which operation dominates
  your actual profile, and does vectorizing it improve the complete run?

## Go find out

- How to **measure** instead of guessing: timing blocks of code, and a profiler. What does a
  profile of your program tell you?
- What is **caching**, and what's the risk of caching something that depends on the input?
- Which hardware is the SDK using on your machine (CPU, GPU, other)? How might the reviewer's
  machine differ, and what does that mean for your safety margin?

## Optimize the measured bottleneck

For greedy selection, checking candidates in descending score order until the first legal
candidate is found preserves the selection rule, provided the search can reach every eligible
candidate. A fixed shortlist that omits legal lower-ranked candidates can change behavior.

Known output text may reduce inference work, but any shortcut must preserve state advancement,
escaping, token boundaries, and model-led function/argument decisions. Neither shortcut is a
requirement. Compare the complete run before and after, including correctness.

**Checkpoint:** record setup, model, constraint-checking, and total times. Note hardware,
batch size, and failed requests. Revert an optimization that regresses correctness.

## Resources

- [Python profilers (cProfile)](https://docs.python.org/3/library/profile.html)
- [NumPy docs](https://numpy.org/doc/stable/)

---

[Folder overview](../README.md) · [Build route](../build-route.md) · [Previous guide](08-io-and-errors.md) · [Next guide](10-testing-and-debugging.md)
