# 13 — Common pitfalls (as questions)

Each of these is a place where people lose time or points. They're questions on purpose —
answer them by experimenting.

**Tokens and scores**

- Does your code treat a token's *string in the vocabulary file* as if it were *real text*? What
  does a leading-space token look like in the file, versus what it means?
- Is there any position in the score list that has no vocabulary string? What does your mask
  do with it?
- Did you decode only the *generated* part, or the prompt too?
- Are you modifying a list of scores in place that you still need later?

**Masks and structure**

- Is a token allowed because it's valid *on its own*, or because it keeps the *whole output* on a
  valid path? Are those the same?
- What happens at a token that ends one piece of structure and starts another?
- When is a number or a string considered finished? What if the model never ends it?
- What if the mask blocks **every** token? Do you detect that?
- Does your decoder ever depend on the model producing an end-of-text token to finish?

**Types and values**

- What does your output look like for `integer` vs `number` parameters?
- How does a backslash in a regex travel from the model's output to the final file?
- What happens with a quote inside a string value? An empty string? A very long string?
- Are numbers like `0.5`, `-3`, `1e5`, or a huge value handled the way you think?

**Prompting**

- Did you ever test with a function set that is *not* the example one?
- Does the prompt mention things that only exist in the example functions?
- What does your prompt cost, in tokens, on every single request?

**Robustness**

- What does your program do with an empty input list? A prompt that's an empty string? A missing
  key? An unreadable file?
- If one prompt raises an exception, what happens to the others?
- If a result file is written after failures, does it still contain complete valid JSON?
- Are skipped or rejected requests still counted when you report semantic accuracy?

**Project hygiene**

- Does a fresh clone work with just `uv sync` and the run command?
- Does `make lint` pass on the whole repo? Did you rely on a `|| true` that hides failures?
- Is anything generated (output, caches, virtual environment) committed by accident?
- Does every function have type hints and a docstring?
- Does your README contain **every** required section?

## Use failures to locate a boundary

Start with the observed symptom and the state immediately before it. Determine whether the
failure belongs to input, token representation, grammar, schema progress, semantic choice,
stopping, or writing. A wrong answer and an invalid record require different investigations.

**Checkpoint:** explain one bug as symptom, cause, affected boundary, and evidence that the
fix worked. Avoid changing prompting to cover a demonstrated parser defect.

---

[Folder overview](../README.md) · [Build route](../build-route.md) · [Previous guide](12-readme-and-defense.md)
