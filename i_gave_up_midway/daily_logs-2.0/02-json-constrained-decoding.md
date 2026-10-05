# JSON Constrained Decoding — Valid JSON Is Only the Beginning

After connecting the existing pieces into a running program, I needed to look more carefully at what my constraints were actually guaranteeing.

The program could produce readable output. That was progress. But readable output, valid JSON, and a correct function call are three different things.

Today was about the middle one: making sure the generation loop respects JSON syntax before it accepts a token.

## Separating the Responsibilities

I already had a JSON state machine and masking code. I reused them instead of rebuilding everything just because I had changed my workflow.

The important part was understanding which checks belonged where.

The JSON machine understands syntax: strings, quotes, brackets, commas, numbers, literals, and whether a value is complete.

It does not need to know that my output should contain a field named `prompt`. A prefix check against an allowed field name belongs to the schema constraints.

For example, these are both valid JSON:

```json
1
```

```json
{}
```

Neither one is the function-call record I need.

That distinction helped me stop treating every incomplete result as a failure of the JSON parser.

## A Token Can Contain More Than One Character

The model gives me token candidates, but my JSON machine reasons about characters.

A candidate might contain a quote, some text, and another structural character. Checking only its first character would miss what the rest of the token does.

So candidate evaluation has to simulate the whole decoded token against a copy of the current state. If any part becomes illegal, the candidate must be rejected.

That simulated state is temporary. Considering a candidate must not change the real generation state, especially when that candidate is rejected.

Only the selected token advances the live state.

This is also why parsing the finished output is useful for inspection but cannot replace constrained decoding. By then, generation has already happened.

## The Number-Whitespace Bug

One concrete bug was that the grammar could accept something like:

```text
[1 2]
```

There is no comma between the values, so this is invalid JSON.

The problem was in how the number state handled whitespace. Whitespace after a completed number needs to end that number and move the machine into the state that expects a delimiter. It should not leave the machine ready to keep extending the same number.

There is another detail: whitespace cannot make an unfinished exponent valid. A number that has started an exponent still needs the remaining number syntax before it can finish.

Fixing that boundary made the machine distinguish a completed number from an incomplete one, rather than treating whitespace as harmless everywhere.

Apparently a space can cause drama too.

## Completion and Failure Need Different Outcomes

The generation loop also needs to distinguish:

- a complete JSON result,
- a result cut off by the generation limit,
- and a state with no legal next token.

A limit does not magically turn partial output into a successful result.

Likewise, if every candidate is rejected, generation must report that failure. The guard added during the earlier integration work stayed important here: selecting a token anyway would hide the actual constraint problem.

Each request also needs fresh state. A completed object or a partial string from one request cannot become the starting point of the next one.

## What the Checks Actually Showed

The grammar/regression checks passed: **248 tests**. The focused generation checks also passed: **8 tests**.

The real-model batch still showed the limits of the current implementation:

- 7 responses parsed as JSON,
- 3 of those contained only the prompt,
- 4 were empty objects,
- and 4 requests reported an explicit no-valid-next-token failure.

Those outcomes were recorded rather than quietly called complete function calls. The remaining dead ends still needed investigation.

## What I Learned

I now have a clearer boundary between JSON syntax and the function-call contract.

The grammar can tell me whether a candidate is syntactically legal. It cannot tell me whether the output contains all the fields my application needs or whether the model understood the request.

The next work is to connect the runtime schema to the same generation loop so a parseable result also has the required structure.
