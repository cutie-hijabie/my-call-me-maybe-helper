# Day 8 — Part 1: Chapter 5 Is Finally Dead 🎉

Today was supposed to be “finish Chapter 5.”

And technically... that is what happened.

It just involved optimization, tests, a benchmark that scared me for a minute, and discovering that maybe loading an entire LLM **20 TIMES** is not the fastest approach imaginable.

## Where We Started

Chapter 5 already had the important pieces working:

* constrained generation
* JSON masking
* masked argmax instead of normal argmax
* updating Jason after every selected token
* real generations producing valid JSON

But the chapter wasn't actually done yet.

The remaining problem was **performance optimization**.

My masking function was checking tokens using `validate_candidate()`, which means copying the JSON state machine and testing the token character by character.

Correct? Yes.

Cheap? Absolutely not.

So the goal became:

> Reject obviously impossible tokens cheaply first, and only send the survivors through full validation.

## The Cheap Filter Era

I added state-specific checks before `validate_candidate()`.

For states where the next possible characters are predictable, I can look at the beginning of the token and immediately reject obviously impossible candidates.

I optimized states like:

* `EXPECT_COLON`
* `IN_STRING_ESCAPE`
* `AFTER_VALUE`
* `EXPECT_KEY`
* `DONE`
* `START`
* `EXPECT_VALUE`
* `IN_ARRAY`

I deliberately **didn't** try to force the same optimization onto everything.

`IN_NUMBER`, `IN_STRING`, and `IN_LITERAL` are much more context-dependent, so those still go through full validation.

The important part wasn't “optimize every single state.”

It was:

> Do cheap work where cheap work is actually safe.

## Then We Tried Very Hard to Break It

A separate optimization test file was added because I did not trust myself enough to just look at the code and declare victory.

The tests checked things like:

* whitespace before valid characters
* invalid closing brackets
* multi-character tokens
* string escapes
* empty objects
* values inside arrays
* tokens that begin correctly but become invalid later
* states intentionally left to full validation

The main rule was simple:

> The optimized masking result must still agree with Jason.

And somehow:

**18 optimization tests passed.**

Then the full test suite passed too.

So optimization had not quietly destroyed JSON while I wasn't looking.

Excellent.

## But Is the Optimization Actually Doing Anything?

Next problem:

Just because I wrote a “cheap filter” doesn't mean it is actually saving work.

So we measured it.

For one representative `EXPECT_COLON` case:

```text
total mapped tokens: 18
full validate_candidate calls: 6
cheaply rejected: 12
```

So **12 out of 18 tokens were rejected before full validation**.

That was exactly what I needed to prove: the optimization wasn't decorative. It was genuinely reducing expensive validation work.

We also tested that obviously invalid tokens never even reached `validate_candidate()`.

For example, while Jason is waiting for a colon:

```text
hello
```

gets murdered immediately.

As it should.

## The 344-Second Incident

Then came the Chapter 5 performance requirement:

> 20 generations should finish in under 5 minutes.

We ran the benchmark.

And got:

```text
20 constrained generations: 344.04 seconds
```

🙂

That is **5 minutes and 44 seconds**.

Chapter 5 said no.

At first this looked like the optimization still wasn't fast enough.

Except the terminal output had a tiny clue.

The model was loading.

Again.

And again.

And again.

**Twenty times.**

The problem wasn't that constrained decoding took 344 seconds.

The benchmark was basically doing:

```text
load entire model
generate once

load entire model
generate once

load entire model
generate once

...

WHY IS THIS SLOW???
```

Very scientific.

## Back Into `generate_constrained()` We Go

The problem was architectural.

`generate_constrained()` was creating its own `LLMWrapper`.

That meant every call owned a completely new model.

But the model doesn't need to change between prompts.

So I changed the responsibility:

**Before:**

`generate_constrained()` created everything it needed.

**Now:**

The caller creates the model once and passes it into generation.

Then I noticed the exact same thing with `vocab_map`.

The vocabulary also doesn't magically change between generations.

So why was I rebuilding that every time?

Out it went too.

Now the reusable pieces are created once:

```text
model
vocab_map
```

and passed into `generate_constrained()`.

The things that actually belong to one generation — like Jason and the prompt's token IDs — stay inside.

Much cleaner.

## The Redemption Benchmark

Tests were updated for the new generation interface.

Then we ran the Chapter 5 tests again.

First proper benchmark:

```text
20 constrained generations: 18.27 seconds
```

EXCUSE ME???

From:

**344.04 seconds**

to:

**18.27 seconds**

because apparently loading an LLM twenty times was not an optimization strategy.

Then we ran everything again after adding the final verification test:

```text
20 constrained generations: 12.92 seconds
```

Six tests.

Six passed.

Chapter 5 wanted under **300 seconds**.

We gave it **12.92**.

I'll take it.

## The Final Proof

The last test was specifically about proving where an invalid token gets excluded.

We gave `"hello"` a higher logit than `":"` while Jason was in `EXPECT_COLON`.

Normally, argmax would love `"hello"`.

But constrained decoding doesn't care about its hopes and dreams.

The cheap filter sees that `"hello"` cannot begin a valid continuation in `EXPECT_COLON`, changes its logit to `-inf`, and it never even reaches full `validate_candidate()`.

So now I can point to the exact moment an invalid token disappears:

**inside masking, before argmax chooses the next token.**

That is the entire point of constrained decoding.

## Chapter 5: DONE ✅

Today ended with:

* optimization added without breaking Jason
* 18 dedicated optimization tests passing
* full validation calls measurably reduced
* 20 real generations producing parseable JSON
* model loading moved out of individual generations
* vocab map building moved out too
* benchmark dropped from **344.04s → 18.27s → 12.92s**
* explicit proof that invalid tokens are removed before selection
* Chapter 5 officially finished

The funny part is that today's biggest performance improvement wasn't some genius token-filtering algorithm.

It was basically:

> maybe don't load the whole model twenty times.

Anyway.

Next Up: Chapter 6 👀

Chapter 5 was about making sure the model can only generate valid JSON.

Chapter 6 takes that further: now the JSON doesn't just need to be syntactically valid — it needs to match the structure we actually asked for.

So we're moving from:

“Is this valid JSON?”

toward:

“Is this the right kind of JSON?”

Which means Chapter 6 is where we start dealing with schemas and using them to constrain what the model is allowed to generate.

Jason survived Chapter 5.

Now we're giving him standards.

**Chapter 6 next.**
