# Days 4–7 – JSON state machine

This part took me four days.

The goal was to build the JSON grammar layer that will eventually control what the model is allowed to generate. Before token masking can happen, I first needed a state machine that understands JSON one character at a time.

This ended up being much bigger than I expected. JSON looks simple when you read it normally, but when you have to validate it character by character, every tiny rule suddenly matters.

## What I worked on

I built a state machine that keeps track of where the parser currently is while JSON is being generated.

Instead of validating a finished JSON string at the end, the machine processes one character at a time and decides whether that character is legal based on the current state.

The states represent situations such as:

* waiting for the beginning of a value
* waiting for an object key
* waiting for a colon
* waiting for a value
* currently inside a string
* currently inside a string escape
* currently reading a number
* currently reading a literal
* waiting for a comma or closing bracket after a value
* currently inside an array
* finished with a complete top-level JSON value

That became the foundation for everything else in this chapter.

## Objects and arrays

One of the first things I had to track was nested objects and arrays.

A single state is not enough for this, because when something closes, the parser needs to know what it is returning to. I used a bracket stack to remember which structures are still open.

This became especially important with nested values such as objects inside arrays, arrays inside objects, and multiple levels of nesting.

I also ran into an interesting object-specific problem: an empty object is valid, but a trailing comma is not.

That means a closing brace can sometimes be valid while the parser is waiting for a key, but only if the object was just opened. Once a comma has appeared, another key is required.

I added state to distinguish those situations.

## Strings

Strings ended up having several separate rules.

I needed to distinguish between a string being used as an object key and a string being used as a value, because closing the string leads to different next states.

A key string should lead to waiting for a colon.

A value string should lead to waiting for the end of the surrounding structure.

A top-level string should finish the JSON value completely.

I also added handling for escaped characters inside strings.

This includes normal JSON escapes as well as Unicode escapes.

Unicode escapes were interesting because after the Unicode marker appears, exactly four hexadecimal characters must follow before the parser can return to normal string processing.

Another edge case I caught later was raw control characters inside strings. A literal newline inside a JSON string is invalid even though an escaped newline is valid.

That was one of those rules that is easy to forget until a test exposes it.

## Numbers

Numbers were probably the most annoying part of the grammar.

I had to keep track of several things at once:

* whether at least one digit had appeared
* whether the number started with zero
* whether a decimal point had already appeared
* whether an exponent had appeared
* whether an exponent sign had appeared
* whether the parser was currently waiting for a required digit

This is where a lot of small invalid cases appeared.

For example, a leading zero cannot be followed by another normal digit.

A decimal point has to be followed by a digit.

An exponent also has to be followed by a digit, possibly after a sign.

One edge case that caught a bug was a number with a decimal point immediately followed by an exponent. The parser originally allowed the exponent even though the decimal portion had not received its required digit.

That forced me to think more carefully about the difference between “a digit has appeared somewhere in the number” and “a digit is required right now.”

## JSON literals

I added support for the three JSON literals:

* true
* false
* null

The machine tracks which literal is currently being read and which character should appear next.

The important part was that the literal has to be validated incrementally. A partial literal can still be a valid continuation even though it is not yet a complete JSON value.

That distinction became important again later when I started validating whole token strings.

## Whitespace

JSON whitespace can appear between structural parts of the document, so I added support for spaces, tabs, carriage returns, and newlines outside strings.

Whitespace inside a string is treated as part of the string instead.

This sounds small, but without it something as normal as an object containing spaces around its key, colon, or value would fail.

## Candidate validation

Once the character-level transition logic was stable, I added candidate validation.

This matters because the model does not generate one character at a time. It generates tokens, and one token can contain multiple characters.

So it is not enough to look at only the first character of a token.

Candidate validation takes the entire candidate string and feeds it through the state machine character by character.

If any character becomes invalid, the candidate is rejected.

A very important design decision was that validation happens on a copy of the current state machine.

Testing a possible candidate should not move the real parser forward.

This means I can ask, “Would this token be legal from here?” without changing the actual generation state.

I specifically tested candidate validation both from the beginning of JSON and from the middle of a partially built structure.

I also tested that the original machine remains unchanged afterward.

## Legal next characters

For debugging, I added a way to inspect which JSON-relevant characters are currently legal.

