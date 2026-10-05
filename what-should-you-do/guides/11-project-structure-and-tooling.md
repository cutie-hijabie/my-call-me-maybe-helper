# 11 — Project structure and tooling

## What it's for

This project is graded partly on how it's set up: dependencies, linting, typing, docstrings,
the Makefile, and what's in the repository. Checking these early helps avoid discovering a setup or compliance gap after generation works.

## Think about

**The repository**

- Which files and folders does the subject say the repository **must** contain? Make a checklist
  from the Submission chapter.
- What does the subject say must **not** be in the repository? How do you make sure generated
  output never gets committed? What does the `.gitignore` need to cover?
- Where does the provided `llm_sdk` go, and why?

**Dependencies with uv**

- The reviewer and the moulinette will **just run `uv sync`**. What does that tell you about
  which files must be committed so that command works from a clean checkout?
- Which packages does *your* code import? Which does the SDK import internally? How will the
  SDK's own requirements end up installed when only `uv sync` is run? (Look at what the SDK ships
  with it.)
- Read the "forbidden" list in 4.3.1 *exactly*. What may your own files import? What may not?
  Be ready to explain the line you drew.
- Which Python version does the subject require? What does your project declare, and does the
  reviewer's machine need anything special?

**Makefile**

- Which rules are mandatory, and which is optional? What must each one do?
- The subject lists the exact flags for `lint`. Does `make lint` pass when run on the **whole
  repository** — including the copied SDK folder? What are your options if the SDK code isn't
  clean? Investigate the actual failures and applicable requirements. Running only on `src/`
  does not demonstrate that the subject's whole-repository commands passed.

**Code quality**

- What does flake8 check, and how do you configure it if you need to?
- What does mypy's `--disallow-untyped-defs` mean in practice for every function you write?
- What is a good docstring under PEP 257? What do the subject's two named styles (Google, NumPy) look like?
- What is a context manager, and where in this project should you use one?

## Go find out

- How does `uv` create the environment, and what does the **lock file** record?
- What does `python -m src` need inside `src/`?
- What does `.PHONY` do in a Makefile, and which of your rules should be phony?

## Check the exact required commands

Version 1.2 requires the install, run, debug, clean, and lint Makefile targets. Its lint
commands cover the repository: `flake8 .` and `mypy .` with `--warn-return-any`,
`--warn-unused-ignores`, `--ignore-missing-imports`, `--disallow-untyped-defs`, and
`--check-untyped-defs`. A stricter optional target does not replace those required checks.

Keep the Python version requirement (3.10 or later), dependency declarations/lockfile, the supplied
SDK, input examples, typing, and docstrings in view. A locally working environment is not enough.

**Checkpoint:** create a clean checkout, run `uv sync`, and follow the documented commands.
Check actual exit codes and repository contents; do not hide failed tools.

## Resources

- [uv docs](https://docs.astral.sh/uv/)
- [flake8](https://flake8.pycqa.org/)
- [mypy docs](https://mypy.readthedocs.io/)
- [PEP 257 — Docstring conventions](https://peps.python.org/pep-0257/)
- [GNU Make manual](https://www.gnu.org/software/make/manual/make.html)

---

[Folder overview](../README.md) · [Build route](../build-route.md) · [Previous guide](10-testing-and-debugging.md) · [Next guide](12-readme-and-defense.md)
