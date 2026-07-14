---
title: "Moore and Mealy Machines — State Diagrams & Tables"
course: "6.004"
topic_number: 11
prerequisites: ["6.004 Topic 10 — Edge-Triggered D Flip-Flop (Master-Slave Construction)", "6.004 Topic 08 — Bistability & the SR Latch"]
status: done
---

# Moore and Mealy Machines — State Diagrams & Tables

## Why this matters

Topic 10's flip-flop stores exactly one bit, and its entire behavior is summarized by one equation: $Q^+=D$ (the next state equals whatever $D$ was at the triggering edge). Real systems need to remember *many* bits at once, and the next value of that whole collection needs to depend not just on "whatever the input was" but on some designed relationship between the current stored bits and the current input — e.g. "output a 1 only after the input has shown two consecutive 1s," a behavior no single flip-flop's $Q^+=D$ equation can express, since it needs to react differently to the *same* current input depending on what happened before. This note supplies the formal language for describing such behavior precisely, before any circuit is built: the **finite state machine (FSM)**, expressed as a **state diagram** or an equivalent **state transition table**, in two standard flavors — **Moore** and **Mealy** — that differ in exactly one structural choice (whether output depends on state alone, or on state and input together) with real, derivable consequences for output timing. Skip this topic and Topic 12 (the design *procedure* for turning a specification into such a diagram) and Topic 13 (turning the diagram into actual flip-flops and gates) have no formal object to operate on.

## Builds on

- [[6.004-computation-structures/notes/10-edge-triggered-d-flip-flop-master-slave]] — the flip-flop's characteristic equation $Q^+=D$ and its edge-triggered timing discipline (state changes only at a clock edge, never in between); this note generalizes $Q^+=D$ to $Q^+=\delta(Q,\text{input})$ for a general next-state function $\delta$, and relies on edge-triggering to make "the state at time $t$" and "the state at time $t+1$" well-defined, non-overlapping notions.
- [[6.004-computation-structures/notes/08-bistability-sr-latch]] — the definition of **state** as a circuit's internally-held value(s), not determined by external inputs alone; this note's "present state" is exactly that concept, now formalized as a named element of a finite set rather than a single stored bit.

## Core definitions

- **Finite state machine (FSM)** — a formal model of a system whose behavior at any moment is fully determined by (a) a **present state**, drawn from a finite set of possible states, and (b) the **current input**; formally a 5-tuple $(S, I, O, \delta, \lambda)$ (with an implicit designated **initial state** $s_0\in S$), where $S$ is the finite set of states, $I$ the finite set of possible input values (the **input alphabet**), $O$ the finite set of possible output values (the **output alphabet**), $\delta: S\times I \to S$ the **next-state function** (also called the **transition function**), and $\lambda$ the **output function**, whose signature differs between the Moore and Mealy variants defined below.
- **Present state / next state** — the state the machine occupies at the current clock cycle (present state, often written $Q$ or $s$) versus the state it will occupy at the *next* clock cycle (next state, written $Q^+$ or $s^+$), computed as $s^+=\delta(s,x)$ for current input $x$ — directly generalizing Topic 10's $Q^+=D$, with $D$ now replaced by the output of a next-state *function* of both $s$ and $x$, rather than a raw external input.
- **State diagram** — a directed graph representation of an FSM: one node per state in $S$, and one directed edge from state $s$ to state $\delta(s,x)$ for every state $s$ and input value $x\in I$, each edge labeled with the input $x$ that causes that transition (and, for a Mealy machine, also labeled with the output $\lambda(s,x)$ produced during that transition, conventionally written "$x/o$" on the edge).
- **State transition table** — a tabular representation of the identical information as a state diagram: one row per state, one column per input value, each cell containing the next state $\delta(s,x)$ (and, for Mealy machines, the associated output $\lambda(s,x)$); a state diagram and its state transition table are two notations for the exact same function $\delta$ (and $\lambda$), never containing different information from each other.
- **Moore machine** — an FSM whose output function has signature $\lambda: S \to O$ — the output depends on the present state *alone*, not on the current input. Consequently every state in a Moore state diagram is itself labeled with a fixed output value (rather than each transition edge carrying its own output label).
- **Mealy machine** — an FSM whose output function has signature $\lambda: S\times I \to O$ — the output depends on *both* the present state and the current input. Consequently, in a Mealy state diagram, it is the transition *edges* that are labeled with outputs (since the output can differ depending on which input triggered the transition out of a given state), not the states themselves.
- **Synchronous FSM** — an FSM whose state updates ($s \to \delta(s,x)$) all happen simultaneously, exactly once per clock cycle, at a clock edge — the model this note (and the whole rest of this course) assumes, made physically well-defined specifically because Topic 10's edge-triggered flip-flops (not Topic 09's level-sensitive latches) are used to hold the state, avoiding any race-through ambiguity about exactly when "the next state" takes effect.

