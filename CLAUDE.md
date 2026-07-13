# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A personal, self-taught curriculum vault for relearning EE and CS from scratch, structured as MIT OCW course numbers (18.01, 6.002, 6.006, etc.). There is no source code, build system, or test suite — every file is Markdown notes. "Working in this repo" means generating or reviewing lecture-note files against the rubric below, not writing/running software.

## Structure

- Each course lives in `<course-number>-<short-name>/` (e.g. `6.002-circuits-and-electronics/`).
  - `README.md` — course description, prerequisites, feeds-into, and a topic table with a `not started` / `in progress` / `done` status per topic.
  - `notes/NN-topic-slug.md` — one file per lecture/topic, zero-padded and sequential, matching the topic order in that course's README.
  - `notes/00-glossary.md` — running list of every term/symbol introduced in the course, with definition and the note that first defined it. Updated whenever a new note introduces a term.
- `README.md` (root) — master index of all courses across Phases 0–5, with the overall learning order. EE phases (0–4) come before CS (Phase 5); math/physics prerequisites are pulled in just-in-time when a later course needs them rather than front-loaded.

## The three governing docs — read before generating or reviewing any note

- **CONVENTIONS.md** — the zero-assumed-knowledge rule: never say "as you know / obviously / clearly / recall that" without the actual recap; every term defined inline or linked to the glossary on first use; every derivation shows every algebraic step; results borrowed from a not-yet-completed course get a self-contained inline recap, not just a link. Also covers notation consistency (fixed symbol per quantity within a course, cross-course clashes flagged explicitly) and the `[[course/notes/NN-topic-slug]]` cross-link style.
- **NOTE_TEMPLATE.md** — the required section structure for every note, in order: frontmatter (title, course, topic_number, prerequisites, status), Why this matters, Builds on, Core definitions, Intuition, Derivation/formalism, Worked examples (≥2), Common pitfalls, Self-check (questions then answers in a separate section), Summary/cheat sheet, Used later in.
- **RUBRIC.md** — the checklist a note must pass before its status flips to `done` in the course README (correctness, zero-assumed-knowledge, completeness, consistency). A topic's status only moves to `done` after this passes, not after a first draft exists — intermediate state is `in progress`. Higher-stakes notes (derivations, numeric examples) need a second, independent verification pass before flipping to `done`.

## Working conventions

- When writing a new note, follow NOTE_TEMPLATE.md's section order exactly and check it against RUBRIC.md before suggesting a status change.
- Update the relevant `00-glossary.md` whenever a note introduces a new term or symbol.
- Update the course `README.md` topic table and the root `README.md` phase table when a topic's status changes.
- Keep symbol usage consistent with what's already in a course's glossary — don't introduce a competing notation for a quantity that already has one.
