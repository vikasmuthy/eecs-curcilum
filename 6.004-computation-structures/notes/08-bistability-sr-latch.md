---
title: "Bistability & the SR Latch"
course: "6.004"
topic_number: 08
prerequisites: ["6.004 Topic 04 — Static CMOS Gates", "6.002 Topic 04 — Nonlinear Elements, Digital Abstraction, MOSFET Model"]
status: done
---

# Bistability & the SR Latch

## Why this matters

Every circuit built in Topics 03–07 is **combinational**: its output at any instant is a pure function of its current inputs, with no dependence on the past. That's sufficient for arithmetic and routing, but it cannot build *memory* — a circuit that, once set to a value, keeps that value even after the inputs that set it are removed. This note introduces the mechanism that makes memory possible: wiring a gate's output back into another gate's input (**feedback**), arranged so the resulting circuit has more than one **stable state** it can sit in indefinitely without any input driving it there. This is a genuinely new idea, not a combinational-logic variant — the value stored depends on *history*, not just the present inputs, which breaks the "output is a pure function of current inputs" assumption every earlier topic relied on. Skip this topic and there is no path to the D latch (Topic 09), flip-flop (Topic 10), or any register/memory element the datapath (Topics 25 onward) depends on.

## Builds on

- [[6.004-computation-structures/notes/04-static-cmos-gates]] — NOR and NAND gates, used below as the two cross-coupled halves of the SR latch; also reuses the fact that a static CMOS gate's output is driven to a well-defined rail (Topic 04, Section 1) whenever exactly one of its pull-up/pull-down networks conducts, which is what makes a *stable* state well-defined here (the output stays at a valid, actively-driven logic level, not merely "whatever it happened to be").
- [[6.002-circuits-and-electronics/notes/04-nonlinear-elements-digital-abstraction-mosfet-model]] — the digital abstraction and the notion of gain (output-change/input-change ratio) restoring a degraded signal toward a clean logic level; recapped and reused below as the intuitive reason feedback around a gain-$>1$ element can lock onto one of several self-consistent states rather than settling to a single "compromise" value.

## Core definitions

