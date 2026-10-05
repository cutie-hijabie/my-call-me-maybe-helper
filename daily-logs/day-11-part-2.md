# Day 11 --- Part 2: Apparently "Valid JSON" Was the Easy Part

Part 1 ended with a pretty important realization: getting the model to
produce *valid* JSON is not the same thing as getting it to produce the
JSON I actually need.

Today was basically me taking that realization and making it everyone's
problem.

The goal sounds innocent enough: the output has a fixed outer shape.
There are only a few legal top-level keys, and each stage of generation
should narrow what the model is allowed to do without secretly making
the decision for it.

Simple.

It was not simple.

## The Outer Keys Problem

I needed the top-level object to stop behaving like generic JSON.

Generic JSON is happy with basically any valid string as a key. My
project is not.

There is a tiny known set of legal outer keys, so I started treating
those keys as **candidates** instead of just hoping the model spells the
structure correctly.

That distinction ended up mattering a lot.

The mask is not supposed to pick the semantic answer for the model. It
is supposed to remove structurally impossible choices. The model still
gets logits for the surviving options and makes the actual choice from
those.

That became the rule I kept coming back to:

> the mask restricts possibilities; the model chooses among the legal
> possibilities.

Once I had that straight in my head, the outer object stopped feeling
like "hardcoding the answer" and started feeling like what it actually
is: enforcing the output schema.

## Prefixes, Because Tokens Refuse to Be Convenient

Of course a tokenizer is under absolutely no obligation to hand me one
neat token containing one complete key.

A key can arrive whole, or in pieces.

So checking only complete strings is useless.

The candidate system has to understand that a partial key can still be
legal if it is the beginning of at least one allowed key. A partial
sequence that can no longer become any legal key has to disappear from
the logits before selection.

There is also a second rule hiding in there:

A prefix is enough **while the key is unfinished**.

The moment the generated token closes the key, "close enough" stops
being acceptable. At that point it has to be an exact candidate.

That sounds obvious written down.

It took considerably more staring at quote characters to make it feel
obvious in the implementation.

## The Quote Situation™

There are two annoyingly different cases.

Sometimes generation is already inside a string and a later token
contains the quote that closes it.

Other times one token can contain enough of the string syntax that the
key begins and ends in that same candidate token.

Those cases cannot be treated identically.

This was one of those bugs where the condition looks completely
reasonable until you ask what state the JSON machine is in *before* the
candidate is accepted.

The lesson: when doing constrained decoding, "what state am I in?" is
not enough.

You also have to ask:

> Is this state describing the world before this candidate, or after it?

That question has now caused me enough suffering that I will probably
remember it forever.

## Outer Keys Are Not Parameter Keys

Another thing I had to keep very explicit: a key is not automatically an
*outer* key.

Later, the parameters object contains its own keys. Those names come
from the selected function definition, not from the fixed outer
structure.

So the candidate-key restriction has to know which object depth it is
operating in.

Otherwise the same mechanism that correctly rejects random top-level
keys will eventually start rejecting perfectly legal parameter names.

Fun.

The JSON state machine already tracks nesting, so I used that
information instead of inventing another parser on top of the parser.

## Remembering What We Just Generated

Masking can decide whether the next candidate is legal, but the schema
state also needs memory.

I added state for:

-   the remaining legal outer-key candidates,
-   the partial outer key currently being built,
-   and the current completed outer key.

The last one became important surprisingly quickly.

Once a key is complete, its partial buffer needs to be cleared so the
next key can start from an empty string. But clearing it means I would
immediately lose the information about which key had just been
generated.

So the completed key gets remembered separately before the partial state
is reset.

That gives later logic context like:

> I am not just generating *a value*. I am generating the value
> belonging to this specific outer key.

Which is exactly what I need next.

Completed outer keys also stop being candidates. A schema that requires
one copy of a field should not quietly allow the model to generate it
again later just because the JSON would still parse.

## The First Stage Now Actually Means Something

The schema state already had stages, starting with the prompt stage.

Previously that mostly described where generation was conceptually.

Now it is beginning to actively constrain what can happen.

During the prompt stage, the legal top-level key is narrowed further. It
is not enough for the model to choose any key from the outer candidate
set. The generated key has to agree with the stage it is currently in.

So there are now layers of restriction:

1.  generic JSON says whether a token is syntactically possible,
2.  the outer schema says whether it can still form a legal top-level
    key,
3.  the current stage narrows that legal set further.

The model still generates the surviving token.

The constraints just keep it inside the contract.

## Then I Remembered the Prompt Value Exists

After spending all that time making sure the key itself is correct,
there is still the value.

And this value is different from the function name and parameters: it is
already known. It is the original request that came from the input file.

That meant I needed to separate two things that are very easy to
casually call "the prompt":

-   the original user request,
-   the larger instruction text eventually sent to the model.

Those are not interchangeable.

The output needs the original request.

So the schema state now starts carrying that original request as
generation context, and I started building the same kind of
partial-value tracking I used for candidate keys.

The basic idea is familiar by now: while the value is incomplete, a
candidate must still be capable of becoming the required value; when the
value closes, partial agreement is no longer enough.

I am intentionally not writing out the full mechanism here because this
is exactly the kind of part that is worth working through yourself.

## Where I Stopped

I stopped mid-way through wiring the prompt value into the schema-state
update logic.

The masking side can now identify the relevant prompt value context and
reason about candidates against the original request.

The missing piece is making the state tracker correctly accumulate the
accepted value character by character without accidentally treating JSON
syntax as part of the actual prompt text.

Which sounds small.

Which is exactly why I am stopping before tired-me turns it into a
three-hour bug.

## Things I Actually Learned Today

-   Schema constraints and semantic decisions are not the same thing.
-   A finite candidate set can still leave the model responsible for the
    choice.
-   Prefix validity and exact completion are two separate checks.
-   Token boundaries cannot be assumed to line up with JSON boundaries.
-   Quote handling gets weird very quickly when a token can contain more
    than one structural character.
-   Nested keys need different candidate sources from top-level keys.
-   State needs to remember completed context, not just partial text.
-   "Prompt" can mean the model instruction or the original request, and
    mixing those up is asking for pain.
-   If I am tired enough to write a state condition and immediately
    distrust it, it is time to go home.

## Current Status

The outer candidate-key machinery is mostly in place.

The prompt stage now has a real structural constraint instead of merely
existing as an enum value.

The original request is being brought into schema state so its output
value can be constrained next.

Tests are intentionally waiting until the prompt block is complete,
because I want to test the whole section as a unit instead of stopping
after every microscopic change.

Next session: finish the prompt-value state tracking, finish the prompt
section, handle the transition into the function-name stage, and then
make the tests try to ruin everything.

As they should.
