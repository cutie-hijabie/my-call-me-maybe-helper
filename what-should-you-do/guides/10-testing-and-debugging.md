# 10 — Testing and debugging

## What it's for

You have no reference implementation and a small model that behaves oddly. Focused tests and real runs help separate defects in your own logic from model behavior.
Test components in isolation, then verify the connected journey.

## Think about

**Test the logic without the model**

- Most of your code (reading files, building the mask, tracking structure, validating output)
  doesn't need the real model at all. How could you test the decoder's rules with a **fake tiny
  vocabulary** and **made-up scores**? What would a test like that assert?
- Fast tests run constantly; a test that loads a 0.6B model takes a while. Which tests need the
  real model, and how few can you get away with?

**Test the behavior**

- The subject's testing section names edge cases: **empty strings, large numbers, special
  characters, wrong types, ambiguous prompts, functions with multiple parameters**. For each
  one, write down the expected outcome *first*.
- The review changes the function set and the prompts. Invent your **own function definitions**
  (different names, different parameters, other types) and prompts. Does everything still work?
- How will you measure the **accuracy** target (90%+)? You need prompts for which you know the
  expected answer. How many? Count function choice and arguments together, and include failed
  requests in the denominator. Accepted equivalence rules should be explicit.
- How do you check every saved output for both JSON and schema compliance? Parsing is one
  check, not proof of the full contract. Python's default parser accepts `NaN`/`Infinity` and
  keeps the last value of a repeated key, so include stricter checks where the contract requires them.

**Debug the decoder**

- When something goes wrong in generation, what do you want to *see*? At a given step: the
  text so far, which tokens were allowed, which was picked, and why?
- If the model picks something unexpected, is it the model's choice, your mask, or your
  parsing of the result? How can you tell the three apart?

## Go find out

- `pytest` or `unittest`: fixtures, parametrized tests, how to organize a tests folder.
- How to use Python's debugger (`pdb`) — the Makefile needs a `debug` rule for it.
- How to measure elapsed time for the whole run and compare to the 5-minute limit.

## Finish each capability with evidence

Run and inspect behavior while implementing, then finish the capability with focused tests
and regressions for changed shared behavior. Keep logic tests fast and separate from real-model
integration runs. A large historical test count does not verify later edits.

**Checkpoint:** report attempted requests, correct function-and-argument records, failures,
structural violations, and whole-run time separately. Use different definitions as well as
different request wording.

## Resources

- [pytest](https://docs.pytest.org/)
- [pdb](https://docs.python.org/3/library/pdb.html)
- [unittest](https://docs.python.org/3/library/unittest.html)
- [Python JSON compliance details](https://docs.python.org/3/library/json.html#standard-compliance-and-interoperability)

---

[Folder overview](../README.md) · [Build route](../build-route.md) · [Previous guide](09-performance.md) · [Next guide](11-project-structure-and-tooling.md)
