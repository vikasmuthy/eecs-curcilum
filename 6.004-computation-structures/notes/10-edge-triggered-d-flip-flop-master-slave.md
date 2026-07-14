---
title: "Edge-Triggered D Flip-Flop (Master-Slave Construction)"
course: "6.004"
topic_number: 10
prerequisites: ["6.004 Topic 09 — D Latch and the Transparency Problem", "6.004 Topic 08 — Bistability & the SR Latch"]
status: done
---

# Edge-Triggered D Flip-Flop (Master-Slave Construction)

## Why this matters

Topic 09 ended by hand-patching the transparency problem for one specific case (a 2-latch shift register) using two complementary clocks, $\text{CLK}$ and $\overline{\text{CLK}}$, wired to two separate latches. This note takes exactly that patch and repackages it as a single, self-contained component — the **edge-triggered D flip-flop** — so that every future topic in this course (registers, Topic 25; pipeline stages, Topics 31–33) can treat "store one bit, sampled once per clock cycle" as a primitive, without every designer having to re-derive the complementary-clock trick from scratch or risk wiring latches together incorrectly. The core claim this note proves is that Topic 09's fix, viewed as a single 2-latch unit with one external clock, behaves as if it samples $D$ at a single *instant* (a clock edge) rather than over an interval — which is precisely the property "level-sensitive" latches lack and every synchronous digital system (everything from Topic 24 onward) is built to assume.

## Builds on

- [[6.004-computation-structures/notes/09-d-latch-transparency-problem]] — the D latch's transparent/hold behavior (Section 1's Claims 2–3) and the transparency problem itself; this note's entire construction is Topic 09 Self-check Questions 7–8's complementary-clock fix, now formalized as a standalone component with a single external clock input instead of two independently-supplied clock signals.
- [[6.004-computation-structures/notes/08-bistability-sr-latch]] — the underlying bistable core inside each D latch, and the Hold/Set/Reset case analysis reused (via Topic 09) inside both stages of the construction below.

## Core definitions

- **Edge-triggered device** — a sequential circuit whose output changes (captures a new value) only at the instant a control signal transitions between logic levels (an **edge**), not throughout any sustained level; contrast with **level-sensitive** (Topic 09), where the device is live for an entire high or low interval.
- **Rising edge / positive edge** — the instant a clock signal $\text{CLK}$ transitions from $0$ to $1$.
- **Falling edge / negative edge** — the instant $\text{CLK}$ transitions from $1$ to $0$.
- **Positive-edge-triggered D flip-flop** — an edge-triggered memory element with data input $D$, clock input $\text{CLK}$, and output $Q$, satisfying: $Q$ takes on the value $D$ had *at the moment of the most recent rising edge* of $\text{CLK}$, and holds that value unchanged at every other time (including while $\text{CLK}$ is at a steady $0$ or steady $1$, and even if $D$ changes freely in between edges).
- **Master-slave construction** — the standard implementation of an edge-triggered flip-flop from two level-sensitive D latches (Topic 09) in series, called the **master** (first stage, enabled while $\text{CLK}=0$ in the positive-edge-triggered version derived below) and the **slave** (second stage, enabled while $\text{CLK}=1$), with the master's output feeding the slave's input and the two stages driven by complementary clock phases ($\text{CLK}$ and $\overline{\text{CLK}}$) so they are never simultaneously transparent — exactly Topic 09's fix, now named and fixed as a single reusable two-stage unit.
- **Characteristic equation** — the compact notation $Q^{+} = D$ (read: "the flip-flop's *next* state, $Q^+$, equals $D$'s value at the triggering edge"), used to describe an edge-triggered device's behavior independent of any particular gate-level implementation; contrasted below with the D latch's transparent behavior, which has no analogous "next state" notion since its output isn't defined only at discrete moments.

## Intuition

