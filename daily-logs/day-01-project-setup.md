# Day 1 – Project setup and SDK wrapper

Today was all about getting the project structure right and actually talking to the SDK for the first time. No constraints yet, just making sure I can load the model and get logits out.

**What I got done:**

- Created `src/` folder, dropped the SDK into `llm_sdk/` at the root, and made sure imports work.
- Set up `uv` with `pyproject.toml` and locked dependencies. The lockfile keeps everyone on the same versions.
- Added a Makefile with `install`, `run`, `debug`, `clean`, `lint`, `fclean`, `re`. `install` runs `uv sync`, `clean` deletes `__pycache__` and `.mypy_cache`. `fclean` nukes the `.venv` entirely. `re` does a full rebuild. The subject requires these, might as well get them out of the way now.

**Setup decisions worth explaining:**

The SDK comes with its own dependencies (`torch`, `transformers`, `huggingface-hub`, etc.). Added those to `pyproject.toml`. Also pinned `numpy` to `<2.0` because the newer versions break compatibility with Python 3.10 — and the subject requires 3.10 or later. Easy to miss if you're developing on a newer Python yourself.

Made sure the SDK folder was at the root and not nested. The `__init__.py` has to be directly inside `llm_sdk/` so `from llm_sdk import Small_LLM_Model` actually works.

**The wrapper class I built (`src/llm_wrapper.py`):**

The SDK returns tensors and other objects that are annoying to work with directly.
The SDK methods are just encode, get_logits_from_input_ids, and a vocab file path getter — the wrapper passes through to those but hides the tensor types and exposes plain Python lists instead. That makes the rest of the code easier to work with.

**Tip:** Before you start writing your wrapper, look carefully at what each SDK method actually takes as parameters and what it returns. They're not all the same shape. One returns a tensor, another returns a list of floats, and one just gives you a string path. Think about what format you want each of them to return to the rest of your code. Once you know what you're working with, you can decide how to handle each one consistently.

If you're not sure what type something returns, print it. For example:

```python
print(type(self.llm.encode("hello")))
```

**Also:**

Added a `.gitignore` with the usual Python stuff, and explicitly excluded `data/output/` since the subject says not to commit generated output.

**The repo structure right now:**

The root has the SDK folder, the src folder, the Makefile, pyproject.toml with its lockfile, a python-version file, and the README. Clean and minimal.

**Tomorrow:**

Now that I can get logits for a single token step, I need to build the loop that keeps calling the model, selecting tokens, and feeding them back in. Unconstrained for now — JSON masking comes later.
