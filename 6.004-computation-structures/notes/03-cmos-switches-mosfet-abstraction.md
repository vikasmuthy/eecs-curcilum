---
title: "CMOS Switches & the MOSFET-as-Switch Abstraction"
course: "6.004"
topic_number: 03
prerequisites: ["6.004 Topic 02 — Canonical Forms & Simplification", "6.002 Topic 04 — Nonlinear Elements, Digital Abstraction, MOSFET Model"]
status: done
---

# CMOS Switches & the MOSFET-as-Switch Abstraction

## Why this matters

Topics 01 and 02 built an entire toolbox — Boolean algebra, truth tables, canonical SOP/POS, K-map minimization — for manipulating logic functions as pure symbols. None of it says how to build a physical object that actually computes one of these functions. This note supplies the missing physical primitive: a **voltage-controlled switch**, realized by a MOSFET, whose open/closed state is set by a single bit. It then shows a purely mechanical rule for wiring such switches together — **series connection realizes AND, parallel connection realizes OR** — that turns any Boolean expression built from literals into a concrete network topology. Skip this topic and Topic 04 (static CMOS gates) has no device-level foundation to build the inverter, NAND, and NOR gates on; skip the series/parallel rule specifically and there is no principled way to go from "I minimized this K-map" to "here is the transistor network that implements it."

## Builds on

- [[6.004-computation-structures/notes/01-boolean-algebra-logic-functions]] — AND, OR, NOT, literals (a variable or its complement), and the convention that any Boolean expression is built by composing these three operations.
- [[6.004-computation-structures/notes/02-canonical-forms-simplification]] — sum-of-products (SOP) and product-of-sums (POS) expressions; this note's switch networks turn out to directly realize both forms as network topologies (parallel-of-series for SOP-style expressions, series-of-parallel for POS-style expressions).
- [[6.002-circuits-and-electronics/notes/04-nonlinear-elements-digital-abstraction-mosfet-model]] — the NMOS switch-level model and the digital abstraction (logic 0/1 as voltage bands). That note's status is `in progress`, not `done`, so per this vault's zero-assumed-knowledge rule the relevant result is recapped in full below rather than just linked.

## Core definitions

- **NMOS (n-channel enhancement-mode MOSFET)** — recapped from 6.002 Topic 04: a three-terminal (gate $G$, drain $D$, source $S$; body terminal ignored below) device. Define $v_{GS} = v_G - v_S$. In the **switch-level model**, the device is an open switch between $D$ and $S$ when $v_{GS} < V_{Tn}$ ("cutoff"), and a closed switch when $v_{GS} \ge V_{Tn}$ ("on"), where $V_{Tn} > 0$ is the NMOS **threshold voltage**. The gate draws no current ($i_G = 0$) in this model.
- **$V_{Tn}$, $V_{Tp}$** — this note splits 6.002 Topic 04's single symbol $V_T$ into two, because that note only ever discussed one device type. $V_{Tn}$ is the NMOS threshold voltage (by convention positive, e.g. $\approx 1\ \text{V}$ for a $5\ \text{V}$ logic family). $V_{Tp}$ is the **PMOS** threshold voltage, defined below (by convention negative). This is a notation refinement, not a new quantity — $V_{Tn}$ *is* the $V_T$ of 6.002 Topic 04, renamed here only because a second device with its own threshold now exists in the same discussion.
- **PMOS (p-channel enhancement-mode MOSFET)** — a second three-terminal device (terminals $G, D, S$, body ignored), built from complementary semiconductor regions to the NMOS (physics not derived here, exactly as 6.002 Topic 04 declined to derive NMOS physics — both are taken as terminal ("black-box") models). Using the same voltage definition $v_{GS} = v_G - v_S$: the PMOS is a closed switch between $D$ and $S$ when $v_{GS} \le V_{Tp}$, and an open switch when $v_{GS} > V_{Tp}$, where $V_{Tp} < 0$ (e.g. $\approx -1\ \text{V}$). Gate current $i_G = 0$, same idealization as NMOS.
- **CMOS (complementary MOS)** — the naming for circuits built using *both* device types together, exploiting the fact that their on/off conditions are complementary (opposite) responses to the same gate voltage. This note derives exactly what "complementary" means below.
- **Ideal switch abstraction** — a further idealization on top of the switch-level model above, used for all of Topic 04 onward: discard $R_{ON}$ entirely and treat a "closed" switch as a perfect (zero-resistance) short and an "open" switch as a perfect (infinite-resistance) break, with the open/closed state determined purely by a Boolean control bit rather than an analog gate voltage. This is the right level of detail for answering "does this network of switches connect node $A$ to node $B$?" — the question Topic 04 and onward actually needs answered. The finite-$R_{ON}$, analog-voltage version of the switch model is reserved for later timing/delay analysis (Topic 22).
- **Control literal** — the Boolean condition (a variable $X$ or its complement $\bar X$) under which a given ideal switch is closed. Realized directly by device choice: an NMOS switch driven by bit $X$ is closed exactly when $X = 1$ (contributing literal $X$); a PMOS switch driven by the same bit $X$ is closed exactly when $X = 0$ (contributing literal $\bar X$) — derived formally below.
- **Connectivity function** $C(x_1, \dots, x_n)$ — for a two-terminal network of switches whose control bits are $x_1, \dots, x_n$, the Boolean function that equals $1$ exactly for those bit assignments under which the network provides at least one conducting path between its two terminals, and $0$ for every assignment under which it does not.
- **Series composition** — two switch (sub)networks, the first between terminals $A$ and $M$, the second between $M$ and $B$, sharing only the intermediate node $M$ (i.e., forming a single path $A \to M \to B$ with no other connection to $M$).
- **Parallel composition** — two switch (sub)networks, both connected directly between the same pair of terminals $A$ and $B$, forming two independent candidate paths between them.

