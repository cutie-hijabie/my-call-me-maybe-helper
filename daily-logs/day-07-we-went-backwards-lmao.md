# Day 7 -- We fixed generation... and somehow ended up back in Chapter 3

Today actually started with **generation**, not vocabulary.

Before continuing Chapter 5, I wanted to make sure my constrained
generation loop was in a good place. The overall flow was already there:

``` text
prompt
→ encode
→ get logits
→ mask invalid tokens
→ choose the highest valid token
→ update the JSON state
→ append the token
→ check whether generation should stop
```

I reviewed it and fixed the max-length condition so it checks whether
the length is **greater than or equal to** the limit instead of only
checking for an exact match.

There are still a couple of cleanup things I want to come back to later,
but the generation structure itself was good enough to move forward.

So, back to **Chapter 5: masking invalid tokens**.

## Then Chapter 5 broke

The goal of Chapter 5 is to connect the pieces I already built.

The model gives me logits, the vocabulary map turns token IDs into
strings, and the JSON state machine decides whether each candidate token
can legally come next. Anything invalid gets masked before token
selection.

My smaller tests were working, so I tried the real model.

And then:

``` text
KeyError: 151643
```

The model was returning **151936 logits**, while my vocabulary map only
contained **151643 entries**, ending at ID `151642`.

Masking was trying to look up every possible token ID in the vocabulary
map. As soon as it reached `151643`, there was nothing there.

At first, this looked like a Chapter 5 bug.

The obvious solution would have been to say: if the ID isn't in the
vocabulary map, mask it.

But that felt suspicious.

Before deciding that `151643` was invalid, I wanted to know what
`151643` actually was.

And that is how Chapter 5 sent me all the way back to **Chapter 3**.

## Surprise: Chapter 3 wasn't actually finished

In **Chapter 3: Vocabulary and Token-to-String Mapping**, I had built my
ID-to-string dictionary using `vocab.json`.

I thought that was the complete tokenizer vocabulary.

It wasn't.

Qwen also has added tokenizer tokens stored in `tokenizer_config.json`
under `added_tokens_decoder`.

Those include things like:

-   end-of-text and message boundary tokens
-   tool-call tokens
-   thinking tokens
-   other special/added tokenizer tokens

And, very importantly, those tokens occupy IDs:

``` text
151643–151668
```

So `151643`, the exact ID that crashed Chapter 5, was a **real token**.

My masking code wasn't wrong for expecting a token there.

My Chapter 3 vocabulary map was incomplete.

If I had immediately changed Chapter 5 to mask every missing ID, I would
have hidden the actual problem.

## Fixing Chapter 3 properly

I changed the vocabulary-building logic so the same ID-to-string map
includes both:

-   the normal tokens from `vocab.json`
-   the added tokens from `tokenizer_config.json`

The added-token data already tells me which IDs exist and what their
token strings are, so I didn't need to hardcode a range manually.

After the fix, the vocabulary map changed from:

``` text
151643 entries
highest ID: 151642
```

to:

``` text
151669 entries
highest ID: 151668
```

I also checked some of the new mappings directly, including the
end-of-text, end-of-message, and thinking tokens.

They were there.

Chapter 3 was fixed.

## Back to Chapter 5... again

Then I ran the real generation again.

The old error was gone.

Instead of:

``` text
KeyError: 151643
```

I got:

``` text
KeyError: 151669
```

Which sounds like I broke something again, but this was actually exactly
what I wanted to see.

It proved that the missing real tokenizer tokens were now being handled
correctly.

The situation is now:

``` text
0–151668      = actual tokenizer tokens
151669–151935 = model output positions with no tokenizer entry
```

So the model's logits are larger than the tokenizer's actual vocabulary.

That means the remaining IDs are genuinely a **Chapter 5 problem**.

## What clicked today

Today was basically a lesson in not trusting the location of the crash.

We started by cleaning up generation.

Then Chapter 5 crashed.

Then investigating the Chapter 5 crash revealed that Chapter 3 wasn't as
complete as I thought.

So we went backwards, fixed Chapter 3, tested it, and came back to
Chapter 5 with a completely different and much clearer problem.

The biggest things I learned were:

-   the model's logits size and tokenizer vocabulary size do not have to
    match
-   `vocab.json` was not the whole tokenizer vocabulary for this model
-   a missing dictionary entry does not automatically mean the token is
    invalid
-   sometimes the right debugging move is to go backwards instead of
    patching forward
-   if an error changes after a fix, that change can be evidence that
    the fix actually worked

## Next

Now I am **actually** back in Chapter 5.

The next job is to make masking handle the model output positions that
do not correspond to real tokenizer tokens.

Those IDs should never be able to win token selection.

Once that is handled, I can continue testing real constrained generation
and finally keep moving through Chapter 5.

Hopefully without accidentally discovering that Chapter 2 has been
plotting against me this whole time.
