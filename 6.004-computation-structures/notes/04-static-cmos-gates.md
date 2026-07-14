---
title: "Static CMOS Gates (Inverter, Pull-Up/Down Networks, NAND/NOR, Complex Gates)"
course: "6.004"
topic_number: 04
prerequisites: ["6.004 Topic 03 — CMOS Switches & the MOSFET-as-Switch Abstraction"]
status: done
---

# Static CMOS Gates (Inverter, Pull-Up/Down Networks, NAND/NOR, Complex Gates)

## Why this matters

Topic 03 ended with a real gap: series/parallel switch networks can realize any AND/OR tree of literals, but cannot invert a compound (multi-literal) subexpression — there's no way to wire switches alone into something that computes $\overline{A+B}$. This note closes that gap by combining *two* switch networks (one built from PMOS, one from NMOS) around a shared output node, in a specific topology — the **pull-up/pull-down** structure — that turns "connect the output to $V_{DD}$ under condition $F$, and to ground under condition $\bar F$" into an actual inverting logic gate. Every standard combinational gate used from here on (NAND, NOR, and general "complex gates") is one instance of this single pattern. Skip this topic and there is no way to build an actual inverter, and therefore no way to realize any Boolean function that isn't already a pure AND/OR tree of literals — which, by De Morgan's laws (6.004 Topic 01), is most of them.

## Builds on

- [[6.004-computation-structures/notes/03-cmos-switches-mosfet-abstraction]] — the ideal switch abstraction, NMOS/PMOS on-conditions (NMOS closed for control bit $1$, PMOS closed for control bit $0$), and the series$=$AND / parallel$=$OR composition rules. This note places one NMOS-only network and one PMOS-only network, each built using those exact rules, around a shared output node.
- [[6.004-computation-structures/notes/01-boolean-algebra-logic-functions]] — De Morgan's laws, used below to show algebraically why a network satisfying "PMOS network conducts exactly when NMOS network doesn't" is guaranteed to exist for any function built from the pull-down network's structure.

## Core definitions

- **Output node** — the single node whose voltage this note treats as the gate's output signal, distinct from the fixed supply rails $V_{DD}$ (logic 1) and ground/$GND$ (logic 0, written $0\ \text{V}$).
- **Pull-down network (PDN)** — a switch network, built entirely from NMOS switches using the series/parallel composition rules of Topic 03, connecting the output node to ground. Its connectivity function is written $F_{PD}$.
- **Pull-up network (PUN)** — a switch network, built entirely from PMOS switches, connecting the output node to $V_{DD}$. Its connectivity function is written $F_{PU}$.
- **Static CMOS gate** — a circuit consisting of exactly one PDN and one PUN sharing the same output node and the same set of input control bits, satisfying the **complementary-network condition** defined and derived below: $F_{PU} = \overline{F_{PD}}$.
- **Complementary-network condition** — the requirement that, for every input combination, exactly one of \{PUN conducts, PDN conducts\} holds (never both, never neither). This is what guarantees the output node is always driven to a well-defined logic level rather than left floating or short-circuited.
- **Floating node** — a node connected to neither $V_{DD}$ nor ground (both PUN and PDN open for the current input); its voltage is undefined by the circuit (it retains whatever charge was last on it), which is why the complementary-network condition forbids this case in a *static* gate.
- **Contention / crowbar condition** — a node connected to *both* $V_{DD}$ and ground simultaneously (both PUN and PDN conducting for the current input); this would force significant current directly from supply to ground through both networks, which the complementary-network condition also forbids.
- **CMOS inverter** — the smallest static CMOS gate: PDN is a single NMOS switch controlled by input $A$; PUN is a single PMOS switch controlled by the same input $A$; output taken from the shared node.
- **NAND gate** — a static CMOS gate whose PDN realizes the AND of its inputs (inputs in series, since AND = series per Topic 03) and whose PUN realizes the OR of the complemented inputs (inputs in parallel), computing $\overline{\text{AND}}$ overall.
- **NOR gate** — a static CMOS gate whose PDN realizes the OR of its inputs (inputs in parallel) and whose PUN realizes the AND of the complemented inputs (inputs in series), computing $\overline{\text{OR}}$ overall.
- **Complex gate** — a static CMOS gate whose PDN realizes an arbitrary series/parallel AND/OR tree $F_{PD}$ (not just a single AND or OR of literals), with the PUN built as its exact dual network (defined and derived below), computing $\overline{F_{PD}}$.
- **Dual network** — given a series-parallel PDN, the network obtained by replacing every series connection with a parallel connection (and vice versa) and every NMOS switch controlled by literal $L$ with a PMOS switch controlled by the complemented literal $\bar L$ — proved below to always satisfy the complementary-network condition.

