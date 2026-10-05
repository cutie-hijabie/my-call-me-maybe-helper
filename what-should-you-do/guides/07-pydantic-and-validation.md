# 07 — Pydantic and validation

## What it's for

The subject requires that **all classes use pydantic for validation**. Pydantic lets you describe
data as Python classes with typed fields, and then checks real data against that description,
raising clear errors when it doesn't fit.

## Think about

- Which pieces of data in this project have a *shape*? (Function definitions, a single
  parameter description, a prompt entry, an output entry…) Which of them deserve a class?
- What fields does each class have? Which are required, which optional?
- Input files come from outside and may be wrong: missing keys, wrong types, empty lists,
  extra junk. What should happen when validation fails? Who sees the error?
- The subject says **all classes**. Do not silently rewrite that requirement as "input models
  only." Consider it while choosing class responsibilities. Keep data validation separate from
  incremental token masking, and identify any compliance question early.
- Can you use pydantic to validate your *own output* before writing it? How does that relate
  to the subject's validation rules?
- How do pydantic models relate to the type hints and `mypy` checks the subject demands?

## Go find out

- The difference between creating a model normally and **validating** raw data into one.
- What a **ValidationError** contains, and how to show it to a user as a readable message instead
  of a stack trace.
- How to add custom checks (validators), and how to forbid or allow extra fields.
- How a model turns back into plain data you can write as JSON.
- Default coercion versus strict validation: would a numeric string be converted when your
  contract requires a number? Check the behavior for the exact model, type, and input mode you use.
- Why validating a final Python object does not enforce legal next tokens during generation.

## Validation and masking have different jobs

Boundary validation checks loaded or completed data. Incremental masking decides which
next token is legal before selection. Document how those responsibilities connect without
assuming one automatically implements the other. This guide does not prescribe particular
state classes or a refactor of an existing implementation.

**Checkpoint:** identify each validation boundary, its error handling, and whether conversion
could hide a wrong input type. Explain how token constraints remain enforced during generation.

## Resources

- [Pydantic docs](https://docs.pydantic.dev/latest/)
- [Strict mode](https://docs.pydantic.dev/latest/concepts/strict_mode/)
- [Conversion table](https://docs.pydantic.dev/latest/concepts/conversion_table/)
- [Python typing module](https://docs.python.org/3/library/typing.html)
- [mypy docs](https://mypy.readthedocs.io/)

---

[Folder overview](../README.md) · [Build route](../build-route.md) · [Previous guide](06-prompt-design.md) · [Next guide](08-io-and-errors.md)
