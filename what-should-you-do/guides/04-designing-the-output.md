# 04 — Designing the output

## What it's for

The output for each prompt has three parts: the original **prompt**, the function **name**, and
the **parameters**. Two of those need the LLM: *which function?* and *which values?* This guide is
about how to split that into steps, and how the constraints differ between them.

## Think about

**Choosing the function**

- The set of possible function names is **finite and known** before you generate anything. What
  does that imply about which tokens are allowed while the name is being written?
- Suppose you've generated part of a name. Which names are still possible? What does that tell
  you about what may come next?
- Do two function names ever share a beginning? What happens at the point where they diverge?
- How will the model "know" which function fits a prompt? What does it need to be shown?
  (See [06 — Prompt design](06-prompt-design.md).)

**Choosing the arguments**

- Different functions have different parameters. **Which part of the output determines the
  constraints on another part?** How will your design know the selected function when it
  checks the parameters? Distinguish your chosen generation order from JSON object ordering.
- For each parameter, what is its name and its type? Do you let the model write the name, or do
  you already know it?
- Functions can have several parameters. How do you make sure *all* required ones appear, exactly
  once, in a sensible order?
- Do you need the model for every character of the output, or are some parts completely forced
  by the schema? What are the trade-offs (accuracy, speed, simplicity) of letting the model
  "write" the forced parts versus inserting them yourself?

**Putting it together**

- The output format has *exactly* three keys per entry and no extras. Where does the final
  structure get assembled — inside the decoder, or after?
- If a request fails, how will you report it without writing an incompatible result record?
  The subject specifies successful records, not a complete failure-record format. Document
  your policy. Post-generation parsing or retries do not replace constraints before selection.

## Go find out

- Re-read section 5.4 (Output file format) and 5.4.2 (Validation rules) of the subject. Make a
  checklist of every rule, and tick it off against your design.
- What should happen when a prompt doesn't clearly match any function ("ambiguous prompts" are
  explicitly named in the subject's testing section)?

## Preserve the original request

The output's `prompt` field contains the original input request, not the longer model
instruction. Its JSON spelling may contain escapes, but parsing it must restore the original
text exactly. Keep ownership of that text explicit wherever the record is assembled.

**Checkpoint:** use a request with quotes and backslashes. Compare the parsed output value
to the original request and check that exactly the required outer fields are present.

## Resources

- [json.org](https://www.json.org/)

---

[Folder overview](../README.md) · [Build route](../build-route.md) · [Previous guide](03-constrained-decoding.md) · [Next guide](05-schema-and-types.md)
