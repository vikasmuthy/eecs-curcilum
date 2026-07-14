---
name: drill-bank
description: Generate a large bank of medium-to-hard, worded (scenario-based) practice problems for one specific topic note in this EECS curriculum vault, with zero solutions provided. Use when a topic is done/in-progress and the user wants deliberate, effortful drilling to build muscle memory, not another guided walkthrough.
metadata:
  tags: eecs-curriculum, notes, pedagogy, drilling
---

# Build a topic drill bank

This is deliberately the opposite of a note's Self-check section: no answers, no hints, no worked steps. The point is productive struggle — the user works these alone, over time, and finds out whether they're right the same way a working engineer does: by checking their own answer against the topic's self-consistency method, not by looking one up.

## Scope

One drill bank targets exactly one topic note (course + topic number or slug), given as the invocation argument (e.g. "6.002 KVL/KCL" or "6.004 topic 2"). Do not generate banks for multiple topics in one invocation.

## Before writing

1. Read the target note in full (`<course>/notes/NN-topic-slug.md`) — its Core definitions, Derivation, Worked examples (especially their "Check" steps), and Common pitfalls. The problems must be answerable using *only* what that note (and its prerequisites) actually established — no reaching for material from later topics.
2. Read `<course>/notes/00-glossary.md` for this course, to keep notation identical to what the note uses (same symbols for the same quantities).
3. Note the topic's **intrinsic self-verification method** — the way the note's own worked examples confirm an answer is right (e.g., KCL closure check, perfect induction against a truth table, dimensional/limiting-case sanity check, re-deriving via an independent path). Every drill bank needs one of these stated up front; it is not an answer, it's a way to know if you're lying to yourself.

## Writing the drill bank

- File: `<course>/notes/NN-topic-slug.drills.md`, sibling to the note itself. Not part of NOTE_TEMPLATE.md's structure — this is a separate artifact.
- Open with one short paragraph: which self-verification method applies to this topic's problems, stated once, not per-problem.
- Problems must be **worded** — a scenario or description in prose that the user has to first translate into the formal representation (a circuit from a component/connection description, a function from a behavioral spec, etc.), not an already-formalized expression to manipulate. This mirrors how real problems arrive, not how textbook drills do.
- Difficulty: medium-to-hard only. Skip anything a Self-check "Easy" question in the note already covers — this bank exists for the harder tier, not general review.
- **Zero solutions, zero hints, zero worked steps.** Just numbered problems. If a problem needs specific numeric values, state them precisely (units included) so the problem is well-posed, but do not show or imply the path to the answer.
- Volume: aim for a large bank (up to ~100), but scale to how many genuinely distinct hard scenarios the topic actually supports — don't pad a narrow topic with reskinned duplicates just to hit a number. Generate in batches (e.g. 4×25) rather than one pass, and deliberately vary the scenario/cover story across batches so problems don't start blurring into the same setup with different numbers swapped in.
- Do not create an answer key anywhere, hidden or otherwise, unless the user explicitly asks for one in a given invocation.

## After writing

- Do not touch the topic's status in the course README or the note's frontmatter — a drill bank is independent of the RUBRIC.md verification pipeline and never changes done/in-progress status.
- Report back the problem count and a one-line description of how the batches vary, so the user can judge coverage before grinding through it.
