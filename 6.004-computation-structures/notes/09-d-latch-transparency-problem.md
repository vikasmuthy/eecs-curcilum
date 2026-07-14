---
title: "D Latch and the Transparency Problem"
course: "6.004"
topic_number: 09
prerequisites: ["6.004 Topic 08 — Bistability & the SR Latch", "6.004 Topic 05 — Multiplexers, Decoders, Encoders", "6.004 Topic 04 — Static CMOS Gates"]
status: done
---

# D Latch and the Transparency Problem

## Why this matters

Topic 08's SR latch is memory, but it's an awkward interface: two separate control lines, a forbidden input combination that has to be avoided by discipline rather than by the circuit itself, and no single, obvious signal for "now is the moment to remember this bit." This note builds the **D (data) latch** — the same bistable core, but wrapped so it takes exactly one data bit and one **clock/enable** signal, with the sensible rule "while enabled, output follows input; while disabled, output holds." That single design gives this note its second, more consequential job: showing precisely *why* "while enabled, output follows input" — called **transparency** — is a genuine problem the moment latches are chained together to build a multi-bit register or a pipeline, not just an inconvenience. Understanding exactly what breaks under transparency is the direct motivation for Topic 10's edge-triggered flip-flop, which resolves it.

## Builds on

- [[6.004-computation-structures/notes/08-bistability-sr-latch]] — the SR latch's cross-coupled-gate bistable core, its two stable states, and its forbidden input combination; this note wraps that exact core with extra gating logic to eliminate the forbidden combination and add a single-bit data interface.
- [[6.004-computation-structures/notes/05-multiplexers-decoders-encoders]] — reused below in an alternative D latch construction (the transmission-gate/MUX-based latch), directly reusing Topic 05's transmission gate as a controlled pass element.
- [[6.004-computation-structures/notes/04-static-cmos-gates]] — AND, NAND, and inverter gates, used to build the gating logic between the D/clock inputs and the SR latch's $S,R$ inputs.

## Core definitions

- **D latch (data / transparent latch)** — a bistable memory element with one data input $D$, one control input $\text{CLK}$ (variously called clock, enable, or gate — this note uses $\text{CLK}$), and outputs $Q,\overline Q$, satisfying: while $\text{CLK}=1$, $Q$ tracks $D$ continuously (whenever $D$ changes, $Q$ changes to match, essentially immediately, subject to real gate delay); while $\text{CLK}=0$, $Q$ holds whatever value it had at the moment $\text{CLK}$ fell to $0$, regardless of subsequent changes to $D$.
- **Level-sensitive** — describes a latch's behavior as governed by the *level* (the sustained value, $0$ or $1$) of its control signal, not by a specific instant of transition; a D latch is level-sensitive because its entire "transparent" phase lasts for as long as $\text{CLK}=1$ holds, not for one instant.
- **Transparency** — the property, during the interval $\text{CLK}=1$, that the latch's output is not merely *eventually* equal to $D$ but *continuously* follows every change in $D$ throughout that interval, as though $D$ were connected almost directly to $Q$ (through the latch's internal gate delays) — the latch is "transparent" to $D$ during this window, in the sense that $D$'s changes pass through visibly to $Q$ in real time rather than being sampled once.
- **Transparency problem (race-through)** — the failure mode that occurs when the output of one D latch (or any logic derived combinationally from it) is connected — directly or through combinational gates — to the data input of a second latch that is enabled ($\text{CLK}=1$) at the same time as the first: because the first latch is transparent during that whole interval, a change can propagate through *both* latches within a single enabled interval, corrupting the second latch's stored value with data that was only ever meant to pass through one latch at a time. Precisely characterized and demonstrated in the Derivation and Worked Examples below.
- **Setup time / hold time (introduced informally here, formalized in Topic 22)** — the interval of time, respectively, before and after the moment $\text{CLK}$ falls (or, for edge-triggered devices, the clock edge) during which $D$ must remain stable for the latch to reliably capture the correct value; mentioned here only to flag that a latch's data-capture behavior is not instantaneous in reality, even though this note's Boolean-level treatment idealizes it as if it were.

## Intuition

