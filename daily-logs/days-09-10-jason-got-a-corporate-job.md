# Days 09 + 10 --- Jason Got a Corporate Job and Now He Has Rules 🧃📋

Okay so. Day 8 ended with Jason finally having standards.

He could already stop the model from producing broken JSON, and then
Chapter 6 walked into the room and basically said:

> cute. but is it the **right** JSON?

And that is where Days 9 and 10 happened.

These two days were mostly about taking the generic JSON guardrails we
already built and teaching them about the actual function definitions
that arrive at runtime. Not "this looks like JSON, ship it." More like
"this is valid JSON **and** you are only allowed to say things that
exist in the schema, thank you very much."

Very calm. Very normal. Definitely no tiny state machines supervising
other tiny state machines.

------------------------------------------------------------------------

## Day 09 --- The Schema Started Having Opinions

We picked Chapter 6 back up from the exact point where Day 8 stopped: we
already had the runtime function definitions validated and we had
started building a separate schema-tracking state.

The first job was getting from:

**the model chose a function**

to:

**okay, what parameters is that function actually allowed to have?**

That sounds extremely obvious until you remember that the reviewer can
replace the function-definition file.

So we cannot know the function names ahead of time.

We cannot know the parameter names ahead of time.

We cannot quietly build special cases around the examples we were given.

Everything has to come from the runtime definitions.

Which means Chapter 6 became a lot of asking:

**"What is allowed *right now*, based on what has already been
generated?"**

### Chosen function → allowed parameters

Once a function name is complete, the schema tracker can find that
function inside the loaded definitions and retrieve its parameter
definitions.

That gave us the next important relationship:

**chosen function → its own parameter dictionary**

Not everybody's parameters.

Not a hardcoded list.

Not "well, the sample file has `a`, `b`, and `name`, so surely..."

No.

The function that was actually selected determines what parameter names
become legal.

This is one of those pieces that is tiny when you describe it and
absolutely foundational when you think about what the evaluator is going
to do to us.

------------------------------------------------------------------------

## Parameter names got the same treatment as function names

We already had the idea of prefix-constraining function names.

So if the runtime functions are something like:

-   one function beginning with `fn_g...`
-   another beginning with `fn_r...`

then while the model is spelling the name, we keep only prefixes that
can still become a real runtime function.

For parameters, we built the same idea.

The schema tracker keeps the partial parameter name currently being
generated and asks:

**Does at least one parameter belonging to the chosen function still
begin with this?**

If yes, the token can stay alive.

If no, goodbye. Into the `-inf` void you go.

Then, once the partial name exactly matches a declared parameter, the
schema state can recognize which parameter we are currently dealing
with.

This matters because the parameter name tells us what kind of value is
supposed to come next.

So now the chain started looking like:

**function → parameter → expected type**

And suddenly Chapter 6 started feeling less like "a couple extra checks"
and more like an actual second constraint system sitting beside Jason.

------------------------------------------------------------------------

## We also met the Extremely Annoying Closing Quote Problem™

Tokenizer tokens do not care about our emotional wellbeing.

A token is not guaranteed to contain one neat little character.

It might contain part of a name.

It might contain the end of a name **and** the closing quote.

It might contain punctuation with it.

So when checking a possible function-name continuation, we could not
blindly append the entire token and ask whether the result was a prefix.

Because a perfectly good final token could effectively look like:

**the last letters of the function name + the closing quote**

The quote is JSON syntax.

It is not part of the function name.

That distinction became important.

So the masking logic learned to reason about the piece before the
closing quote, and to treat a closing quote as a stronger claim:

If the token closes the name, the accumulated name must now be an
**exact** runtime function name.

A prefix is no longer enough.

Same basic principle for parameter names.

While the string is still open, a valid prefix is okay.

When it closes, we need the real thing.

Jason handles whether the JSON string itself is syntactically legal.

The schema layer handles whether the thing inside that string is legal
for this schema.

Two bouncers. Different guest lists.

------------------------------------------------------------------------

## Then values entered the chat

Once we could identify the current parameter, we added the beginning of
type-aware masking.

The provided runtime definitions currently use string and number
parameter types, so we started there.

The idea is not that SchemaState chooses the value.

It absolutely does not.

The model still has to generate the actual argument.

