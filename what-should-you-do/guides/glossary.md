# Glossary

Quick definitions. Where a term matters for a design decision, the relevant guide goes deeper.

- **Token** — a chunk of text the model works with: part of a word, a whole word, punctuation, a
  space plus a word, and so on. Not the same as a character, and not the same as a word.
- **Tokenizer** — the component that turns text into tokens (encode) and back (decode).
- **Token ID** — the integer that stands for a token. The model only ever sees IDs.
- **Vocabulary** — the complete list of tokens the model knows, with their IDs.
- **Logits** — the raw scores the model gives to *every token in the vocabulary* for "what comes
  next". Higher means more likely. They are not probabilities yet.
- **Softmax** — the function that turns logits into probabilities. You often don't need it just to pick the best token.
- **Greedy decoding** — choosing the highest-scoring eligible token at each step.
- **Constrained decoding** — restricting token eligibility before selection according to
  grammar/schema rules. Correct constraints protect accepted prefixes; successful completion
  is still necessary for a final record.
- **Mask** — the "allowed / not allowed" information for every token at one step.
- **Schema** — a description of what valid data looks like (which keys, which types).
- **State machine** — a model of "where am I in the structure right now, and what may come next?".
- **Function calling** — producing a function *name* and *arguments*, rather than the answer itself.
- **Original request** — the input request that must be preserved in the output's `prompt` field.
- **Model instruction/context** — the larger text given to the model, including task instructions,
  definitions, and the request. It is distinct from the output's original-request field.
- **Prompt** — an overloaded term: check whether it means the original request or model context.
- **Pydantic model** — a data model with declared fields and validation rules. Validation can
  include conversion; strictness depends on configuration and the specific type/input mode.
- **Valid prefix** — unfinished text that can still be extended into an allowed complete result.
- **Structural validity** — the completed data follows the syntax and schema.
- **Semantic accuracy** — the function and arguments match the intended request.
- **State machine and stack** — transition rules track local progress; nesting memory tracks
  matching containers where the grammar permits nested values.
- **Moulinette** — 42's automatic grader.

---

[Folder overview](../README.md) · [Build route](../build-route.md)