**An edge-triggered flip-flop is "Topic 09's two-complementary-clock fix," given a name and a single clock pin.** Nothing new is invented here at the level of components — it is exactly two D latches, exactly the complementary-clock wiring Topic 09 Self-check Question 7 worked out, packaged so that from the *outside* it looks like it has one clock input (the two internal phases, $\text{CLK}$ and $\overline{\text{CLK}}$, are generated internally from that one input by a single inverter) and behaves as if the whole 2-latch pair "samples $D$ instantaneously" at the moment $\text{CLK}$ rises.

**Why two latches, not one, are needed to get edge-triggered behavior out of level-sensitive parts.** A single D latch is fundamentally level-sensitive (Topic 09) — there is no way to make one latch alone ignore $D$ except during a single instant, because "enabled" and "disabled" are its only two modes, and "enabled" necessarily lasts for a nonzero interval (however short) in any real circuit. The master-slave trick sidesteps this by never actually needing a true single-instant sample: instead, it arranges that whichever latch is currently transparent, the *other* one is holding, so a value can only ever cross from master to slave during the narrow "changeover" moment when the roles swap — and because the slave is genuinely blind to the master's changes during the whole rest of the master's transparent phase, only the master's value *at that changeover point* (i.e., right when $\text{CLK}$ transitions) ever makes it through to the visible output $Q$. That changeover point is exactly the clock edge.

**The half of the pulse where the master is "listening" is invisible from the outside.** During the first half of a clock cycle (master transparent, slave holding), the master's internal node is busy tracking $D$'s every wiggle — exactly like a plain D latch — but none of that is visible on the flip-flop's actual output $Q$, because the slave is frozen. Only at the instant the roles swap does whatever the master happened to be holding (i.e., $D$'s value right at that instant) get handed to the slave, which then displays it and refuses to change again until the *next* such handoff. This is why the flip-flop's *external* behavior looks like a single instantaneous sample, even though internally it's built entirely from Topic 09's continuously-tracking, level-sensitive latches.

## Derivation / formalism

### 1. The master-slave wiring, and mapping it onto Topic 09's fix directly

Let Master $=$ a D latch (Topic 09) with data input $D$ (the flip-flop's external data input) and enable $\overline{\text{CLK}}$ (i.e., transparent while $\text{CLK}=0$). Let Slave $=$ a second D latch with data input $=$ Master's output $Q_M$, and enable $\text{CLK}$ (transparent while $\text{CLK}=1$). The flip-flop's external output is the Slave's output, $Q \triangleq Q_S$.

This is exactly Topic 09 Self-check Question 7's fix (Master $\leftrightarrow$ "Latch 1," Slave $\leftrightarrow$ "Latch 2," with the earlier note's $\text{CLK}$ swapped for $\overline{\text{CLK}}$ on the first stage — the specific assignment of which stage gets $\text{CLK}$ vs. $\overline{\text{CLK}}$ is what determines whether the resulting device triggers on the rising or falling edge, addressed in Section 3). By that question's own answer (already proved in Topic 09, reused here rather than re-derived): whenever $\text{CLK}=0$, the Master is transparent ($Q_M=D$, tracking continuously) while the Slave holds (frozen at whatever it last captured); whenever $\text{CLK}=1$, the Master holds (frozen at whatever $D$-value it had right when $\text{CLK}$ rose) while the Slave is transparent, so $Q_S = Q_M$ throughout — and since $Q_M$ is itself frozen during this entire interval, $Q_S$ takes on that one frozen value and doesn't change again until the next time $\text{CLK}$ falls and rises.

### 2. Formal proof that $Q$ changes only at rising edges of $\text{CLK}$, and captures $D$'s value at that instant