## Intuition

**A MOSFET, at the switch level, is a relay whose coil is the gate.** An ordinary electromechanical relay has a control coil and a pair of contacts; energize the coil and the contacts close. An NMOS switch is the same idea with one refinement: the "coil" is a voltage (the gate, relative to the source) rather than a current, and it takes no continuous current to hold ($i_G = 0$), only a voltage level. NMOS closes when that control voltage is *high* — exactly the switch-level rule recapped above.

**PMOS is the same relay wired to close on the opposite signal.** Its terminal equation is identical in form to NMOS's ($v_{GS}$ compared against a threshold), but the inequality flips direction and the threshold itself is negative. The practical consequence, derived rigorously below for the common case where a switch's source terminal is tied to a fixed supply rail: an NMOS switch driven by bit $X$ conducts when $X=1$; a PMOS switch driven by the *same* bit $X$ conducts when $X=0$. Nothing about the bit itself was inverted — only the *device's response* to it flipped. This is the entire content of the word "complementary" in CMOS: it names a pair of devices with opposite responses to the same control signal, which is what will let Topic 04 build an inverter from exactly one NMOS and one PMOS with their gates tied together, no separate "NOT gate device" required.

**Wiring switches in series is physically the same thing as ANDing their conditions.** Current can only flow $A \to B$ through two switches in series if *both* provide a path — miss either one and the chain is broken. Wiring switches in parallel is physically the same thing as ORing their conditions: current can flow through *either* branch, so the pair conducts if at least one does. Because a control literal is just "0 or 1, determined by a device and a bit," and because series/parallel composition directly implements AND/OR of those literals, building a switch network for a target Boolean expression becomes a purely mechanical translation: draw the expression's AND/OR structure as a network topology, and place one NMOS or PMOS switch per literal depending on whether it's uncomplemented or complemented.

**This mechanism can only build "positive" combinations of literals, not invert a whole subexpression.** Series/parallel composition gives you AND and OR of literals for free, and complementation of a *single variable* for free (device choice), but there is no way to build $\overline{A+B}$ (the complement of a two-variable subexpression) by composing switches this way — inverting a compound expression is an active operation that needs an amplifying device, not just wiring. That gap is exactly what Topic 04's inverter (and, from it, NAND/NOR) fills.

## Derivation / formalism

### 1. Recap: the NMOS switch-level model (from 6.002 Topic 04)

