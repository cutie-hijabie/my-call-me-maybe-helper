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

The SDK returns tensors and other objects that are annoying to work with directly. I made a thin wrapper that exposes plain Python types instead:

- `wrapper_encode(text: str) -> List[int]` – returns a flat list of token IDs. `.tolist()[0]` flattens the tensor into a list.
- `wrapper_get_logits_from_input_ids(input_ids: List[int]) -> List[float]` – returns raw logits as a list of floats. Length = vocab size, which for Qwen3-0.6B is around 150k.
- `wrapper_get_path_to_vocab_file() -> str` – returns the path to the vocab JSON file.

Tested each one with tiny inputs. `wrapper_encode("hello world")` gave me 5 or 6 IDs, not 2 — that's the subword tokenizer splitting things up. Good to see it in action early.

**Also:**

Added a `.gitignore` with the usual Python stuff, and explicitly excluded `data/output/` since the subject says not to commit generated output.

**The repo structure right now:**
"""
.
├── llm_sdk/
│ ├── init.py
│ ├── pyproject.toml
│ └── uv.lock
├── src/
│ └── llm_wrapper.py
├── Makefile
├── pyproject.toml
├── .python-version
├── README.md
└── uv.lock
"""

**Tomorrow:**

Now that I can get logits for a single token step, I need to build the loop that keeps calling the model, selecting tokens, and feeding them back in. Unconstrained for now — JSON masking comes later.