**Claim: for the wiring of Section 1, $Q$ (the Slave's output) only ever changes value at the instant $\text{CLK}$ transitions $0\to1$ (a rising edge), and the new value it takes is $D$'s value at that same instant.**

**Proof, by cases on $\text{CLK}$'s behavior:**

*While $\text{CLK}$ stays at $0$ (no transition):* Master is transparent (tracks $D$), Slave holds (Topic 09 Claim 3, applied to the Slave with its enable $=\text{CLK}=0$). Since the Slave's enable is $0$ throughout this stretch, $Q_S=Q$ does not change, regardless of what the Master ($Q_M$, tracking $D$) is doing. So $Q$ is constant during any interval where $\text{CLK}$ stays at $0$.

*While $\text{CLK}$ stays at $1$ (no transition):* Master holds (Topic 09 Claim 3, enable $=\overline{\text{CLK}}=0$ throughout), so $Q_M$ is frozen at whatever value it had the instant $\text{CLK}$ became $1$. Slave is transparent (enable $=\text{CLK}=1$), so $Q=Q_S=Q_M$ continuously — but since $Q_M$ itself is frozen (not changing) throughout this stretch, $Q$ is also constant (equal to that one frozen $Q_M$ value) for the entire stretch, even though the Slave is "transparent" in the level-sensitive sense — it simply has nothing changing to be transparent *to*.

*At the instant $\text{CLK}$ transitions $0\to1$ (rising edge):* Immediately before the transition (Master still transparent, per the "$\text{CLK}$ stays at $0$" case just above, holding right up to the transition instant), $Q_M$ equals $D$'s value at that instant (Topic 09 Claim 2, transparency). The transition switches the Master to hold (freezing $Q_M$ at exactly that value — $D$'s value at the transition instant) and simultaneously switches the Slave to transparent, so $Q$ immediately takes on this newly-frozen $Q_M$ value, i.e. $Q$ becomes $D$'s value as of the rising-edge instant. This is the one and only point in the cycle where a *new* value can reach $Q$, per the two preceding cases (which showed $Q$ is constant throughout both the $\text{CLK}=0$ stretch and the $\text{CLK}=1$ stretch individually).

*At the instant $\text{CLK}$ transitions $1\to0$ (falling edge):* Slave switches from transparent to hold — freezing $Q$ at whatever it currently is (no new information enters, since the Slave is simply locking in its current value as it stops tracking $Q_M$). Master switches from hold to transparent, resuming tracking of $D$ — but this doesn't affect $Q$ at all, since $Q$ only reflects $Q_M$ while the Slave is transparent, and the Slave has just stopped being transparent.

Combining all four cases: $Q$ only changes at $0\to1$ transitions of $\text{CLK}$, and the value it changes *to* is exactly $D$'s value at that transition instant. This is exactly the Core definition of a positive-edge-triggered D flip-flop, and it is the formal justification for the characteristic equation $Q^+=D$ (interpreting $Q^+$ as "the value $Q$ takes immediately after the next rising edge"). $\blacksquare$

### 3. Why the Master/Slave clock assignment determines trigger polarity

Section 1's assignment (Master enabled by $\overline{\text{CLK}}$, Slave enabled by $\text{CLK}$) produced positive-edge triggering (Section 2). Swapping the assignment — Master enabled by $\text{CLK}$, Slave by $\overline{\text{CLK}}$ — produces, by the fully symmetric argument (relabel $\text{CLK}\leftrightarrow\overline{\text{CLK}}$ throughout Section 2's proof, which is valid since the proof never used any property of $\text{CLK}$ beyond "some signal and its complement, assigned to the two stages"), a **negative-edge-triggered** flip-flop: $Q$ changes only at $1\to0$ transitions, capturing $D$'s value at that instant. This is why real flip-flop components come in both polarities, and why datasheets/schematics must specify which — it's a direct, one-bit consequence of which of the two complementary phases drives the master vs. the slave.

## Worked examples

### Example 1 — Full timing trace: positive-edge-triggered flip-flop tracking a changing $D$

Using Section 1's construction, trace $D, \text{CLK}, Q_M, Q$ through two full clock cycles, with $D$ changing at various points (including mid-level, to demonstrate the flip-flop ignores those changes except exactly at rising edges):

