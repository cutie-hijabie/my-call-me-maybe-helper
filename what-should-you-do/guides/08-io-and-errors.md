# 08 — Input, output and errors

## What it's for

Your program is run by peers and grading tools, potentially with different input files
and custom command-line arguments. The subject is strict: **it must never crash
unexpectedly and must always give clear error messages.**

## Think about

**The command line**

- The subject gives the exact way your program is launched and the three optional arguments.
  Which names do they have, and what are the defaults when one is missing?
- How do you parse command-line arguments properly instead of by hand?
- What does `python -m src` require of the `src` folder? (Find out what makes a package
  runnable this way.)

**Reading files**

- What can go wrong when opening an input file? List at least five things: missing, unreadable,
  a directory instead of a file, empty, not JSON, valid JSON but the wrong shape…
- For each of them: what should the user see, and what should the exit behavior be?
- How do you guarantee files are closed even if something fails halfway?

**Writing the output**

- The subject says one output file. Which path is the *default*? Read sections 4.3.2 and 5.4
  carefully. Version 1.2 names `data/output/function_calling_results.json` as the result path;
  some illustrations shorten it. A custom `--output` path is a separate CLI choice.
- Does the output directory exist when you start? What if it doesn't? What if the file already
  exists?
- Separate a valid file containing some successful records from a file truncated halfway
  through its JSON syntax. What is your batch policy, and when do you write the result?
  If preserving an existing file during a write failure matters, investigate atomic replacement.
- Is the output valid JSON *even if zero prompts succeeded*?

**Per-prompt failures**

- If one prompt fails, should the whole run stop? What's the best behavior for the user, and
  what do the subject's rules ("never crash", "clear error messages") suggest?

## Go find out

- `argparse`: required vs optional arguments, defaults, and help text.
- What is a **context manager** and why does the subject prefer them for files?
- How do you write JSON out *properly* (not by gluing strings together)?
- What does it mean to exit with a non-zero status, and when is that appropriate?

## Make partial success visible

Load and validate inputs before model setup where possible. Initialize shared resources
once and create fresh state for each request. If your policy continues after a request failure,
report it clearly and count it alongside successes; do not manufacture a normal function call.

A no-match request and a valid call are different outcomes. Any rejection mechanism should
leave semantic judgment to the model rather than request-specific Python heuristics. The
subject does not specify every failure outcome, so record your policy instead of claiming it
is the only compliant one.

**Checkpoint:** check bad input before model loading, a request failure followed by a success,
an output-write failure, and a zero-success batch. Match the saved file and exit status to
what your documentation promises.

## Resources

- [argparse](https://docs.python.org/3/library/argparse.html)
- [`python -m` (command line docs)](https://docs.python.org/3/using/cmdline.html#cmdoption-m)
- [The `with` statement](https://docs.python.org/3/reference/compound_stmts.html#with)
- [Python `json` module](https://docs.python.org/3/library/json.html)

---

[Folder overview](../README.md) · [Build route](../build-route.md) · [Previous guide](07-pydantic-and-validation.md) · [Next guide](09-performance.md)
