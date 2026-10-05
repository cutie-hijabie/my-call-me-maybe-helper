# 03 — Constrained decoding

## What it's for

This is the heart of the project. The subject describes it in four steps: the model produces
scores; you work out which tokens are **valid**; you rule out the others; you choose among what's
left. Correct constraints keep the accepted sequence on a legal path. A **completed** result
can then satisfy the grammar and schema; an unfinished prefix is still not a valid final record.

The hard part isn't the four steps. It's the middle one: *"which tokens are valid right now?"*

## Think about

- "Valid" compared to *what*? You need some description of "where we are" in the output. What
  information would you have to remember to answer "what may come next?" — position in the
  structure, the current key, the current function, whether you're inside a string…?
- **Tokens are not characters.** A single token can be several characters long, and one token can
  cross a structural boundary (for example, end a string *and* start the next part). How do you
  decide whether a token is acceptable *as a whole*? What are you really testing?
- If a token is only valid as the *beginning* of something that could still become valid, should
  it be allowed? Where's the line?
- What should you do to a token's score to make it impossible to pick? Why does that value
  work with the "pick the best" operation?
- What if, at some step, **no** token is valid? Can that happen? How would you prevent it, and
  how would you notice if it did?
- What if exactly **one** token is valid? Do you still need the model's opinion?
- The vocabulary has well over a hundred thousand tokens. Checking all of them every step
  from scratch is expensive. What could you work out *once* and reuse?
- The subject says the mask must enforce **both** JSON validity **and** the schema. What's the
  difference between those two? Give an example of something that is valid JSON but breaks
  the schema.

## Do this on paper first

Invent a **tiny vocabulary** of about ten tokens that can build a very small structure — for
instance one key and one string value. Write the tokens and their IDs. Now pretend to be the
decoder:

- At each step, list which tokens are allowed and which are blocked, and *why*.
- Pretend the model "wants" a blocked token. What happens?
- Add a token that straddles a boundary (say, one that closes a value and opens the next
  piece). Is it allowed? When?
- When do you stop?

The exercise gives you a small case you can reason about. The real implementation still needs
careful handling of nesting, escapes, token boundaries, state advancement, and completion.

## Go find out

- What is a **state machine**, and how could one describe "where am I in the JSON"?
  General nested JSON also needs nesting memory, such as a stack; a finite list of states alone
  cannot track arbitrarily deep matching brackets.
- What is a **prefix**, and what does "this text is a valid *prefix* of the language" mean?
- Read about **structured / guided generation** for LLMs to see how others describe the same
  problem. *Read for ideas, then close the tab and design your own* — the subject forbids
  libraries that do this for you (such as outlines).

## Candidate state is temporary

Evaluate candidates without changing live grammar or schema state. The selected candidate
must then advance live state using the same rules used during simulation. Check every part
of a token contribution, including a boundary crossed halfway through it.

**Checkpoint:** use a tiny vocabulary to reject a token whose first part is legal and later
part is not. Confirm that a rejected candidate leaves both live states unchanged.

## Resources

- [json.org](https://www.json.org/) — the full JSON grammar
- [RFC 8259 — JSON](https://datatracker.ietf.org/doc/html/rfc8259)
- [Efficient Guided Generation for LLMs (paper)](https://arxiv.org/abs/2307.09702) — optional, conceptual background.

---

[Folder overview](../README.md) · [Build route](../build-route.md) · [Previous guide](02-generation-loop.md) · [Next guide](04-designing-the-output.md)