The schema layer only says:

**"This parameter expects a string, so the value needs to begin like a
JSON string."**

or:

**"This parameter expects a number, so do not start giving me a quoted
string."**

This distinction matters a LOT because we are not trying to sneak
semantic selection into Chapter 6.

Chapter 6 is about legality.

Chapter 7 is where the model has to actually understand the prompt well
enough to choose the right function and arguments.

We are building the fence right now.

We are not choosing where the model walks inside it.

------------------------------------------------------------------------

## Day 10 --- Two State Machines, One Token, Absolutely No Drama Whatsoever

Day 10 was where the schema pieces stopped being isolated helpers and
started getting wired into the real generation path.

Before this, we had lots of focused tests where we manually put Jason
into a particular state and asked:

"Okay, if we are here, does SchemaState behave correctly?"

Useful.

Necessary.

Also slightly suspicious.

Because production does not teleport Jason into convenient test states.

The model chooses a token.

Jason processes it.

The schema tracker has to stay synchronized with what just happened.

And masking needs the schema tracker to be correct before the next token
is chosen.

So we added an `update` flow to SchemaState.

------------------------------------------------------------------------

## SchemaState learned to move through the output

The schema tracker now has stages for the important parts of the
expected call structure.

It starts around the prompt portion, then moves toward the function
name, then into parameters.

While the function-name value is being generated, it accumulates the
partial function name and recognizes it when it becomes complete.

Once we reach the parameters object, it starts tracking parameter keys
instead.

When a parameter name is complete, SchemaState knows which parameter is
active.

When we move on to the next key, that partial parameter tracking gets
reset so the next parameter can be read cleanly.

And we tested the little transitions individually:

-   moving from the prompt stage toward the name stage,
-   building a partial function name,
-   not accidentally treating the literal top-level `"name"` key as the
    function name,
-   moving from the function-name stage into parameters,
-   tracking a parameter name,
-   not confusing the outer `"parameters"` key with an actual function
    parameter,
-   resetting between parameters,
-   and successfully tracking the next parameter after that reset.

This was the point where SchemaState started feeling like an actual
state tracker instead of a bag of helper methods.

------------------------------------------------------------------------

## Important: SchemaState does NOT collect the generated argument values

We stopped here for a minute because it would be very easy to make
SchemaState responsible for too much.

It needs to know:

**which function?**

**which parameter?**

**what type is expected?**

It does **not** need to store the model's actual generated value.

The model generates the value.

Jason makes sure it is valid JSON.

The schema-aware mask makes sure the value is compatible with what the
parameter allows.

Keeping those responsibilities separate is basically the theme of this
entire chapter.

Every time two responsibilities start looking suspiciously cuddly, we
separate them again.

------------------------------------------------------------------------

## Then schema-aware masking got plugged into the real mask

The generic JSON mask from Chapter 5 is still the first line of defense.

A token first has to survive Jason.

If Jason says the token cannot legally continue the current JSON, there
is no reason to spend more time asking the schema layer about it.

It is already dead.

So invalid JSON candidates get rejected and skipped before the schema
checks.

For candidates that survive, the schema layer can apply the more
specific rules depending on where we are:

-   function-name constraints while generating the function name,
-   parameter-name constraints while generating parameter keys,
-   type constraints when beginning a parameter value.

This is the composability Chapter 6 was aiming for.

JSON grammar did not get replaced.

Schema constraints did not get shoved inside Jason.

They cooperate.

Jason says:

**"Can this exist in JSON?"**

Schema says:

**"Can this exist in *our* JSON?"**

I love them. They are coworkers now.

Possibly coworkers who communicate exclusively through
passive-aggressive tickets, but coworkers.

------------------------------------------------------------------------

## And then we had to wire SchemaState into generation

This part caused a tiny architecture detour.

The generation loop already had a clear order from Chapter 5:

model logits → masking → choose token → advance Jason → append token →
check whether we are done.

Now SchemaState also needs to learn about the token that was actually
chosen.

The important word there is **chosen**.

We cannot update SchemaState while testing candidate tokens during
masking, because masking looks at loads of possible tokens.

If evaluating a candidate mutated the real schema state, merely
*considering* a token would change generation.

That would be cursed.

So the schema state only updates after the model has actually selected a
token.

