---
title: "FSM Design Procedure (State Encoding, Next-State/Output Synthesis)"
course: "6.004"
topic_number: 12
prerequisites: ["6.004 Topic 11 — Moore and Mealy Machines", "6.004 Topic 02 — Canonical Forms & Simplification"]
status: done
---

# FSM Design Procedure (State Encoding, Next-State/Output Synthesis)

## Why this matters

Topic 11 gave a precise language for *specifying* sequential behavior — state diagrams and tables, with states as abstract, named elements ("$A$," "$B$," ...) — but named states are not something a circuit can be built from directly; a real circuit only has wires carrying bits. This note supplies the missing translation step: assign each abstract state a concrete bit pattern (a **state encoding**), which turns the symbolic $\delta$ and $\lambda$ functions of Topic 11 into ordinary truth tables over Boolean variables — at which point Topic 02's entire canonical-form and K-map machinery applies directly, producing minimized **next-state equations** and **output equations**. This is the last purely-logical step before Topic 13 wires those equations to actual flip-flops and gates; skip it and there is no principled path from "here is my state diagram" to "here is the Boolean expression my circuit needs to compute."

## Builds on

- [[6.004-computation-structures/notes/11-moore-mealy-machines]] — the FSM 5-tuple $(S,I,O,\delta,\lambda)$, state diagrams/tables, and the Moore/Mealy distinction; this note's entire procedure operates on exactly this input, turning it into Boolean equations.
- [[6.004-computation-structures/notes/02-canonical-forms-simplification]] — canonical SOP, K-map minimization, and don't-cares; reused directly and without modification, once the state table has been converted into an ordinary multi-output truth table over Boolean variables.

## Core definitions

- **State variable** — one of the $k$ Boolean signals $Q_{k-1},\dots,Q_0$ used to represent which state an FSM currently occupies; the vector $(Q_{k-1},\dots,Q_0)$ at any moment is the **encoded present state**, and $(Q_{k-1}^+,\dots,Q_0^+)$ is the **encoded next state**.
- **State encoding (state assignment)** — an injective function $\text{code}: S \to \{0,1\}^k$ assigning each abstract state a distinct $k$-bit pattern (a **code word**), where $k$ is chosen large enough that $2^k \ge |S|$ (derived precisely in Section 1 below); "injective" here means distinct states always get distinct code words, since the encoding must be reversible (no two different states may share one code word, or the circuit could never distinguish them).
- **Unused code word** — a $k$-bit pattern not assigned to any state by the encoding, arising whenever $2^k > |S|$ (i.e., $|S|$ is not itself a power of $2$); treated as a **don't-care** row in every next-state/output truth table, per Topic 02's don't-care concept, since the FSM (by design/assumption) is never actually in an unused code word.
- **Encoded transition table** — the state transition table of Topic 11, rewritten with every symbolic state name replaced by its code word, and with the present-state code split into its individual bit columns $Q_{k-1},\dots,Q_0$ (alongside the existing input columns) — an ordinary truth table over $k+|I\text{'s encoding}|$ Boolean input variables, with $k$ next-state-bit output columns (and, for Mealy, output-bit columns as well).
- **Next-state equations** — one Boolean expression per state variable, $Q_i^+ = f_i(Q_{k-1},\dots,Q_0,\ x_1,\dots,x_m)$ (where $x_1,\dots,x_m$ are the encoded input bits), each derived by treating that state variable's column of the encoded transition table as an ordinary Topic 02 truth table and minimizing it.
- **Output equations** — analogously, one Boolean expression per output bit, derived the same way from the encoded table's output column(s); for a Moore machine these depend only on $Q_{k-1},\dots,Q_0$ (no input variables, matching $\lambda:S\to O$'s signature from Topic 11), while for a Mealy machine they may depend on both state and input bits (matching $\lambda:S\times I\to O$).

## Intuition

**Encoding turns a symbolic table into an ordinary multi-output truth table, and nothing else changes.** Topic 02 already showed how to go from a truth table to a minimized Boolean expression — the only thing Topic 11's abstract state names were missing was *being* bits in the first place. Once every state has a fixed bit pattern, the state table's "row: (current state, input) $\to$ (next state, output)" structure is *exactly* the shape of a truth table with several output columns (one per next-state bit, one per output bit) — nothing conceptually new is required, only bookkeeping to expand symbolic names into bit columns.

