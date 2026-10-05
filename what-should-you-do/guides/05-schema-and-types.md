# 05 — Schema and types

## What it's for

`functions_definition.json` supplies the runtime function definitions. It tells you each function's name, description,
parameters (with types), and return type. Each **type** has its own rules for what text is
valid — so each needs its own "what may come next?" logic.

## Think about

**Reading the definition**

- What shape does a function definition have? What lives under `parameters`? Where does the type
  of each parameter live?
- Which types actually occur? Look through the example file, and *also* think about the ones the
  subject names ("number, string, boolean, etc."). Will your code survive a type you didn't
  expect? What should it do?
- Does the **return type** matter for producing the call? Why or why not?

**Strings**

- What makes a character illegal *inside* a JSON string? How does a string end?
- A string value may contain quotes, backslashes, unicode… What does valid escaping look like?
  What happens if the model wants to emit a raw quote in the middle of a value?
- Consider a regular expression as an argument: it often contains backslashes. How does it look as
  JSON *text*, versus how does it look once it's *parsed*? Which one is the "real" value?
- Can a string be empty? How does your decoder handle it?
- When the model is writing a string, how do you know it's **done**? The model must close it —
  but what stops it from going on forever?

**Numbers and integers**

- What's the JSON grammar for a number? Which characters may start it? Which may follow?
  (The grammar on json.org is small. Read it.)
- When is a number "finished"? Numbers have no closing delimiter — what token ends one?
- How do you distinguish an integer parameter from a number parameter? JSON has a number
  grammar, while your runtime type contract determines which numeric forms or values qualify.
- Does the output need to keep `2` as an integer and `2.5` as a decimal? Check the subject's
  example and type requirements. Avoid assuming that one example defines every accepted numeric form.
- What about very large numbers, negative numbers, or leading zeros?

**Booleans (and anything else)**

- What are the only two valid texts for a boolean? How is that like the function-name problem?
- If a new type appeared tomorrow (say an array), where in your design would you add support — and
  how many places would you have to touch? (The review may ask for a small change like this.)

## Go find out

- The exact JSON grammar for *numbers* and for *string escapes* on [json.org](https://www.json.org/).
- How does Python's `json` module turn text into numbers and back? Does `1.0` round-trip as an
  integer or a float?
- What is **Unicode escaping** in JSON, and when would the model need it?

## Separate syntax, runtime types, and meaning

A numeric-looking token is not enough to validate a number. Whitespace can end a complete
number, but cannot complete an unfinished exponent. Once a value ends, delimiters must be
handled in the correct state. `NaN` and `Infinity` are not JSON number literals.

Decide which runtime types your subject/definition contract supports. Report unsupported
schemas clearly; do not silently let arbitrary JSON through. Use the loaded definitions for
parameter names and requiredness rather than inferring a richer schema format that is absent.

**Checkpoint:** check a zero-parameter function, missing/duplicate keys, a wrong value type,
an unfinished exponent, and the missing-comma case `[1 2]`.

## Resources

- [json.org](https://www.json.org/)
- [Python `json` module](https://docs.python.org/3/library/json.html)
- [RFC 8259: numbers and strings](https://datatracker.ietf.org/doc/html/rfc8259)

---

[Folder overview](../README.md) · [Build route](../build-route.md) · [Previous guide](04-designing-the-output.md) · [Next guide](06-prompt-design.md)