| Event | $D$ | $\text{CLK}$ | $Q_M$ (Master, tracks $D$ while $\text{CLK}=0$; frozen while $\text{CLK}=1$) | $Q$ (Slave/output, tracks $Q_M$ while $\text{CLK}=1$; frozen while $\text{CLK}=0$) |
|---|---|---|---|---|
| Start | 0 | 0 | 0 (tracking $D$) | 0 (held from before) |
| $D\to1$ (still $\text{CLK}=0$) | 1 | 0 | 1 (tracks new $D$ immediately — Master transparent) | 0 (unchanged — Slave holding) |
| Rising edge ($\text{CLK}:0\to1$) | 1 | 1 | 1 (frozen at $D$'s value right at the edge) | **1** (captures $Q_M$ — this is the moment $Q$ updates) |
| $D\to0$ (still $\text{CLK}=1$) | 0 | 1 | 1 (unchanged — Master now holding, ignores this $D$ change) | 1 (Slave transparent, but $Q_M$ isn't changing, so $Q$ doesn't either) |
| Falling edge ($\text{CLK}:1\to0$) | 0 | 0 | 0 (Master resumes tracking, immediately grabs current $D=0$) | 1 (frozen — Slave stops tracking right as this happens) |
| Rising edge ($\text{CLK}:0\to1$) | 0 | 1 | 0 (frozen at $D=0$, the value right at this edge) | **0** (captures new $Q_M$) |

Boldface marks the two moments $Q$ actually changes — both, and only, at rising edges of $\text{CLK}$, exactly matching Section 2's proof; note the mid-pulse change of $D$ from $1\to0$ while $\text{CLK}=1$ (fourth row) has *no* effect on $Q$, in sharp contrast to Topic 09 Example 2's plain D latch chain, where a mid-pulse change did propagate through.

### Example 2 — Contrast with Topic 09's broken 2-latch shift register, same $D_1$ sequence

Re-run Topic 09 Example 2's exact input sequence ($D_1$: starts $0$, becomes $1$ mid-pulse, stays $1$) through a single edge-triggered flip-flop instead of two independently-clocked plain D latches, to make the fix's effect concrete.

Topic 09 Example 2 found: $Q_2$ ends the pulse at $1$, incorrectly matching $Q_1$'s current-pulse value (transparency problem — the intended one-cycle delay collapsed to zero).

Using this note's flip-flop instead (single device, internal master-slave): the "capture instant" is the rising edge that begins the pulse. Per Section 2's proof, $Q$ becomes $D$'s value *exactly at that rising edge* — which, per Topic 09 Example 2's own trace, was $D_1=0$ (the value at $t_1$, pulse start) — and then $Q$ stays at $0$ for the rest of the pulse, *not* changing to $1$ when $D_1$ changes mid-pulse (Example 1 above, fourth row, shows this exact mid-pulse-ignore behavior). So the edge-triggered flip-flop correctly captures the *pre-pulse* value of $D$ and holds it steady through the rest of the pulse — precisely the one-cycle-delayed sampling a shift register needs, achieved here by a single component with one clock input, rather than Topic 09's two-latch-with-manually-supplied-complementary-clocks patch.

## Common pitfalls

- **Believing an edge-triggered flip-flop is a fundamentally different kind of primitive from a D latch, rather than two D latches wired a specific way.** Section 1's construction is exactly two ordinary Topic 09 D latches; the "edge-triggered" behavior is an emergent property of the complementary-clock wiring (Section 2's proof), not a new kind of transistor-level mechanism — same conclusion Topic 09 itself foreshadowed with its own complementary-clock fix.
- **Assuming the flip-flop's output can be affected by $D$ changing at any point during the high half of the clock cycle.** Section 2's second case and Example 1's fourth row show explicitly that once the rising edge has passed, the Master is holding and completely deaf to further changes in $D$ until the *next* rising edge — a change in $D$ during the "wrong" half-cycle simply never reaches $Q$, not even partially or transiently.
- **Getting the trigger polarity backward.** Which edge a master-slave flip-flop triggers on depends entirely on which stage (master or slave) is wired to $\text{CLK}$ directly versus $\overline{\text{CLK}}$ (Section 3) — swapping this swaps positive-edge for negative-edge triggering; always check (or explicitly state) which convention a given schematic/datasheet uses rather than assuming.
- **Forgetting that the master stage is itself a plain, fully transparent D latch during its enabled half-cycle.** It's tempting to think of "the master" as already edge-triggered on its own — it is not; by itself the master has exactly Topic 09's ordinary level-sensitive behavior, and it is only the *combination* with the oppositely-clocked slave that produces edge-triggered behavior for the pair as a whole.
- **Confusing the characteristic equation $Q^+=D$ with a claim that $Q$ instantaneously equals $D$ at all times.** $Q^+=D$ specifically describes the value $Q$ takes on *after the next triggering edge*, not a continuous relationship — unlike the D latch's transparent-phase behavior ($Q=D$ literally, in real time, while enabled), the flip-flop's $Q$ and $D$ are equal only at (and briefly after) triggering edges, and otherwise $Q$ can differ from $D$'s current value for the rest of the cycle (as in Example 1's fourth row, $D=0$ but $Q=1$).
- **Using two D latches with the *same* clock phase (not complementary) and calling the result a flip-flop.** That configuration is exactly Topic 09's broken 2-latch chain (Topic 09 Example 2) — it is the *complementary* clocking (Section 1, one stage on $\text{CLK}$, the other on $\overline{\text{CLK}}$) that is essential; using the same clock signal on both stages reproduces the transparency problem rather than curing it.

