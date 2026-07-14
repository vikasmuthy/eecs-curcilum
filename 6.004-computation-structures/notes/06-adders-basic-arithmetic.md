---
title: "Adders (Half/Full Adder, Ripple-Carry) and Basic Arithmetic"
course: "6.004"
topic_number: 06
prerequisites: ["6.004 Topic 05 — Multiplexers, Decoders, Encoders", "6.004 Topic 04 — Static CMOS Gates", "6.004 Topic 02 — Canonical Forms & Simplification"]
status: done
---

# Adders (Half/Full Adder, Ripple-Carry) and Basic Arithmetic

## Why this matters

Every combinational block built so far (Topics 03–05) routes or selects bits; none of them *compute* with the bits as numbers. Arithmetic is the first place this course treats a group of wires not as $n$ independent bits but as a single binary-encoded number, and addition is the base case every other arithmetic operation (subtraction, multiplication, the ALU of Topic 07) is built from. This note derives a 1-bit adder from the truth table of elementary-school addition-with-carry, then shows how chaining $n$ of them (ripple-carry) scales the same idea to $n$-bit numbers — the exact same "chain small identical blocks" pattern that will reappear structurally once the datapath topics (25 onward) chain register-file cells and ALU bit-slices.

## Builds on

- [[6.004-computation-structures/notes/02-canonical-forms-simplification]] — canonical SOP/K-map minimization, used below to derive minimal-literal expressions for a full adder's sum and carry-out from its truth table.
- [[6.004-computation-structures/notes/04-static-cmos-gates]] — NAND/NOR/complex gates; every adder circuit below is realized as ordinary combinational gates from this note.
- [[6.004-computation-structures/notes/05-multiplexers-decoders-encoders]] — no direct reuse of MUX/decoder circuitry here, but this note continues that note's theme of building a reusable, parameterized building block ("one 1-bit adder, replicated $n$ times") rather than a fixed one-off circuit.

## Core definitions

- **Binary place-value number** — an $n$-bit unsigned integer $A = A_{n-1}A_{n-2}\cdots A_1A_0$ (each $A_i \in \{0,1\}$) represents the value $\sum_{i=0}^{n-1} A_i\cdot 2^i$; $A_0$ is the **least significant bit (LSB)**, $A_{n-1}$ the **most significant bit (MSB)**.
- **Half adder** — a combinational circuit with two 1-bit inputs $A, B$ and two outputs, **sum** $S$ and **carry-out** $C_{out}$, computing $A+B$ as a 2-bit result ($C_{out}$ is the $2^1$ place, $S$ is the $2^0$ place); "half" because it has no provision for an incoming carry from a lower bit position.
- **Full adder** — a combinational circuit with three 1-bit inputs, $A$, $B$, and **carry-in** $C_{in}$ (a carry arriving from the next-less-significant bit position), and two outputs, $S$ and $C_{out}$, computing $A+B+C_{in}$ as a 2-bit result. This is the actual reusable building block, since every bit position except the very least significant one has an incoming carry to account for.
- **Ripple-carry adder** — an $n$-bit adder built from $n$ full adders, bit position $i$'s $C_{out}$ wired directly to bit position $i+1$'s $C_{in}$, with bit position $0$'s $C_{in}$ tied to $0$ (since there is no carry into the least significant bit); the name reflects that a carry generated at a low bit position must propagate ("ripple") through every higher bit position's full adder before the final result is valid.
- **Overflow (unsigned)** — for an $n$-bit ripple-carry adder, the final full adder's $C_{out}$ (i.e. $C_{out}$ out of bit position $n-1$) signals that the true sum exceeds $2^n-1$, the largest value representable in $n$ unsigned bits; this bit is often exposed as an extra output, $C_n$.
- **Propagation delay of a ripple-carry chain** — informally introduced here (full timing analysis is deferred to Topic 22): the worst-case time for a ripple-carry adder's output to become valid, dominated by the carry signal propagating sequentially through all $n$ full adders in the worst case (e.g. adding $\underbrace{11\cdots1}_{n}$ to $1$, where the carry ripples all the way from bit $0$ to bit $n-1$).

## Intuition

