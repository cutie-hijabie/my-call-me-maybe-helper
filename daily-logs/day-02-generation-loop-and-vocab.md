# Day 2 – Generation loop and vocabulary mapping

Today I built the unconstrained generation loop and loaded the vocabulary file. Still no constraints yet — just getting the raw pieces working.

**What I got done:**

- Built the autoregressive generation loop from scratch. No `.generate()` helper, just raw calls to the SDK in a loop. It takes a prompt, encodes it to token IDs, then repeatedly asks the model for the next token's logits, picks the highest one, appends it, and repeats until it hits a maximum length.

- Tested the loop on a simple open-ended prompt and decoded the final token sequence back to text. It produced coherent natural language — not perfect, but readable. That confirmed the loop was actually working.

- Loaded the vocabulary file using the SDK's path getter. Opened the JSON file and looked at its structure. The keys are strings, the values are integers. That's the opposite of what I needed — I want to look up a token ID and get its string form. So I built a reverse mapping: token IDs as keys, strings as values.

- Tested the mapping by looking up a few token IDs manually. Token ID 1 maps to a double quote (`"`). That's going to be important later when I'm checking JSON grammar character by character.

**Building the generation loop:**

The loop itself is simple: encode the prompt, then repeatedly call `get_logits` with the current token sequence, pick the highest logit, append it, and repeat. Argmax is just finding the index with the highest value — no probabilities, no sampling.

One thing that took me a moment: the model has no memory between calls. You have to pass the entire sequence of token IDs every single time, not just the latest one. If you only pass the latest token, the model has no context and produces gibberish.

I used a maximum length as the stopping condition. Later I'll add EOS token detection too, once I figure out what the EOS token ID is for this model.

**Loading the vocabulary:**

The SDK gives you a path to the vocab file. I opened it and saw the structure: strings as keys, integers as values. I needed the reverse — look up a token ID and get its string. So I built a dictionary mapping integers to strings.

The string representations have some quirks. Some tokens have leading characters that indicate spaces. I noticed this when I peeked at a few entries. That's going to be important later because I'll be checking JSON syntax character by character, and the token's full string — spaces and all — is what I need to validate.

**A few things worth knowing:**

- The vocab mapping should be built once and reused. It's just a dictionary, so lookups are fast.
- The generation loop is deterministic — same prompt, same output every time.
- Decoding the final token sequence back to text is a good sanity check. If it looks like real text, the loop is working.

**One more thing:**

I also peeked at the tokenizer using the `transformers` library to find the EOS token ID. The SDK doesn't give you a direct way to get it, so loading the tokenizer separately and checking its EOS token and ID gives you a clean stopping condition if you want to stop when the model naturally ends a response.

**What clicked today:**

The generation loop is just `encode` → `get_logits` → `argmax` → `append` → repeat. That's the engine. Everything else — JSON grammar, schema constraints, masking — will plug into this loop by modifying the "pick the highest token" step.

The vocab file is the bridge between numeric token IDs and the actual text they represent. Having that bridge ready makes the next steps much easier.

**Tomorrow:**

JSON state machine. I need to track what characters are legal at every point during generation, according to JSON syntax rules. That means checking strings character by character and keeping track of where I am — inside an object, inside a string, expecting a colon, etc.