- **Feedback (in a digital circuit)** — a wiring pattern in which a gate's output is connected, directly or through other gates, back to one of that same gate's inputs (or an input of a gate earlier in the chain leading to it), so the circuit's own output influences its own future input.
- **State** — for a circuit with feedback, the actual voltage(s)/logic value(s) currently present on the feedback wire(s); unlike a combinational circuit's output, the state is not determined by the current external inputs alone.
- **Stable state (equilibrium)** — an assignment of logic values to every node in a feedback circuit that is *self-consistent*: if every gate in the circuit is evaluated using these values as its inputs, every gate's output reproduces the same values already assumed — the circuit, once in this state, has no tendency to change, since each gate's actual output matches what its own inputs already say it should be.
- **Bistable circuit** — a feedback circuit with (at least) two distinct stable states, both self-consistent per the definition above, such that external inputs can be used to force the circuit into either one, and after those inputs are removed (returned to some fixed, "leave me alone" value) the circuit *remains* in whichever state it was last forced into.
- **Metastable state** — an equilibrium of a feedback circuit's equations (in the idealized model) that is *not* practically stable: unlike the digital abstraction's usual account (which will treat this as one of a bistable circuit's other outcomes), an infinitesimal perturbation drives the circuit away from this point toward one of the genuinely stable states rather than back to it. Flagged here as a real physical phenomenon in actual latches, not resolved with the ideal-gate model used for the rest of this note (further timing consequences are out of scope until Topic 23).
- **Cross-coupled gates** — two 2-input gates (NOR-NOR or NAND-NAND, both combinations used below) wired so gate 1's output feeds one of gate 2's inputs, and gate 2's output feeds one of gate 1's inputs, forming a 2-gate feedback loop; the other input of each gate is left free as an external control input.
- **SR latch (set-reset latch)** — a bistable circuit built from two cross-coupled gates with external inputs conventionally named $S$ (set) and $R$ (reset), and two outputs conventionally named $Q$ and $\overline Q$ (intended to be complements of each other in normal operation), whose behavior is: asserting $S$ (with $R$ not asserted) drives $Q$ to $1$; asserting $R$ (with $S$ not asserted) drives $Q$ to $0$; with neither asserted, $Q$ holds whatever value it last had (memory).
- **Forbidden / restricted input combination** — the input combination (for a given SR latch implementation) under which both $S$ and $R$ are simultaneously asserted, breaking the intended $Q=\overline Q$ complementary relationship and/or leading to unpredictable behavior when the assertion is released; derived and analyzed explicitly below rather than merely asserted as a rule to memorize.

## Intuition

**Feedback lets a circuit "remember" by describing itself, not just react.** Every gate built so far computes today's output from today's inputs only. Feed a gate's output back into its own input path, and the question "what is the output right now" no longer has a unique answer determined purely by external signals — it depends on what the output *already was*, because that old value is now one of the inputs determining the new value. This circularity is exactly what "memory" requires: a fact that persists because the circuit is actively re-confirming it to itself, moment to moment, rather than because some external signal keeps asserting it.

**Two cross-coupled gates naturally settle into one of two "each half confirms the other" states.** Picture two NOR gates, gate A's output feeding gate B's input and vice versa. Suppose gate A's output happens to be $0$: this makes gate B see the input "A's output $=0$," and (with the other input at its idle value) gate B's own output becomes $1$; that $1$ feeds back into gate A, and (checking gate A's own truth table with that input) gate A's output does indeed come out $0$ — self-consistent, as defined above. Swap the roles (A outputs $1$, B outputs $0$) and the same self-confirming pattern holds with the values swapped. Two different, both entirely self-consistent, "everyone agrees with everyone else" configurations — that's exactly what bistability means, and neither configuration requires any external input to keep confirming it, which is why this circuit can remember.

**The digital abstraction's gain argument is what rules out the loop "settling" at some in-between voltage.** Recall from 6.002 Topic 04: a gate with gain $>1$ in its transition region *actively pushes* a slightly-off voltage further toward whichever clean rail it's closer to, rather than passing the imprecision through unchanged. Around a feedback loop built from such gain-$>1$ gates, this pushing effect compounds every time around the loop — any voltage that isn't already exactly at one of the true digital equilibria gets nudged further from the "undecided middle" and closer to one of the two clean rail-to-rail states on each pass, which is the physical (not just Boolean-algebra) reason real cross-coupled gates snap decisively into one of the two stable digital states rather than settling at some analog compromise.

## Derivation / formalism

### 1. Cross-coupled NOR gates: writing and solving the self-consistency equations

Let gate A and gate B each be 2-input NOR gates (Topic 04's NOR: output $=\overline{X+Y}$). Wire: gate A's inputs are $R$ (external) and $Q_B$ (gate B's output); gate A's output is named $Q$. Gate B's inputs are $S$ (external) and $Q$ (gate A's output); gate B's output is named $\overline Q$. So:
$$Q = \overline{R + \overline Q} \qquad \overline Q = \overline{S+Q}$$

**Case $S=0, R=0$ (both idle):** the equations become $Q = \overline{0+\overline Q} = \overline{\overline Q} = Q$ (using the double-complement identity, $\overline{\overline X}=X$, proved in Topic 07 Self-check Question 6) — a tautology, true for *either* value of $Q$. Likewise $\overline Q = \overline{0+Q}=\overline Q$, also a tautology. So with both inputs idle, *both* $Q=0,\overline Q=1$ and $Q=1,\overline Q=0$ are self-consistent solutions — exactly the two stable states of a bistable circuit (Core definitions), confirming algebraically what the Intuition section argued informally: this circuit does not have a single forced answer, it holds onto whichever of the two states it's already in.

**Case $S=0, R=1$ ("reset" asserted):** $Q = \overline{1+\overline Q} = \overline 1 = 0$ (dominance/null law, Topic 01: $1+X=1$ for any $X$, then NOT of $1$ is $0$) — forced to $0$ regardless of $\overline Q$'s prior value. Then $\overline Q = \overline{0+Q} = \overline{0+0}=\overline 0 = 1$. Unique solution: $Q=0,\overline Q=1$. This is the "reset" behavior named in the Core definitions.

**Case $S=1, R=0$ ("set" asserted):** by the symmetric argument (swap the roles of the two equations), $\overline Q = \overline{1+Q}=0$ forced, then $Q=\overline{0+0}=1$. Unique solution: $Q=1,\overline Q=0$. The "set" behavior.

**Case $S=1,R=1$ (both asserted — the forbidden combination):** $Q=\overline{1+\overline Q}=0$ and $\overline Q = \overline{1+Q}=0$ — *both* outputs forced to $0$ simultaneously, violating the intended $Q=\overline{\overline Q}$ complementary relationship (Core definitions) entirely; this is why the combination is called forbidden/restricted, derived here directly from the gate equations rather than asserted by convention.

### 2. Why memory (holding a value with no input asserted) actually works, not just "is possible in principle"

Section 1's $S{=}0,R{=}0$ case showed *both* states satisfy the self-consistency equations — but self-consistency alone doesn't yet prove the circuit stays wherever it last was, only that it *could* stay. The missing piece: Sections 1's Set/Reset cases showed that asserting $S$ or $R$ drives the circuit to a *unique* forced state (no ambiguity while the input is asserted). When the asserted input is released back to $0$ (returning to the $S{=}0,R{=}0$ case), the circuit's actual node voltages at that instant already equal one of the two self-consistent solutions found in Section 1 (specifically, whichever one Set or Reset just forced) — and since both of Section 1's $0,0$-case solutions are genuinely self-consistent equilibria (every gate's actual output already matches what feeds back into it), there is no driving force pushing the circuit away from the state it's already sitting in. It stays, by the very definition of a self-consistent equilibrium — the circuit isn't being told to hold $Q$, it's simply not being told to do anything else, and "do nothing else" is itself a valid, self-reinforcing equilibrium at whichever value $Q$ already has.

### 3. The dual construction: cross-coupled NAND gates

By an argument fully parallel to Section 1 (swap NOR for NAND, and swap the roles of $0$/$1$ using duality, 6.004 Topic 01), cross-coupled NAND gates ($Q = \overline{R\cdot \overline Q}$, $\overline Q=\overline{S\cdot Q}$) realize the same set/reset/hold behavior, but with the external inputs **active-low**: asserting a $0$ (not a $1$) on the "reset" input forces $Q=0$, asserting a $0$ on "set" forces $Q=1$, and *both idle at $1$* (not $0$) is the hold condition — worked out fully in Worked Example 2 below. The forbidden combination for the NAND version is therefore both inputs held at $0$ simultaneously (the active-low mirror of the NOR version's both-at-$1$ forbidden case).

## Worked examples

### Example 1 — Cross-coupled NOR SR latch: full sequence trace (set, hold, reset, hold)

Using Section 1's equations ($Q=\overline{R+\overline Q}$, $\overline Q = \overline{S+Q}$), trace the latch through a sequence of inputs applied one after another, each new row's starting $Q,\overline Q$ taken from the previous row's result (this is exactly what "state" means — the next output depends on the previous output, not just the current $S,R$):

| Step | $S$ | $R$ | Case (Section 1) | Resulting $Q$ | Resulting $\overline Q$ |
|---|---|---|---|---|---|
| 1 | 1 | 0 | Set | 1 | 0 |
| 2 | 0 | 0 | Hold (both solutions exist; circuit stays at the value it already had, per Section 2) | 1 (unchanged from step 1) | 0 |
| 3 | 0 | 1 | Reset | 0 | 1 |
| 4 | 0 | 0 | Hold | 0 (unchanged from step 3) | 1 |
| 5 | 1 | 0 | Set | 1 | 0 |

This trace directly exhibits the memory property: step 2 and step 4 both have identical external inputs ($S{=}0,R{=}0$) but produce *different* outputs ($Q{=}1$ vs. $Q{=}0$) — impossible for any purely combinational circuit (whose output is a pure function of current inputs, Topic 01–07's entire framework), and only possible because the latch's output depends on its history (which state it was driven into by the most recent Set or Reset).

### Example 2 — Cross-coupled NAND SR latch: verifying the active-low behavior

Using Section 3's equations, $Q=\overline{R\cdot\overline Q}$, $\overline Q = \overline{S\cdot Q}$ (note: here $S,R$ are NAND-latch inputs, conventionally written $\overline S,\overline R$ in many textbooks to emphasize active-low, but kept as $S,R$ here for direct notational continuity with Section 1 — flagged explicitly since this is a genuine notational choice, not a universal convention).

**Case $S=1,R=0$ ("reset" input asserted, active-low — i.e. driven to $0$):** $Q = \overline{0\cdot\overline Q} = \overline 0 = 1$ (null law: $0\cdot X=0$ for any $X$, Topic 01) — forced regardless of $\overline Q$. Then $\overline Q = \overline{1\cdot 1} = \overline 1 = 0$. Result: $Q=1,\overline Q=0$.

Wait — check this against the intended "reset drives $Q$ to 0" behavior: this case, with $R=0$ asserted (active-low), produced $Q=1$, not $Q=0$. This confirms the NAND latch's polarity really is the opposite of the NOR latch's: here it is asserting $S$ (not $R$) at $0$ that should reset. Recomputing with **$S=0,R=1$**: $\overline Q = \overline{0\cdot Q} = \overline 0=1$, then $Q=\overline{1\cdot1}=\overline1=0$. Result: $Q=0,\overline Q=1$ — this is the true "reset" case for the NAND latch, confirming Section 3's claim that the NAND latch's control-signal roles are swapped relative to the NOR latch (asserting $R$ at $0$ sets, asserting $S$ at $0$ resets, mirroring — not matching label-for-label — the NOR latch's $S$/$R$ roles). This mismatch, caught here by direct calculation rather than assumed, is exactly why real NAND-based SR latches are conventionally drawn with inputs already labeled $\overline S,\overline R$ rather than reusing $S,R$ unchanged.

**Case $S=1,R=1$ (both idle, active-low convention $\Rightarrow$ both held at logic 1):** $Q=\overline{1\cdot\overline Q}=\overline{\overline Q}=Q$ (double-complement identity) and $\overline Q = \overline{1\cdot Q}=\overline Q$ — both tautologies, confirming this is indeed the hold condition, exactly as Section 3 claimed, with both external inputs idle at $1$ rather than $0$.

## Common pitfalls

- **Treating the SR latch as a combinational circuit and trying to write a single-row truth table for it.** Its output at $S{=}0,R{=}0$ genuinely depends on history (Example 1, steps 2 vs. 4) — there is no valid truth table with just $S,R$ as inputs; the state ($Q$'s previous value) is effectively a third, internal input that a truth table would need to include explicitly (a "characteristic table," conventionally written with $Q_{\text{next}}$ as a function of both $S,R$ *and* $Q_{\text{current}}$ — not covered in depth here, deferred to Topic 09's D latch which formalizes this table style).
- **Forgetting the NAND-latch's control-signal polarity is inverted relative to the NOR-latch's, as shown concretely in Example 2.** Applying NOR-latch intuition (asserting a $1$ on $S$ sets) directly to a NAND-latch circuit produces exactly backward behavior — always re-derive (or explicitly recall) which specific gate type is in use before reasoning about which input value asserts which behavior.
- **Applying $S=R=1$ to a NOR latch (or $S=R=0$ to a NAND latch) and expecting a well-defined "both set and reset" result.** Section 1 showed this forces $Q=\overline Q=0$ (NOR case) — a state that isn't even a valid logic-level pair (both outputs at the same value, violating the intended complementary relationship) — and Common practice additionally flags (though this note's ideal-gate equations alone don't fully capture it) that the transition *out* of this forbidden state, when the inputs are released, can be unpredictable/race-prone in a real circuit, connecting to the metastability concept defined above. Never rely on this combination in a design.
- **Assuming bistability requires exactly two cross-coupled NOR or NAND gates specifically.** The two-NOR and two-NAND constructions here are the standard minimal cases, but the underlying principle (a feedback loop with gain $>1$ per pass, per the Intuition section's argument) is more general; don't over-generalize "SR latch" to mean "the only possible bistable circuit," but also don't assume every feedback loop is automatically bistable without checking its self-consistency equations the way Section 1 did.
- **Confusing metastability with the ordinary hold state.** The ordinary hold state ($S{=}R{=}0$ for NOR, both self-consistent equilibria found in Section 1) is a *genuinely* stable resting point — the circuit stays there indefinitely with no tendency to move. Metastability (Core definitions) is a *different*, third equilibrium point that exists in a more detailed (non-ideal-gate) analysis and is inherently unstable to the slightest perturbation; conflating "the latch is holding a value" with "the latch is metastable" reverses their meanings.

## Self-check

### Questions

1. (Easy) In a cross-coupled NOR SR latch, what value does $Q$ take when $S=1,R=0$ is applied?
2. (Easy) True or false: a combinational circuit's output can depend on the sequence of inputs it received in the past, not just the current input. Justify in one sentence using this note's definitions.
3. (Medium) Starting from $Q=0,\overline Q=1$ (a valid hold state for a NOR SR latch), apply $S=0,R=1$. Using Section 1's equations, determine the resulting $Q,\overline Q$, and state whether this changed the latch's state.
4. (Medium) For a NAND-based SR latch (Section 3), what input combination corresponds to "hold" (memory, no forced change), and what values are $S,R$ held at during hold?
5. (Medium) Explain, using Section 2's argument, why releasing $R$ back to $0$ after a NOR latch has just been reset (so $Q=0,\overline Q=1$) does not cause $Q$ to spontaneously change.
6. (Hard) Verify, by direct substitution into $Q=\overline{R+\overline Q}$ and $\overline Q=\overline{S+Q}$ (the NOR latch's equations), that $Q=1,\overline Q=1$ is *not* a self-consistent solution when $S=0,R=0$ (i.e., show at least one of the two equations fails to hold for this assignment) — confirming that not every assignment of $Q,\overline Q$ is a valid stable state, only the two identified in Section 1.
7. (Hard) A NOR SR latch is in the forbidden state $S=1,R=1$ (so $Q=\overline Q=0$, per Section 1). Both inputs are released to $S=0,R=0$ simultaneously. Using the equations from Section 1's $S{=}0,R{=}0$ case, explain why the immediate post-release state ($Q=\overline Q=0$, inherited from the forbidden state) is *not* itself a stable solution to the $S{=}0,R{=}0$ equations, and connect this to why the actual next state the circuit settles into is not fully predictable from the ideal Boolean equations alone (i.e., why this connects to the metastability concept flagged in Core definitions, even though this note's model can't fully resolve which of the two genuine stable states it ends up in).
8. (Hard) Using the gain argument from Intuition (6.002 Topic 04's gain concept, restoring a degraded signal toward a clean rail), explain in your own words why an *odd* number of inverters wired in a single feedback loop (e.g. 1 inverter, output fed straight back to its own input) does *not* settle into a stable digital state the way 2 cross-coupled NOR/NAND gates do, even though it's also "feedback." (Hint: check whether a single self-consistent Boolean solution even exists for 1 inverter's loop equation $Q=\overline Q$.)

### Answers

1. $Q=1$ (the "set" case, Section 1).
2. False for a combinational circuit — by definition (established across Topics 03–07 and explicitly restated in this note's Why This Matters), a combinational circuit's output is a pure function of its *current* inputs only, with no dependence on history; a circuit whose output depends on input history (like the SR latch) is, by this note's definitions, not combinational but a feedback/state-holding circuit.
3. $Q=\overline{R+\overline Q} = \overline{1+1}=\overline1=0$ (using the prior $\overline Q=1$ momentarily on the right side of the equation, then solving) — actually more directly: this is exactly Section 1's Reset case ($S{=}0,R{=}1$), which forces $Q=0,\overline Q=1$ unconditionally, regardless of the prior state. Since the latch was already at $Q=0,\overline Q=1$ before this input was applied, the result is unchanged — the state was already $0$, and Reset simply reconfirms it.
4. Hold for the NAND latch is $S=1,R=1$ (both inputs held at logic $1$) — the opposite polarity from the NOR latch's hold condition ($S=0,R=0$), as derived in Section 3 and confirmed in Example 2's second case.
5. Per Section 1, at $S{=}0,R{=}0$ both $Q=0,\overline Q=1$ and $Q=1,\overline Q=0$ are self-consistent equilibria — every gate's actual output already matches what its own inputs (fed back from the other gate) say it should produce. Just after Reset, the latch's actual node values are $Q=0,\overline Q=1$, which *is* one of those two self-consistent equilibria; per Section 2's argument, a circuit already sitting at a self-consistent equilibrium has, by definition, no gate whose output disagrees with its inputs, hence no driving force to change — so releasing $R$ (returning to the tautological $0,0$-case equations) leaves the circuit exactly where it already was.
6. Check $Q=\overline{R+\overline Q}$ with $S{=}0,R{=}0,Q=1,\overline Q=1$: $Q\stackrel{?}{=}\overline{0+\overline Q} = \overline{0+1}=\overline1=0$. But the assumed value was $Q=1$, and the equation demands $Q=0$ — contradiction ($1\ne0$). So $Q=1,\overline Q=1$ fails the first equation and is not a self-consistent solution, confirming (as Section 1 asserted but did not need to separately disprove every other possible assignment for) that only the two identified assignments are valid stable states at $S{=}R{=}0$; $Q=\overline Q=1$ is simply not one of them (nor, by a symmetric check, is $Q=\overline Q=0$ — that combination was only shown as the *forced* result specifically at $S{=}R{=}1$, the forbidden case, not at $S{=}R{=}0$).
7. Substituting $Q=\overline Q=0$ into the $S{=}0,R{=}0$ equations: $Q\stackrel{?}{=}\overline{0+\overline Q}=\overline{0+0}=\overline0=1$. But the assumed value is $Q=0$, and the equation demands $Q=1$ — a contradiction, exactly like Question 6's check. So the instant-of-release state ($Q=\overline Q=0$, inherited unchanged from the forbidden case since voltages don't change instantaneously) is *not* itself a valid equilibrium of the just-restored $S{=}0,R{=}0$ equations — the circuit is momentarily sitting somewhere that isn't self-consistent, meaning some gate's actual output does *not* yet match its fed-back input, so the circuit *will* change. Which of the two true stable states ($Q{=}0,\overline Q{=}1$ or $Q{=}1,\overline Q{=}0$) it moves to depends on which of the two gates' analog transition happens to complete first — a race that the ideal, instantaneous Boolean-equation model used throughout this note has no mechanism to predict, since the model assumes gates evaluate instantaneously and doesn't represent the finite, comparably-sized delays of the two gates whose relative timing actually decides the outcome. This is precisely the physical mechanism behind the metastability concept flagged (not resolved) in Core definitions.
8. A single inverter's feedback loop equation is $Q=\overline Q$ — but by definition, $\overline Q$ is *never* equal to $Q$ for either Boolean value ($\overline 0=1\ne0$; $\overline1=0\ne1$), so this equation has *no* self-consistent Boolean solution at all, unlike the SR latch's $S{=}R{=}0$ case (Section 1), which had two. Physically (via the gain argument), the inverter's output is always being pushed toward the *opposite* rail from wherever its input currently sits, and since its own output is its own input, it's chasing a target that always just flipped away from it — there is no rail-to-rail voltage where "output matches what feeds back as input" the way there was for two cross-coupled gates (where gate B's inversion of gate A's inversion brings the signal back in phase with itself, not out of phase). This is exactly why a *pair* of cross-coupled inverting gates (an even number of inversions around the loop) is the standard bistable construction, while a lone inverter (an odd number) instead tends toward oscillation or an ill-defined in-between voltage rather than locking onto a clean digital state — consistent with the fact that no Boolean equilibrium exists for it in the first place.

## Summary / cheat sheet

**NOR SR latch** ($Q=\overline{R+\overline Q}$, $\overline Q=\overline{S+Q}$, active-high):

| $S$ | $R$ | Behavior |
|---|---|---|
| 0 | 0 | Hold (both self-consistent states valid; circuit keeps its prior $Q$) |
| 0 | 1 | Reset: $Q=0$ |
| 1 | 0 | Set: $Q=1$ |
| 1 | 1 | Forbidden: $Q=\overline Q=0$ (breaks complementary relationship) |

**NAND SR latch** ($Q=\overline{R\cdot\overline Q}$, $\overline Q=\overline{S\cdot Q}$, active-low): same behavior with every input polarity flipped — hold at $S{=}R{=}1$, forbidden at $S{=}R{=}0$.

**Key facts:** a stable state is a self-consistent solution to the feedback equations (every gate's output matches its own fed-back input); a bistable circuit has 2 such solutions when inputs are idle; feedback with an *even* number of inversions around the loop (2 cross-coupled gates) admits Boolean solutions, an *odd* number (1 inverter) does not.

## Used later in
- [[6.004-computation-structures/notes/09-d-latch-transparency-problem]] — the SR latch's cross-coupled core is reused unmodified, gated by $S=D\cdot\text{CLK}, R=\overline D\cdot\text{CLK}$ so the forbidden state becomes structurally unreachable; the two stable-state/hold argument is reused directly to explain D-latch hold behavior.
- [[6.004-computation-structures/notes/10-edge-triggered-d-flip-flop-master-slave]] — the Hold/Set/Reset case analysis is reused (via the D latch) inside both stages of the master-slave flip-flop construction.
- [[6.004-computation-structures/notes/11-moore-mealy-machines]] — the definition of state (a circuit's internally-held value, not determined by external inputs alone) is reused directly as an FSM's "present state" concept.
- [[6.004-computation-structures/notes/13-implementing-fsms-flip-flops]] — this note's unclocked feedback/self-consistency framing is contrasted directly with an FSM's clocked feedback, explaining why the FSM architecture never faces an analogous "does a solution exist" question inside its next-state logic.