**A half adder is just elementary-school addition of two single digits, in base 2.** Adding two 1-bit numbers can produce a result up to $1+1=2=10_2$ — a 2-bit result. The low bit of that result is the sum bit; the high bit is the carry — exactly what a grade-school addition table for single digits (extended to base 2, where the only "carry" case is $1+1$) would show.

**A full adder is the same idea, but modeling what actually happens at every column of a multi-digit addition.** When you add two multi-digit numbers by hand, every column (except the very first) has *three* things to add: the two digits in that column, plus whatever carried in from the column to the right. A full adder is precisely "the circuit for one column of hand addition," and that's why it — not the half adder — is the block that gets replicated to build an $n$-bit adder: only the very lowest column (bit $0$) genuinely has just two inputs; every other column needs the third, carry-in input.

**Ripple-carry is "wire the columns together, in order, the same way you'd process them by hand."** When adding by hand, you work right-to-left, carrying each column's overflow into the next. A ripple-carry adder does exactly this in hardware: chain $n$ full adders, feed each one's carry-out directly into the next one's carry-in, and tie the very first (bit $0$) carry-in to $0$ (no carry comes from "column $-1$"). The tradeoff this exposes, and the reason the name emphasizes "ripple": bit position $n-1$'s output isn't valid until the carry has had time to physically propagate through all $n-1$ full adders before it, which is slow for large $n$ — a limitation flagged here and revisited once propagation delay is formally covered (Topic 22).

## Derivation / formalism

### 1. Half adder: truth table to minimal expressions

Half adder inputs $A,B$; outputs $S,C_{out}$, defined by the requirement that the 2-bit number $C_{out}S$ equals the true sum $A+B$ (as an integer, $0,1,$ or $2$):

| $A$ | $B$ | $A+B$ (integer) | $C_{out}$ | $S$ |
|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 0 |
| 0 | 1 | 1 | 0 | 1 |
| 1 | 0 | 1 | 0 | 1 |
| 1 | 1 | 2 | 1 | 0 |

Reading $S$'s column as a canonical SOP (Topic 02): $S = \bar A B + A\bar B$. This is exactly the **XOR** pattern encountered in Topic 04's Self-check Question 8 (there written $A\bar B+\bar AB$, the same expression with terms reordered) — it does not simplify further on a K-map (the two $1$-cells are diagonal, non-adjacent, as already noted in that Topic 04 answer). So:
$$S = A \oplus B \quad \text{(notation: } \oplus \text{ denotes XOR, defined as } X\oplus Y = X\bar Y+\bar XY\text{)}$$
Reading $C_{out}$'s column: only row $A{=}1,B{=}1$ is $1$, giving directly (no simplification possible for a single-minterm function of 2 variables other than leaving it as the minterm itself):
$$C_{out} = AB$$

### 2. Full adder: truth table to minimal expressions via K-map

Full adder inputs $A,B,C_{in}$; outputs $S, C_{out}$, defined by requiring the 2-bit number $C_{out}S$ to equal the integer $A+B+C_{in}$ (ranging $0$ to $3$):

| $A$ | $B$ | $C_{in}$ | $A+B+C_{in}$ | $C_{out}$ | $S$ |
|---|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 1 | 0 | 1 |
| 0 | 1 | 0 | 1 | 0 | 1 |
| 0 | 1 | 1 | 2 | 1 | 0 |
| 1 | 0 | 0 | 1 | 0 | 1 |
| 1 | 0 | 1 | 2 | 1 | 0 |
| 1 | 1 | 0 | 2 | 1 | 0 |
| 1 | 1 | 1 | 3 | 1 | 1 |