Terminals $G, D, S$; define $v_{GS} = v_G - v_S$. The model states:
$$
\text{NMOS: closed (on) if } v_{GS} \ge V_{Tn}; \qquad \text{open (off) if } v_{GS} < V_{Tn}, \qquad V_{Tn} > 0.
$$
Gate current is zero in this model ($i_G = 0$): the control terminal draws no current, only sets a voltage.

### 2. The PMOS switch-level model, stated symmetrically

By the same style of terminal definition, with the same $v_{GS} = v_G - v_S$:
$$
\text{PMOS: closed (on) if } v_{GS} \le V_{Tp}; \qquad \text{open (off) if } v_{GS} > V_{Tp}, \qquad V_{Tp} < 0.
$$
Equivalently, defining $v_{SG} = -v_{GS} = v_S - v_G$ (source-to-gate voltage, the natural variable for this device since it turns on when the source is *high relative to* the gate): PMOS is on when $v_{SG} \ge -V_{Tp} = |V_{Tp}|$. Both forms say the same thing; the $v_{GS}$ form is used below so NMOS and PMOS can be compared using one shared variable.

### 3. Why "NMOS on for bit 1, PMOS on for bit 0" follows from the terminal model (not asserted by fiat)

Consider the common digital-design configuration: a device's source terminal is tied to a fixed rail, and its gate is driven by a logic signal that takes one of two rail voltages, $0\ \text{V}$ (representing bit $0$) or $V_{DD}$ (representing bit $1$, the supply voltage), consistent with the digital abstraction of 6.002 Topic 04.

**NMOS, source tied to $0\ \text{V}$ (ground), gate driven by bit $X$:**
$$v_{GS} = V(X) - 0\ \text{V} = V(X)$$
where $V(X)$ denotes the rail voltage encoding bit $X$ ($V(0)=0\ \text{V}$, $V(1)=V_{DD}$).
- If $X = 0$: $v_{GS} = 0\ \text{V}$. Since $V_{Tn} > 0$, $v_{GS} = 0 < V_{Tn}$ — **off**.
- If $X = 1$: $v_{GS} = V_{DD}$. A basic device-sizing requirement for any usable logic family is $V_{DD} > V_{Tn}$ (otherwise the device could never be driven on); assuming this, $v_{GS} = V_{DD} \ge V_{Tn}$ — **on**.

So an NMOS switch with a grounded source and gate driven by bit $X$ is closed exactly when $X = 1$: it contributes the literal $X$.

**PMOS, source tied to $V_{DD}$, gate driven by bit $X$:**
$$v_{GS} = V(X) - V_{DD}$$
- If $X = 1$: $v_{GS} = V_{DD} - V_{DD} = 0\ \text{V}$. Since $V_{Tp} < 0$, $v_{GS} = 0 > V_{Tp}$ — **off**.
- If $X = 0$: $v_{GS} = 0 - V_{DD} = -V_{DD}$. Assuming $|V_{Tp}| < V_{DD}$ (the PMOS analogue of the sizing requirement above), $-V_{DD} \le V_{Tp}$ — **on**.

So a PMOS switch with its source tied to $V_{DD}$ and gate driven by bit $X$ is closed exactly when $X = 0$: it contributes the literal $\bar X$.

This is a derived fact, not a new postulate: both device types obey one shared terminal equation ($v_{GS}$ vs. a threshold), and the opposite signs of $V_{Tn}$ and $V_{Tp}$ are the *entire* reason their digital on-conditions come out complementary.

### 4. Series composition realizes AND

Let switch network $S_1$ connect terminal $A$ to intermediate node $M$, closed exactly when control condition $C_1$ holds; let $S_2$ connect $M$ to terminal $B$, closed exactly when $C_2$ holds, with $M$ touching nothing else. A conducting path exists from $A$ to $B$ if and only if both a path exists from $A$ to $M$ *and* a path exists from $M$ to $B$ — there is no other way to cross from $A$ to $B$. By perfect induction over the four combinations of $(C_1, C_2)$:

