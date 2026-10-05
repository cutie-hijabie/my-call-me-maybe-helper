# Semantic Accuracy and Performance — Correct, but Please Finish Today

The generated records finally had the structure I needed. Now I had to check two other things: whether they meant the right thing, and whether the program could finish the batch in a reasonable amount of time.

These turned out to be connected. Changing the model context could affect both the answers and how much generation work the program performed.

## Structure Does Not Prove Meaning

The constraints can prevent an unavailable function name or a parameter with the wrong type.

They cannot automatically tell me that the model chose the right available function or extracted the intended text from the request.

That decision still belongs to the model.

I kept that boundary explicit while working on the prompt. I did not want request-specific Python rules choosing functions or extracting arguments just to make the examples pass.

## Giving the Model Useful Context

I worked on the model instructions and how the runtime function definitions were presented.

The useful context included function descriptions and parameter information. Shorter text is helpful only if the model still has enough information to understand what each function does.

The original request also stayed separate from the instruction text, with JSON serialization used where the request needed to be represented safely.

I compared actual generated function names and argument values, rather than assuming a cleaner-looking prompt must be better.

## Measuring the Expensive Parts

An early private-style batch produced 10 complete records and one explicit failure. The successful generations alone took about **22 minutes and 47 seconds**.

That was nowhere near the whole-batch target.

I measured model inference and candidate checking separately. The bottleneck changed as the masking work improved: eventually model inference was taking most of the time.

This also corrected an assumption about the hardware. The measured runs were using the **CPU**. Having a GPU available somewhere does not mean the model is using it.

## Checking Candidates in Model Order

For greedy selection, I do not need to fully validate every vocabulary token before choosing the highest-scoring legal one.

Candidates can be checked from highest model score downward. Once the first valid candidate is found, lower-scoring candidates cannot beat it under the same selection rule.

That reduced candidate-checking work while keeping legality checks before selection and preserving the model's ranking.

I also reduced the definition context after the model had already selected a function. At that point, its argument generation needs the selected function's definition rather than the full list of unrelated functions.

## Known Text Was Another Source of Work

The outer field names and the original request were already known. Making the model repeatedly predict that text added inference work without adding a semantic decision.

The generation shortcut copied known text while keeping the JSON and schema state synchronized. The model continued to choose the function and argument values.

Token boundaries still mattered. A candidate that crossed into a copied section could not be allowed to duplicate or skip part of it.

This was an optimization of the existing journey, and it needed the same structural checks as ordinary generation.

## An Optimization That Went Backwards

One attempted optimization bundle made performance worse and produced an argument such as:

```text
b: 3e+69
```

That can look like a legal number while being a completely wrong answer to the request.

I reverted that bundle. A proposed speed improvement is still an experiment until the output and measurements support it.

Closing Firefox and other applications also noticeably improved a later CPU run. One observed batch finished in **160.14 seconds**, with about **147.22 seconds** spent on model work and **3.66 seconds** on selection/validation. Its displayed calls looked correct; formal grading remained a separate check.

## The Graded Results

The public set scored **11/11: 100%**.

The private set scored **10/11: 90.9%**, with its 11 requests taking **232.11 seconds**.

All private records passed the structural checks and preserved their original prompts. The remaining semantic error was an extracted value of `system database` where the expected value was `system`.

The regression checks also passed: **440 tests in 6.25 seconds**, with lint clean.

## What I Learned

I need separate evidence for structure, meaning, and speed. Improving one does not prove the others improved too.

The measured accuracy reached the target on these sets, and the private batch finished within five minutes. The extraction mistake stayed in the record because passing the target does not make that mistake disappear.
