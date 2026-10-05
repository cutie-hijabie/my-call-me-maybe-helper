# 02 — The generation loop

## What it's for

A language model generates **one token at a time**. To get a whole answer you repeat: score the
next token, pick one, append it, score again. Before you constrain anything, make sure you can
run this plain loop yourself.

## Think about

- What is the sequence you feed the model on the *first* step? On the *second*?
- How do you pick "the best" token from a list of scores? What's the simplest operation that
  does it, and do you even need to compute probabilities?
- Greedy selection removes sampling randomness. What else could affect repeatability: model
  configuration, numerical behavior, tied scores, or a different environment?
- When does the loop **stop**? There are at least two different reasons. (Hint: think about
  what you know about the *structure* of the answer, versus a safety net.)
- What is the text you got at the end? How do you get from a list of generated IDs back to a string,
  and which part of the sequence should be decoded — everything, or only what you generated?
- What happens if you never stop? What safety net do you want?

## Go find out

- What is the difference between **greedy decoding**, **sampling**, and **temperature**? Why is
  greedy a useful baseline here? Remember that equivalent outputs or ambiguous requests can exist.
- What is a **KV cache**, and does the provided SDK let you use one? What does that mean for your
  speed budget? (See [09 — Performance](09-performance.md).)

## Keep stopping and success separate

A generation limit is a safety bound, not a successful-completion rule. End-of-text alone
also does not establish that the required record is complete. Once constraints are connected,
explicitly handle the case where no candidate is legal; taking a maximum over entirely blocked
scores can still return an index.

**Checkpoint:** observe completion, truncation, and no-legal-candidate behavior separately.
Confirm that only generated IDs are decoded as the response and state is fresh per request.

## Resources

- [The Illustrated GPT-2](https://jalammar.github.io/illustrated-gpt2/)
- [Hugging Face: how to generate text](https://huggingface.co/blog/how-to-generate) — for the concepts of greedy vs sampling only.

---

[Folder overview](../README.md) · [Build route](../build-route.md) · [Previous guide](01-llm-sdk-and-tokens.md) · [Next guide](03-constrained-decoding.md)
