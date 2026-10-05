# What Should You Do?

This is the approach I would recommend after working through Call Me Maybe, getting stuck, and changing the way I organized the work.

My daily logs record what happened as I learned. This folder gathers the concepts, questions, and checkpoints that would have helped me see the project more clearly from the beginning.

It gives you a route through the project and references to investigate. You still choose your architecture, write your code, and explain why it works.

**Start with the subject supplied to you.** These notes were checked against version 1.2. If your subject or SDK differs, use your actual requirements and public interfaces.

## Start here

1. Read [the big picture](guides/00-big-picture.md) to understand the request journey.
2. Follow [the build route](build-route.md). It connects the guides to something you can build, run, and verify.
3. Use the [glossary](guides/glossary.md) whenever a term is unclear.
4. Return to the relevant guide when you need to investigate a concept or debug a boundary.

The guide numbers organize topics. They are not an instruction to finish every topic in that order before running anything.

If you already have code, map it onto the route. Reuse what works and verify the behavior you are relying on. You do not need to restart your project to use this folder.

## The recommended route

| Working toward | What you should be able to observe | Read alongside it |
| --- | --- | --- |
| Setup and boundaries | Validated inputs and a project environment you can reproduce | [Output ownership](guides/04-designing-the-output.md), [validation](guides/07-pydantic-and-validation.md), [I/O](guides/08-io-and-errors.md), [tooling](guides/11-project-structure-and-tooling.md) |
| A running request | One request reaches the real model and returns readable generated text | [SDK and tokens](guides/01-llm-sdk-and-tokens.md), [generation](guides/02-generation-loop.md) |
| JSON constraints | Completed output parses; illegal token candidates are rejected before selection | [Constrained decoding](guides/03-constrained-decoding.md) |
| Runtime schema constraints | Records use the required fields and the loaded function/parameter definitions | [Output design](guides/04-designing-the-output.md), [schema and types](guides/05-schema-and-types.md) |
| Meaning and speed | Correct function/argument choices measured separately from validity and runtime | [Prompt design](guides/06-prompt-design.md), [performance](guides/09-performance.md) |
| Complete batch behavior | Results are saved and failures have deliberate, visible outcomes | [I/O and errors](guides/08-io-and-errors.md) |
| Review readiness | A clean checkout runs, required checks pass, and your explanation matches your code | [Testing](guides/10-testing-and-debugging.md), [tooling](guides/11-project-structure-and-tooling.md), [README and review](guides/12-readme-and-defense.md) |

For the detailed goals, decisions, observations, and checks, use [the build route](build-route.md).

## How to work through a section

Understand the goal, identify the relevant files, make the necessary decisions, implement a useful piece of behavior, and run it. Finish that section with focused verification and regressions for anything you changed.

Tests without the real model help isolate your own logic. Real-model runs show integration and semantic behavior. You need both kinds of evidence; neither replaces the other.

Keep a short decision record: what you chose, why, whether it comes from the subject or your own design, and what evidence would make you revisit it.

When a run fails, locate the boundary: input, model access, token representation, JSON grammar, schema progress, semantic choice, stopping, or output writing. Fix the demonstrated problem before expanding the scope.

## Topic guides

| Guide | Focus |
| --- | --- |
| [00 — Big picture](guides/00-big-picture.md) | The complete journey and its responsibilities |
| [01 — LLM SDK and tokens](guides/01-llm-sdk-and-tokens.md) | Public model access and token representation |
| [02 — Generation loop](guides/02-generation-loop.md) | Selection, advancement, and stopping |
| [03 — Constrained decoding](guides/03-constrained-decoding.md) | Candidate legality before selection |
| [04 — Designing the output](guides/04-designing-the-output.md) | Record ownership and dynamic function choices |
| [05 — Schema and types](guides/05-schema-and-types.md) | Parameter names, value types, and completion |
| [06 — Prompt design](guides/06-prompt-design.md) | Context for semantic accuracy |
| [07 — Pydantic and validation](guides/07-pydantic-and-validation.md) | Input/output boundaries and validation behavior |
| [08 — Input, output, and errors](guides/08-io-and-errors.md) | Files, CLI, batch results, and failure policy |
| [09 — Performance](guides/09-performance.md) | Measurements and optimization trade-offs |
| [10 — Testing and debugging](guides/10-testing-and-debugging.md) | Evidence for logic, integration, and meaning |
| [11 — Project structure and tooling](guides/11-project-structure-and-tooling.md) | Dependencies, required commands, and repository contents |
| [12 — README and review](guides/12-readme-and-defense.md) | Documentation, explanations, and small changes |
| [13 — Common pitfalls](guides/13-common-pitfalls.md) | Questions to revisit when something goes wrong |
| [Glossary](guides/glossary.md) | Terms used throughout the folder |

## Requirements and choices

Runtime definitions, model-led function selection, constraints before selection, the output contract, and the required tools come from the subject.

Class names, internal file boundaries, greedy selection, field generation order, and where you represent known output text are design decisions. Evaluate those decisions against the subject and actual results. An example in this folder is not automatically a requirement.

For failures that the subject does not define with a complete output policy, document your choice and its consequences. A skipped request must not disappear from your accuracy calculation.

## Using AI and references

These materials began as an AI-assisted draft and were revised around my project experience and the supplied subject. Verify explanations against the linked documentation and your actual SDK. Review any content you reuse so you can explain and take responsibility for it.

The references include documentation for packages the subject forbids importing into your implementation. Reading a tokenizer explanation is different from using that package to do the project's work for you.

Ask peers to review your reasoning too, especially where a working example might be hiding a wrong assumption.
