# A Practical Build Route

Use this route to grow one running program. Each section adds a capability to the same request journey.

The subject governs the final result. Early observations can be incomplete or wrong while still showing that a particular connection works. Record those limits clearly.

## 1. Establish the contract and setup

**Goal:** know what enters the program, what leaves it, and what environment the reviewer will use.

**Understand first:** [the big picture](guides/00-big-picture.md), [output ownership](guides/04-designing-the-output.md), [Pydantic](guides/07-pydantic-and-validation.md), [I/O](guides/08-io-and-errors.md), and [tooling](guides/11-project-structure-and-tooling.md).

**Decide:**

- Where will loading and input validation happen?
- Who preserves the original request? Who builds the larger model instruction?
- What will generation return, and who will assemble and write the final record?
- Which resources can be initialized once, and which state belongs to one request?
- What are the supported definition types, and how will unsupported definitions be reported?

**Build:** establish dependency configuration, a runnable entry point, and input loading/validation. Give the files clear responsibilities without requiring any particular module names. Read the subject's class/Pydantic requirement while designing those responsibilities, rather than leaving it until final review.

**Observe:** inspect the loaded definitions and original requests. Report bad input before expensive model initialization where possible.

**Verify at the end:** correct inputs load, malformed or missing inputs produce useful errors, and the documented environment setup works. These checks do not need the model.

**Carry forward:** validated runtime data, explicit ownership boundaries, and a working environment.

## 2. Connect one real request

**Goal:** get a request through public model access and back as readable generated text.

**Understand first:** [SDK and tokens](guides/01-llm-sdk-and-tokens.md) and [the generation loop](guides/02-generation-loop.md).

**Decide:** how token IDs reach the scoring method, how you identify the generated part of the sequence, and what stops generation. Inspect the public SDK methods actually supplied to you; do not assume capabilities from a different wrapper.

**Build:** connect the entry point, model setup, token representation, instruction construction, and generation loop. Keep original-request text separate from model context.

If you already have constraints, preserve useful work. Observing a basic run does not require deleting it.

**Observe:** display the original request and generated response separately. Identify whether the run finished, reached a limit, or failed.

**Verify at the end:** public SDK access, response decoding, generated-token boundaries, and explicit stopping/failure behavior. Repeat a request with fresh state.

**Carry forward:** a traceable model journey. This does not yet prove JSON syntax, the output schema, or meaning.

## 3. Enforce JSON syntax during selection

**Goal:** reject candidates that would break JSON grammar before accepting a token.

**Understand first:** [constrained decoding](guides/03-constrained-decoding.md). Try its tiny-vocabulary exercise, then read the JSON grammar.

**Decide:** what grammar state you need, how nesting is tracked, how candidate simulation leaves live state untouched, and how completion differs from a valid unfinished prefix.

**Build:** evaluate the whole token contribution, restrict eligibility, select a legal continuation, and advance live state consistently. Account for token representations that span byte or structural boundaries.

**Observe:** inspect generated text and the state at completion or failure. An empty object can demonstrate valid syntax while still failing the final output contract.

**Verify at the end:** multi-character candidates, quote/escape handling, literals, nesting, number delimiters, and state preservation when candidates are rejected. Include a candidate that becomes illegal halfway through. Check no-legal-token and generation-limit outcomes.

Parsing is a useful independent check after generation. Remember that Python's default JSON parser accepts some extensions; a successful parse alone does not establish every JSON or schema rule.

**Carry forward:** the grammar and running loop. Record schema-related failures separately rather than trying to solve every problem inside the JSON machine.

## 4. Add the runtime function-call schema

**Goal:** completed records have the required fields, the exact original request, and parameters matching a loaded function definition.

**Understand first:** [output design](guides/04-designing-the-output.md), [schema and types](guides/05-schema-and-types.md), and [validation boundaries](guides/07-pydantic-and-validation.md).

**Decide:** which parts of the record are generated or represented as known text, and how the selected function supplies the parameter constraints. A convenient generation order is a design choice; JSON object ordering is not the schema itself.

**Build:** derive legal function and parameter names from runtime definitions. Track partial and completed names, used keys, required arguments, and value types. Keep grammar state and schema progress synchronized.

