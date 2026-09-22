# Day 11 Part 1 --- Chapter 6 Had One Last Trap Door

Chapter 6 was *basically* done.

Which, as I have now learned, is one of the most dangerous sentences I
can say in this project.

I came into today with schema-aware masking working, a very large pile
of passing tests, and enough confidence to think this was going to be
mostly verification and then we could move on.

It was not mostly verification.

It was verification, finding weird edge cases, fixing them, finding
another thing the old tests never proved, fixing *that*, and then
finally getting to close Chapter 6 for real.

So. Normal day.

## Where We Started

At this point the project could already do a lot more than just generate
valid JSON.

The schema layer knew about the runtime function definitions, tracked
which part of the function call was being generated, restricted function
names, restricted parameter names based on the selected function, and
restricted parameter values based on their declared types.

The existing stress tests were also doing their job very aggressively.

Everything looked good.

But most of the schema tests had been built around small synthetic
definitions, which meant there was still one question I wanted answered
before calling the chapter finished:

**Does all of this actually behave correctly against the real runtime
schema?**

So I made a separate Chapter 6 verification test file and started poking
the system with the actual function definitions.

That is where things got interesting.

## The Prefix That Refused to Die

Most of the real-schema tests passed immediately.

Real function names? Good.

Obviously fake function names? Rejected.

Parameter types? Good.

Parameters belonging to the selected function? Mostly good.

And then one incomplete function name survived when it absolutely should
not have.

The funny part was that the value itself *was* a valid prefix of a real
function name.

That would have been perfectly fine if generation was still in the
middle of the string.

Except it wasn't.

The candidate had already closed the string.

So we had a very specific distinction hiding inside the masking logic:

-   a partial name that is still being generated can be a prefix
-   a name that has finished being generated has to be exact

We already knew this conceptually. We already had tests for it.

The problem was that there was one token shape our old tests had not
fully cornered.

A token could contain enough of the string structure that opening,
content, and closing happened together instead of being spread across
multiple generation steps.

The old logic understood the split version.

The single-token version found the trap door.

## And Then Parameters Did The Exact Same Thing

Naturally, after fixing the function-name version, the parameter tests
exposed basically the same family of problem.

One very short parameter name was being accepted where it should not
have been, because it happened to be the beginning of a longer valid
parameter name.

Again:

still generating? Prefix is fine.

finished generating? Exact match or goodbye.

This was actually a nice moment because the second failure stopped
looking random. The function-name side and parameter-name side were
following the same idea, so once the first bug made sense, the second
one became much easier to reason about.

I am deliberately not putting the exact fix here because this repo is
supposed to help you understand the project, not let you copy the answer
and escape the suffering entirely.

You deserve at least a *little* character development.

## Okay Surely Chapter 6 Is Done Now

No ❤️

After the runtime-schema verification passed, I went back to the actual
Chapter 6 done criteria instead of just trusting the green test suite.

One phrase mattered a lot:

The generated parameters need to contain **exactly** the parameters
required by the selected function.

We had already proven that random undeclared parameter names could not
get in.

But "no extra names" is not automatically the same thing as "exactly the
correct set."

So I started asking more annoying questions.

Can the parameter object close too early?

Can a required parameter disappear?

Can the same valid parameter be generated twice?

The early-close behavior was already protected.

The duplicate question was not.

## We Had Memory Loss

The schema state knew which parameter was currently being handled.

Then, once that parameter was finished and generation moved on, that
current state was cleared so the next parameter could begin.

Which sounds completely reasonable.

Except there was no persistent memory of which parameters had already
been completed.

So when the model reached the next key, the masker could correctly ask:

"Is this a parameter that belongs to the selected function?"

But it could not also ask:

"Have we already used this one?"

That meant a parameter could potentially come back for round two.

Technically valid name.

Wrong overall parameter set.

Very Chapter 6 problem.

So the schema state gained a small amount of long-term memory, and the
parameter masking started taking that history into account when deciding
which names were still available.

I am once again leaving the exact mechanics out.

You can suffer productively.

## The Final Test Pile

After the edge cases were fixed, the Chapter 6 verification tests
passed.

Then I ran the entire project test suite because changing schema state
and masking this late in the chapter is exactly how you accidentally
resurrect a bug from three chapters ago.

Final result:

**414 tests passed.**

No surprise regression party.

No mysterious old JSON failure.

No "why did touching parameters break numbers?"

Just green.

Suspiciously peaceful, honestly.

## Chapter 6: ACTUALLY DONE ✅

Chapter 6 now has the full schema-aware constraint layer sitting on top
of the generic JSON constraints.

The important shift across this chapter was basically:

> valid JSON is not enough.

The generated structure now has to stay inside the runtime function
schema while it is being generated.

Not after.

Not by cleaning it up later.

Not by hoping the model behaves.

The schema constrains what is possible, while the model still makes
choices among the valid possibilities.

And that last sentence is exactly why the next chapter exists.

## Next Up: Chapter 7 👀

Chapter 6 can guarantee that the model chooses **a real function** and
produces **schema-valid parameters**.

It cannot guarantee that the model chose the function that actually
makes sense for the user's request.

A perfectly valid call can still be completely wrong.

Chapter 7 is where we finally start dealing with that.

The model needs enough context to understand the available functions and
the request, and the actual function and argument choices need to come
from the model itself --- not from keyword tricks, regex shortcuts, or
suspicious medieval magic hiding behind the curtain.

So now we move from:

> "Is this output allowed?"

to:

> "Did the model actually understand what we asked?"

This feels extremely safe and unlikely to cause problems.

See you in Part 2.