**Unused code words are a free, natural instance of "don't-care," not a new idea.** If $|S|$ isn't a power of $2$ (e.g. 3 states needing 2 bits, leaving one code word unused, as in Worked Example 2), the FSM is designed to never actually enter that leftover code word — Topic 02 already defined a don't-care as exactly "an input row the specification places no requirement on," and an unused state code is precisely that: the design promises the circuit never sits there, so the corresponding truth-table row can be assigned whatever value makes the resulting equation smallest, with zero risk to correctness (Topic 02, Derivation, don't-care intuition, reused verbatim here).

**The encoding you choose is a real design decision with real consequences, even though this note doesn't derive an optimal one.** Different assignments of bit patterns to the same abstract states generally produce differently-sized minimized equations — Worked Example 2 picks one reasonable encoding and gets clean, small equations, but a different encoding of the *same* 3-state machine could produce a messier result. Finding a *provably* smallest-possible encoding is a harder combinatorial problem this note does not solve; the procedure derived here works correctly for *any* valid encoding (any injective assignment), it just doesn't guarantee the smallest one.

## Derivation / formalism

### 1. The minimum number of state bits

Claim: encoding $|S|$ distinct states as distinct $k$-bit patterns requires $k \ge \lceil \log_2|S|\rceil$.

**Proof.** There are exactly $2^k$ distinct $k$-bit patterns (Topic 06's place-value counting argument: each of $k$ bit positions independently takes $2$ values). An injective encoding needs at least $|S|$ distinct patterns available, one per state, so $2^k \ge |S|$, i.e. $k \ge \log_2|S|$; since $k$ must be a whole number of bits, $k=\lceil\log_2|S|\rceil$ is the smallest value satisfying this. (This is the identical counting argument used in Topic 05 to size a decoder/encoder's bit width for $2^n$ distinguishable index values — the same "how many bits distinguish how many things" question, here applied to states instead of MUX/decoder indices.) $\blacksquare$

### 2. Building the encoded transition table, mechanically

Given a state diagram/table $(S,I,O,\delta,\lambda)$ (Topic 11) and a chosen encoding $\text{code}:S\to\{0,1\}^k$ (with $k$ from Section 1), construct the encoded transition table by: for every state $s\in S$ and input value $x$, look up the row $(s,x)\to(\delta(s,x),\lambda(s,x)\text{ if Mealy})$ from Topic 11's table, and rewrite it as $(\text{code}(s), x) \to (\text{code}(\delta(s,x)), \lambda(s,x)\text{ if Mealy})$. Any $k$-bit pattern not in the image of $\text{code}$ (an unused code word) contributes no row from this construction — those rows are filled in afterward as don't-cares (Section 3). This construction is purely mechanical substitution — no new information is introduced or lost, mirroring Topic 11 Section 1's diagram$\leftrightarrow$table equivalence, now extended one step further to encoded-table$\leftrightarrow$symbolic-table.

### 3. Deriving next-state and output equations via K-map minimization

Once the encoded transition table exists (Section 2), each output column — the $k$ next-state bits $Q_{k-1}^+,\dots,Q_0^+$, plus (for Mealy) the output bits — is, individually, an ordinary Boolean function of the present-state bits and input bits, with a well-defined truth table (one column extracted from the full encoded transition table). Apply Topic 02's canonical-SOP-then-K-map-minimization procedure to *each* column separately: plot that column as a K-map over the $Q_{k-1},\dots,Q_0,x_1,\dots,x_m$ variables, mark unused-code-word rows as don't-cares (Core definitions), find prime implicants, identify essential ones, and read off the minimal SOP — exactly Topic 02 Derivation Section 5's procedure, applied once per output column, with no modification. The resulting minimal expressions are the next-state equations and output equations.

### 4. Encoding choice affects equation size, but not correctness

Different valid encodings (different injective functions $\text{code}$) of the *same* abstract FSM produce, in general, different next-state/output equations after Section 3's minimization — because the K-map's cell layout (which rows are adjacent to which) depends entirely on which bit pattern was assigned to which state, and adjacency is exactly what enables simplification (Topic 02, Derivation Section 4). However, *correctness* is unaffected by which valid encoding is chosen: Section 2's construction is a lossless, mechanical translation regardless of which specific code words are used (any injective assignment satisfies Section 1's requirement), so the resulting circuit — whatever its next-state/output equations turn out to be — will always correctly reproduce the original FSM's behavior once decoded back to the abstract state names, only its transistor/gate count downstream (Topic 13) may differ from one encoding to another.

