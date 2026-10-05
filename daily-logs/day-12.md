# Day 12 — I Got Frustrated and Restarted the Work

Today I restarted the way I am working on Call Me Maybe.

Not the entire project.

I am still keeping and reusing the code I already wrote, but I was getting frustrated with the way I was moving through the project. I kept discovering missing pieces later, going backwards, fixing something, moving forward again, and eventually I stopped feeling like I had a clear picture of what I was actually building.

So I changed the structure of the work.

Instead of thinking about the whole final project at once, I am now splitting it into stages where each stage represents something the program should actually be able to do. I want to know what I am building, why it exists, what files are responsible for it, and what I should be able to observe before I move forward.

## Today's Goal: Connect the Existing Pieces

Today was about getting the pieces I had already written connected into one real journey.

The goal was **not** to make the final function-calling output correct yet.

The goal was to answer a simpler question:

> Can a real request enter my program, reach the real model through the code I already wrote, go through generation, and come back as readable output?

I added the program entry point and connected the existing pieces together.

The program can now:

- load the function definitions
- validate the function definitions
- load the test requests
- initialize the model
- build the vocabulary map
- build a model prompt for each request
- create fresh generation state for each request
- run constrained generation
- separate newly generated token IDs from the original model-prompt token IDs
- decode the generated IDs into readable text

And, most importantly:

**THE PROGRAM RUNSSSSS.**

The output is absolutely not correct yet.

But it runs.

And today, that was the point.

## The `!!!!!!!!!!!!` Incident

Connecting everything immediately exposed a problem.

Generation was taking FOREVER.

At first I did not know whether the model was slow or whether my constraint code was slow, so I measured them separately.

Originally, getting the model logits was taking roughly 0.2–0.3 seconds per generated token, while masking could take around 1.1–1.25 seconds per token.

So masking was doing way more work than I expected.

After working on the performance problem, masking dropped to roughly 0.13 seconds per token.

Much better.

Except the program was still not finishing.

So I printed the actual tokens it was generating.

And I got this:

```text
{
"
!
!
!
!
!
!
```

Amazing.

## Why It Was Screaming

There were two problems interacting with each other.

The first problem was in the schema state for JSON keys.

When the opening quote of a key was generated, the JSON state machine correctly moved into a key string. But because the schema state was updated after that transition, the opening quote itself was being stored as part of the partial key.

So instead of the partial key still being empty immediately after:

```text
{"
```

it contained:

```text
"
```

Then when masking tried to see whether a candidate could continue toward an allowed key such as `prompt`, it was effectively comparing prefixes such as:

```text
"p
```

against:

```text
prompt
```

Nothing matched.

That caused the second problem to appear.

When masking rejected every possible next token, every constrained score could become negative infinity. My generation code still called `max()` anyway.

So generation still selected *something* even though there were actually zero valid choices.

That is where the repeated `!` came from.

The model was not deciding that the correct key was `!!!!!!!!!!!!!!!!`.

The constraints had reached a state where nothing was legal, and generation did not know how to stop when that happened.

I fixed both behaviors:

- the opening JSON quote is no longer stored as part of the actual key prefix
- generation now explicitly fails when masking leaves zero valid next tokens instead of silently selecting a token anyway

That second change made debugging much easier because now a broken constraint gives me a useful error at the exact point where generation has no legal continuation.

## What Happened After the Fix

After fixing the key-prefix problem, the program started producing real constrained output such as:

```json
{"prompt" :"What is the sum of 2 and 3?"}
```

and:

```json
{"prompt" :"What is the sum of 265 and 345?"}
```

Those requests completed in roughly 5–7 seconds during today's run.

Other requests still fail with messages such as:

```text
No valid next token: JSON state=EXPECT_KEY, key prefix=''
```

and some currently produce:

```json
{}
```

That is not surprising yet because my handling for the function name and parameters is not complete.

The important distinction for me is that this does **not** mean the entire program is broken.

The running pipeline works.

The later output constraints are unfinished.

Those are different problems.

## Graceful Failure

I also changed what happens when one request reaches a state with no valid next token.

Before, the exception stopped the whole program and dumped a traceback.

Now each request is handled independently. If one request fails, I get a readable message explaining which request failed and why, and then the program continues with the rest of the input file.

That means one unfinished constraint no longer kills the entire batch.

## What Works at the End of Today

I can now trace the running journey as:

```text
input files
    ↓
validated runtime function definitions
    ↓
original user request
    ↓
model prompt
    ↓
encode into token IDs
    ↓
model logits
    ↓
mask invalid next tokens
    ↓
select the highest valid token
    ↓
update JSON + schema state
    ↓
repeat until generation finishes or reports a real failure
    ↓
separate newly generated IDs
    ↓
decode
    ↓
readable output
```

The final Call Me Maybe output is **not correct yet**.

Function-name handling still needs work.

Parameter handling still needs work.

Some constraints still allow incomplete output such as `{}` and some paths correctly expose that there is currently no valid continuation.

But I finally have the existing pieces connected into one program, and I can see where a failure belongs instead of feeling like everything is broken at once.

That is a much better place to continue from tomorrow.
