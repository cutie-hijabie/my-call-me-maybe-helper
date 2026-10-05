# 06 — Prompt design

## What it's for

Correct constraints enforce syntax and schema on completed output. It doesn't make it **right**. The model still
needs to *understand the task* to pick the right function and extract the right values — and that's
what the prompt is for. The subject says you must not *rely* on prompting alone for structure; it
doesn't say you can't write a good prompt for accuracy.

## Think about

- What does the model need to see, at a minimum, to choose the right function? Names? Descriptions?
  Parameter names? Types? Which of these help the most, and which just add noise (and slow things
  down by making the sequence longer)?
- The model *continues* text. What text should the prompt end with so that the constrained
  output is a natural continuation of it? Where exactly does "your constraint takes over" begin?
- The output is a structure with names and values. How can you make the model *aware* of the
  format it's about to produce without relying on it to produce it?
- How does the model know which part of the sentence is the argument? For "Greet shrek" — what
  is the value? For "Reverse the string 'hello'"? For "Replace all numbers in … with NUMBERS"?
  Does the quoting in the prompt help or hurt?
- Would an example or two help a small model? What are the trade-offs of **few-shot examples**
  (accuracy vs length vs overfitting to the examples)? Careful: the review uses *different*
  functions and prompts.
- Is the prompt the same for every request, or built per request? Which parts are fixed?

## Go find out

- **Qwen3 is a chat model.** What is a **chat template**, and what special markers does Qwen3
  expect around system / user / assistant text? What happens if you ignore them?
- Qwen3 has a "**thinking**" mode. What is it, and how does it affect what the model wants to
  generate first? (Read the model card.)
- Does the encoding function add special tokens by itself? Check the SDK source.

## Evaluate the actual values

Use runtime definitions and the real request rather than example-specific instructions.
Serialize the request safely when embedding it in a format that uses JSON string syntax.
Change one aspect at a time where practical, then compare both values and timing.

**Checkpoint:** a schema-valid call can still be wrong. Inspect the exact extracted string,
number, or expression on unfamiliar examples, and retain failures in the record.

## Resources

- [Qwen/Qwen3-0.6B model card](https://huggingface.co/Qwen/Qwen3-0.6B)
- [Hugging Face: chat templates](https://huggingface.co/docs/transformers/chat_templating) — for the *concept*.
- [Prompt engineering overview (Anthropic docs)](https://docs.claude.com/en/docs/build-with-claude/prompt-engineering/overview)

---

[Folder overview](../README.md) · [Build route](../build-route.md) · [Previous guide](05-schema-and-types.md) · [Next guide](07-pydantic-and-validation.md)
