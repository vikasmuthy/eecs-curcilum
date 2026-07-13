---
name: write-notes
description: Draft or revise a single topic note file (<course>/notes/NN-topic-slug.md) in this EECS curriculum vault, following CONVENTIONS.md, NOTE_TEMPLATE.md, and RUBRIC.md exactly. Use whenever a topic row in a course README needs its note written or an existing note reviewed.
metadata:
  tags: eecs-curriculum, notes, pedagogy
---

# Write or revise a topic note

## Before writing

Read, in this order:
1. `CONVENTIONS.md` — zero-assumed-knowledge rule, file naming, notation, cross-linking.
2. `NOTE_TEMPLATE.md` — the exact required section structure.
3. `RUBRIC.md` — the checklist the note must pass.
4. The target course's `README.md` — get the exact topic title/wording and its position in the topic table (this fixes both the note's title and its `NN` file-number prefix).
5. `<course>/notes/00-glossary.md` — terms and symbols already defined in this course. Reuse them; don't redefine or rename.
6. The immediately preceding note file in the same course, if one exists — inherit its notation exactly (same symbol per quantity) and use it to write an accurate `Builds on` section.

## Writing the note

- File name: `NN-topic-slug.md`, zero-padded, matching the topic's row position in the course README's topic table.
- Follow `NOTE_TEMPLATE.md`'s section order exactly: frontmatter, Why this matters, Builds on, Core definitions, Intuition, Derivation/formalism, Worked examples (≥2), Common pitfalls, Self-check (questions, then answers in their own section), Summary/cheat sheet, Used later in.
- Apply `CONVENTIONS.md`'s zero-assumed-knowledge rule literally: no "as you know / obviously / clearly / it can be shown" without the actual explanation following it; every term defined inline or linked to the glossary on first use; every derivation shows every algebraic step; any result borrowed from a not-yet-finished course gets a self-contained recap inline, not just a link.
- Keep symbol usage identical to what's already in `00-glossary.md` — don't introduce a competing notation for a quantity that already has one in this course.

## After writing

1. Append every new term/symbol introduced to `<course>/notes/00-glossary.md`, with its definition and a link to this note.
2. Set the topic's status in the course `README.md` to `in progress` — never `done`. Flipping to `done` requires the independent verification pass in `RUBRIC.md`'s "Verification method" section, which this skill does not perform.
3. Run the self-check pass below before returning.

## Self-check before finishing (RUBRIC.md, condensed)

- [ ] Every formula/derivation step checked — no algebra errors, no dropped signs/units.
- [ ] Worked examples produce numerically correct answers.
- [ ] Claims match the actual MIT OCW syllabus content for this topic — no invented results.
- [ ] No term used before it's defined (inline or via glossary link).
- [ ] No unexplained "as you know / obviously / clearly / it can be shown."
- [ ] Any result borrowed from an unfinished course is recapped inline, not just linked.
- [ ] All required NOTE_TEMPLATE.md sections present and non-empty; self-check spans easy → hard.
- [ ] Notation matches `00-glossary.md`; cross-links to prerequisite notes are present and correct; frontmatter fully filled in.
