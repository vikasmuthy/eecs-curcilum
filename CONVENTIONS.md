# Vault Conventions

Rules that apply to every course/note in this curriculum. Read this before generating or reviewing any note.

## Zero-assumed-knowledge rule

Every note is written as if the reader has only what's explicitly listed in that note's "Builds on" links — nothing else, no matter how "obvious." Concretely:
- Never write "as you know," "obviously," "clearly," "recall that" (unless immediately followed by the actual recap, not just the phrase).
- Every technical term is either defined inline on first use in a note, or linked to the earlier note/glossary entry that defines it.
- Every derivation shows every algebraic step — no skipped lines with "it can be shown that."
- If a note needs a result from another course (e.g. 6.002 needing a Laplace transform from 18.03 before 18.03 is finished), the result gets a one-paragraph self-contained recap inline, not just a link — the reader shouldn't have to context-switch to understand the current note.

## File structure

- One markdown file per topic in `<course>/notes/`, named `NN-topic-slug.md` (zero-padded, sequential, matching the course README's topic table order).
- `<course>/notes/00-glossary.md` — running list of every term/symbol introduced in that course, with definition and the note where it was first defined. Updated every time a new note introduces a term.
- Frontmatter on every note (see NOTE_TEMPLATE.md).

## Notation

- Inline math: `$...$`. Block math: `$$...$$`.
- Symbol consistency within a course is mandatory (e.g. always `v` for voltage, `i` for current, never switch to `V`/`I` mid-course). Cross-course notation clashes (e.g. `P` meaning power in EE vs. probability in 6.041) get flagged explicitly the first time they'd collide.
- SI units always stated on first numeric example in a note.

## Status tracking

A topic's status in a course README only moves to `done` after it passes the rubric in RUBRIC.md — not just after a first draft exists. Intermediate state is `in progress`.

## Cross-linking

- Every note has a "Builds on" section (links to prerequisite notes/concepts) and, once it exists, a "Used later in" backlink.
- Use `[[course/notes/NN-topic-slug]]`-style references in prose.
