# Note Quality Rubric

A note is only marked `done` in its course README after passing every item below. Use this both when self-checking a note and when running a verification agent pass over it.

## Correctness
- [ ] Every formula/derivation step checked — no algebra errors, no dropped signs/units.
- [ ] Worked examples produce numerically correct answers.
- [ ] Claims match the actual MIT OCW syllabus content for that topic (no invented results).

## Zero-assumed-knowledge
- [ ] No term used before it's defined (inline or via glossary link).
- [ ] No "as you know / obviously / clearly / it can be shown" without the actual explanation following.
- [ ] Any result borrowed from a not-yet-completed course is recapped inline, not just linked.

## Completeness
- [ ] "Why this matters," "Builds on," "Core definitions," "Intuition," "Derivation," at least 2 worked examples, "Common pitfalls," "Self-check" (with answers), "Summary" — all sections present and non-empty.
- [ ] Self-check questions span easy → hard, not all trivial recall.

## Consistency
- [ ] Notation matches the course's `00-glossary.md` (same symbols as prior notes in this course).
- [ ] Cross-links to prerequisite notes are present and correct.
- [ ] Frontmatter fields filled in (title, course, topic_number, prerequisites, status).

## Verification method
1. Self-check pass: re-read the note fresh, tick every box above honestly.
2. For higher-stakes topics (anything with a derivation or numeric example): have a second pass — either you re-derive from scratch without looking at the draft and compare, or an independent review agent checks the note against this rubric and tries to break the worked examples/derivations.
3. Only flip status to `done` in the course README after both passes are clean.
