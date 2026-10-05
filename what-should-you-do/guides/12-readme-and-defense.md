# 12 — README and review

## What it's for

This project's README has **extra required sections** beyond the usual ones, and the review
includes a possible small modification and questions about *why* your decoder works.

## The README

From the subject's "Readme Requirements" chapter, your README must be **in English** and include:

- The exact **italic first line** crediting the author(s) (copy the wording from the subject).
- **Description** — the goal and a brief overview.
- **Instructions** — how to install and run.
- **Resources** — classic references, **plus an honest description of how AI was used** (which
  tasks, which parts).
- **Algorithm explanation** — your constrained decoding approach, *in detail*.
- **Design decisions** — the key choices in your implementation.
- **Performance analysis** — accuracy, speed, reliability.
- **Challenges faced** — what went wrong and how you solved it.
- **Testing strategy** — how you validated it.
- **Example usage** — clear examples of running the program.

### Think about

- Could someone who's never seen the project reproduce your *idea* from the algorithm section
  alone? Where would they get lost?
- For each design decision: what was the alternative, and why did you reject it?
- Do you have real **numbers** for the performance analysis (accuracy, time)? Where did they come from?
- Is your AI paragraph specific? "I asked AI to explain how byte-level BPE works" is a very
  different statement from "AI wrote my mask logic."

## The defense

- Input files **change** during review. Hardcoding based on the examples fails immediately.
- You may be asked for a **small modification**: a minor behavior change, a few lines to
  write or rewrite, or an easy-to-add feature (the subject's examples include adjusting a data
  structure or changing a display). Practice an appropriately small change to your own design.
  Adding a type may be larger than a few minutes depending on the representation; no particular
  feature is guaranteed to be the review task.
- Expect questions like:
  - Which invariants make completed output valid, and how do you handle incomplete generation?
  - What happens when no token is valid at some step?
  - How does your decoder deal with a token that spans two structural pieces?
  - Where exactly does the LLM choose the function?
  - What would you change to make it faster?
- Prepare a diverse set of tests you can run in front of the reviewer.

## Go find out

- Pick any three functions in your code and ask: "Why does this exist? What breaks if I delete it?"
- Explain your decoder out loud to a peer who hasn't seen the code. Where do you stumble? Study that part again.

## Document your own implementation

Explain what the program actually does, including its failure policy and measured limits.
Label architectural decisions as decisions. Keep semantic accuracy, structural validity, and
runtime evidence separate, with the sample size and environment used.

**Checkpoint:** trace one successful request and one handled failure to a peer. Locate function
selection and candidate-state simulation in your explanation, then try a small appropriate
change. Update the README when behavior changes.

## Resources

- [GitHub: About READMEs](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-readmes)

---

[Folder overview](../README.md) · [Build route](../build-route.md) · [Previous guide](11-project-structure-and-tooling.md) · [Next guide](13-common-pitfalls.md)
