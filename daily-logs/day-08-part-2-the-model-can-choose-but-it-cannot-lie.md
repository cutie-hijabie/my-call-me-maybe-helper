# Day 8 — Part 2: The Model Can Choose, But It Cannot Lie

Today was basically about giving the model rules without accidentally giving it the answers.

We started working on the schema-aware part of the project, and I was VERY suspicious of it at first because I do not want to overfit this thing to the provided test files. The reviewer can change the functions, parameters, and prompts, so absolutely nothing here should depend on `fn_add_numbers`, `fn_greet`, or any of the examples we currently have.

The model still has to make the actual decision.

Our job is just to make sure whatever it decides is **allowed by the schema it was given at runtime**.

## First: understanding the input

I started by actually modeling `functions_definition.json` instead of treating it like mysterious JSON soup.

I made Pydantic models for:

- parameter definitions
- return definitions
- individual function definitions
- the entire input file

The slightly confusing part was realizing that the file itself is a list of function definitions, so I ended up using a Pydantic `RootModel` for the whole thing.

Then we tested it against the actual provided file.

**Passed.**

So now the function definitions can be loaded and validated without hardcoding any particular function or parameter.

## Then came the schema state

I made a separate `SchemaState` instead of stuffing more responsibilities into the JSON state machine.

The distinction finally started making sense:

- the JSON state machine cares whether the output is valid JSON
- the schema state cares whether that valid JSON is legal according to the current function definitions

So far it tracks things like:

- where we are in the output
- the chosen function
- the current parameter
- the partially generated function name
- the partially generated parameter name

The JSON machine and schema tracker are going to work next to each other rather than becoming one giant terrifying state machine.

## The model can choose. It just can't invent things.

The first actual constraint I implemented was function-name prefix filtering.

If the model has generated:

`fn_g`

we look at the function names loaded from the current input and ask:

> Which names could this still legally become?

So something like:

`fn_greet` → yes  
`fn_get_weather` → yes  
`fn_reverse_string` → absolutely not babe

Importantly, this does **not** decide which function is correct for the prompt.

It only removes impossible choices.

That distinction matters a LOT because the model is eventually responsible for the semantic decision. We're not sneaking in heuristics like:

> prompt says "add" → use `fn_add_numbers`

Nope.

The available functions come from the runtime input, and the model still has to choose between them.

I also added exact-name recognition so that once the partial name actually equals one of the available function names, the schema state can remember that function as the selected one.

## Tests because I refuse to trust myself

We now have three schema tests:

- the provided function-definition file validates correctly
- function-name prefix filtering works
- incomplete names don't get accepted as complete functions, while exact matches do

And:

**3 passed.**

No explosions today.

## Where I stopped

The function-name side has its first working pieces.

Next time I'm basically doing the same thing on the parameter side: once a function has been chosen, its parameter names come from that function's runtime definition, and the model should only be able to generate parameters that actually belong to it.

Then we'll eventually have to deal with parameter **types**, which is where this starts connecting much more seriously to the JSON constraints.

So Chapter 6 has officially started becoming real code instead of me staring at the blueprint and going "okay but what does that MEAN."

Stopping here because it is 10:30 PM and being a functioning human being unfortunately requires sleep.