## Worked examples

### Example 1 — Encoding Topic 11 Example 1's 2-state Mealy machine

Topic 11 Example 1's "detect two consecutive 1s" Mealy machine has states $S=\{A,B\}$. By Section 1, $k=\lceil\log_2 2\rceil = 1$ bit suffices. Encode $\text{code}(A)=0$, $\text{code}(B)=1$ (state variable $Q$; no unused code words, since $2^1=2=|S|$ exactly).

**Encoded transition table** (from Topic 11's Mealy table, Section 2's substitution; $Z$ denotes the output bit throughout this note's worked examples):

| $Q$ | $x$ | $Q^+$ | $Z$ |
|---|---|---|---|
| 0 | 0 | 0 | 0 |
| 0 | 1 | 1 | 0 |
| 1 | 0 | 0 | 0 |
| 1 | 1 | 1 | 1 |

**Deriving $Q^+$ (Section 3):** the $1$-rows are $(Q,x)=(0,1)$ and $(1,1)$ — both have $x=1$, and $Q$ takes both values across them, so by Topic 02's pairing rule these merge into the single literal $x$: $Q^+ = x$ (no don't-cares needed; already minimal).

**Deriving $Z$ (Section 3):** the only $1$-row is $(Q,x)=(1,1)$ — a single minterm, giving $Z=Qx$ directly (2 literals; only one row is $1$, so no further merging is possible without a don't-care to pair with, and there are none here).

**Result:** $Q^+=x$, $Z=Qx$. This exactly reproduces Topic 11 Example 1's behavior: e.g. tracing input $1,1,0,1,1,1$ from $Q=0$: $Q^+=x$ gives the state sequence $0,1,1,0,1,1,1$ (present state each cycle, prepended with the initial $0$), and $Z=Qx$ evaluated each cycle (using that cycle's present $Q$ and current $x$) reproduces $0,1,0,0,1,1$ — matching Topic 11 Example 1's output trace exactly.

### Example 2 — Encoding Topic 11 Example 2's 3-state Moore machine, with a don't-care

Topic 11 Example 2's Moore machine has states $S=\{A_0,B_0,B_1\}$ ($|S|=3$). By Section 1, $k=\lceil\log_2 3\rceil = 2$ bits (since $2^1=2<3\le 4=2^2$), leaving $2^2-3=1$ unused code word. Encode $\text{code}(A_0)=00$, $\text{code}(B_0)=01$, $\text{code}(B_1)=11$ (state variables $Q_1,Q_0$), leaving $10$ unused.