| $C_1$ | $C_2$ | $S_1$ | $S_2$ | Path $A\to B$? |
|---|---|---|---|---|
| 0 | 0 | open | open | no |
| 0 | 1 | open | closed | no ($S_1$ blocks) |
| 1 | 0 | closed | open | no ($S_2$ blocks) |
| 1 | 1 | closed | closed | yes |

The connectivity function is $1$ in exactly the row where $C_1=1$ and $C_2=1$, which is the truth table of AND (6.004 Topic 01). So:
$$C_{\text{series}}(C_1, C_2) = C_1 \cdot C_2$$

### 5. Parallel composition realizes OR

Let switch networks $S_1$ and $S_2$ both connect the same terminals $A$ and $B$ directly (two independent branches), closed under conditions $C_1$ and $C_2$ respectively. A conducting path exists from $A$ to $B$ if *either* branch is closed (they're independent alternatives, not required to both work). By perfect induction:

| $C_1$ | $C_2$ | $S_1$ | $S_2$ | Path $A\to B$? |
|---|---|---|---|---|
| 0 | 0 | open | open | no |
| 0 | 1 | open | closed | yes (via $S_2$) |
| 1 | 0 | closed | open | yes (via $S_1$) |
| 1 | 1 | closed | closed | yes (either) |

This is exactly the truth table of OR:
$$C_{\text{parallel}}(C_1, C_2) = C_1 + C_2$$

### 6. General claim: nested series/parallel composition realizes any AND/OR tree of literals

Sections 4–5 handled two single switches. The same two rules apply recursively to *subnetworks*, not just individual switches: if $N_1$ has connectivity function $F_1$ and $N_2$ has connectivity function $F_2$ (each possibly itself built from many switches), placing $N_1$ and $N_2$ in series gives a network with connectivity function $F_1 \cdot F_2$, and placing them in parallel gives $F_1 + F_2$ — the derivations in Sections 4–5 never used the assumption that $S_1, S_2$ were single devices, only that each had a well-defined closed/open condition, which any subnetwork's connectivity function provides. Consequently, any Boolean expression built by nesting AND (series) and OR (parallel) of literals — with each literal realized by one NMOS (uncomplemented) or one PMOS (complemented) switch per Section 3 — corresponds to a switch network whose connectivity function is exactly that expression. This includes both canonical shapes from Topic 02: a **parallel-of-series** network (OR of AND-terms) directly realizes an SOP expression; a **series-of-parallel** network (AND of OR-terms) directly realizes a POS expression.

## Worked examples

### Example 1 — Building and verifying a switch network for $F(A,B,C) = AB + C$

**Network construction.** $F$ is an OR of two terms: $AB$ (an AND of two uncomplemented literals) and $C$ (a single uncomplemented literal). By Section 6: realize $AB$ as two NMOS switches in series (control bits $A$ then $B$, both uncomplemented so both NMOS, by Section 3); realize $C$ as one NMOS switch; place the series pair in parallel with the single switch, all between the same two terminals $X$ and $Y$.

**Connectivity function, derived from the composition rules.** Series pair: $C_{AB}(A,B) = A \cdot B$ (Section 4). Parallel with the $C$ switch: $C_{\text{total}}(A,B,C) = C_{AB} + C_{C} = AB + C$ (Section 5). Matches the target $F$ by construction.

**Perfect-induction verification** (all 8 rows, confirming the physical network's connectivity matches $F=AB+C$ exactly):

| $A$ | $B$ | $C$ | $AB$ | Series switches ($A,B$) closed? | Parallel ($C$) switch closed? | Path $X\to Y$? | $F=AB+C$ |
|---|---|---|---|---|---|---|---|
| 0 | 0 | 0 | 0 | no | no | no | 0 |
| 0 | 0 | 1 | 0 | no | yes | yes | 1 |
| 0 | 1 | 0 | 0 | no | no | no | 0 |
| 0 | 1 | 1 | 0 | no | yes | yes | 1 |
| 1 | 0 | 0 | 0 | no | no | no | 0 |
| 1 | 0 | 1 | 0 | no | yes | yes | 1 |
| 1 | 1 | 0 | 1 | yes | no | yes | 1 |
| 1 | 1 | 1 | 1 | yes | yes | yes | 1 |

All 8 rows match. Note this network is a **parallel-of-series** topology, directly mirroring the SOP shape of $F$ (Section 6).

### Example 2 — A network with a complemented literal: $F(A,B,C) = (A+B)\bar C$

**Network construction.** $F$ is an AND of two subexpressions: $(A+B)$ (an OR of two uncomplemented literals) and $\bar C$ (a single complemented literal). By Section 6: realize $(A+B)$ as two NMOS switches in parallel (both uncomplemented); realize $\bar C$ as one **PMOS** switch (complemented literal, per Section 3's derived rule); place the parallel pair in series with the PMOS switch, between terminals $X$ and $Y$.

**Connectivity function.** Parallel pair: $C_{A+B}(A,B) = A + B$ (Section 5). In series with the PMOS switch (closed when $C=0$, i.e. contributing literal $\bar C$): $C_{\text{total}}(A,B,C) = (A+B)\cdot \bar C$ (Section 4). Matches the target $F$.

**Perfect-induction verification:**

| $A$ | $B$ | $C$ | $A+B$ | Parallel ($A,B$) closed? | PMOS ($\bar C$) closed? ($C=0$?) | Path $X\to Y$? | $F=(A+B)\bar C$ |
|---|---|---|---|---|---|---|---|
| 0 | 0 | 0 | 0 | no | yes | no | 0 |
| 0 | 0 | 1 | 0 | no | no | no | 0 |
| 0 | 1 | 0 | 1 | yes | yes | yes | 1 |
| 0 | 1 | 1 | 1 | yes | no | no | 0 |
| 1 | 0 | 0 | 1 | yes | yes | yes | 1 |
| 1 | 0 | 1 | 1 | yes | no | no | 0 |
| 1 | 1 | 0 | 1 | yes | yes | yes | 1 |
| 1 | 1 | 1 | 1 | yes | no | no | 0 |

All 8 rows match. This network is a **series-of-parallel** topology, mirroring the $(A+B)\bar C$ POS-style shape, and it needed exactly one PMOS switch — the one place a complemented literal appeared.

## Common pitfalls

- **Swapping the NMOS/PMOS on-conditions.** NMOS conducts for its control bit $=1$; PMOS conducts for its control bit $=0$. Using an NMOS where the literal is complemented (or vice versa) silently realizes the wrong function — the network still "works" electrically, just computes something else.
- **Forgetting the ideal switch abstraction has thrown away $R_{ON}$.** This model only answers "is there a conducting path" (a yes/no, Boolean question). It cannot be used to compute an actual output voltage, current, or delay — those require the finite-$R_{ON}$ switch-level model (or the full piecewise-linear model) from 6.002 Topic 04, reserved in this vault for Topic 22 onward.
- **Trying to invert a compound subexpression with switches alone.** Series/parallel composition of literal-controlled switches can realize any AND/OR tree of literals (Section 6), but cannot realize the complement of a multi-literal subexpression like $\overline{A+B}$ directly — only single-variable complementation (via device choice) comes for free. Building an inverting function requires an active device arrangement (Topic 04's inverter), not more wiring.
- **Assuming every possible switch network is series-parallel.** Section 6's composition rule only covers networks built by repeated series/parallel nesting. More general ("bridge") topologies exist and can realize functions not expressible as a simple series/parallel tree, but analyzing them needs more machinery than the two composition rules here; every network actually used in this course is series-parallel by construction, so this gap doesn't block anything ahead, but don't assume the simple rule applies to an arbitrary hand-drawn switch diagram.
- **Applying the "source tied to a fixed rail" simplification from Section 3 unconditionally.** That derivation assumed a specific configuration (source at a fixed supply rail, gate driven directly by a logic signal). In a series chain of switches (Example 1, Example 2), an interior switch's source is the intermediate node $M$, whose voltage depends on the rest of the network's state, not a fixed rail — the simple "on for 1 / on for 0" digital rule is still the one used throughout gate design (Topic 04 makes it exact for the specific pull-up/pull-down configurations used there), but it is worth remembering it is a derived simplification of the more general terminal model, not the terminal model itself.
- **Ignoring the body terminal's assumed tie-off.** Both device models above silently assume the body/substrate terminal is tied so that $v_{SB}=0$ (NMOS body to ground, PMOS body to $V_{DD}$), which avoids a secondary "body effect" on the threshold voltage. This is a standard simplification, not a universal truth about all MOSFET circuits — flagged here so it isn't mistaken for a fact about the device rather than a modeling choice.

## Self-check

### Questions

1. (Easy) An NMOS switch has its source grounded and its gate driven by bit $X=0$. Is it open or closed?
2. (Easy) A PMOS switch has its source tied to $V_{DD}$ and its gate driven by bit $X=1$. Is it open or closed?
3. (Easy) Two switches are placed in parallel between terminals $A$ and $B$, with control conditions $X$ and $\bar X$ (the same variable, complemented in one branch). Simplify the connectivity function, and explain in one sentence what this means physically about the network.
4. (Medium) Draw (in words) a switch network realizing $F(A,B,C,D) = AB + CD$, stating which device type is used for each of the four switches and how they're composed.
5. (Medium) For the network in Question 4, write the connectivity function using the series/parallel composition rules (Sections 4–5) and confirm it equals $AB+CD$.
6. (Medium) A network has two switches in series between $A$ and $B$: the first is an NMOS controlled by bit $X$, the second is a PMOS controlled by the *same* bit $X$. Find the connectivity function as a function of $X$, and simplify it using Boolean algebra (6.004 Topic 01). What does the result tell you about this particular series combination?
7. (Hard) Realize $F(A,B,C) = A(B+\bar C)$ as a switch network: state the topology (series/parallel structure), the device used for each literal, and verify the network's connectivity function against $F$ by perfect induction over all 8 rows.
8. (Hard) Using only the terminal equations from Sections 1–2 (not the "source tied to a fixed rail" shortcut of Section 3), determine the on/off state of an NMOS switch whose source is itself the output node of another switch presently sitting at $v_S = 0.2\ \text{V}$ (not exactly ground), with $v_G = V_{DD} = 5\ \text{V}$ and $V_{Tn}=1\ \text{V}$. Is the simple "on iff gate bit is 1" digital rule still exactly correct here, or only approximately so? Explain.

### Answers

1. $v_{GS} = V(0) - 0 = 0\ \text{V} < V_{Tn}$ (since $V_{Tn}>0$) — **open**.
2. $v_{GS} = V(1) - V_{DD} = V_{DD}-V_{DD} = 0\ \text{V} > V_{Tp}$ (since $V_{Tp}<0$) — **open**. (Both Q1 and Q2 land on "open" — Q1 because NMOS wants a high gate to turn on and got a low one; Q2 because PMOS wants a low gate and got a high one.)
3. By the parallel composition rule (Section 5), $C_{\text{parallel}} = X + \bar X = 1$ (identity/complement law from 6.004 Topic 01). Physically: this network is closed for *every* input — it's always a short between $A$ and $B$, regardless of $X$, since one branch or the other is guaranteed to be closed at all times.
4. $F=AB+CD$ is an OR of two AND-terms, all four literals uncomplemented. Realize $AB$ as two NMOS switches in series (controlled by $A$ then $B$); realize $CD$ as two NMOS switches in series (controlled by $C$ then $D$); place the two series pairs in parallel with each other, all between the same two terminals. All four switches are NMOS since no literal is complemented.
5. Series pair 1: $C_{AB} = A\cdot B$ (Section 4). Series pair 2: $C_{CD} = C\cdot D$ (Section 4). Parallel combination: $C_{\text{total}} = C_{AB} + C_{CD} = AB+CD$ (Section 5) — matches $F$ exactly.
6. NMOS contributes literal $X$ (closed iff $X=1$); PMOS contributes literal $\bar X$ (closed iff $X=0$). In series (Section 4): $C_{\text{series}}(X) = X \cdot \bar X$. By the complement law (6.004 Topic 01), $X\cdot\bar X = 0$ — the network is *never* conducting, for either value of $X$. This is the series-composition mirror of Question 3's parallel case: an NMOS and a PMOS sharing the same control bit, wired in series, form a switch pair that can never both be on simultaneously (Section 3 showed their on-conditions are exact complements of each other), so a series connection of the two is permanently open. (This exact complementary pair, wired differently — in Topic 04 — is what builds the CMOS inverter's two halves.)
7. $F = A(B+\bar C)$ is a series composition of a single literal $A$ (NMOS) with a parallel combination of $B$ (NMOS) and $\bar C$ (PMOS). Topology: NMOS switch for $A$, in series with [NMOS switch for $B$ in parallel with PMOS switch for $\bar C$]. Connectivity function: parallel part $= B+\bar C$ (Section 5), then in series with $A$: $C_{\text{total}} = A(B+\bar C)$ (Section 4) — matches $F$ by construction. Perfect-induction check over all 8 rows of $A,B,C$:

   | $A$ | $B$ | $C$ | $B+\bar C$ | $A(B+\bar C)$ |
   |---|---|---|---|---|
   | 0 | 0 | 0 | 1 | 0 |
   | 0 | 0 | 1 | 0 | 0 |
   | 0 | 1 | 0 | 1 | 0 |
   | 0 | 1 | 1 | 1 | 0 |
   | 1 | 0 | 0 | 1 | 1 |
   | 1 | 0 | 1 | 0 | 0 |
   | 1 | 1 | 0 | 1 | 1 |
   | 1 | 1 | 1 | 1 | 1 |

   All rows match $F=A(B+\bar C)$ (nonzero exactly at $A{=}1$ with $B{=}1$ or $C{=}0$).
8. Using the exact terminal equation (Section 1): $v_{GS} = v_G - v_S = 5\ \text{V} - 0.2\ \text{V} = 4.8\ \text{V}$. Compare to $V_{Tn}=1\ \text{V}$: $4.8\ \text{V} \ge 1\ \text{V}$, so the switch is still **on**. The simple digital rule ("on iff gate bit is 1") is only an *approximation* here, valid because $0.2\ \text{V}$ is close enough to the ideal $0\ \text{V}$ ground reference that it doesn't change the on/off conclusion — but the exact terminal model (Section 1–2), not the Section 3 shortcut, is what actually has to be checked whenever a switch's source isn't sitting at a clean rail voltage (e.g. an interior node of a series chain, as flagged in Common pitfalls).

## Summary / cheat sheet

**Device on-conditions** (terminal model, $v_{GS}=v_G-v_S$):
| Device | Closed (on) when | Threshold sign |
|---|---|---|
| NMOS | $v_{GS} \ge V_{Tn}$ | $V_{Tn}>0$ |
| PMOS | $v_{GS} \le V_{Tp}$ | $V_{Tp}<0$ |

**Digital shortcut** (source at fixed rail, gate driven by bit $X$, derived in Section 3): NMOS closed iff $X=1$ (literal $X$); PMOS closed iff $X=0$ (literal $\bar X$).

**Ideal switch abstraction:** closed = zero resistance, open = infinite resistance; state set purely by the control bit. Used for connectivity ("is there a path") questions only, not analog/delay analysis.

**Composition rules:**
- Series $\to$ AND: $C_{\text{series}} = C_1 \cdot C_2$.
- Parallel $\to$ OR: $C_{\text{parallel}} = C_1 + C_2$.
- Nested series/parallel of literal-controlled switches (NMOS for uncomplemented, PMOS for complemented) realizes any AND/OR tree of literals — parallel-of-series realizes SOP; series-of-parallel realizes POS.
- Cannot realize inversion of a compound (multi-literal) subexpression by wiring alone — needs an active device (Topic 04).

## Used later in
- [[6.004-computation-structures/notes/04-static-cmos-gates]] — the pull-down network (PDN) and pull-up network (PUN) of every static CMOS gate are built directly from this note's NMOS-only / PMOS-only series-parallel switch networks; the dual-network construction there is proved by structural induction on this note's series/parallel composition rules.
- [[6.004-computation-structures/notes/05-multiplexers-decoders-encoders]] — the transmission gate combines one NMOS and one PMOS switch from this note (in parallel) to build a controlled bidirectional switch, used for the pass-transistor multiplexer design.