We also caught another small mistake while wiring this up: the schema
update had briefly ended up inside the loop that sends each character of
a token through Jason.

That would mean a token containing several characters could be handed to
SchemaState several times.

Absolutely not.

Jason is character-based.

SchemaState's current update is token-based.

So Jason processes all the characters in the chosen token first, and
SchemaState receives the chosen token once.

------------------------------------------------------------------------

## Runtime parameters stay runtime

There was another little moment where the parameter dictionary almost
became an extra argument passed into generation.

But that would be awkward because the correct parameter dictionary
depends on which function the model eventually chooses.

And we already had the information required to derive it.

So generation keeps the runtime definitions, and once there is a chosen
function, SchemaState can retrieve that function's parameters from those
definitions.

No duplicated source of truth.

No manually carrying around "the parameters" before we even know whose
parameters they are.

Much nicer.

------------------------------------------------------------------------

## The Great Test Explosion of Day 10 💥

Then we ran the entire test suite.

And the terminal basically went:

**FAILED FAILED FAILED FAILED FAILED---**

Twenty-six failures.

Which looked dramatic for approximately four seconds.

Then we actually read them.

The schema tests were passing.

The JSON state tests were passing.

The failures were almost entirely old Chapter 5 tests calling functions
using their old signatures.

We had expanded both the masking and generation APIs so they could
receive the schema state, runtime function names, and definitions.

The old tests obviously did not know that because they were written
before Chapter 6 existed.

So this was not:

**"Chapter 6 destroyed constrained decoding."**

It was:

**"Congratulations, you changed a function signature."**

The suite had 265 tests at this point.

239 were already passing.

The 26 failures all pointed back to that same test-maintenance issue.

So the old Chapter 5 tests were updated with neutral schema context,
allowing them to keep testing exactly what they were originally written
to test: generic JSON constrained decoding.

We did **not** weaken the new implementation just to make old tests
happy.

And then:

# 265 PASSED 🎉

Every single test.

Chapter 5 still alive.

Chapter 6 tests alive.

Jason alive.

Me, technically alive.

A beautiful moment.

------------------------------------------------------------------------

## But green tests do not mean Chapter 6 is done

This is where we are stopping the Days 9 + 10 log.

The architecture is now much more complete than it was at the end of Day
8, but there is one particularly suspicious thing we need to test next.

Most of the SchemaState update tests so far create a convenient Jason
state manually.

Real generation does not do that.

Real generation feeds an entire tokenizer token through Jason first and
then updates SchemaState.

And tokenizer tokens can contain multiple characters at once.

Including quotes.

Including an entire function name.

Including punctuation.

So the next test is an integration test that makes **Jason and
SchemaState process a realistic token sequence together in the same
order used by generation**.

The point is not to prove ourselves right.

The point is to try to catch the synchronization bug before the
evaluator does.

Because there is a very important difference between:

**"all my pieces work"**

and:

**"all my pieces work together."**

And Chapter 6 is officially in that second phase now.

------------------------------------------------------------------------

## Where Chapter 6 stands after Days 9 + 10

We now have the runtime schema represented and validated, function names
constrained from runtime data, chosen-function parameter lookup,
parameter-name prefix constraints, parameter type lookup, basic
value-type constraints, a schema-tracking state, schema-aware masking
layered on top of JSON masking, and SchemaState wired into the
generation loop.

What is still left is the annoying-but-important finishing work:

real synchronization testing, tightening exactly when parameter-name
masking is active, making value-type masking robust against tokenizer
weirdness and whitespace, making sure the parameter set itself is
enforced properly, enforcing the exact expected top-level structure, and
then running the full Chapter 6 Done Criteria against runtime
definitions rather than our sample names.

So no, Chapter 6 is not done yet.

But we are no longer inventing its architecture either.

We are in the **"now try to break it"** era.

Which, historically, is where this project becomes the funniest.

------------------------------------------------------------------------

## Current status

**Chapter 6: Schema-Aware Constraints**

Status: **very much alive, mostly built, currently being interrogated.**

Test suite before the next integration test:

**265 passing.**

Next mission:

Make Jason and SchemaState walk through real generated tokens together
and see whether they remain friends.

I have concerns.

See you after the test. 🫡