**Encoded transition table** (from Topic 11 Example 2's Moore table, Section 2's substitution; unused-code rows marked don't-care per Core definitions):

| $Q_1$ | $Q_0$ | $x$ | $Q_1^+$ | $Q_0^+$ | ($Z$, for reference) |
|---|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 0 | 1 | 0 |
| 0 | 1 | 0 | 0 | 0 | 0 |
| 0 | 1 | 1 | 1 | 1 | 0 |
| 1 | 0 | 0 | $d$ | $d$ | $d$ |
| 1 | 0 | 1 | $d$ | $d$ | $d$ |
| 1 | 1 | 0 | 0 | 0 | 1 |
| 1 | 1 | 1 | 1 | 1 | 1 |

**Deriving $Q_1^+$ (Section 3):** required $1$-rows: $(Q_1Q_0x)=(011)$ and $(111)$; don't-cares at $(100),(101)$. The two required rows share $Q_0=1,x=1$ (differing only in $Q_1$), merging to $Q_0x$ (2 literals) by Topic 02's pairing rule. Checking whether the don't-cares permit further reduction to a single literal $Q_1$: that would require every $Q_1=1$ row ($100,101,110,111$) to be $1$ or don't-care, but row $110$ (state $B_1$, $x=0$) is a *required* $0$-row (not don't-care) — so $Q_1$ alone is invalid, and $Q_1^+=Q_0x$ is the minimal result.

**Deriving $Q_0^+$ (Section 3):** required $1$-rows: $(001),(011),(111)$; don't-cares at $(100),(101)$. Checking the single literal $x$: every row with $x=1$ is $(001),(011),(101),(111)$ — the required rows $(001,011,111)$ are all $1$, and $(101)$ is a don't-care (free to be $1$) — so *every* $x=1$ row is $1$-or-don't-care. Checking $x=0$ rows $(000,010,100,110)$: required values are $0,0,d,0$ — all consistent with $Q_0^+=0$ there. So the single literal $Q_0^+=x$ is valid and minimal, exploiting the don't-cares fully (a direct instance of Topic 02's "don't-cares let a group grow bigger" payoff).

**Deriving $Z$ (Moore output, depends only on $Q_1,Q_0$, Section 3):** $Z=1$ only at $Q_1Q_0=11$ ($B_1$); $Z=0$ at $00,01$; don't-care at $10$. Reading directly off the encoding: $Z=Q_1$ (the output bit simply *is* the high state bit, by the specific encoding chosen — no minimization algebra needed beyond noticing the pattern, though formally this is the K-map result: the required $1$-row and $0$-rows already separate cleanly along $Q_1$, with the don't-care at $Q_1{=}1,Q_0{=}0$ consistent with $Z=Q_1$ giving $1$ there, an acceptable free choice).

**Result:** $Q_1^+=Q_0x$, $Q_0^+=x$, $Z=Q_1$. Tracing input $1,1,0,1,1,1$ from $(Q_1,Q_0)=(0,0)$: cycle 1 ($Q_1Q_0=00,x=1$): $Q_1^+=0\cdot1=0$, $Q_0^+=1$ $\to(0,1)$; cycle 2 ($01,x=1$): $Q_1^+=1\cdot1=1$, $Q_0^+=1\to(1,1)$; cycle 3 ($11,x=0$): $Q_1^+=1\cdot0=0,Q_0^+=0\to(0,0)$; cycle 4 ($00,x=1$): $\to(0,1)$; cycle 5 ($01,x=1$)$\to(1,1)$; cycle 6 ($11,x=1$): $Q_1^+=1\cdot1=1,Q_0^+=1\to(1,1)$. Reading $Z=Q_1$ off the *present*-state bit each cycle (states $00,01,11,00,01,11$ present during cycles 1–6 respectively): $Z=0,0,1,0,0,1$ — matching Topic 11 Example 2's Moore output trace exactly.

## Common pitfalls

- **Forgetting to mark unused code words as don't-cares.** Treating an unused code word's next-state/output row as a *required* $0$ (rather than a don't-care) throws away exactly the simplification opportunity Example 2's $Q_0^+=x$ result depended on — the resulting equation would be needlessly larger, and in the worst case could even be a different (still-correct-if-the-code-is-truly-unreachable, but unnecessarily complex) expression.
- **Using a non-injective encoding (two states sharing one code word).** Section 1's requirement is not optional — if two abstract states mapped to the same bit pattern, the circuit would have no way to distinguish "which abstract state" it's actually representing, silently merging two potentially different behaviors into one; always verify the chosen $\text{code}$ function is one-to-one before proceeding.
- **Deriving next-state equations from the *symbolic* Topic 11 table directly, skipping the encoded table.** Topic 02's K-map/SOP machinery operates on Boolean variables, not named states — the encoding (Section 2) is not a cosmetic step, it's what makes the minimization procedure of Section 3 applicable at all; there is no way to "K-map" a table whose entries are state *names* rather than bit patterns.
- **Assuming the number of state bits $k$ must exactly equal $\log_2|S|$ even when that's not a whole number.** Section 1 derives $k=\lceil\log_2|S|\rceil$ — the ceiling, not the raw logarithm; 3 states need 2 bits (not "1.58 bits"), and the resulting one unused code word is a normal, expected byproduct handled via don't-cares (Example 2), not an error to avoid.
- **Assuming any two valid encodings of the same FSM must produce equally-simple equations.** Section 4 explicitly notes this is false — encoding choice is a real design decision with real consequences for equation (and downstream circuit) size, even though every valid encoding is equally *correct*. Don't treat "I found *an* encoding that works" as "I found *the* encoding that gives the smallest circuit."
- **Confusing a next-state equation's variables with an output equation's variables for a Moore machine.** A Moore output equation (Section 3) is a function of state bits *only*, never input bits (matching $\lambda:S\to O$'s signature from Topic 11) — accidentally including an input variable in a Moore output equation silently turns it into Mealy-style behavior, contradicting the original specification's timing.

## Self-check

### Questions

1. (Easy) An FSM has 5 abstract states. How many state bits $k$ are needed, per Section 1, and how many unused code words result?
2. (Easy) True or false: a Moore machine's output equation may depend on an input variable. Justify in one sentence, citing Topic 11.
3. (Medium) Using Example 1's encoded table, verify $Q^+=x$ and $Z=Qx$ reproduce the correct next-state/output for the specific row $Q=1,x=0$.
4. (Medium) Explain why Section 1's proof concludes $k=\lceil\log_2|S|\rceil$ rather than $k=\lfloor\log_2|S|\rfloor$ (the floor instead of the ceiling) — what would go wrong with the floor for $|S|=5$?
5. (Medium) In Example 2, why was the don't-care at code word $10$ essential to simplifying $Q_0^+$ all the way down to the single literal $x$? Identify specifically which row's required value would have blocked that simplification if $10$ were not a don't-care.
6. (Hard) Suppose Example 2's 3-state machine were instead encoded as $\text{code}(A_0)=00,\text{code}(B_0)=10,\text{code}(B_1)=11$ (leaving $01$ unused, a different encoding than the one used in Example 2). Rebuild the encoded transition table for $Q_1^+$ only, and determine whether $Q_1^+$ simplifies to a single literal under this encoding, contrasting with Example 2's 2-literal result — illustrating Section 4's claim that encoding choice affects equation size.
7. (Hard) A 4-state FSM needs $k=2$ bits with no unused code words (since $2^2=4$ exactly). Explain, using Section 1's proof, why this specific case can *never* benefit from don't-care-based simplification the way Example 2's 3-state machine did, regardless of which valid encoding is chosen.
8. (Hard) Prove that Section 2's encoded-transition-table construction is injective as a mapping from (symbolic row, encoding) pairs to encoded rows — i.e., that two different symbolic rows $(s,x)$ and $(s',x')$ (with $s\ne s'$ or $x\ne x'$) can never produce the identical encoded row $(\text{code}(s),x)=(\text{code}(s'),x')$ — using only the fact that $\text{code}$ is injective (Section 1's requirement).

### Answers

1. $k=\lceil\log_2 5\rceil = 3$ (since $2^2=4<5\le8=2^3$). Unused code words: $2^3-5=3$.
2. False. Per Topic 11's Core definitions, a Moore machine's output function has signature $\lambda:S\to O$ — output is a function of state alone, with no input argument at all; an output equation depending on an input variable would contradict this signature and effectively implement Mealy-style (same-cycle input-dependent) behavior instead.
3. Encoded table row $Q=1,x=0$: $Q^+=0$, $Z=0$ (from Example 1's table). Checking the derived equations: $Q^+=x=0$ — matches. $Z=Qx=1\cdot0=0$ — matches.
4. If $k=\lfloor\log_2|S|\rfloor$ were used for $|S|=5$: $\lfloor\log_2 5\rfloor = \lfloor 2.32\rfloor=2$, giving only $2^2=4$ available code words — fewer than the $5$ states needing distinct codes, violating Section 1's injectivity requirement (at least two states would be forced to share a code word). The ceiling guarantees $2^k\ge|S|$ always; the floor does not.
5. The row at code word $10$ (with either $x=0$ or $x=1$) is precisely one of the two rows with $x=1$ that would otherwise need to be checked ($100$ has $x=0$, so it's actually the $x{=}0$ half; the relevant don't-care for the $x=1$ simplification is specifically row $101$, i.e. $Q_1Q_0x=1,0,1$). Without that row being a don't-care, it would be a *required* value, and Example 2's derivation would need to check whether $Q_0^+=x$ correctly predicts that required value too — since $10$ is an unused state, no such required value exists in the actual specification, which is exactly why it's free to be treated as $1$ (matching $x=1$'s output there) with no risk of contradicting a real requirement, letting the "every $x=1$ row is 1-or-don't-care" argument go through cleanly.
6. New encoding: $\text{code}(A_0)=00,\text{code}(B_0)=10,\text{code}(B_1)=11$, unused $=01$. Rebuilding $Q_1^+$'s rows from Topic 11 Example 2's symbolic table ($A_0\xrightarrow{0}A_0,\ A_0\xrightarrow{1}B_0,\ B_0\xrightarrow{0}A_0,\ B_0\xrightarrow{1}B_1,\ B_1\xrightarrow{0}A_0,\ B_1\xrightarrow{1}B_1$): $(Q_1Q_0x)=(000)\to$next$=00\to Q_1^+=0$; $(001)\to$next$=10\to Q_1^+=1$; $(100)\to$next$=00\to Q_1^+=0$; $(101)\to$next$=11\to Q_1^+=1$; $(010),(011)$ unused, don't-care; $(110)\to$next$=00\to Q_1^+=0$; $(111)\to$next$=11\to Q_1^+=1$. Required $1$-rows: $(001),(101),(111)$; don't-cares $(010),(011)$. Checking single literal $x$: every $x=1$ row is $(001),(011),(101),(111)$ — required rows $(001,101,111)$ all $1$, and $(011)$ is don't-care — so *every* $x=1$ row is $1$-or-don't-care, and every $x=0$ row $(000,010,100,110)$ has required/don't-care values $0,d,0,0$, all consistent with $0$. So under this encoding, $Q_1^+=x$ — a *single* literal, simpler than Example 2's original $Q_1^+=Q_0x$ (2 literals) — a concrete demonstration that a different valid encoding of the identical abstract FSM produced a strictly simpler next-state equation, confirming Section 4's claim.
7. With $|S|=4$ and $k=2$, $2^k=4=|S|$ exactly (Section 1: $\lceil\log_2 4\rceil=2$, and $2^2=4$ has zero slack), so there are zero unused code words for *any* valid encoding — every one of the $2^k=4$ possible code words must be assigned to some real state (an injective function from a 4-element set onto a 4-element codomain is automatically surjective, i.e. bijective, leaving nothing unused). With no unused code words, there are no don't-care rows available in any next-state/output truth table built from this FSM, regardless of which specific bijection is chosen as the encoding — so the "shrink further using don't-cares" step of Topic 02's minimization procedure simply has nothing to work with here; minimization can still find groupings among the *required* rows (ordinary K-map simplification), just never with the extra don't-care-fueled boost Example 2 benefited from.
8. Suppose two symbolic rows $(s,x)$ and $(s',x')$ produce the identical encoded row, i.e. $\text{code}(s)=\text{code}(s')$ and $x=x'$ (the encoded row is the pair of the state's code word and the input value, so equality of encoded rows requires equality in both components). Since $\text{code}$ is injective (Section 1: distinct states must map to distinct code words), $\text{code}(s)=\text{code}(s')$ implies $s=s'$ (injectivity's contrapositive: if $s\ne s'$ then $\text{code}(s)\ne\text{code}(s')$, so equal code words force equal states). Combined with $x=x'$ (already assumed), this gives $s=s'$ and $x=x'$ — i.e., $(s,x)=(s',x')$, meaning the two "different" symbolic rows were not actually different after all. So no two genuinely distinct symbolic rows can produce the same encoded row, proving injectivity of the construction. $\blacksquare$

## Summary / cheat sheet

**State encoding:** injective $\text{code}:S\to\{0,1\}^k$, $k=\lceil\log_2|S|\rceil$ (Section 1). Unused code words (when $2^k>|S|$) become don't-cares.

**Procedure:** (1) choose $k$ and an encoding; (2) build the encoded transition table by mechanical substitution (Section 2); (3) treat each next-state bit and each output bit as its own Topic 02 truth-table column; minimize each via K-map, using unused-code-word rows as don't-cares (Section 3).

**Moore vs. Mealy equations:** Moore output equations are functions of state bits only; Mealy output equations may depend on both state and input bits (Topic 11's $\lambda$ signatures, carried through unchanged).

**Encoding is a real design choice:** different valid (injective) encodings of the same FSM generally yield different-sized minimized equations (Section 4, Self-check Q6) — correctness never depends on which encoding is chosen, but circuit size can.

## Used later in
- [[6.004-computation-structures/notes/13-implementing-fsms-flip-flops]] — the next-state and output equations derived here are wired, unmodified, directly into a physical circuit's next-state logic and output logic blocks.
