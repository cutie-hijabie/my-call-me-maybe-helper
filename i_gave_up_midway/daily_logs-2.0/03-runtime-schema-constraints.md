# Runtime Schema Constraints — Making JSON Mean a Function Call

The JSON constraints gave me a syntax boundary. Now I needed to make the generated output match the actual function-call contract.

An empty object is perfectly happy being JSON. My project is considerably less happy receiving it.

The goal was to produce a complete record containing the original request, an available function name, and parameters that match that function's runtime definition.

## Deciding Who Owns the Record

Before changing individual conditions, I needed to be clear about what generation was responsible for producing.

I chose to generate the full record, with the fields progressing in this order:

```text
prompt → name → parameters
```

That order is an implementation decision that makes the state progression easier to follow. JSON itself does not require that object fields appear in this order.

The original request and the model instruction also have different jobs. The instruction contains context for the model. The output's `prompt` field must preserve the request from the input file.

Mixing those up would produce a very impressive record of the wrong text.

## Two Kinds of State

The JSON state machine tracks syntax.

The schema state tracks progress through the function-call contract: which field is being generated, which function has been selected, which key prefix is being built, and which parameters have already been completed.

The mask uses both kinds of information before selecting a token.

Pydantic was already being used to validate the input definitions. Those validated definitions supply the runtime information used by the constraints. The incremental state machinery has a different responsibility: tracking what can legally come next during generation.

## Candidates Come From the Runtime Definitions

The legal function names come from the loaded definitions, rather than a list of the sample functions written into my masking code.

While a function name is unfinished, its partial text must still be a prefix of at least one available name.

When the name closes, a prefix is no longer enough. It must be a complete available function name.

That matters when names share a beginning: finishing one valid name and continuing toward a longer valid name are different possibilities.

The model still chooses among legal candidates using its scores. The constraints restrict the available names; they do not interpret the request and choose the function themselves.

## Parameters Depend on the Selected Function

Once the function name is complete, its definition becomes the source of the parameter rules.

The parameter keys are different from the fixed outer keys. They depend on the selected function and can change when the input definitions change.

The constraints need to prevent unknown or repeated keys and stop the parameters object from closing while required arguments are missing.

They also need to handle a function with no parameters without waiting for an argument that does not exist.

The supported value types included `string`, `number`, `integer`, `boolean`, and `null`. That means the schema layer has to narrow the JSON grammar according to the current parameter's type.

A value being valid JSON does not automatically make it valid for that parameter.

## Preserving the Request Through JSON Escaping

The original request can contain quotes, backslashes, or other characters that need escaping inside a JSON string.

Its serialized representation can therefore look different from the original text. The important check is that parsing the output restores the exact original request.

That helped separate actual content from the JSON syntax used to represent it.

It also reinforced an earlier lesson: quotes that open or close a string are not part of the key or value being tracked.

## Verification

The combined JSON and schema checks finished with:

```text
428 passed in 2.85s
```

That verification covered the structural work. It did not establish semantic accuracy just because the records now looked much more convincing.

A permitted function can still be the wrong function. A value can have the correct type and still be the wrong value.

## What Changed for Me

I can now follow how the two constraint layers work together: JSON limits the syntax, and runtime definitions limit the function-call structure.

That gives me a much clearer way to classify failures. A missing field, an unknown parameter, and a wrong extracted value are no longer one vague category called “the model is broken.”

The next step is to measure whether the model is choosing the right function and extracting the right arguments inside those structural boundaries.