## Intuition

**An FSM is what you get when you let $Q^+=D$'s right-hand side depend on the current state too, not just a raw external input.** Topic 10's flip-flop has no memory of *why* it holds the value it holds — its next value is simply whatever $D$ says, full stop, with no way to make that decision depend on history beyond the single bit already stored. An FSM keeps a whole state (which can encode arbitrary "memory of history so far," as the worked examples below make concrete) and computes the *next* state as a designed function of both that memory and the current input — this is precisely what makes it possible to build a circuit whose reaction to "input is currently 1" differs depending on whether the *previous* input was also 1 or not, something no purely combinational circuit (Topics 03–07) or bare flip-flop (Topics 08–10) can do on its own.

**A state diagram is a "recipe graph," and a state table is the exact same recipe with the graph flattened into rows and columns.** Neither representation is more fundamental than the other — a state diagram is easier for a human to read at a glance (you can trace a path visually to see what a sequence of inputs does), while a state table is easier to mechanically convert into actual next-state logic (Topic 12's job). Converting between them is purely notational, never a design decision: every arrow in the diagram is one filled cell in the table, and vice versa.

**Moore vs. Mealy is entirely about *when* the output is allowed to see the current input.** In a Moore machine, the output is a property of *where you currently are* (the state) — nothing about the current input can affect it until that input has first caused a transition to a new state (which only happens at the next clock edge, per the synchronous-FSM assumption). In a Mealy machine, the output is a property of *where you are and what's currently happening* — the current input can affect the output the same cycle it's applied, with no clock-edge delay, because the output function reads the input directly, the same way any ordinary combinational circuit (Topics 03–07) would. This is a real, physically meaningful timing difference, not just a stylistic choice — derived precisely in Section 2 below, and demonstrated concretely in the worked examples (the same specification, built both ways, produces outputs shifted by exactly one clock cycle relative to each other).

## Derivation / formalism

### 1. State diagrams and state tables encode identical information

Claim: for any FSM $(S,I,O,\delta,\lambda)$, its state diagram and its state transition table are two notations for the same functions $\delta,\lambda$, and either can be mechanically reconstructed from the other with no information gain or loss.

**Construction (diagram $\to$ table):** for every state $s\in S$ and input $x\in I$, the diagram contains exactly one edge from $s$ labeled $x$ (by $\delta$'s definition as a *function*, i.e. total and single-valued — every $(s,x)$ pair has exactly one next state); write that edge's destination into the table's row-$s$, column-$x$ cell, along with the edge's output label if present (Mealy) or the source state's own label (Moore).

**Construction (table $\to$ diagram):** for every filled cell (row $s$, column $x$) of the table, draw one edge from node $s$ to the node named by that cell's next-state entry, labeled $x$ (and the cell's output entry, for Mealy); for Moore, additionally label each state node with its own fixed output entry (read off a separate output column of the table, indexed only by state, matching $\lambda:S\to O$'s signature).

Both constructions are total (every table cell / every diagram edge is accounted for, since $\delta$ is a total function over all of $S\times I$) and lossless (no two distinct tables produce the same diagram, and vice versa, since the constructions are exact inverses of each other cell-by-cell/edge-by-edge). So the choice between diagram and table is purely a notational convenience, never a different design. $\blacksquare$

### 2. The Moore/Mealy output-timing difference, derived from the synchronous-FSM assumption

Consider an FSM currently in state $s$ (established at the most recent clock edge) receiving input $x$ during the current clock cycle, with the *next* clock edge yet to occur.

**Moore case:** $\lambda(s)$ depends only on $s$, which was already fixed *before* $x$ arrived (state $s$ was set by the previous edge, before this cycle's input existed). So the Moore output visible during the current cycle reflects the state as of the *start* of the cycle — it cannot reflect anything about the current cycle's input $x$ until $x$ has caused a transition to $\delta(s,x)$ at the *next* edge, whereupon $\lambda(\delta(s,x))$ becomes the output for the *cycle after that*. Formally: if input $x$ is applied during cycle $t$ (with state $s$ established at the start of cycle $t$), the output influenced by $x$ first appears as $\lambda(\delta(s,x))$ during cycle $t+1$ — a **one-cycle delay** between an input's arrival and its (indirect, via a state change) effect on the output.

**Mealy case:** $\lambda(s,x)$ is evaluated combinationally from both the state $s$ (fixed at the start of the cycle) and the input $x$ (arriving during the cycle) — exactly like any ordinary combinational circuit's output depends on its current inputs (Topics 03–07), with no flip-flop or clock edge standing between $x$ and $\lambda$. So the output reflecting $x$ appears during the *same* cycle $t$ that $x$ was applied — **zero-cycle delay** between an input and its effect on the output, though the *state* transition triggered by that same $x$ (i.e. $\delta(s,x)$) still only takes effect at the next edge, exactly as in the Moore case.

This is a strict, derivable difference (not a convention or preference): a Moore machine's output during cycle $t$ is a function of the state established through cycle $t-1$'s inputs; a Mealy machine's output during cycle $t$ is a function of state established through cycle $t-1$'s inputs *and* cycle $t$'s own input. Worked Example 2 makes this concrete with a specific input sequence, showing the two machines' output sequences are literally offset by one cycle from each other for the same specification.

### 3. Converting a Mealy machine to an equivalent Moore machine

Given a Mealy machine $(S,I,O,\delta_M,\lambda_M)$, construct a Moore machine as follows: for each *reachable* combination of (a state $s\in S$) and (an output value $o\in O$ that some transition *into* $s$ could produce), create a new Moore state $s_o$ (read: "the Mealy machine is in state $s$, having just produced output $o$ on the way in"). Define the Moore output function $\lambda_{M'}(s_o) = o$ (the output is now a fixed property of which "state+incoming-output" combination we're in — matching the Moore signature $\lambda:S\to O$). Define the new next-state function: $\delta_{M'}(s_o, x) = \delta_M(s,x)_{\lambda_M(s,x)}$ — i.e., transition to the new Moore state that pairs the *Mealy* machine's actual next state $\delta_M(s,x)$ with the output $\lambda_M(s,x)$ that this specific transition would have produced.

**Why this reproduces the Mealy machine's *output sequence*, shifted by exactly one cycle.** By Section 2, the Mealy machine's output during cycle $t$ (given state $s$ at the start of cycle $t$ and input $x$ during cycle $t$) is $\lambda_M(s,x)$ — available immediately, same cycle. In the constructed Moore machine, that same output value $\lambda_M(s,x)$ only becomes available once the machine has *transitioned into* the state $s_o$ that encodes it (since Moore output is read off the *state*, and state transitions only take effect at the next edge, per Section 2's Moore case) — i.e., during cycle $t+1$, one cycle later than the Mealy machine's output for the identical input history. This one-cycle offset is not a flaw in the conversion; it is the exact, unavoidable structural consequence of Section 2's timing derivation (a Moore output *cannot* reflect the current cycle's input, by definition) — the state-splitting construction is the standard way of "storing" a Mealy transition's output as a state property so it becomes readable via $\lambda:S\to O$, at the cost of that one-cycle delay. Worked Example 2 demonstrates this offset on a concrete input sequence.

### 4. Converting a Moore machine to an equivalent Mealy machine

The reverse direction is comparatively trivial: given a Moore machine $(S,I,O,\delta,\lambda_{Moore})$ with $\lambda_{Moore}:S\to O$, define a Mealy output function $\lambda_{Mealy}(s,x) \triangleq \lambda_{Moore}(s)$ for every $x\in I$ — i.e., simply ignore the input argument. Since $\lambda_{Mealy}$ never actually depends on $x$, it satisfies the Mealy signature $S\times I\to O$ while behaving exactly like the original Moore output, with the same (zero-cycle, since the "input dependence" is vacuous) timing as the Moore machine — no state-splitting or extra states are needed, and no timing offset is introduced, since a Moore machine's output is trivially a special case of a Mealy machine's output that happens not to use the input.

## Worked examples

### Example 1 — "Detect two consecutive 1s" as a Mealy machine: state diagram, table, and a trace

**Specification:** a single-bit input stream arrives one bit per clock cycle; output $1$ during any cycle whose input, together with the *immediately preceding* cycle's input, forms the pattern "$1,1$" (two consecutive 1s, output asserted on the second 1); output $0$ otherwise. Overlapping occurrences count (e.g. input $1,1,1$ should output $0,1,1$ — the third 1 forms a new "two consecutive 1s" pair with the second).

**States:** $S=\{A,B\}$ where $A$ = "have not just seen a 1" (i.e., the most recent input, if any, was $0$, or this is the very start), $B$ = "just saw a 1" (the most recent input was $1$). Initial state $s_0=A$.

**Mealy transition/output table:**

| State | Input $0$: next state / output | Input $1$: next state / output |
|---|---|---|
| $A$ | $A$ / $0$ | $B$ / $0$ |
| $B$ | $A$ / $0$ | $B$ / $1$ |

(Reading row $B$, input $1$: we were already in "just saw a 1," and now see another 1 — this *is* the second of two consecutive 1s, so output $1$, and remain in state $B$ since the most recent input is still a 1, ready to detect the *next* possible pair.)

**State diagram (in words):** node $A$ has a self-loop labeled "$0/0$" and an edge to $B$ labeled "$1/0$"; node $B$ has an edge to $A$ labeled "$0/0$" and a self-loop labeled "$1/1$".

**Trace for input sequence $1,1,0,1,1,1$ (cycle by cycle, starting in state $A$):**

| Cycle | Present state | Input $x$ | Output $\lambda_M(s,x)$ | Next state $\delta_M(s,x)$ |
|---|---|---|---|---|
| 1 | $A$ | 1 | 0 | $B$ |
| 2 | $B$ | 1 | 1 | $B$ |
| 3 | $B$ | 0 | 0 | $A$ |
| 4 | $A$ | 1 | 0 | $B$ |
| 5 | $B$ | 1 | 1 | $B$ |
| 6 | $B$ | 1 | 1 | $B$ |

Output sequence: $0,1,0,0,1,1$ — matches the specification (1s at cycles 2, 5, 6, exactly where the current input and the immediately preceding one are both 1; cycle 6's input of 1 pairs with cycle 5's input of 1, an overlapping detection, correctly flagged per the "overlapping occurrences count" requirement).

### Example 2 — Converting Example 1's Mealy machine to Moore, and comparing output sequences on the same input

Apply Section 3's construction to Example 1's Mealy machine. Reachable (state, incoming-output) pairs: starting from $s_0=A$ (treat the initial state's "incoming output" as a fixed placeholder, say $0$, since no transition has occurred yet), the reachable pairs turn out to be $A_0$ (in $A$, arrived via an output-$0$ transition — the only way to reach $A$, per the table), $B_0$ (in $B$, arrived via an output-$0$ transition — from $A$ on input 1), and $B_1$ (in $B$, arrived via an output-$1$ transition — the self-loop on $B$ for input 1). Moore states: $S'=\{A_0, B_0, B_1\}$, with $\lambda_{Moore}(A_0)=0$, $\lambda_{Moore}(B_0)=0$, $\lambda_{Moore}(B_1)=1$ (each state's output is exactly the subscript, by construction).

**Moore transition table** (derived via Section 3's $\delta_{M'}(s_o,x)=\delta_M(s,x)_{\lambda_M(s,x)}$):

| State | Input $0$: next state | Input $1$: next state |
|---|---|---|
| $A_0$ | $A_0$ (was $\delta_M(A,0)=A$, output $0$) | $B_0$ (was $\delta_M(A,1)=B$, output $0$) |
| $B_0$ | $A_0$ (was $\delta_M(B,0)=A$, output $0$) | $B_1$ (was $\delta_M(B,1)=B$, output $1$) |
| $B_1$ | $A_0$ (was $\delta_M(B,0)=A$, output $0$) | $B_1$ (was $\delta_M(B,1)=B$, output $1$) |

**Trace for the identical input sequence $1,1,0,1,1,1$, starting in state $A_0$:**

| Cycle | Present state | Input $x$ | Output $\lambda_{Moore}(s)$ | Next state |
|---|---|---|---|---|
| 1 | $A_0$ | 1 | 0 | $B_0$ |
| 2 | $B_0$ | 1 | 0 | $B_1$ |
| 3 | $B_1$ | 0 | 1 | $A_0$ |
| 4 | $A_0$ | 1 | 0 | $B_0$ |
| 5 | $B_0$ | 1 | 0 | $B_1$ |
| 6 | $B_1$ | 1 | 1 | $B_1$ |

Output sequence: $0,0,1,0,0,1$. **Compare to Example 1's Mealy output $0,1,0,0,1,1$:** the Moore sequence is exactly the Mealy sequence shifted right by one cycle (Mealy's cycle-2 output of $1$ appears as Moore's cycle-3 output; Mealy's cycle-5 and cycle-6 outputs of $1,1$ appear as Moore's cycle-6 output of $1$ followed by what would be cycle 7's output, not shown in this 6-cycle trace but predictable as $1$ from state $B_1$ persisting) — a direct numeric confirmation of Section 3's one-cycle-delay claim.

## Common pitfalls

- **Forgetting that $\delta$ must be a total function — every state needs an outgoing transition for every possible input.** A state diagram with a state that has no drawn edge for some input value $x$ is an incomplete specification, not a valid FSM per this note's definition (Section 1's diagram $\leftrightarrow$ table equivalence assumes every cell is filled); "no transition drawn" is not the same as "self-loop" or "stay put" unless that self-loop is explicitly drawn.
- **Labeling a Mealy machine's *states* with outputs (Moore-style) instead of its *transitions*.** Since a Mealy output can differ depending on which input triggered a given transition out of the same state, a single fixed per-state output label cannot capture Mealy behavior — the output information belongs on the edges, not the nodes, precisely because $\lambda_{Mealy}$'s signature includes $x$ as an argument.
- **Assuming Moore-to-Mealy and Mealy-to-Moore conversions are symmetric in cost.** Section 4 (Moore$\to$Mealy) needs no new states and no timing change; Section 3 (Mealy$\to$Moore) generally *does* need new states (one per reachable state+incoming-output combination — Example 2 turned 2 Mealy states into 3 Moore states) and *does* introduce a one-cycle output delay. The two directions are not mirror images of each other in cost, only in the sense that both produce an "equivalent" machine by some definition of equivalence.
- **Treating "equivalent" (Section 3) as meaning "identical, cycle-for-cycle."** The Mealy-to-Moore conversion preserves the *sequence of output values*, but shifted by one clock cycle (Example 2) — a system depending on exact same-cycle output timing (e.g. feeding the FSM's output combinationally into something else that same cycle) would behave differently if a Mealy machine were swapped for its "equivalent" Moore version without accounting for that shift.
- **Confusing a state diagram/table's *input alphabet* size with the *number of bits* needed to encode it, or the *number of states* with the number of flip-flops needed to store them.** This note deliberately treats states and inputs as abstract, named elements of finite sets (e.g. "$A$," "$B$," not yet any particular bit pattern) — the question of *how many bits* are needed to represent $|S|$ states, and how that maps onto actual flip-flops, is Topic 12's (state encoding) and Topic 13's (implementation) job, not something this note's abstract formalism answers.
- **Forgetting the synchronous-FSM assumption (Core definitions) is doing real work.** This note's entire "present state / next state, exactly one transition per clock cycle" framing depends on edge-triggered storage (Topic 10) to make "the current cycle" and "the next cycle" well-defined, non-overlapping notions; built instead from Topic 09's level-sensitive latches, the whole state-diagram formalism's cycle-by-cycle bookkeeping would be undermined by exactly the transparency problem Topic 09 identified.

## Self-check

### Questions

1. (Easy) In an FSM's state transition table, what does the cell in row $s$, column $x$ contain?
2. (Easy) True or false: in a Mealy machine, a state's output is fixed regardless of the current input. Justify in one sentence.
3. (Medium) A Moore machine's state $s$ was entered at the start of cycle $t$. During cycle $t$, input $x$ arrives. In which cycle does an output reflecting $x$'s influence first appear, and why?
4. (Medium) Using Example 1's Mealy table, trace the output sequence for input sequence $0,1,1,0,1$ starting from state $A$.
5. (Medium) Explain why converting a Moore machine to an equivalent Mealy machine (Section 4) never requires adding new states, while the reverse conversion (Section 3) generally does.
6. (Hard) A Mealy machine has 3 states, and every state can be reached via transitions producing 2 different possible incoming-output values. Using Section 3's construction, how many states does the equivalent Moore machine have at most, and why "at most" rather than "exactly"?
7. (Hard) Does applying Section 4's Moore-to-Mealy conversion and then Section 3's Mealy-to-Moore conversion to a Moore machine necessarily reproduce a machine with the same number of states as the original? Either prove it always does, or give a small counterexample and explain, in terms of Section 3's splitting rule, exactly what causes the state count to change.
8. (Hard) Design a Moore machine (state diagram or table) directly — not by converting Example 1's Mealy machine — that detects "two consecutive 1s" with the same specification as Example 1 (overlapping occurrences count). Verify your machine against the same input sequence used in Example 1 ($1,1,0,1,1,1$) and confirm your output sequence matches Example 2's converted-Moore-machine output ($0,0,1,0,0,1$).

### Answers

1. The next state $\delta(s,x)$ — i.e., which state the machine transitions to if it is currently in state $s$ and receives input $x$ (and, for a Mealy machine, also the output $\lambda(s,x)$ produced during that transition).
2. False. In a Mealy machine, $\lambda(s,x)$ depends on both the state $s$ and the current input $x$ — by definition (Core definitions, Mealy machine), the output can differ for the same state depending on which input is currently applied (as seen directly in Example 1: state $B$ produces output $0$ on input $0$ but output $1$ on input $1$).
3. Cycle $t+1$. Per Section 2's Moore case: $\lambda(s)$ for cycle $t$ depends only on $s$, which was fixed *before* $x$ arrived; $x$ can only affect the output indirectly, by first causing a transition to $\delta(s,x)$ at the next clock edge (the boundary between cycle $t$ and $t+1$), after which $\lambda(\delta(s,x))$ becomes the output during cycle $t+1$.
4. Starting state $A$: cycle 1, input $0$: $A\to A$/output $0$. Cycle 2, input $1$: $A\to B$/output $0$. Cycle 3, input $1$: $B\to B$/output $1$. Cycle 4, input $0$: $B\to A$/output $0$. Cycle 5, input $1$: $A\to B$/output $0$. Output sequence: $0,0,1,0,0$.
5. Section 4's conversion sets $\lambda_{Mealy}(s,x)=\lambda_{Moore}(s)$ for every $x$ — since this new output function never actually depends on the input value, it can be defined directly on the *existing* set of states $S$, with no need to distinguish "which input value led here," because the output was never going to depend on that anyway. Section 3's conversion, going the other way, has to encode information the Mealy machine's output function *does* depend on (which specific transition/input produced a given output) into the *state* itself (since Moore states must fully determine output), which generally requires splitting one Mealy state into multiple Moore states — one per distinct incoming-output value that state can be reached with.
6. At most $3\times2=6$ states (Section 3: one new state per reachable (Mealy state, incoming-output) combination, and with 3 states each reachable via 2 distinct output values, the upper bound is the product). "At most," not "exactly," because some of these 6 combinations might not actually be *reachable* in practice (e.g. if a particular state can only ever be entered via transitions producing one specific output value, despite the problem stating "2 different possible" values in the abstract — the actual reachable count depends on the specific transition structure, and Section 3's construction only creates states for combinations that genuinely occur along some path from the initial state).
7. **No, not necessarily** — the round trip can produce *more* states than the original. Section 3's splitting rule creates one new Moore state per reachable (destination state, incoming-output) pair, where the incoming output on a transition $\delta(s',x)=s$ is $\lambda_{Mealy}(s',x)$. After Section 4's conversion, $\lambda_{Mealy}(s',x)=\lambda_{Moore}(s')$ — a function of the *source* state $s'$, not the destination $s$. So if a single destination state $s$ is reachable via transitions from two (or more) source states $s'_1,s'_2$ with *different* Moore outputs ($\lambda_{Moore}(s'_1)\ne\lambda_{Moore}(s'_2)$), then $s$ receives two different incoming-output labels and gets split into two separate Moore states in the round trip, even though the original machine needed only one state for $s$. Concrete counterexample: states $A$ (output $0$), $B$ (output $1$), with transitions $A\xrightarrow{0}B$, $A\xrightarrow{1}A$, $B\xrightarrow{0}B$, $B\xrightarrow{1}A$. State $A$ is reached both from itself (via input $1$, source output $\lambda_{Moore}(A)=0$) and from $B$ (via input $1$, source output $\lambda_{Moore}(B)=1$) — two different incoming outputs, so $A$ splits into $A_0,A_1$. Symmetrically $B$ is reached from $A$ (input $0$, output $0$) and from itself (input $0$, output $1$) — $B$ splits into $B_0,B_1$. The round-trip Moore machine ends up with 4 states ($A_0,A_1,B_0,B_1$), double the original 2 — so the round trip does not, in general, preserve state count; it depends on whether any state has multiple differently-labeled source states feeding into it.
8. One valid Moore construction: states $A_0$ ("have not just seen a 1," output $0$), $B_0$ ("just saw one 1, no pair yet," output $0$), $B_1$ ("just completed a pair of consecutive 1s," output $1$) — this is exactly Example 2's converted machine, and any direct Moore design solving the same specification with minimal states will necessarily match it (up to state renaming), since a Moore machine correctly implementing "output 1 exactly one cycle after a 1-1 pair is confirmed, with the one-cycle Moore delay accounted for" needs to distinguish these same 3 histories to produce the correct one-cycle-delayed output sequence. Tracing $1,1,0,1,1,1$ from $A_0$ reproduces Example 2's table exactly, giving output sequence $0,0,1,0,0,1$ — matching.

## Summary / cheat sheet

**FSM** $=(S,I,O,\delta,\lambda)$: $\delta:S\times I\to S$ (next-state function); $\lambda$ differs by flavor.

**Moore**: $\lambda:S\to O$ (output = property of state alone; states labeled with output). **Mealy**: $\lambda:S\times I\to O$ (output = property of state + current input; transitions labeled with output).

**Timing (Section 2):** Moore output reflects an input one cycle after it arrives (via a state transition first); Mealy output reflects an input the same cycle it arrives (combinationally).

**Conversions:** Moore$\to$Mealy (Section 4): trivial, $\lambda_{Mealy}(s,x)=\lambda_{Moore}(s)$, no new states, no timing shift. Mealy$\to$Moore (Section 3): split each state into one per reachable (state, incoming-output) pair; introduces states and a one-cycle output delay.

**State diagram $\leftrightarrow$ state table:** lossless, purely notational (Section 1) — never a different design choice.

## Used later in
- [[6.004-computation-structures/notes/12-fsm-design-procedure]] — the FSM 5-tuple, state diagrams/tables, and the Moore/Mealy output-signature distinction are all reused directly as the input this note's encoding procedure operates on.