**A D latch is an SR latch with a translator bolted on the front that makes the forbidden state structurally impossible.** Topic 08's SR latch required the *user* to never assert $S$ and $R$ together. A D latch instead takes a single data bit $D$ and mechanically derives $S$ and $R$ from it so that they are *always* complementary whenever the latch is actually writable ($\text{CLK}=1$) — $S=D, R=\overline D$ — which makes $S=R=1$ structurally unreachable rather than merely "a case the designer promises not to trigger." This is the same kind of move Topic 04 made turning a bare pair of switch networks into a *guaranteed*-complementary PDN/PUN pair: push a correctness property into the wiring topology itself instead of leaving it as an operating discipline.

**"Transparent while enabled" sounds convenient and is exactly the thing that breaks chaining.** If a D latch's whole point is to *remember* a bit past the moment it was written, you'd like to feed one latch's output into the next latch's input and have each one grab a snapshot in turn — but a transparent latch doesn't take a snapshot at a single instant; for the entire time $\text{CLK}=1$, its output is a live, real-time copy of its input, gate delays aside. If two latches sharing the same clock are chained data-out-to-data-in, then during the shared enabled interval, the first latch is transparently passing $D$ through to its output, which is the second latch's input, which the second latch is *also* transparently passing through — data can "race through" both latches in one clock pulse, exactly as if the two latches briefly become a single wire. Whatever value the chain settles on by the end of the pulse depends on the *whole history* of $D$ during the pulse, not on any one clean sample — a genuinely broken abstraction for building, say, an $n$-bit shift register (where each stage is supposed to hold last cycle's value while receiving this cycle's new value from the stage before it, not the same cycle's value).

## Derivation / formalism

### 1. Gating an SR latch into a D latch

Take the NOR-based SR latch (Topic 08, Section 1: $Q=\overline{R+\overline Q}$, $\overline Q=\overline{S+Q}$). Derive $S,R$ from $D,\text{CLK}$ using two AND gates (Topic 04):
$$S = D \cdot \text{CLK} \qquad R = \overline D \cdot \text{CLK}$$
($\overline D$ obtained from a single inverter on $D$, Topic 04.)

**Claim 1: whenever $\text{CLK}=1$, $S$ and $R$ are always complementary (never both $1$, matching Topic 08's forbidden-state analysis, but now guaranteed rather than merely required).** At $\text{CLK}=1$: $S=D\cdot1=D$, $R=\overline D\cdot1=\overline D$ (identity law, Topic 01). Since $D$ and $\overline D$ are complements by definition, $S$ and $R$ are complements whenever $\text{CLK}=1$ — the forbidden $S=R=1$ case (Topic 08) requires $D=\overline D$, which is never true for any single bit value. So the forbidden state is structurally unreachable through this gating, for any value of $D$.

**Claim 2: whenever $\text{CLK}=1$, the latch behaves exactly as $Q=D$ (transparent).** From Claim 1, $S=D,R=\overline D$ exactly. If $D=1$: $S=1,R=0$ — Topic 08's Set case, forcing $Q=1=D$. If $D=0$: $S=0,R=1$ — Topic 08's Reset case, forcing $Q=0=D$. Either way $Q=D$ — transparent, matching the Core definition.

**Claim 3: whenever $\text{CLK}=0$, the latch holds its previous value regardless of $D$.** At $\text{CLK}=0$: $S=D\cdot0=0$ (null law) and $R=\overline D\cdot 0=0$, for *any* value of $D$ — so regardless of what $D$ is doing, the underlying SR latch always sees $S=R=0$, Topic 08's Hold case. By Topic 08 Section 2's argument, the latch remains at whichever self-consistent state it last held. This proves the hold behavior does not depend at all on $D$'s value while $\text{CLK}=0$ — exactly the Core definition's requirement.

### 2. Alternative construction: transmission-gate D latch

Using Topic 05's transmission gate directly (rather than gating an SR latch): connect $D$ through a transmission gate $TG_1$ (controlled by $\text{CLK}$: closes when $\text{CLK}=1$) into the input of a pair of cross-coupled inverters (the simplest 2-inverter bistable loop consistent with Topic 08's "even number of inversions admits a Boolean solution" argument — here realizing the storage core, functionally equivalent to Topic 08's cross-coupled NOR/NAND pair but built from inverters plus a feedback transmission gate $TG_2$, controlled by $\overline{\text{CLK}}$, that closes the loop only when $TG_1$ is open). This is a direct hardware analogue of Section 1's Claims 2–3: while $\text{CLK}=1$, $TG_1$ is closed (passing $D$ straight through to the storage node, transparent) and $TG_2$ is open (feedback loop broken, so the stored inverter pair isn't fighting the new data); while $\text{CLK}=0$, $TG_1$ opens (disconnecting $D$) and $TG_2$ closes (reconnecting the feedback loop, so the cross-coupled inverters hold whatever value was last written). This construction is mentioned here (not worked in full circuit detail) because it is the standard implementation used inside real flip-flops built in Topic 10, and because it makes the "transparent when enabled, feedback-isolated when disabled" behavior visible as two literal, mutually-exclusive switch paths rather than as SR-latch algebra.

### 3. The transparency problem, formalized

Consider two D latches, Latch 1 and Latch 2, sharing the same clock signal $\text{CLK}$, wired so Latch 1's output $Q_1$ feeds Latch 2's data input $D_2$ (i.e., $D_2 = Q_1$, possibly through some combinational logic, but consider the direct-wire case first for clarity).

**Claim: while $\text{CLK}=1$, $Q_2$ transparently tracks the *original* $D_1$, not a stable snapshot of it.** By Section 1 Claim 2, while $\text{CLK}=1$: $Q_1 = D_1$ (Latch 1 is transparent) — continuously, for the whole interval, including if $D_1$ changes partway through. Simultaneously, since $\text{CLK}=1$ also enables Latch 2: $Q_2 = D_2 = Q_1$ (Latch 2 is transparent too, tracking whatever is currently on its own data input). Substituting: $Q_2 = Q_1 = D_1$, continuously, for the entire enabled interval. So *any* change to $D_1$ at *any* point during the shared enabled pulse propagates all the way through to $Q_2$ before the pulse ends — the two latches have transiently become, in effect, a single piece of wire from $D_1$ to $Q_2$, for exactly as long as $\text{CLK}=1$ holds. This is the race-through effect named in the Core definitions, now derived directly from Section 1's Claim 2 applied twice in series rather than merely asserted.

**Why this specifically breaks a shift register (the standard motivating use case).** A 2-stage shift register wants: at the end of each clock pulse, $Q_2$ should hold whatever $D_1$ was at the *start* of that same pulse (data shifted one stage down), while $Q_1$ should hold the *new* $D_1$ (this cycle's fresh input) — i.e., the two stages are supposed to end the cycle holding two genuinely different values (last cycle's data and this cycle's data). But Section 3's claim shows $Q_2$ actually ends up equal to $Q_1$ (both equal to whatever $D_1$ was at the very end of the pulse) whenever they share a clock and are directly chained — the "shift" is lost; both stages converge to the same value instead of preserving the one-stage offset a shift register requires.

## Worked examples

### Example 1 — Single D latch: transparency and hold, numeric trace

Using Section 1's gated construction, trace a D latch through a changing $D$ signal while $\text{CLK}$ is first high, then low:

| Time step | $D$ | $\text{CLK}$ | $S=D\cdot\text{CLK}$ | $R=\overline D\cdot\text{CLK}$ | Resulting $Q$ |
|---|---|---|---|---|---|
| $t_1$ | 0 | 1 | 0 | 1 | 0 (Reset case, $Q=D$) |
| $t_2$ | 1 | 1 | 1 | 0 | 1 (Set case, $Q=D$ — changed immediately, since transparent) |
| $t_3$ | 0 | 1 | 0 | 1 | 0 (Reset case again — $Q$ tracked $D$'s drop instantly) |
| $t_4$ | 1 | 0 | 0 | 0 | 0 (Hold case — $D$ changed but $\text{CLK}=0$, so ignored, per Section 1 Claim 3) |
| $t_5$ | 0 | 0 | 0 | 0 | 0 (Hold — still ignoring $D$) |

Note $t_1$–$t_3$: $Q$ changes every single time $D$ changes, in real time, while $\text{CLK}=1$ — exactly the transparency property (Section 1, Claim 2). Note $t_4$: even though $D$ jumped to $1$, $Q$ stayed at $0$ because $\text{CLK}$ had already fallen — exactly Claim 3's hold behavior, confirming $Q$'s value is "whatever it was at the moment $\text{CLK}$ fell," here $0$ (from $t_3$), not influenced by $D$'s post-fall value at all.

### Example 2 — Two chained D latches sharing a clock: the transparency problem in a numeric trace

Latch 1: data input $D_1$, output $Q_1$. Latch 2: data input $D_2=Q_1$ (directly wired), output $Q_2$. Both share $\text{CLK}$. Trace one enabled pulse during which $D_1$ changes partway through:

| Time step | $D_1$ | $\text{CLK}$ | $Q_1$ (tracks $D_1$, Section 1 Claim 2) | $D_2=Q_1$ | $Q_2$ (tracks $D_2$, same Claim 2) |
|---|---|---|---|---|---|
| $t_1$ (pulse starts) | 0 | 1 | 0 | 0 | 0 |
| $t_2$ (mid-pulse) | 1 | 1 | 1 | 1 | 1 |
| $t_3$ (pulse ends, $\text{CLK}\to0$) | 1 | 0 | 1 (held) | — | 1 (held) |

**What a correct 2-stage shift register would want instead:** by the end of the pulse, $Q_2$ should reflect $D_1$'s value from *before* this pulse (last cycle's data, now shifted into stage 2), while $Q_1$ reflects $D_1$'s value *during* this pulse (this cycle's new data, in stage 1) — two different values. **What actually happened:** $Q_2$ ended at $1$, tracking $D_1$'s mid-pulse change all the way through to Latch 2's output within the *same* pulse, exactly matching $Q_1$'s final value — the intended one-stage delay between $Q_1$ and $Q_2$ collapsed to zero. This is a direct numeric instance of Section 3's general claim ($Q_2=Q_1=D_1$ throughout the shared enabled interval), and it is precisely the failure mode the edge-triggered flip-flop (Topic 10) is built to eliminate.

## Common pitfalls

- **Believing "the latch samples $D$ once when $\text{CLK}$ goes high" instead of tracking continuously.** This note's derivation (Section 1, Claim 2; Example 1) shows the opposite: a D latch is live/transparent for the *entire* high interval, not sampling once at the rising instant — that single-sample behavior is what an edge-triggered device (Topic 10) provides, and conflating the two is exactly the mistake that leads designers into the transparency problem.
- **Assuming the transparency problem only occurs with a literal direct wire between two latches' data and output.** Section 3's derivation goes through unchanged if $D_2$ is any combinational function of $Q_1$ (e.g. $D_2 = f(Q_1)$ for some gate network $f$) rather than a bare wire, since combinational logic (Topics 03–07) is itself instantaneous/continuously-tracking by definition — inserting gates between the latches does not break the race-through, it just changes $D_2$'s formula from $Q_1$ to $f(Q_1)$, and the same "changes propagate through within one pulse" argument applies to $f(Q_1)$ exactly as it did to the bare $Q_1$.
- **Thinking two latches sharing a clock is inherently broken in every circumstance.** The transparency problem specifically requires a data path from one latch's output back around (directly or combinationally) to another same-clock latch's *input* during the shared enabled window; two independent D latches with unrelated data inputs, both driven by the same clock, are perfectly fine — nothing races through because there's no data path connecting them.
- **Treating $S=D\cdot\text{CLK}, R=\overline D\cdot\text{CLK}$ as the only possible D-latch gating.** It's one standard, easily-derived construction (Section 1); Section 2's transmission-gate version achieves the identical Core-definition behavior via entirely different circuitry (switches and cross-coupled inverters, not an SR latch plus AND gates) — both are valid, and neither is "the" canonical D latch to the exclusion of the other.
- **Forgetting real gate/switch delay when reasoning about "instantaneous" transparency.** This note's Boolean-level treatment (as throughout the course so far) treats gate evaluation as instantaneous for clarity of the logical argument; real transparency and race-through happen subject to actual propagation delays through each gate/switch, which is exactly why setup/hold time (flagged in Core definitions, not derived here) becomes a precise, non-idealized concern once Topic 22 introduces real timing.

## Self-check

### Questions

1. (Easy) A D latch has $D=1,\text{CLK}=1$. What is $Q$, and by which SR-latch case (Topic 08) does this follow?
2. (Easy) A D latch has $\text{CLK}=0$, and $D$ changes from $0$ to $1$ while $\text{CLK}$ stays at $0$. Does $Q$ change? Why or why not?
3. (Medium) Using Section 1's gating equations, compute $S$ and $R$ for $D=0,\text{CLK}=0$, and state which SR-latch case this triggers.
4. (Medium) Explain, in terms of Section 1 Claim 1, exactly why the SR latch's forbidden state can never occur in a properly gated D latch, no matter what value $D$ takes.
5. (Medium) Two D latches share a clock, with $D_2=Q_1$ directly wired (as in Example 2). During one enabled pulse, $D_1$ starts at $1$, changes to $0$ midway, then changes back to $1$ just before the pulse ends. What is $Q_2$ at the end of the pulse? Justify using Section 3's claim.
6. (Hard) A single D latch (not chained to anything else) has its own output $Q$ fed back into its own $D$ input, i.e. $D=Q$, while $\text{CLK}=1$. Using Section 1 Claim 2 ($Q=D$ while transparent), analyze what happens — is this a case the latch's equations can resolve to a definite value, or does it create an ambiguous/unstable situation? Relate your answer to Topic 08's single-inverter-loop argument (Self-check Question 8 of that note).
7. (Hard) Design a fix for the 2-latch transparency problem (Example 2) using only components already introduced in this course (D latches, inverters) — without yet introducing anything from Topic 10 — by using *two different, complementary clock signals* for the two latches instead of one shared clock. State what those two clock signals should be and explain, using Section 1's Claims 2–3, why this prevents both latches from being transparent at the same time.
8. (Hard) Prove that your fix from Question 7 correctly implements one-cycle shift-register behavior: using $\text{CLK}$ and $\overline{\text{CLK}}$ as the two latches' respective enables, show that $Q_2$ at the end of a $\text{CLK}$ pulse equals $D_1$'s value from *before* that pulse started (i.e., $Q_1$'s previously-held value), not $D_1$'s value during the current pulse — contrasting directly with Example 2's broken result.

### Answers

1. $Q=1$. This is Section 1 Claim 2's transparent case: $S=D\cdot\text{CLK}=1\cdot1=1$, $R=\overline D\cdot\text{CLK}=0\cdot1=0$ — Topic 08's Set case, forcing $Q=1$.
2. No, $Q$ does not change. By Section 1 Claim 3, whenever $\text{CLK}=0$, $S=D\cdot0=0$ and $R=\overline D\cdot0=0$ regardless of $D$'s value — the underlying SR latch always sees Hold ($S=R=0$) while $\text{CLK}=0$, so it retains whatever value it already had, completely independent of $D$'s changes during this interval.
3. $S=D\cdot\text{CLK}=0\cdot0=0$. $R=\overline D\cdot\text{CLK}=1\cdot0=0$. Both $0$ — Topic 08's Hold case, regardless of $D=0$ specifically (matches Section 1 Claim 3: at $\text{CLK}=0$, $S=R=0$ for *any* $D$, not just $D=0$).
4. Section 1 Claim 1 shows that whenever $\text{CLK}=1$, $S=D$ and $R=\overline D$ exactly — and $D,\overline D$ are complements of each other by the basic definition of complementation (6.004 Topic 01), so $S,R$ are always complements of each other whenever $\text{CLK}=1$. The forbidden SR state requires $S=R=1$ simultaneously, which would require $D=\overline D=1$ — impossible for any single bit, since a bit and its complement can never both equal $1$ (the complement law, Topic 01). When $\text{CLK}=0$, $S=R=0$ always (Claim 3) — also not the forbidden $1,1$ case. So for every possible combination of $D$ and $\text{CLK}$, $S,R$ never reach $1,1$ — the forbidden state is structurally excluded by the gating, not merely avoided by discipline.
5. $Q_2=1$ at the end of the pulse. By Section 3's claim, $Q_2=Q_1=D_1$ continuously throughout the entire enabled interval — so $Q_2$ simply tracks whatever $D_1$'s value is at each instant, ending the pulse equal to $D_1$'s *final* value before $\text{CLK}$ falls, which is $1$ (the last of the three stated changes), regardless of the intermediate excursion to $0$ midway.
6. With $D=Q$ and $\text{CLK}=1$, Section 1 Claim 2 gives $Q=D=Q$ — a tautology, not a contradiction, but also not a value-determining equation (it's satisfied by *any* value of $Q$, exactly like Topic 08's $S{=}R{=}0$ tautology case). Unlike Topic 08's genuine SR-latch hold case, though, there is no second, independent equation here forcing a unique answer — this setup has effectively turned the D latch's transparent pass-through into its own feedback loop, with the number of inversions around the loop depending on the specific internal gate structure used to build the D latch (Section 1's SR-latch-based construction has an *even* number of inversions from $D$ through to $Q$ when traced through the underlying NOR gates, similar to Topic 08's stable 2-gate loop) — so, by the same reasoning as Topic 08 Self-check Question 8, this is likely to behave as another (degenerate) bistable loop admitting a Boolean solution for either value of $Q$, rather than the "no solution at all" case of a single bare inverter loop; but pinning down the *exact* behavior requires knowing the specific gate-level delay structure, which is beyond this note's ideal, delay-free Boolean model — flagged here as a genuinely underdetermined case at this level of abstraction, not a fully resolved one.
7. Use $\text{CLK}$ for Latch 1's enable and $\overline{\text{CLK}}$ (a single inverter on $\text{CLK}$) for Latch 2's enable. By Section 1 Claims 2–3: whenever $\text{CLK}=1$ (Latch 1 transparent, tracking $D_1$), $\overline{\text{CLK}}=0$, so Latch 2 is in its Hold case (Claim 3) — not transparent, ignoring its input $D_2=Q_1$ entirely and just retaining its own previously-held value. Conversely whenever $\text{CLK}=0$, Latch 1 holds (not transparent) while $\overline{\text{CLK}}=1$ makes Latch 2 transparent, now free to track $Q_1$ (which is itself frozen, since Latch 1 is holding). Since $\text{CLK}$ and $\overline{\text{CLK}}$ are complements, they're never simultaneously $1$ (complement law again) — so the two latches are *never* both transparent at the same time, which is exactly the condition Section 3's race-through derivation required ($Q_1=D_1$ and $Q_2=D_2$ holding *simultaneously* during a shared interval); with that simultaneity broken, Section 3's chained-equality argument no longer applies.
8. During a $\text{CLK}=1$ pulse: Latch 1 is transparent, so $Q_1$ tracks $D_1$'s value throughout this pulse (ending the pulse equal to $D_1$'s value at the pulse's end); simultaneously $\overline{\text{CLK}}=0$, so Latch 2 is holding (Claim 3), meaning $Q_2$ stays exactly at whatever value it held from *before* this pulse started — which, by the same argument applied to the *previous* $\text{CLK}=0$ phase (during which $\overline{\text{CLK}}=1$ made Latch 2 transparent, tracking $Q_1$, while $Q_1$ itself was frozen at Latch 1's Hold value from the pulse before that), is exactly $Q_1$'s value as it stood at the *end of the previous $\text{CLK}=1$ pulse* — i.e., $D_1$'s value from before the current pulse. So at the end of the current pulse: $Q_1 = D_1$(current pulse, new data), and $Q_2 = Q_1$(as of before the current pulse) $=D_1$(previous pulse, old data) — two genuinely different values, correctly reproducing the one-stage shift-register delay, in direct contrast to Example 2's collapsed result where $Q_2$ ended up equal to $Q_1$'s *current*-pulse value instead of its prior one.

## Summary / cheat sheet

**D latch** ($S=D\cdot\text{CLK}$, $R=\overline D\cdot\text{CLK}$ into an SR latch, or equivalently the transmission-gate construction): while $\text{CLK}=1$, $Q=D$ (transparent, continuously tracking); while $\text{CLK}=0$, $Q$ holds its last value regardless of $D$ (forbidden SR state structurally unreachable, since $S,R$ are always complementary when live and both $0$ when disabled).

**Transparency problem (race-through):** two same-clock latches chained $D_2=Q_1$ (or $D_2=f(Q_1)$ through combinational logic) collapse into $Q_2=Q_1=D_1$ throughout the shared enabled interval — the intended one-stage delay is lost.

**Fix demonstrated here (without Topic 10's flip-flop):** drive the two latches with complementary clocks ($\text{CLK}$, $\overline{\text{CLK}}$) so they are never simultaneously transparent — correctly reproduces one-cycle shift-register delay. (Topic 10 generalizes this idea into a single edge-triggered device built from two opposite-clocked latches internally.)

## Used later in
- [[6.004-computation-structures/notes/10-edge-triggered-d-flip-flop-master-slave]] — the complementary-clock fix worked out in this note's Self-check Questions 7-8 is formalized as the master-slave construction, packaged as a single edge-triggered component with one external clock pin.
- [[6.004-computation-structures/notes/14-propagation-delay-setup-hold-time]] — setup time and hold time, named informally here, are given precise quantitative definitions and derived constraints.
