---
name: write-motivation
description: Draft or revise a course's MOTIVATION.md in this EECS curriculum vault — the one real problem the course solves, the topic arc as acts, and bridging questions into the next course. Use when a course has no MOTIVATION.md yet, or when reviewing one for wrong mental models.
metadata:
  tags: eecs-curriculum, notes, pedagogy
---

# Write a course MOTIVATION.md

Reference example — read it first to match tone and structure before writing a new one: `6.004-computation-structures/MOTIVATION.md`.

This file is a sibling of the course's `README.md` (not inside `notes/`). It is narrative, not rigorous — the opposite register from a topic note. A topic note (see the `write-notes` skill) derives things step by step; this file makes the reader want to do that derivation.

## Before writing

1. Read the root `README.md` to see the course's phase, what precedes it, and its `Feeds into` line.
2. Read the course's own `README.md` — its topic table, `Prerequisites`, and `Feeds into` lines.
3. Read `CONVENTIONS.md` — the zero-assumed-knowledge rule still applies loosely here (no unintroduced jargon without a quick gloss), even though derivation rigor doesn't.

## Structure

### 1. The problem
State exactly **one** concrete, real problem the whole course exists to solve — not a list of what the course covers. If you find yourself writing "this course covers X, Y, and Z," stop and find the single underlying problem that X, Y, and Z are all in service of.

### 2. What to expect
Reframe the topic table as a sequence of **acts**, grouping consecutive topics. Every act must end by naming the specific wall/limitation that the *previous* act's abstraction hits — the wall is what motivates the next act existing at all. Don't just summarize what each act covers; say why it had to come after the one before it.

### 3. Bridging questions
Write 3–5 questions that are **not answerable using only this course's material**. Each one should aim at a specific downstream course (pick candidates from the course's `Feeds into` line). Before finalizing each question, check: can it actually be answered with what's taught in *this* course? If yes, that's a bug — rewrite it so it genuinely requires the next course.

## Correctness constraint — do not plant wrong mental models

This is the failure mode that matters most in this file, more than in ordinary notes, because motivation prose reaches for simplifications and analogies. Every time you simplify something:
- State explicitly **which dimension/layer** is being simplified (don't let a simplification of one axis silently read as if a different axis were also simplified).
- Canonical example of the bug this guards against: describing a MOSFET as a "switch" without noting that voltage remains a continuous function of continuous time — that silently implies time itself became discrete, which is false and contradicts what the reader will learn in 6.003. The fix, in `6.004-computation-structures/MOTIVATION.md`, is the paragraph that separates "digital discretizes the value (codomain)" from "the clock is what discretizes time (domain)."
- If you're not sure whether a simplification is safe, prefer stating the fuller truth in one extra sentence over a clean but misleading line.

## Self-check before finishing

- [ ] Exactly one problem stated in section 1, not a covers-X-Y-Z list.
- [ ] Every act in section 2 names the wall the previous act hit.
- [ ] Every bridging question in section 3 genuinely requires next-course material — verified, not assumed.
- [ ] Every simplification explicitly flags which layer/dimension is simplified.
- [ ] No new file other than `<course>/MOTIVATION.md` was touched (this skill does not edit READMEs, notes, or CONVENTIONS/RUBRIC/TEMPLATE).
