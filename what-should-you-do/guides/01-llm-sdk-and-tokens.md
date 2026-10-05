# 01 — The LLM SDK and tokens

## What it's for

The subject provides a small wrapper package (`llm_sdk`) around the model. It's your only
interface to the model, and it deliberately exposes **low-level pieces** — token IDs in,
scores out — so that *you* control generation.

## What the SDK gives you

Read the SDK's source and docstrings yourself, but in broad strokes it offers: a way to turn text
into token IDs, a way to turn IDs back into text, a function that returns the model's scores for the
next token, and paths to files describing the tokenizer (including the vocabulary file).

## Think about

- What is the *type* of what the encoding method returns? What type does the scoring
  function expect? What has to happen in between — and how do you do it **without importing
  PyTorch yourself** (the subject forbids it)?
- The scoring function returns scores for **the next token only**. If you want the score for
  the token after that, what must you do?
- How many numbers come back per call? What does each position in that list correspond to?
- The subject forbids using **private** methods and attributes of the SDK. How do you recognize
  a private one in Python?
- Inspect how the supplied scoring method uses the sequence and whether its public interface
  exposes caching. Measure how context length affects runtime; do not assume another SDK's behavior.
- The vocabulary file maps tokens to IDs. Which direction do you need for generation — ID to
  text, text to ID, or both?
- What does a token look like in the vocabulary file? Does it look like real text? (The subject
  mentions a special symbol standing for a leading space.)

## Go find out

- What are **byte-level BPE** tokenizers, and why do their vocabularies contain strange
  characters for spaces and newlines? What mapping turns those strange characters back into
  real text? (Search: *GPT-2 byte-level BPE bytes to unicode*.)
- Can a single character (for example an emoji or an accented letter) be split across several
  tokens? What does a token containing only *part* of a character decode to?
- Compare **the length of the list of scores** with **the number of entries in the vocabulary
  file**. Are they equal? What do you do about score positions that have no entry?
- What are **special tokens** (like end-of-text), and are they in the vocabulary file?
- What do logits look like numerically? Can they be negative? What does a very large negative
  number (or negative infinity) mean when you later pick the "best" one?

## Inspect representation before using it

A vocabulary spelling is not necessarily the text it contributes to the response. Byte-level
tokens may represent only part of a Unicode character, so independently decoding tokens and
joining their strings is not generally equivalent to decoding the whole sequence. Investigate
the public SDK behavior and account for this boundary in your representation.

**Checkpoint:** inspect a plain token, a leading-space token, a structural token, and a
non-ASCII example. Explain any difference between vocabulary spelling and decoded output.

## Resources

- [Hugging Face: tokenizer summary](https://huggingface.co/docs/transformers/tokenizer_summary) (concepts only)
- [Hugging Face Tokenizers: components](https://huggingface.co/docs/tokenizers/components) — byte-level representation and decoding
- [NumPy: argmax](https://numpy.org/doc/stable/reference/generated/numpy.argmax.html) — ranking without computing probabilities
- [NumPy docs](https://numpy.org/doc/stable/) — you'll want vector operations for scores.

---

[Folder overview](../README.md) · [Build route](../build-route.md) · [Previous guide](00-big-picture.md) · [Next guide](02-generation-loop.md)