## Intuition

**A static CMOS gate is two switch networks fighting to be the only one that can win — by construction, never both.** The pull-down network's job is to pull the output all the way down to ground when a certain Boolean condition holds; the pull-up network's job is to pull the output all the way up to $V_{DD}$ under the *opposite* condition. Because they're built to be exact complements of each other, exactly one of them is ever "live" for a given input, so the output is always cleanly driven to one of the two rails — never left floating, never fought over by both networks at once.

**Why this inverts.** In Topic 03, an NMOS switch driven by literal $L$ is closed when $L=1$; the complementary PMOS switch driven by the *same underlying variable but wired to realize the complemented network* is closed exactly when the PDN is open. If the PDN conducts (output pulled to ground $=$ logic 0) precisely when some function $F$ is true, then the PUN — being the PDN's exact structural opposite — conducts (output pulled to $V_{DD}$ $=$ logic 1) precisely when $F$ is false. The gate's output is therefore $\overline{F}$: whatever Boolean function the pull-down network computes, the gate as a whole computes its complement. This is the mechanism, not a coincidence: it's *why* every basic CMOS gate (inverter, NAND, NOR) is inherently inverting, and why building a non-inverting function (like a plain AND or OR gate) always costs an extra inverter stage tacked onto a NAND or NOR.

**The dual-network construction is De Morgan's laws made physical.** Building the PUN as the PDN's structural dual (series$\leftrightarrow$parallel, literal $\leftrightarrow$ complemented literal) is exactly what De Morgan's laws (6.004 Topic 01) say algebraically: complementing an AND-of-terms expression turns each AND into an OR and complements each literal, and vice versa. Topic 03 already showed series realizes AND and parallel realizes OR; swapping series/parallel while complementing every literal is the network-topology translation of applying De Morgan's twice (once for the outer connective, distributing the complement across it, and once per literal).

## Derivation / formalism

### 1. The complementary-network condition guarantees a well-defined output

Claim: if $F_{PU} = \overline{F_{PD}}$ for all input combinations, then for every input, exactly one of \{PUN conducting, PDN conducting\} holds.