## Self-check

### Questions

1. (Easy) In a positive-edge-triggered D flip-flop, if $\text{CLK}$ has been steady at $1$ for a while and $D$ changes, does $Q$ change? Why or why not?
2. (Easy) Which stage (master or slave) is transparent while $\text{CLK}=0$, in the Section 1 construction?
3. (Medium) Using Section 2's case analysis, explain what happens to $Q_M$ and $Q$ at the instant of a *falling* edge ($\text{CLK}:1\to0$) — does $Q$ change at this instant? Does $Q_M$?
4. (Medium) A negative-edge-triggered flip-flop (Section 3's swapped assignment) has $D=1$ at the moment of a falling edge, and $D$ then changes to $0$ before the next falling edge. What is $Q$ immediately after the falling edge, and does it change again before the next falling edge?
5. (Medium) Explain in one or two sentences, referencing the Intuition section, why the master-slave flip-flop only needs one external clock signal even though internally it requires two complementary clock phases.
6. (Hard) Using Example 1's trace as a template, construct a full timing trace for a *negative*-edge-triggered flip-flop (Master enabled by $\text{CLK}$, Slave by $\overline{\text{CLK}}$, per Section 3) given the same $D,\text{CLK}$ sequence as Example 1. State at which rows $Q$ changes, and confirm they are exactly the falling-edge rows rather than the rising-edge rows.
7. (Hard) Prove, using Section 2's proof structure (the four cases: $\text{CLK}$ steady at 0, steady at 1, rising edge, falling edge), that a positive-edge-triggered flip-flop's output $Q$ is completely unaffected by any number of extra transitions in $D$ that occur strictly between two consecutive rising edges of $\text{CLK}$ — i.e., only $D$'s value at the rising edge itself matters, no matter how many times $D$ wiggled in between.
8. (Hard) Explain why chaining two edge-triggered D flip-flops (not plain D latches) directly, $D_2=Q_1$, both sharing the *same* (non-complementary) clock signal, does *not* suffer from Topic 09's transparency problem — i.e., why this direct chain correctly implements a 2-cycle shift register, in contrast to Topic 09 Example 2's broken 2-latch chain. Use Section 2's proof (specifically, that $Q$ only changes exactly at the rising edge, never during the rest of the cycle) as the basis of your argument.

### Answers

1. No, $Q$ does not change. Per Section 2's second case ("$\text{CLK}$ stays at $1$"), while $\text{CLK}=1$ the Master is holding (frozen since the moment $\text{CLK}$ rose) and the Slave, though transparent, is only tracking the Master's already-frozen value — so no new information from $D$ can reach $Q$ until the next rising edge.
2. The Master (Section 1: Master's enable is $\overline{\text{CLK}}$, so it is transparent exactly when $\text{CLK}=0$).
3. Per Section 2's fourth case: at the falling edge, the Slave switches from transparent to hold, freezing $Q$ at its current value — $Q$ does *not* change at this instant, it simply locks in whatever it already was. Simultaneously, $Q_M$ (the Master) switches from hold to transparent and immediately begins tracking $D$ again — so $Q_M$ *can* change at/after this instant (following $D$), but this has no effect on $Q$ since the Slave is no longer listening to $Q_M$.
4. By the negative-edge analogue of Section 2 (swap $\text{CLK}\leftrightarrow\overline{\text{CLK}}$ throughout): $Q$ takes on $D$'s value exactly at the falling edge, which is $1$. Since the flip-flop only updates $Q$ at falling edges (by the swapped Section 2 proof, the direct analogue of the positive-edge case), the later change of $D$ to $0$ (occurring sometime before the *next* falling edge, but not exactly at a falling edge) has no effect — $Q$ stays at $1$ until the next falling edge occurs.
5. Because the two complementary phases, $\text{CLK}$ and $\overline{\text{CLK}}$, are generated internally from the single external $\text{CLK}$ input by one inverter (implicit in the construction, exactly as Topic 04's inverter generates a complemented control signal elsewhere in this course) — the user/designer only ever supplies one clock wire; the second, complementary phase is manufactured inside the flip-flop component itself, not something the external circuit needs to provide separately (unlike Topic 09's Question 7 fix, which assumed both phases were already separately available signals).
6. Reusing Example 1's exact $D,\text{CLK}$ sequence unmodified would have every capture instant land on the same value ($D=0$ both times), so no bit of $Q$ would actually be seen to flip — that wouldn't genuinely demonstrate negative-edge capture. Instead, take Example 1's sequence with $\text{CLK}$ complemented throughout (equivalent to swapping which stage is Master vs. Slave, per Section 3), which produces two real captures with different values:

   | Event | $D$ | $\text{CLK}$ | $Q_M$ (tracks $D$ while $\text{CLK}=1$; frozen while $\text{CLK}=0$) | $Q$ (tracks $Q_M$ while $\text{CLK}=0$; frozen while $\text{CLK}=1$) |
   |---|---|---|---|---|
   | Start | 0 | 1 | 0 (Master transparent, tracks $D=0$) | 0 (held from before) |
   | $D\to1$ (still $\text{CLK}=1$) | 1 | 1 | 1 (Master transparent, tracks new $D$ immediately) | 0 (unchanged — Slave holding) |
   | Falling edge ($\text{CLK}:1\to0$) | 1 | 0 | 1 (frozen at $D$'s value right at the edge) | **1** (Slave becomes transparent, captures $Q_M=1$ — this is the update moment) |
   | $D\to0$ (still $\text{CLK}=0$) | 0 | 0 | 1 (unchanged — Master now holding, ignores this $D$ change) | 1 (Slave transparent, but $Q_M$ isn't changing, so $Q$ doesn't either) |
   | Rising edge ($\text{CLK}:0\to1$) | 0 | 1 | 0 (Master resumes tracking, immediately grabs current $D=0$) | 1 (frozen — Slave stops tracking right as this happens) |
   | Falling edge ($\text{CLK}:1\to0$) | 0 | 0 | 0 (frozen at $D=0$, the value right at this edge) | **0** (Slave becomes transparent, captures new $Q_M=0$) |

   $Q$ changes only at the two falling-edge rows (marked in bold) — a genuine $0\to1$ flip followed by a genuine $1\to0$ flip — confirming negative-edge triggering with an actual demonstrated state change, the mirror image of Example 1 (where the analogous flips occurred only at rising edges).
7. Any transitions of $D$ occurring strictly between two rising edges fall into Section 2's "steady at $1$" case (while $\text{CLK}=1$), "steady at $0$" case (while $\text{CLK}=0$), or the falling-edge case (the single instant where $\text{CLK}$ transitions $1\to0$ inside that span) — all three were proven to leave $Q$ unchanged (the first two directly, since Slave/Master hold as argued below; the falling-edge case per Section 2's fourth case, which shows the Slave merely locks in its already-current value with no new information entering). Concretely: during a "steady at $1$" stretch, the Slave is disabled, ignoring the Master entirely regardless of how many times the Master's own tracked value $Q_M$ changes in response to $D$; during a "steady at $0$" stretch, the Master is frozen/holding throughout, so even though $D$ might wiggle, $Q_M$ itself doesn't change, meaning the Slave — tracking $Q_M$ — has nothing new to track either. Since every instant strictly between two rising edges falls into one of these three cases (together they cover the entire span, with only the two bounding rising edges themselves excluded by "strictly between"), and all three cases leave $Q$ unchanged, $Q$ cannot change at all during that span, regardless of how many times $D$ transitions — proving the claim.
8. This can be shown without appealing to propagation delay at all, using only Section 2's zero-delay Boolean model. Consider the $\text{CLK}=0$ interval immediately preceding the shared rising edge. Per Section 2's first case ("$\text{CLK}$ stays at $0$"), Flip-flop 1's output $Q_1$ is provably constant throughout that entire interval — nothing changes it while $\text{CLK}=0$. Since $D_2=Q_1$, $D_2$ is therefore also constant throughout that same interval, not just in the instant right before the edge; whatever value $D_2$ holds is the value it has held continuously since the previous rising edge, namely $Q_1$'s value as set by *that* previous edge (last cycle's data). At the shared rising edge itself, Flip-flop 2 captures $D_2$'s value "at the edge" (Section 2's proof) — but by the constancy just established, this is simply $D_2$'s already-settled value throughout the whole preceding interval, with nothing timing-dependent left to resolve; there is no live race between Flip-flop 1's update and Flip-flop 2's capture to adjudicate, because $D_2$ never depended on Flip-flop 1's *new* (post-edge) value in the first place — it only ever depended on $Q_1$'s old, already-constant value from before the edge. Simultaneously, Flip-flop 1 captures $D_1$'s current value at this same edge into a new $Q_1$. So Flip-flop 2 ends up holding $Q_1$'s *previous* value (last cycle's data) while Flip-flop 1 simultaneously updates to $D_1$'s *new* value (this cycle's data) — exactly the two-different-values, one-cycle-offset behavior a shift register requires, with no race-through, because (unlike Topic 09's level-sensitive latches, transparent for a whole interval during which a value could ripple through both stages within one enabled period) each flip-flop only ever looks at its input at one instant per cycle, and — per this zero-delay argument, not a delay-based one — $D_2$'s sampled value was already fixed well before the edge, with no dependency on Flip-flop 1's simultaneous update.

## Summary / cheat sheet

**Positive-edge-triggered D flip-flop:** $Q^+=D$ (next state equals $D$ at the triggering rising edge); $Q$ constant everywhere else, completely ignoring $D$ between edges (Section 2, Self-check Q7).

**Master-slave construction:** Master D latch (enable $\overline{\text{CLK}}$) $\to$ Slave D latch (enable $\text{CLK}$), Slave's output $=$ flip-flop's $Q$. Swap the two stages' clock assignment to get negative-edge triggering instead (Section 3).

**Why it works:** the two stages are never simultaneously transparent (complementary clocks) — Topic 09's transparency-problem fix, generalized and packaged as a single component with one external clock pin (the second phase generated internally by one inverter).

**Contrast with D latch:** D latch tracks $D$ continuously for a whole enabled interval (level-sensitive); flip-flop samples $D$ once, only at the triggering edge (edge-triggered) — this is exactly what makes flip-flops safe to chain on a shared clock (Self-check Q8), unlike plain D latches (Topic 09's transparency problem).

## Used later in
- [[6.004-computation-structures/notes/11-moore-mealy-machines]] — the characteristic equation $Q^+=D$ generalizes directly to an FSM's next-state function $s^+=\delta(s,x)$, and edge-triggering is what makes "present state" and "next state" well-defined, non-overlapping notions for a synchronous FSM.
- [[6.004-computation-structures/notes/13-implementing-fsms-flip-flops]] — Self-check Question 8's zero-delay, no-race-through argument for two chained flip-flops is generalized directly to $k$ simultaneously-updating flip-flops in a state register, proving the general FSM architecture's correctness.