The idea is simple: take a small set of characters that matter to JSON grammar, test each one as a candidate, and keep the ones that are valid.

This includes structural characters, number characters, literal starters, whitespace, escape characters, Unicode escape characters, and hexadecimal digits.

This is not meant to be the final high-performance masking system.

It is mainly a way to inspect the grammar and understand what the machine believes is legal at a particular point.

That distinction is important because the actual masking work belongs to the next chapter.

## Trace debugging

I also added a trace helper because debugging a state machine without being able to see its internal movement is painful.

For every character in an input string, the trace records:

* the character that was processed
* the resulting parser state
* whether the character was accepted
* which JSON-relevant characters would be legal next

The trace works on a copy of the machine as well, so inspecting a possible input does not mutate the real parser.

This made it much easier to see exactly where the machine was going wrong instead of only seeing that an entire string failed.

## Testing the state machine

This chapter became very test-heavy.

I built a large collection of valid and invalid JSON examples and kept adding edge cases whenever something looked suspicious.

By the end of the chapter, the suite had more than two hundred passing tests.

The tests covered normal JSON as well as intentionally unpleasant inputs.

Some of the more useful bugs only appeared after the basic grammar was already passing.

One example was a top-level number followed by a closing array bracket.

The JSON was obviously invalid, but instead of returning false, the state machine crashed because it tried to inspect the top of an empty bracket stack.

That led me to audit places where the parser accessed the bracket stack and make sure the stack actually contained something first.

A similar problem appeared with a closing object brace.

Another interesting bug appeared with a comma after a top-level number.

The parser correctly handed the comma to the next state, and that next state correctly rejected it.

But the original branch ignored that rejection and returned success anyway.

That meant the invalid input was left in a state that looked complete.

The fix was conceptual more than complicated: when one state delegates processing of a character to another state, the result of that delegated transition has to be preserved.

I also tested the same idea with top-level literals to make sure the bug was not hiding elsewhere.

## Resetting the machine

Near the end, I reviewed the reset behavior.

Originally, resetting only the main state would not have been enough because the machine contains much more information than just the state enum.

It also tracks things such as open brackets, string context, number flags, literal progress, Unicode escape progress, and whether an empty object can currently close.

I updated reset so that a reset machine has exactly the same internal state as a brand-new machine.

I tested this by deliberately putting the parser into a complicated nested state, resetting it, creating a fresh machine, and comparing their internal state.

## What took the most time

The hard part was not writing transitions.

The hard part was realizing that JSON grammar is full of context.

A character can be valid in one place and completely invalid one character later.

A closing brace can mean “valid empty object,” “finished object value,” or “invalid trailing comma,” depending on how the parser arrived there.

A zero can be a perfectly valid number by itself but make the next digit illegal.

A plus sign is invalid at the beginning of a number but valid immediately after an exponent marker.

A newline is valid whitespace outside a string but invalid as a raw character inside one.

Most of the work was turning those contextual rules into state that the machine could remember.

## What clicked

The biggest thing that clicked for me was that constrained generation does not start with the model.

It starts with a parser.

Before I can tell the model which tokens are allowed, I need something that can answer a much simpler question reliably:

“Given everything generated so far, would this next piece of text keep the output valid?”

The state machine is that piece.

Another thing that clicked was why candidate validation must process the whole token.

A token may contain several characters, and the first character being valid says nothing about whether the rest of the token remains valid.

The state after each character matters.

I also started seeing the difference between parser state and model state much more clearly.

The model cares about token IDs and logits.

The JSON state machine does not care about the model at all. It only cares about characters and grammar.

Keeping those responsibilities separate made the architecture much easier to reason about.

## Chapter 4 result

By the end of these four days, I had a JSON grammar state machine that can process JSON character by character, track nested structures, handle strings and escapes, validate numbers and literals, accept legal whitespace, reject malformed continuations, validate multi-character candidates without mutating the real parser, expose legal next characters for debugging, trace state transitions, and reset cleanly.

Most importantly, I still have not added token masking yet.

That belongs to the next stage.

## Next

The next chapter is where this state machine finally connects back to the model.

The goal will be to look at the model's vocabulary, test possible token strings against the current grammar state, and mask tokens that would make the JSON invalid.

That is where constrained decoding actually begins.