**Proof.** $F_{PD}$ and $F_{PU}=\overline{F_{PD}}$ are complements of each other as Boolean functions, so for any input assignment, exactly one of $F_{PD}=1, F_{PU}=1$ can hold (they can never both be $1$, since $X$ and $\bar X$ are never simultaneously $1$ — the complement law from 6.004 Topic 01 — and they can never both be $0$, since $X+\bar X=1$ always, the same complement law's dual form). By definition, $F_{PD}=1$ means the PDN conducts (output shorted to ground) and $F_{PU}=1$ means the PUN conducts (output shorted to $V_{DD}$). So exactly one of "PDN conducts" / "PUN conducts" holds for every input, ruling out both the floating case (neither conducts) and the contention case (both conduct). $\blacksquare$

### 2. The CMOS inverter computes NOT

PDN: single NMOS switch, control literal $A$ (so $F_{PD} = A$, by Topic 03's device rule — NMOS closed iff its control bit is 1). PUN: single PMOS switch, same control input $A$ (so $F_{PU} = \bar A$, PMOS closed iff its control bit is 0).

Check the complementary-network condition: $F_{PU} = \bar A = \overline{F_{PD}}$ — satisfied for every value of $A$ (trivially, since there's only one literal). The output is pulled to ground ($=0$) when $F_{PD}=A=1$, and pulled to $V_{DD}$ ($=1$) when $F_{PU}=\bar A = 1$, i.e. when $A=0$. So:
$$\text{Output} = \begin{cases} 0 & A=1 \\ 1 & A=0\end{cases} = \bar A$$
which is exactly the NOT function (6.004 Topic 01). This is the smallest possible static CMOS gate: 1 NMOS + 1 PMOS.

### 3. The dual-network construction, and why it always satisfies the complementary-network condition

**Construction.** Given any PDN built by Topic 03's series/parallel composition rules from literals $L_1, \dots, L_k$ (each literal a variable or its complement, each realized as one NMOS switch per Topic 03's device rule — recall a PDN is NMOS-only), define its **dual network** by: (a) replacing every series connection with parallel and every parallel connection with series, throughout the whole nested structure, and (b) replacing every NMOS switch for literal $L_i$ with a PMOS switch for the complemented literal $\bar L_i$.

**Claim: the dual network's connectivity function is $\overline{F_{PD}}$.**

**Proof, by structural induction on the PDN's series/parallel nesting.**

*Base case (single literal):* PDN is one NMOS switch for literal $L$, so $F_{PD} = L$. Its dual is one PMOS switch for $\bar L$, whose connectivity function (Topic 03's PMOS device rule, generalized from a single variable to a literal by the same argument) is $\bar L = \overline{F_{PD}}$. Holds.

*Inductive step (series):* Suppose sub-networks $N_1, N_2$ have connectivity functions $F_1, F_2$, and (inductive hypothesis) their duals $N_1', N_2'$ have connectivity functions $\overline{F_1}, \overline{F_2}$. Let $N = N_1$ series $N_2$, so $F = F_1 F_2$ (Topic 03, Section 4). Its dual $N'$, by construction, is $N_1'$ *parallel* $N_2'$ (series flips to parallel), so $N'$'s connectivity function is $\overline{F_1} + \overline{F_2}$ (Topic 03, Section 5, applied to the dual sub-networks). By De Morgan's law (6.004 Topic 01), $\overline{F_1}+\overline{F_2} = \overline{F_1 F_2} = \overline{F}$. So $N'$'s connectivity function is $\overline{F}$, matching the claim.

*Inductive step (parallel):* Symmetric. $N = N_1$ parallel $N_2 \Rightarrow F = F_1+F_2$. Dual $N' = N_1'$ series $N_2' \Rightarrow$ connectivity $=\overline{F_1}\cdot\overline{F_2}$, which by the other De Morgan's law equals $\overline{F_1+F_2} = \overline F$. Matches.

Since every series-parallel PDN is built by a finite number of series/parallel compositions starting from single literals, and both the base case and both inductive steps preserve "dual network computes the complement," by induction the dual network of *any* series-parallel PDN computes $\overline{F_{PD}}$ — exactly the complementary-network condition (Section 1) needed for a valid static CMOS gate. $\blacksquare$

This single theorem is why Section 2's inverter, and the NAND/NOR gates below, all "just work": the PUN is never designed independently — it's mechanically read off the PDN by this dual construction.

### 4. NAND gate

Target function: $F(A,B) = \overline{AB}$ (NOT-AND). Design the PDN to realize the function being complemented, $AB$: two NMOS switches in series (control $A$, then $B$) — series realizes AND, matching Topic 03 Section 4. $F_{PD}=AB$.

Apply the dual construction (Section 3): series $\to$ parallel, each literal complemented. PUN: two PMOS switches in parallel, one controlled by $A$ (PMOS on literal $A$ contributes $\bar A$ per Topic 03's device rule) and one by $B$ (contributes $\bar B$). $F_{PU} = \bar A + \bar B$.

**Check via De Morgan's:** $\overline{F_{PD}} = \overline{AB} = \bar A + \bar B$ (6.004 Topic 01's De Morgan's law) $=F_{PU}$. Condition satisfied. Gate output $= \overline{F_{PD}} = \overline{AB}$ — NAND, as intended.

### 5. NOR gate

Target function: $F(A,B) = \overline{A+B}$ (NOT-OR). Design the PDN to realize $A+B$: two NMOS switches in parallel. $F_{PD} = A+B$.

Dual construction: parallel $\to$ series, literals complemented. PUN: two PMOS switches in series, controlled by $A$ then $B$ (each PMOS contributing its complemented literal). $F_{PU} = \bar A \cdot \bar B$.

**Check:** $\overline{F_{PD}} = \overline{A+B} = \bar A\bar B$ (De Morgan's) $= F_{PU}$. Condition satisfied. Gate output $=\overline{A+B}$ — NOR.

### 6. Complex gates: an arbitrary series-parallel PDN, dualized

Nothing in Sections 3–5 required the PDN to be a single AND or OR — Section 3's theorem was proved for *any* series-parallel network. So for any Boolean function $G$ expressible as a series-parallel AND/OR tree of literals (Topic 03, Section 6), building the PDN directly from that tree and the PUN as its dual yields a single static CMOS gate computing $\overline{G}$ — no restriction to 2-input NAND/NOR. This is what "complex gate" means: e.g. a PDN realizing $AB+CD$ (the network from Topic 03's Self-check Question 4, built from NMOS) dualizes (series$\to$parallel, parallel$\to$series, literals complemented) to a PUN realizing $(\bar A+\bar B)(\bar C+\bar D)$, giving a single gate computing $\overline{AB+CD} = (\bar A+\bar B)(\bar C+\bar D)$ directly — verified worked out fully in Example 2 below.

## Worked examples

### Example 1 — NAND gate, full truth-table verification

Using Section 4's design ($F_{PD}=AB$ via series NMOS, $F_{PU}=\bar A+\bar B$ via parallel PMOS):

| $A$ | $B$ | $F_{PD}=AB$ | PDN conducts? | $F_{PU}=\bar A+\bar B$ | PUN conducts? | Output | $\overline{AB}$ |
|---|---|---|---|---|---|---|---|
| 0 | 0 | 0 | no | 1 | yes | 1 | 1 |
| 0 | 1 | 0 | no | 1 | yes | 1 | 1 |
| 1 | 0 | 0 | no | 1 | yes | 1 | 1 |
| 1 | 1 | 1 | yes | 0 | no | 0 | 0 |

Every row: exactly one of \{PDN, PUN\} conducts (complementary-network condition holds, as guaranteed by Section 3), and the resulting output matches $\overline{AB}$ in all 4 rows.

### Example 2 — A 4-input complex gate: dualizing $F_{PD}=AB+CD$

PDN (from Topic 03, Self-check Question 4): NMOS switches for $A,B$ in series, in parallel with NMOS switches for $C,D$ in series. $F_{PD}=AB+CD$.

**Dualize (Section 3):** outer parallel $\to$ series; each inner series $\to$ parallel; every literal complemented. Result: [PMOS for $\bar A$ parallel PMOS for $\bar B$] in series with [PMOS for $\bar C$ parallel PMOS for $\bar D$]. Connectivity function, built up the same way Topic 03 built the original: inner parallels give $\bar A+\bar B$ and $\bar C + \bar D$; series of those two gives $F_{PU} = (\bar A+\bar B)(\bar C+\bar D)$.

**Algebraic check via De Morgan's (applied twice, matching Section 3's inductive proof structure):**
$$\overline{F_{PD}} = \overline{AB+CD} = \overline{AB}\cdot\overline{CD} \quad \text{[De Morgan's, outer]} = (\bar A+\bar B)(\bar C+\bar D) \quad \text{[De Morgan's, each factor]} = F_{PU}$$
Matches $F_{PU}$ derived directly from the dual network topology — confirms Section 3's general theorem on this concrete 4-variable case.

**Partial truth-table check** (4 representative rows out of 16, chosen to cover both $F_{PD}=1$ cases and two $F_{PD}=0$ cases):

| $A$ | $B$ | $C$ | $D$ | $F_{PD}=AB+CD$ | Output (should be $\overline{F_{PD}}$) | $F_{PU}=(\bar A+\bar B)(\bar C+\bar D)$ |
|---|---|---|---|---|---|---|
| 1 | 1 | 0 | 0 | 1 | 0 | 0 |
| 0 | 0 | 1 | 1 | 1 | 0 | 0 |
| 1 | 0 | 1 | 0 | 0 | 1 | 1 |
| 0 | 0 | 0 | 0 | 0 | 1 | 1 |

All 4 rows: output $=\overline{F_{PD}}=F_{PU}$, consistent with Section 3's theorem for every input, not just these four (the theorem was proved for all $2^4=16$ combinations at once via structural induction, so a full 16-row table would add no additional confidence beyond confirming the induction's mechanics — these 4 rows are enough to sanity-check both a $1$-producing and a $0$-producing case on each side of the OR).

## Common pitfalls

- **Forgetting a static CMOS gate is inherently inverting.** The PUN is always the *dual* of the PDN (Section 3), which mechanically produces $\overline{F_{PD}}$, never $F_{PD}$ itself. There is no direct way to build a plain (non-inverting) AND or OR gate as a single static CMOS stage — that requires a NAND/NOR followed by a separate inverter (two gate delays, not one).
- **Building the PUN by copying the PDN's topology instead of dualizing it.** A PUN built with the *same* series/parallel structure as the PDN (just swapping NMOS for PMOS, without swapping series$\leftrightarrow$parallel or complementing literals) will generally violate the complementary-network condition — it can produce floating or contention states for some inputs, since it's no longer computing $\overline{F_{PD}}$.
- **Complementing the outer connective but not each literal (or vice versa) when dualizing by hand.** Section 3's induction needs *both* steps together (flip series/parallel *and* complement every literal) at every level of nesting — doing only one produces an incorrect dual. This mirrors the exact way De Morgan's laws require complementing both the connective and every literal, never just one.
- **Assuming the complementary-network condition is automatically satisfied by any two networks that happen to look "opposite."** It's only guaranteed for the specific dual-network construction of Section 3 (formal series/parallel-swap-plus-literal-complement); an arbitrary PUN, even one that looks structurally similar to the PDN's complement, needs to be checked (or built via the construction) rather than assumed correct.
- **Ignoring that this is called a *static* CMOS gate for a reason.** The complementary-network condition (Section 1) is what makes the gate's output well-defined at every moment in steady state (hence "static") without relying on stored charge; later note types (e.g. dynamic/clocked logic) relax this and are out of scope here — don't assume every gate topology in later courses/notes has to satisfy this exact condition.
- **Confusing "PDN realizes $F_{PD}$" with "gate output is $F_{PD}$."** The pull-down network's own connectivity function $F_{PD}$ is the condition under which the output is pulled to *ground* (logic 0); the gate's actual output as a logic value is $\overline{F_{PD}}$, not $F_{PD}$ itself — easy to drop the final complement when reading a design off its PDN.

## Self-check

### Questions

1. (Easy) What are the two conditions ("floating" and "contention") that the complementary-network condition rules out, and which network(s) (PUN, PDN) are active in each?
2. (Easy) A PDN realizes $F_{PD} = X$ (a single NMOS switch). What is the gate's output as a function of $X$, and what is this gate called?
3. (Medium) Design a 3-input NAND gate: state the PDN topology, the PUN topology (via the dual construction), and the resulting Boolean function. Verify the complementary-network condition holds using De Morgan's laws.
4. (Medium) A PDN realizes $F_{PD} = A(B+C)$ (NMOS for $A$ in series with a parallel pair of NMOS for $B,C$). Find the dual PUN's topology and connectivity function, and state the overall gate's output function.
5. (Medium) Verify your answer to Question 4 using De Morgan's laws applied to $\overline{A(B+C)}$, and confirm it matches the PUN's connectivity function.
6. (Hard) Explain, using Section 1's proof, why a designer who builds a PUN satisfying $F_{PU} = F_{PD}$ (instead of $F_{PU}=\overline{F_{PD}}$) by mistake gets a gate with contention for every input where $F_{PD}=1$ and floating for every input where $F_{PD}=0$.
7. (Hard) Using the structural induction of Section 3, dualize the PDN for $F_{PD} = (A+B)C$ step by step (identify the outer connective first, dualize it, then recurse into the sub-network), stating the PUN topology and connectivity function at each step of the recursion.
8. (Hard) A 2-input XOR function is $F(A,B) = A\bar B + \bar A B$. Explain why this function's *pull-down* network cannot be built as a simple series-parallel tree using only the literals $A,B,\bar A,\bar B$ without repeating a variable's true and complemented form in a way that still obeys Topic 03's literal-per-switch rule — construct the PDN using repeated literals if needed, and identify whether the resulting circuit is still a valid minimal-transistor-count design (you do not need to prove minimality, just reason about the transistor count compared to NAND/NOR).

### Answers

1. **Floating**: neither PUN nor PDN conducts, so the output node is connected to neither rail (undefined voltage). **Contention**: both PUN and PDN conduct simultaneously, shorting $V_{DD}$ to ground through both networks. The complementary-network condition (Section 1) guarantees exactly one of the two networks conducts for every input, ruling out both.
2. Output $=\overline{F_{PD}} = \bar X$. This is the CMOS inverter (Section 2), the smallest static CMOS gate.
3. PDN: three NMOS switches in series, controlled by $A,B,C$ — realizes $F_{PD}=ABC$ (series$=$AND, extended to 3 literals). Dual PUN (series$\to$parallel, each literal complemented): three PMOS switches in parallel, controlled by $A,B,C$ — realizes $F_{PU}=\bar A+\bar B+\bar C$. Check: $\overline{ABC} = \bar A+\bar B+\bar C$ by De Morgan's law (extended to 3 variables, applying the 2-variable law twice: $\overline{ABC}=\overline{(AB)C}=\overline{AB}+\bar C = \bar A+\bar B+\bar C$), matching $F_{PU}$. Gate output: $\overline{ABC}$ — a 3-input NAND.
4. Outer connective of $F_{PD}=A(B+C)$ is series ($A$ in series with the sub-network $B+C$). Dualize outer: series$\to$parallel, so the PUN's outer structure is a PMOS for $\bar A$ **in parallel** with the dual of the $(B+C)$ sub-network. That sub-network's connective is parallel ($B,C$), so its dual is series: PMOS for $\bar B$ in series with PMOS for $\bar C$. Full PUN: PMOS($\bar A$) in parallel with [PMOS($\bar B$) in series with PMOS($\bar C$)]. Connectivity function: inner series $=\bar B\bar C$; outer parallel with $\bar A$: $F_{PU} = \bar A + \bar B\bar C$. Gate output: $\overline{A(B+C)} = \bar A+\bar B\bar C$.
5. $\overline{A(B+C)} = \bar A + \overline{(B+C)}$ [De Morgan's, outer AND$\to$OR with complemented factors] $= \bar A + \bar B\bar C$ [De Morgan's again, inner OR$\to$AND]. Matches $F_{PU}=\bar A+\bar B\bar C$ from Question 4 exactly.
6. If $F_{PU}=F_{PD}$ (not complemented), then whenever $F_{PD}=1$: PDN conducts (by definition) *and* $F_{PU}=1$ means PUN also conducts — both networks active simultaneously $=$ contention (Section 1's proof: with $F_{PU}=F_{PD}$ instead of $\overline{F_{PD}}$, the "exactly one of $F_{PD}=1,F_{PU}=1$" argument breaks because they're now equal, not complementary — they're either both 1 or both 0 for any given input, never exactly one). Whenever $F_{PD}=0$: PDN doesn't conduct, and $F_{PU}=F_{PD}=0$ means PUN doesn't conduct either — both open $=$ floating. So every input lands in exactly one of the two forbidden cases, confirming the mistake breaks the gate for *all* inputs, not just some.
7. Outer connective of $(A+B)C$ is series: $(A+B)$ in series with $C$. **Step 1 (outer):** series$\to$parallel; dualize each side: dual of $(A+B)$ (parallel of $A,B$) is series of $\bar A,\bar B$; dual of $C$ (single literal) is a PMOS for $\bar C$. **Step 2 (assemble):** outer parallel of [series($\bar A,\bar B$)] and [$\bar C$]: PUN topology is [PMOS($\bar A$) series PMOS($\bar B$)] in parallel with [PMOS($\bar C$)]. Connectivity function: inner series $=\bar A\bar B$; parallel with $\bar C$: $F_{PU}=\bar A\bar B+\bar C$. (Check via De Morgan's: $\overline{(A+B)C} = \overline{(A+B)}+\bar C = \bar A\bar B+\bar C$ — matches.)
8. XOR's minimal SOP is $A\bar B+\bar AB$ (no further K-map simplification possible — this is one of the classic "no adjacent-pair reduction" functions, since on a 2-variable K-map the two 1-cells for XOR are diagonal, not adjacent). Realizing this as a series-parallel PDN needs, per the SOP structure, a parallel-of-series network: [NMOS($A$) series NMOS($\bar B$)] in parallel with [NMOS($\bar A$) series NMOS($B$)] — this uses 4 NMOS switches, with $A$ appearing once as an NMOS (for the literal $A$) and once more implicitly needed as $\bar A$ (requiring an inverted copy of $A$ as a separate input signal, since Topic 03's rule realizes a *literal*, and $\bar A$ is a different literal from $A$ even though it's "the same variable"). This is a valid series-parallel network (each branch is literal-controlled, composed via series/parallel exactly as Topic 03 permits), but it costs 4 transistors in the PDN alone (plus a 4-transistor dual PUN, or more commonly XOR is built from other gate combinations in practice) — substantially more than a 2-input NAND's 2 PDN transistors, illustrating that not every Boolean function is "cheap" in the direct series-parallel-plus-dual construction even though the construction always works.

## Summary / cheat sheet

**Static CMOS gate structure:** PDN (NMOS only) connects output to ground under condition $F_{PD}$; PUN (PMOS only) connects output to $V_{DD}$ under condition $F_{PU}$; gate output $= \overline{F_{PD}}$ whenever $F_{PU}=\overline{F_{PD}}$ (complementary-network condition — rules out floating/contention).

**Dual-network construction** (mechanical PUN-from-PDN recipe): swap every series$\leftrightarrow$parallel, complement every literal (NMOS$\to$PMOS on the complemented literal). Proved correct by structural induction + De Morgan's laws.

| Gate | PDN | PUN | Output |
|---|---|---|---|
| Inverter | NMOS($A$) | PMOS($A$) | $\bar A$ |
| NAND($A,B$) | NMOS($A$) series NMOS($B$) | PMOS($A$) parallel PMOS($B$) | $\overline{AB}$ |
| NOR($A,B$) | NMOS($A$) parallel NMOS($B$) | PMOS($A$) series PMOS($B$) | $\overline{A+B}$ |
| Complex gate | any series-parallel AND/OR tree $F_{PD}$ | dual of the PDN | $\overline{F_{PD}}$ |

## Used later in
- [[6.004-computation-structures/notes/05-multiplexers-decoders-encoders]] — the NAND-NAND multiplexer/decoder realization reuses the fact that static CMOS gates are inherently inverting (forcing the double-De-Morgan's construction), and the inverter is reused directly to generate the complemented control signal for each transmission gate.
- [[6.004-computation-structures/notes/06-adders-basic-arithmetic]] — the half/full adder's AND, OR, and XOR outputs are realized as ordinary combinational gates from this note.
- [[6.004-computation-structures/notes/07-comparators-intro-alus]] — bitwise AND/OR gate arrays are used directly as two of the ALU's parallel candidate operations.
- [[6.004-computation-structures/notes/08-bistability-sr-latch]] — NOR and NAND gates are cross-coupled (output-to-input feedback) to build the first memory element; reuses the fact that a static CMOS gate's output is always actively driven to a well-defined rail.