The output's fixed field names describe the contract. The function names and parameter definitions must remain dynamic. The model must make the semantic function and argument choices.

**Observe:** inspect a complete record. Parse it and compare its prompt to the original request, including quotes and backslashes. Compare its function and parameter types to the loaded definitions.

**Verify at the end:** unknown/duplicate keys, missing arguments, shared name prefixes, zero-parameter functions, supported types, and changed definitions. Confirm rejected candidates do not change the live selected-function or parameter state.

**Carry forward:** structurally compliant records. A legal function or typed value can still be semantically wrong.

## 5. Measure meaning and runtime

**Goal:** improve function selection and argument extraction while keeping structural constraints intact and meeting the runtime budget.

**Understand first:** [prompt design](guides/06-prompt-design.md) and [performance](guides/09-performance.md).

**Decide:** what runtime context the model needs, how to count a correct answer, and how to measure the full run. Keep your development examples separate from cases you use to evaluate generalization.

**Build:** an instruction built from the actual request and definitions. Measure model calls, candidate evaluation, setup, and whole-batch elapsed time. Change one relevant thing at a time where possible.

Optimize demonstrated bottlenecks. Precomputed token information, fewer candidate checks, shorter context, or known-text shortcuts need evidence that they preserve the selection rule and the output contract. Do not use request keywords or regular expressions to choose semantic answers.

**Observe:** inspect the actual function and argument values, not only whether the output parses. Compare timings under recorded hardware and conditions.

**Verify at the end:** score function choice and arguments together on varied requests and changed function sets. Count failed requests in the denominator. Recheck structural validity after prompt or generation changes.

Version 1.2 targets at least 90% semantic accuracy, fully parseable/schema-compliant saved output, and the complete test batch in under five minutes. Record sample size and conditions alongside results; a small successful set is limited evidence.

**Carry forward:** measured results and known remaining mistakes. A performance target being met does not make individual errors disappear.

## 6. Complete the batch and failure paths

**Goal:** make the whole application useful with default/custom paths, persistent results, and clear failure behavior.

**Understand first:** [input, output, and errors](guides/08-io-and-errors.md).

**Decide:** how a per-request failure affects the remaining batch, the result file, and the exit status. The output schema does not define a normal error record, so do not quietly insert an incompatible one. Keep failed requests visible in diagnostics and measurements.

**Build:** complete the CLI, initialize shared resources once, create fresh state per request, collect complete validated records, and write the result array. Handle input, setup, generation, and write failures at their own boundaries.

**Observe:** run multiple requests, inspect the saved file, and compare attempted/saved/failed counts. Try a failure followed by a valid request to see whether your documented continuation policy is actually followed.

**Verify at the end:** defaults and custom paths, nested output directories, malformed/missing files, whitespace-only requests, setup failures, request failures, and write failures. Define whether an empty input list is accepted, and check the output when no request succeeds. Avoid testing filesystem permissions in a way that gives misleading results under a privileged account.

**Carry forward:** working end-to-end behavior and a documented failure policy. Finishing with partial successes is a deliberate outcome, not proof that every request passed.

## 7. Verify the project you will submit

**Goal:** reproduce the program from a clean checkout and explain its actual implementation.

**Understand first:** [testing](guides/10-testing-and-debugging.md), [tooling](guides/11-project-structure-and-tooling.md), [README and review](guides/12-readme-and-defense.md), and [common pitfalls](guides/13-common-pitfalls.md).

**Build:** finish the required README, dependency declarations, Makefile commands, typing, docstrings, and repository cleanup. Include the subject's required input examples and SDK directory; exclude generated output and environment/cache files.

**Observe:** follow your own install/run instructions from a clean checkout. Demonstrate a successful request and a handled failure. Explain the boundary where the model makes its function choice.

**Verify at the end:** run the relevant regression suite, the subject's exact lint/type-check commands, real-model accuracy/validity checks, and whole-batch timing. Reuse current evidence unless new changes invalidate it. Check dependency setup using the reviewer's `uv sync` workflow.

Practice a small modification appropriate to your design and explain what changes and why. A particular new feature is not guaranteed to be the review task.

**Finish with:** the command/environment used, results, known limitations, and any unresolved subject requirement. Leave gaps visible rather than marking them complete because the latest demo worked.
