# 00 — Big picture

## What it's for

You're building a **translator**: it takes a human sentence
("What is the sum of 40 and 2?") plus a list of available functions, and outputs a
machine-readable call — *which function*, with *which arguments* — **without answering the
question**.

Everything in the project comes from one uncomfortable fact:

> A small model, asked politely for JSON, often fails. So instead of asking, you **control what
> it is allowed to say**, one token at a time.

## The pieces

Think of the program as a pipeline you can build and test one stage at a time:

1. **Read** the function definitions and the prompts (and survive bad input).
2. **Prepare** something the model can continue from.
3. **Generate** the answer token by token — but only from the tokens that keep the output valid.
4. **Parse and validate** what came out.
5. **Write** the results in the exact format required.

Each stage should be something you can explain and test on its own.

## Think about

- What does the subject mean by "the function to call should be chosen using the LLM, not with
  heuristics"? What would count as a heuristic, and why would it defeat the purpose?
- At the public scoring-method boundary, you receive next-token scores. How do those scores
  become generated text, and where do you restrict eligibility?
- Why does requesting JSON in a prompt fail to enforce the grammar before selection?
- The input files change during review: different prompts, *different function sets*. What
  does that rule out about how you design things?
- Which parts of your solution depend on the *specific* functions in the example file? Which
  must not?

## Go find out

- What is the difference between a model that *answers* and a system that does *function
  calling*? Where does the actual computing happen afterwards, and is that your job?
- Roughly how big is the model's vocabulary? You'll want that number in your head.

## Make the responsibilities visible

Separate shared setup from one-request work. Model/vocabulary initialization can be shared;
JSON progress, schema progress, selected-function context, and generated output start fresh
for each request. Draw that boundary before choosing file names.

**Checkpoint:** trace one original request from its input file to a saved result, and name
which component owns each transition. Distinguish known structure from model-led meaning.

## Resources

- [The Illustrated GPT-2](https://jalammar.github.io/illustrated-gpt2/)
- [Qwen/Qwen3-0.6B model card](https://huggingface.co/Qwen/Qwen3-0.6B)

---

[Folder overview](../README.md) · [Build route](../build-route.md) · [Next guide](01-llm-sdk-and-tokens.md)