**$S$ via K-map** (rows $A$, columns $BC_{in}$ in Gray order $00,01,11,10$; from Topic 02's method):

| | $BC_{in}{=}00$ | $BC_{in}{=}01$ | $BC_{in}{=}11$ | $BC_{in}{=}10$ |
|---|---|---|---|---|
| $A{=}0$ | 0 | 1 | 0 | 1 |
| $A{=}1$ | 1 | 0 | 1 | 0 |

Every $1$-cell is isolated (no two adjacent $1$-cells anywhere, including wraparound — check: $(A,BC)=(0,01)$ and $(0,10)$ are not adjacent since $BC{=}01$ and $BC{=}10$ differ in both bits; similarly for every other pair). So no grouping is possible beyond single cells — $S$ has no simpler SOP than its canonical form:
$$S = \bar A\bar BC_{in} + \bar AB\bar C_{in} + A\bar B\bar C_{in} + ABC_{in}$$
This can be written compactly using XOR (verified in Worked Example 1 below by expanding back to the truth table):
$$S = A \oplus B \oplus C_{in}$$

**$C_{out}$ via K-map:**

| | $BC_{in}{=}00$ | $BC_{in}{=}01$ | $BC_{in}{=}11$ | $BC_{in}{=}10$ |
|---|---|---|---|---|
| $A{=}0$ | 0 | 0 | 1 | 0 |
| $A{=}1$ | 0 | 1 | 1 | 1 |

Three prime implicants, each a pair: cells $(A,BC_{in})=(0,11)$ and $(1,11)$ (same column, both rows) share $BC_{in}{=}11$, differing only in $A$ — group gives $BC_{in}$; cells $(1,01)$ and $(1,11)$ (same row $A{=}1$, adjacent columns $01,11$) share $A{=}1, C_{in}{=}1$, differing in $B$ — group gives $AC_{in}$; cells $(1,11)$ and $(1,10)$ (same row, adjacent columns $11,10$) share $A{=}1,B{=}1$, differing in $C_{in}$ — group gives $AB$. All three pairs are needed since cell $(0,11)$ (minterm for $A{=}0,B{=}1,C_{in}{=}1$) is only covered by the $BC_{in}$ group, cell $(1,01)$ only by $AC_{in}$, and cell $(1,10)$ only by $AB$ — each of these three is therefore an essential prime implicant (Topic 02's terminology). Together:
$$C_{out} = AB + BC_{in} + AC_{in}$$
This is the **majority function**: $C_{out}=1$ exactly when at least two of the three inputs are $1$, matching the truth table directly (verified in Worked Example 1).

### 3. Building the full adder from two half adders (a common, gate-count-motivated decomposition)

Rather than realizing $S$ and $C_{out}$ directly as 3-input complex gates (Topic 04), a full adder is conventionally built from two half adders plus one OR gate, because it reuses an already-derived block. Let HA1 take inputs $A,B$, producing sum $S_1 = A\oplus B$ and carry $C_1=AB$ (Section 1). Feed $S_1$ and $C_{in}$ into a second half adder HA2, producing sum $S_2 = S_1\oplus C_{in}$ and carry $C_2 = S_1\cdot C_{in}$.

**Claim: $S_2 = S$ (the full adder's sum, Section 2).** $S_2 = S_1\oplus C_{in} = (A\oplus B)\oplus C_{in}$. XOR is associative (a fact usable here since $\oplus$ is a standard Boolean operation with the standard properties; verified directly by perfect induction over all 8 rows in Worked Example 1 below rather than assumed), so $(A\oplus B)\oplus C_{in} = A\oplus B\oplus C_{in} = S$ from Section 2. Matches.

**Claim: $C_1 + C_2 = C_{out}$ (the full adder's carry, Section 2).** $C_1+C_2 = AB + S_1 C_{in} = AB + (A\oplus B)C_{in} = AB+(A\bar B+\bar AB)C_{in} = AB+A\bar BC_{in}+\bar ABC_{in}$. Compare against Section 2's $C_{out}=AB+BC_{in}+AC_{in}$: expand $BC_{in} = BC_{in}(A+\bar A) = ABC_{in}+\bar ABC_{in}$ (distributive, then complement/identity, Topic 01) and $AC_{in}=AC_{in}(B+\bar B)=ABC_{in}+A\bar BC_{in}$, so $C_{out} = AB + ABC_{in}+\bar ABC_{in}+ABC_{in}+A\bar BC_{in}$. This has two copies of $ABC_{in}$ (from expanding $BC_{in}$ and $AC_{in}$ separately); the idempotent law ($X+X=X$, Topic 01) merges them into one, giving $C_{out} = AB+ABC_{in}+\bar ABC_{in}+A\bar BC_{in}$. Now the remaining $AB+ABC_{in}$ pair simplifies via $AB+ABC_{in} = AB(1+C_{in}) = AB\cdot1 = AB$ (distributive, then dominance/null law $1+C_{in}=1$, then identity, Topic 01), collapsing the expression to $C_{out}=AB+\bar ABC_{in}+A\bar BC_{in}$. This matches $C_1+C_2 = AB+A\bar BC_{in}+\bar ABC_{in}$ term-for-term. So the two-half-adder-plus-OR construction is algebraically identical to the direct K-map result. $\blacksquare$

### 4. Ripple-carry $n$-bit adder

Given $n$ full adders $FA_0, \dots, FA_{n-1}$, wire: $FA_i$'s inputs are $A_i, B_i$ (bit $i$ of each operand) and $C_{in}^{(i)}$; $FA_i$'s $C_{out}$ output is connected directly to $FA_{i+1}$'s $C_{in}^{(i+1)}$ input, for every $i=0,\dots,n-2$; and $C_{in}^{(0)} = 0$ (tied to ground — no carry enters below bit $0$). The final output is the $n$-bit sum $S_{n-1}\cdots S_0$ plus an overflow bit $C_n = FA_{n-1}$'s $C_{out}$.

**Correctness, by induction on bit position.** *Base case ($i=0$):* $FA_0$ computes $A_0+B_0+0 = A_0+B_0$, correctly producing the true sum's bit-0 place and any carry out of bit 0 — matching ordinary place-value addition with no incoming carry. *Inductive step:* assume $FA_0,\dots,FA_{i-1}$ have correctly produced $S_0,\dots,S_{i-1}$ and a correct carry $C_{in}^{(i)}$ out of bit position $i-1$ (i.e., $C_{in}^{(i)}$ equals the actual carry that place-value addition of $A,B$ would produce out of column $i-1$). Then $FA_i$, by Section 2's definition, computes $A_i+B_i+C_{in}^{(i)}$ exactly — which is precisely what column $i$ of a hand-addition would need to process (the two digits in that column plus the incoming carry) — producing the correct $S_i$ and a correct carry $C_{out}^{(i)}=C_{in}^{(i+1)}$ for the next column. By induction, every $S_i$ and the final $C_n$ are correct for $i=0,\dots,n-1$. $\blacksquare$

## Worked examples

### Example 1 — Full adder: verifying the XOR sum and majority carry-out by perfect induction

Verify $S = A\oplus B\oplus C_{in}$ and $C_{out}=AB+BC_{in}+AC_{in}$ against every row of Section 2's truth table:

| $A$ | $B$ | $C_{in}$ | $A\oplus B$ | $(A\oplus B)\oplus C_{in}$ | $AB{+}BC_{in}{+}AC_{in}$ | Matches table? |
|---|---|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 0 | 0 | $S{=}0,C_{out}{=}0$ ✓ |
| 0 | 0 | 1 | 0 | 1 | 0 | $S{=}1,C_{out}{=}0$ ✓ |
| 0 | 1 | 0 | 1 | 1 | 0 | $S{=}1,C_{out}{=}0$ ✓ |
| 0 | 1 | 1 | 1 | 0 | 1 | $S{=}0,C_{out}{=}1$ ✓ |
| 1 | 0 | 0 | 1 | 1 | 0 | $S{=}1,C_{out}{=}0$ ✓ |
| 1 | 0 | 1 | 1 | 0 | 1 | $S{=}0,C_{out}{=}1$ ✓ |
| 1 | 1 | 0 | 0 | 0 | 1 | $S{=}0,C_{out}{=}1$ ✓ |
| 1 | 1 | 1 | 0 | 1 | 1 | $S{=}1,C_{out}{=}1$ ✓ |

All 8 rows match Section 2's truth table exactly, confirming both compact formulas and (in the $C_{out}$ column) that $C_{out}$ is $1$ precisely when at least two of $A,B,C_{in}$ are $1$ (the majority function, as claimed).

### Example 2 — 4-bit ripple-carry addition, full numeric trace

Add $A=0111_2$ ($7_{10}$) and $B=0001_2$ ($1_{10}$) using a 4-bit ripple-carry adder ($FA_0$ processes the LSB). This example is chosen specifically to exhibit a carry rippling through multiple stages.

| Bit $i$ | $A_i$ | $B_i$ | $C_{in}^{(i)}$ | $S_i = A_i\oplus B_i\oplus C_{in}^{(i)}$ | $C_{out}^{(i)} = A_iB_i+B_iC_{in}^{(i)}+A_iC_{in}^{(i)}$ |
|---|---|---|---|---|---|
| 0 | 1 | 1 | 0 | $1\oplus1\oplus0=0$ | $1{\cdot}1+1{\cdot}0+1{\cdot}0=1$ |
| 1 | 1 | 0 | 1 | $1\oplus0\oplus1=0$ | $1{\cdot}0+0{\cdot}1+1{\cdot}1=1$ |
| 2 | 1 | 0 | 1 | $1\oplus0\oplus1=0$ | $1{\cdot}0+0{\cdot}1+1{\cdot}1=1$ |
| 3 | 0 | 0 | 1 | $0\oplus0\oplus1=1$ | $0{\cdot}0+0{\cdot}1+0{\cdot}1=0$ |

Result: $S_3S_2S_1S_0 = 1000_2$, with final carry-out $C_4=C_{out}^{(3)}=0$. As an integer: $1000_2 = 8_{10}$, matching $7+1=8$ exactly. Note the carry generated at bit $0$ ($C_{out}^{(0)}=1$) rippled all the way through bits $1$ and $2$ before finally being absorbed (produced $S_3=1$, $C_{out}^{(3)}=0$) at bit $3$ — a direct illustration of the "ripple" the adder's name refers to, and of why this input pair is a worst-case-ish (though not the absolute worst case) input for propagation delay.

## Common pitfalls

- **Using a half adder for every bit position of a multi-bit adder.** Only bit position $0$ has no incoming carry; every other position needs a full adder (3 inputs) to correctly account for the carry rippling in from the position below. Using half adders throughout silently drops every carry after bit $0$, corrupting the result for any addition that produces an intermediate carry.
- **Forgetting to tie $C_{in}^{(0)}$ to $0$.** The very first full adder in a ripple-carry chain still has a $C_{in}$ input; leaving it floating (rather than explicitly tied to logic $0$) produces an undefined or incorrect sum, since Section 4's correctness induction has its base case built on $C_{in}^{(0)}=0$ specifically.
- **Confusing the carry-out of the *whole* adder ($C_n$, out of the top bit) with an internal ripple carry ($C_{out}^{(i)}$ for $i<n-1$).** Only $C_n$ signals unsigned overflow (the true sum exceeded what $n$ bits can hold); the internal carries are just intermediate wiring between adjacent full adders, not overflow indicators on their own.
- **Assuming ripple-carry addition is "instant."** Each full adder's output can only be correct once its $C_{in}$ is valid, and $C_{in}$ for bit $i$ comes from bit $i-1$'s computation — so the whole adder's worst-case delay grows with the chain length $n$ (formally: propagation delay, deferred to Topic 22). This is a real hardware cost, not just an implementation detail, and it's why faster (non-ripple) adder architectures exist in real designs, even though they're out of scope for this note.
- **Misapplying XOR's algebra by treating $\oplus$ as if it had the same simplification rules as AND/OR.** $A\oplus B \ne A\cdot B$ and $A\oplus B \ne A+B$ in general (verify e.g. at $A{=}B{=}1$: $A\oplus B=0$ but $A+B=1$ and $AB=1$) — XOR is its own operation with its own truth table ($\bar AB+A\bar B$), only coinciding with OR when at most one input is $1$.
- **Mixing up the majority-function form of $C_{out}$ with a "any two or more" *count* circuit for more than 3 inputs.** The full adder's $C_{out}=AB+BC_{in}+AC_{in}$ formula is specific to exactly 3 inputs; generalizing "majority of $k$ things" to $k>3$ needs a different, larger circuit, not a direct extension of this formula.

## Self-check

### Questions

1. (Easy) Compute the half adder's $S$ and $C_{out}$ for $A=1,B=0$.
2. (Easy) A full adder has inputs $A=0,B=1,C_{in}=1$. Using the majority-function description, what is $C_{out}$ without consulting the truth table numerically — just by counting how many inputs are 1?
3. (Medium) Verify, via perfect induction over $A,B$ (2 rows suffice at fixed $C_{in}=0$, extend to all 8 if needed), that setting $C_{in}=0$ in the full adder's formulas for $S$ and $C_{out}$ reduces them exactly to the half adder's $S$ and $C_{out}$ from Section 1.
4. (Medium) A 4-bit ripple-carry adder computes $A+B$ where $A=1111_2$ and $B=0001_2$. Trace every bit position's $S_i$ and $C_{out}^{(i)}$ (as in Worked Example 2), and state the final 4-bit sum plus overflow bit $C_4$.
5. (Medium) Explain, using Section 4's inductive correctness argument, why a ripple-carry adder would compute the *wrong* answer at bit position 2 if bit position 1's full adder were (by a wiring bug) missing its $C_{in}$ connection entirely (tied to 0 regardless of the true incoming carry).
6. (Hard) Using the two-half-adder decomposition of Section 3, draw (in words) the complete gate-level structure of one full adder, counting the total number of 2-input gates used (each half adder uses one XOR-equivalent circuit and one AND gate; recall XOR itself needs multiple gates if built from Topic 04's NAND/NOR primitives — you may treat XOR as a single primitive gate for this count, but state that assumption).
7. (Hard) An 8-bit ripple-carry adder computes $A+B$ for $A=11111111_2$ ($255_{10}$) and $B=00000001_2$ ($1_{10}$). Without tracing every bit, predict (a) the final 8-bit sum, (b) whether $C_8$ (overflow) is asserted, and (c) explain in one sentence why this input pair produces the worst-case carry-propagation delay for an 8-bit ripple-carry adder.
8. (Hard) Prove algebraically, using only the distributive, complement, and identity laws (6.004 Topic 01) — not perfect induction — that $C_1+C_2$ from Section 3's two-half-adder construction equals $AB+BC_{in}+AC_{in}$, filling in every step (this is the same claim proved in Section 3, but redo it showing the distributive expansions explicitly rather than citing the result).

### Answers

1. $S = A\oplus B = 1\oplus 0 = 1$. $C_{out}=AB=1\cdot0=0$.
2. Two of the three inputs ($B=1,C_{in}=1$) are $1$ — that's a majority (2 out of 3) — so $C_{out}=1$.
3. Full-adder $S$ at $C_{in}=0$: $S=A\oplus B\oplus 0 = A\oplus B$ (since XOR with $0$ leaves a value unchanged — verify: $0\oplus0=0,0\oplus1=1,1\oplus0=1,1\oplus1=0$, i.e. $X\oplus0=X$ for both values of $X$) — matches half adder's $S=A\oplus B$ (Section 1). Full-adder $C_{out}$ at $C_{in}=0$: $C_{out}=AB+B(0)+A(0) = AB+0+0=AB$ (null law, Topic 01) — matches half adder's $C_{out}=AB$ (Section 1). Both reduce exactly, confirming the full adder generalizes the half adder (setting $C_{in}=0$ recovers it).
4. 

   | $i$ | $A_i$ | $B_i$ | $C_{in}^{(i)}$ | $S_i$ | $C_{out}^{(i)}$ |
   |---|---|---|---|---|---|
   | 0 | 1 | 1 | 0 | 0 | 1 |
   | 1 | 1 | 0 | 1 | 0 | 1 |
   | 2 | 1 | 0 | 1 | 0 | 1 |
   | 3 | 1 | 0 | 1 | 0 | 1 |

   Final sum $S_3S_2S_1S_0=0000_2$, $C_4=1$. As a check: $1111_2+0001_2 = 15+1=16 = 10000_2$, a 5-bit result; the 4-bit adder's outputs $C_4S_3S_2S_1S_0 = 10000_2$ correctly represent this — $C_4=1$ correctly signals the unsigned overflow beyond 4 bits.
5. Section 4's inductive correctness proof requires bit $i$'s full adder to receive the *actual* carry produced by bit $i-1$ as its $C_{in}$ — that's the inductive hypothesis being carried forward at each step. If bit $1$'s $C_{in}$ is wired to a fixed $0$ instead of bit $0$'s real $C_{out}^{(0)}$, the inductive hypothesis is violated starting at $i=1$: bit $1$ computes $A_1+B_1+0$ instead of $A_1+B_1+C_{out}^{(0)}$, which is wrong whenever $C_{out}^{(0)}$ was actually $1$. Since bit $2$'s correctness in the induction depends on bit $1$ having produced a *correct* carry, and bit $1$'s carry is now potentially wrong, bit $2$ (and every bit above it) inherits the corrupted carry and can also be wrong — a single broken carry link corrupts every subsequent bit position, not just the one where the bug is.
6. HA1: computes $S_1=A\oplus B$ (1 XOR gate, treated as primitive) and $C_1=AB$ (1 AND gate) — 2 gates. HA2: takes $S_1, C_{in}$, computes $S_2=S_1\oplus C_{in}$ (1 more XOR gate) and $C_2=S_1\cdot C_{in}$ (1 more AND gate) — 2 gates. Final OR gate combines $C_1+C_2=C_{out}$ — 1 gate. Total: 2(XOR)+2(AND)+1(OR) $=5$ two-input gates, under the stated assumption that XOR is available as a single primitive gate (if instead XOR had to be built from NAND/NOR per Topic 04, each XOR would expand into several more gates, increasing this count — flagged here as the assumption's limit).
7. (a) Sum $=00000000_2$. (b) $C_8=1$ (asserted). (c) This input pair forces the carry generated at bit $0$ (from $1+1=10_2$) to propagate through every single one of the remaining 7 full adders before the final result is valid, because every higher-order bit of $A$ is also $1$ (so each stage's carry-in of $1$ combined with $A_i{=}1,B_i{=}0$ again produces a carry-out of $1$, re-triggering the same worst case at the next stage) — this is the longest possible carry chain for an 8-bit ripple-carry adder, hence the worst-case propagation delay case.
8. $C_1+C_2 = AB + S_1C_{in}$ where $S_1=A\bar B+\bar AB$ (Section 1's half-adder sum, expanded). Substitute: $C_1+C_2 = AB+(A\bar B+\bar AB)C_{in}$. Distribute (AND over the parenthesized OR, Topic 01's distributive law): $=AB+A\bar BC_{in}+\bar ABC_{in}$. Now expand the target $AB+BC_{in}+AC_{in}$ using the identity $X = X(Y+\bar Y)=XY+X\bar Y$ (distributive + complement + identity, Topic 01) on the two terms not already matching: $BC_{in}=BC_{in}(A+\bar A)=ABC_{in}+\bar ABC_{in}$ and $AC_{in}=AC_{in}(B+\bar B)=ABC_{in}+A\bar BC_{in}$. So $AB+BC_{in}+AC_{in} = AB+ABC_{in}+\bar ABC_{in}+ABC_{in}+A\bar BC_{in}$. The term $ABC_{in}$ appears twice; by the idempotent law ($X+X=X$, Topic 01), $ABC_{in}+ABC_{in}=ABC_{in}$, collapsing the expression to $AB+ABC_{in}+\bar ABC_{in}+A\bar BC_{in}$. Now note $AB+ABC_{in} = AB(1+C_{in}) = AB\cdot1 = AB$ by the dominance/null law ($1+C_{in}=1$) then identity — so this collapses further to $AB+\bar ABC_{in}+A\bar BC_{in}$, exactly matching $C_1+C_2=AB+A\bar BC_{in}+\bar ABC_{in}$ derived above (term order differs only cosmetically). Both sides equal $AB+A\bar BC_{in}+\bar ABC_{in}$, proving $C_1+C_2=AB+BC_{in}+AC_{in}$ algebraically. $\blacksquare$

## Summary / cheat sheet

**Half adder** ($A,B\to S,C_{out}$): $S=A\oplus B$, $C_{out}=AB$.

**Full adder** ($A,B,C_{in}\to S,C_{out}$): $S=A\oplus B\oplus C_{in}$, $C_{out}=AB+BC_{in}+AC_{in}$ (majority function). Built from 2 half adders + 1 OR gate.

**Ripple-carry $n$-bit adder:** chain $n$ full adders, $C_{out}^{(i)}\to C_{in}^{(i+1)}$, $C_{in}^{(0)}=0$. Correct by induction on bit position. Overflow (unsigned) $\iff C_n=1$ (the top carry-out). Worst-case delay grows with $n$ (formal treatment: Topic 22).

## Used later in
- [[6.004-computation-structures/notes/07-comparators-intro-alus]] — the adder is reused verbatim (unmodified hardware) to build a subtractor via two's complement negation, and the adder's final carry-out $C_n$ is reinterpreted as the borrow/no-borrow signal for unsigned comparison.
- [[6.004-computation-structures/notes/14-propagation-delay-setup-hold-time]] — the two-half-adder full-adder decomposition's carry-chain path is used directly to quantify the ripple-carry adder's propagation and contamination delay, formalizing this note's informally-flagged "delay grows with $n$" claim.
