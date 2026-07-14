---
title: "Comparators and Intro ALUs"
course: "6.004"
topic_number: 07
prerequisites: ["6.004 Topic 06 — Adders (Half/Full Adder, Ripple-Carry) and Basic Arithmetic", "6.004 Topic 05 — Multiplexers, Decoders, Encoders", "6.004 Topic 04 — Static CMOS Gates"]
status: done
---

# Comparators and Intro ALUs

## Why this matters

Topic 06 built one arithmetic operation (addition) as a dedicated circuit. Real processors need many operations — add, subtract, AND, OR, and comparisons like "is $A$ less than $B$" — selectable at runtime by a control code, not wired as separate fixed circuits chosen at design time. This note does two things: first, it shows that subtraction and several comparisons are not new circuits at all but *reinterpretations* of the adder from Topic 06, via **two's complement** representation, so almost no new arithmetic hardware is needed; second, it combines the adder with the Topic 04 logic gates and the Topic 05 multiplexer into a single **arithmetic logic unit (ALU)** — one circuit that performs whichever operation a control code selects. This is the last purely combinational building block before the course turns to memory (Topics 08–10) and the ALU built here reappears as-is inside the single-cycle datapath (Topics 25–30).

## Builds on

- [[6.004-computation-structures/notes/06-adders-basic-arithmetic]] — the full adder and $n$-bit ripple-carry adder; this note reuses the adder unmodified and shows subtraction/comparison can be obtained from it via input manipulation, not a new circuit.
- [[6.004-computation-structures/notes/05-multiplexers-decoders-encoders]] — the MUX, used below to select among several operation results (or among several input transformations) based on a control code, exactly the way Topic 05's select lines chose among data inputs.
- [[6.004-computation-structures/notes/04-static-cmos-gates]] — bitwise AND/OR/XOR/NOT gates, used as ALU operations alongside arithmetic.

## Core definitions

- **Two's complement representation** — an encoding of signed integers in $n$ bits where the value represented by bit pattern $A_{n-1}\cdots A_0$ is $-A_{n-1}\cdot 2^{n-1} + \sum_{i=0}^{n-2}A_i2^i$ (i.e., the same unsigned place-value sum as Topic 06, except the MSB's place value is *negative*). $A_{n-1}$ is still called the MSB but now also directly indicates sign: $A_{n-1}=1$ for negative values, $0$ for non-negative.
- **Two's complement negation (additive inverse)** — for an $n$-bit two's complement value $A$, the bit pattern representing $-A$; derived below to equal $\overline{A}+1$ (bitwise complement of every bit, then add $1$), often abbreviated "invert and add one."
- **Subtractor (via adder reuse)** — a circuit computing $A-B$ by instead computing $A + (\overline{B}+1)$, i.e. an ordinary $n$-bit adder fed $A$ and $\overline{B}$ (bitwise-complemented $B$) with the adder's normally-grounded $C_{in}^{(0)}$ tied to $1$ instead of $0$ — reusing Topic 06's adder verbatim, no new arithmetic circuit.
- **Comparator** — a combinational circuit producing one or more of the relations "$A=B$," "$A<B$," "$A>B$" (in some fixed representation, unsigned or two's complement) as single-bit outputs, given two $n$-bit inputs $A,B$.
- **Zero-detect** — the output signal (used for equality/comparison) that is $1$ exactly when every bit of some $n$-bit value is $0$; realized as a single NOR of all $n$ bits (or equivalently the complement of an $n$-input OR).
- **Arithmetic logic unit (ALU)** — a single combinational circuit with two $n$-bit data inputs $A,B$, a $k$-bit **operation code (opcode)**, and one $n$-bit output, that computes one of several possible functions of $A,B$ (e.g. $A+B$, $A-B$, $A\text{ AND }B$, $A\text{ OR }B$, ...), selected by the opcode via a multiplexer.
- **Bit-slice** — one 1-bit-wide "vertical slice" of an $n$-bit-wide ALU, containing one full adder plus the bitwise-logic gates needed for bit position $i$; an $n$-bit ALU is $n$ identical bit-slices wired side by side (carries rippling between adjacent slices, exactly as in Topic 06), the same "replicate a small identical block" pattern Topic 06 introduced for the adder alone.

## Intuition

**Two's complement makes subtraction "free" once you already have an adder.** Topic 06 built an adder assuming unsigned numbers. Two's complement is a *reinterpretation* of the same bit patterns (same adder circuit, same wires) such that negation becomes a cheap bit-level operation (complement every bit, add one) rather than requiring a fundamentally different circuit — and once negation is cheap, $A-B$ is literally just $A+(-B)$, computed by the *same* adder hardware with $B$'s bits complemented and the carry-in forced to $1$ (supplying the "+1" of negation for free, using a wire that was otherwise tied to a fixed $0$ and is now tied to a fixed $1$ instead). This is why real processors have one adder/subtractor circuit, not two.

**Comparisons piggyback on subtraction.** Once $A-B$ is computable, "is $A=B$" is just "is $A-B$ zero," and (for two's complement) "is $A<B$" is closely related to the *sign* of $A-B$ (with a subtlety around overflow, addressed in the Common Pitfalls below, since the naive "just look at the sign bit" rule breaks exactly when the subtraction itself overflows). No separate comparison circuit is needed beyond the subtractor plus a small amount of extra logic examining its outputs.

**An ALU is "put a MUX after a pile of circuits that all run in parallel, and let the opcode pick which answer to keep."** Rather than a genuinely programmable circuit that reconfigures its wiring at runtime (as a general-purpose reconfigurable device would), the ALUs at this level of the course compute *every* candidate operation's result simultaneously, all the time, on dedicated hardware for each — and simply select which one is exposed as the output, via the opcode driving a MUX's select lines (Topic 05). This trades some wasted switching activity (the unused operations' results are computed and discarded every cycle) for simplicity: no operation-specific control sequencing is needed, since every result is always ready.

## Derivation / formalism

### 1. Why "invert and add one" computes two's complement negation

Claim: for an $n$-bit two's complement value $A$ (as an integer, per the Core definitions weighting), $\overline{A}+1 \pmod{2^n}$ represents $-A$.

**Proof.** Consider $A + \overline{A}$, bit by bit: for every bit position $i$, $A_i + \overline{A_i} = 1$ (a bit and its complement, summed as integers, always give $1$ — direct from the complement law $X+\bar X=1$ of Boolean algebra reinterpreted arithmetically, since exactly one of $A_i,\overline{A_i}$ is $1$). So, treating $A$ and $\overline A$ as $n$-bit unsigned patterns and adding them with an $n$-bit adder (Topic 06) column by column with no cross-column carry needed (each column independently sums to exactly $1$, never $2$, so there is never a carry to propagate — verified by the bit-level fact just shown), $A+\overline{A} = \underbrace{11\cdots1}_n{}_2 = 2^n - 1$ (the all-ones pattern, whose unsigned value is $2^n-1$ by definition of place value). Therefore:
$$A + \overline{A} + 1 = 2^n$$
Modulo $2^n$ (which is exactly the wraparound behavior of an $n$-bit adder — any carry out of the top bit, $C_n$ in Topic 06's notation, is simply discarded/dropped off the $n$-bit result), $2^n \equiv 0$, so:
$$A + (\overline{A}+1) \equiv 0 \pmod {2^n}$$
This says $\overline A + 1$ is exactly the value that, added to $A$ (mod $2^n$, i.e. using ordinary $n$-bit adder wraparound), gives $0$ — which is precisely the defining property of an additive inverse, $-A$. So $\overline{A}+1$ represents $-A$ in two's complement. $\blacksquare$

### 2. Subtraction via adder reuse

Given the adder of Topic 06 and Section 1's negation result, $A-B = A+(-B) = A+(\overline B+1)$. Realize this by feeding the $n$-bit adder's $B$-input port with $\overline{B}$ (an inverter on each of $B$'s $n$ bits — $n$ extra inverters total, negligible compared to the adder itself) and tying $C_{in}^{(0)}$ (which Topic 06 tied to $0$ for plain addition) to $1$ instead. By Topic 06 Section 4's correctness argument applied verbatim with $B$ replaced by $\overline B$ and $C_{in}^{(0)}=1$: the adder computes $A+\overline{B}+1$, which by Section 1 equals $A+(-B)=A-B \pmod{2^n}$ — exactly two's complement subtraction, using the identical adder hardware as addition, differing only in whether $B$'s bits are pre-inverted and whether $C_{in}^{(0)}$ is $0$ or $1$ (both easily selected by a single control bit, e.g. via a bank of $n$ 2-input XOR gates in place of the plain inverters: $B_i \oplus \text{sub} $ gives $B_i$ unchanged when $\text{sub}=0$ and $\overline{B_i}$ when $\text{sub}=1$, since $X\oplus 0=X$ and $X\oplus1=\bar X$ — direct from Topic 06's XOR truth table — and the same $\text{sub}$ bit is wired straight into $C_{in}^{(0)}$).

### 3. Equality comparator: zero-detect on $A \oplus B$ (bitwise)

Claim: $A=B$ (as $n$-bit patterns) if and only if every bit of $A\oplus B$ (bitwise XOR, position by position) is $0$.

**Proof.** By Topic 06's XOR truth table, $A_i\oplus B_i = 0$ exactly when $A_i=B_i$ (both agree), and $=1$ exactly when $A_i\ne B_i$. So every bit of $A\oplus B$ is $0$ iff $A_i=B_i$ for every position $i$, iff $A$ and $B$ are identical as bit patterns — which is the definition of $A=B$. $\blacksquare$ Realize "every bit of $A\oplus B$ is $0$" with a single $n$-input NOR gate (Topic 04's complex gate, or built from smaller NOR/NAND stages) taking all $n$ XOR outputs: $\text{Equal} = \overline{(A_0\oplus B_0)+(A_1\oplus B_1)+\cdots+(A_{n-1}\oplus B_{n-1})}$ — this NOR is the **zero-detect** circuit.

### 4. Unsigned "less-than" via the subtractor's borrow (final carry-out)

For *unsigned* comparison, compute $A-B$ using Section 2's subtractor and examine the final carry-out $C_n$ (Topic 06's overflow bit, here reinterpreted). Claim: for unsigned $A,B$, $A<B$ if and only if $C_n=0$ (i.e., the subtraction "borrowed," conventionally signaled in two's-complement subtraction by the *absence* of a final carry-out — the mirror image of Topic 06's unsigned-addition overflow convention).

**Justification (sketch consistent with Topic 06's framework, not re-deriving two's complement arithmetic from scratch since that is standard machine-arithmetic content, but connecting it explicitly to what's already been derived):** $A-B = A+\overline B + 1$ (Section 2). If $A \ge B$ (unsigned), then $A-B\ge 0$ and the true (unbounded-precision) sum $A+\overline B+1$ is $\ge 2^n$ (since $\overline B = 2^n-1-B$ by Section 1's construction, so $A+\overline B+1 = A + 2^n-1-B+1 = 2^n + (A-B) \ge 2^n$ when $A\ge B$), meaning the $n$-bit adder genuinely overflows and produces $C_n=1$. If $A<B$, then $A-B<0$, so $A+\overline B+1 = 2^n+(A-B) < 2^n$, meaning no overflow occurs and $C_n=0$. So $C_n=0 \iff A<B$ (unsigned), as claimed.

### 5. A minimal 4-operation ALU: structure

Build an $n$-bit ALU with 2-bit opcode $\text{Op}_1\text{Op}_0$ selecting among 4 operations: $00\to A\text{ AND }B$, $01\to A\text{ OR }B$, $10\to A+B$, $11\to A-B$. Structure: compute all four results in parallel on dedicated hardware — an $n$-bit bitwise AND (n 2-input AND gates, Topic 04), an $n$-bit bitwise OR ($n$ 2-input OR-equivalent gates), and one $n$-bit adder/subtractor (Section 2's structure, with the "sub" control wire tied to $\text{Op}_0$ specifically so that $\text{Op}=10$ gives sub$=0$ (addition) and $\text{Op}=11$ gives sub$=1$ (subtraction) — matching the table). Feed all four $n$-bit results into an $n$-bit-wide 4:1 MUX (Topic 05, generalized bit-for-bit: $n$ separate 4:1 MUXes, one per output bit position, all sharing the same select lines $\text{Op}_1\text{Op}_0$) to produce the single $n$-bit output.

## Worked examples

### Example 1 — Two's complement negation and subtraction, 4-bit numeric trace

Let $n=4$. Represent $A=5$ as $0101_2$ and $B=3$ as $0011_2$. Compute $A-B$ via Section 2's method.

**Step 1 — Negate $B$ (Section 1).** $\overline{B} = \overline{0011_2} = 1100_2$. Add $1$: $1100_2+1 = 1101_2$. Check: as an unsigned value, $1101_2=13$; and indeed $13 = 2^4-3 = 16-3$, consistent with $\overline B+1$ representing $-3$ mod $16$ (Section 1's proof, with $n=4$, $2^n=16$).

**Step 2 — Add $A + (\overline B+1)$ using the 4-bit adder from Topic 06, $C_{in}^{(0)}=1$ (per Section 2 — note this replaces the ordinary $0011_2$ addend with $1101_2$ and also sets carry-in to 1, both consequences of the subtractor wiring):**

$A=0101_2=5$, $\overline B+1 = 1101_2 = 13$. Ordinary unsigned addition: $5+13=18 = 10010_2$ (5 bits). Truncated to 4 bits (dropping the carry out of the top, per Section 1's mod-$2^n$ argument): result $=0010_2 = 2$, with $C_4=1$ (a carry was generated).

**Check against expected result:** $A-B = 5-3=2$. The 4-bit adder produced $0010_2=2$ — matches. Per Section 4, since $A=5\ge B=3$ (unsigned), we expect $C_4=1$ (no borrow) — matches the computed $C_4=1$.

### Example 2 — 4-operation ALU trace and the unsigned less-than check

Using Section 5's ALU with $n=4$, $A=0011_2$ ($3$), $B=0101_2$ ($5$):

| $\text{Op}_1\text{Op}_0$ | Operation | Computation | Result |
|---|---|---|---|
| 00 | AND | $0011\ \text{AND}\ 0101$ (bitwise) | $0001_2=1$ |
| 01 | OR | $0011\ \text{OR}\ 0101$ (bitwise) | $0111_2=7$ |
| 10 | $A+B$ | $3+5$ | $1000_2=8$ |
| 11 | $A-B$ | $3+\overline{0101}+1 = 3+1010_2+1 = 3+11=14=1110_2$; check: $3-5=-2$, and $-2 \bmod 16=14=1110_2$ | $1110_2$ (interpreted as $-2$ in two's complement, since its MSB is 1) |

**Less-than check (Section 4) for the $A-B$ case:** the 4-bit adder computing $3+10_2\text{(binary for }\overline B=1010_2\text{)}+1$: true unbounded sum $=3+10+1=14 < 16=2^4$, so no overflow, $C_4=0$. Per Section 4, $C_4=0 \iff A<B$ — and indeed $3<5$, confirming the unsigned less-than logic correctly flags this case using only the subtractor's existing carry-out wire, no extra arithmetic circuit.

## Common pitfalls

- **Using the sign bit of $A-B$ directly as "$A<B$" without checking for overflow, in signed (two's complement) comparison.** In genuinely *signed* comparison (distinct from the unsigned case derived in Section 4), the naive rule "negative result means $A<B$" fails exactly when the subtraction itself overflows the representable signed range (e.g. subtracting a very negative number from a very positive one in a narrow bit width) — correct signed comparison needs to examine both the result's sign bit *and* the overflow condition together, a refinement standard machine-arithmetic references cover but which is beyond this note's derived scope (only unsigned comparison via Section 4 is fully derived here).
- **Forgetting that "invert and add one" is a two's-complement-specific trick.** It relies on the specific place-value weighting of two's complement (Section 1's negative-MSB weighting) and the wraparound (mod $2^n$) behavior of a fixed-width adder; it does not generalize to, e.g., ordinary unsigned arithmetic (where there is no such thing as negation at all) or to other signed representations like sign-magnitude (not covered in this note).
- **Wiring the "sub" control bit only into the $B$-inverters and forgetting $C_{in}^{(0)}$, or vice versa.** Section 2's construction needs *both* changes simultaneously (complement $B$'s bits AND set $C_{in}^{(0)}=1$) to correctly compute $\overline B+1$; doing only one produces neither addition nor subtraction correctly.
- **Assuming an ALU literally "chooses which circuit to run."** As emphasized in Intuition, the ALU structure derived here (Section 5) computes *every* operation's result on dedicated, always-active hardware, every cycle, and only the final MUX selects which result is exposed — no operation-specific circuit is ever "turned off," which has real power/switching-activity consequences in an actual chip (out of scope here, but worth not misunderstanding the mechanism).
- **Confusing the equality comparator's zero-detect NOR (Section 3) with the arithmetic zero result of a subtraction.** Both can be used to test $A=B$ (zero-detect directly on $A\oplus B$, or checking whether $A-B=0$ via the subtractor's own output), and they agree in result, but they are two different circuits reaching the same answer — don't assume one "is" the other, or that only one of the two derivations is "the" correct method.
- **Treating the carry-out convention for subtraction as identical to addition's.** Topic 06 defined $C_n=1$ as unsigned-addition overflow (sum too big). For the subtractor built via Section 2, $C_n=1$ instead means *no borrow occurred* ($A\ge B$ unsigned) — the same wire, same adder hardware, but an inverted-sense interpretation depending on whether the operation is add or subtract; conflating the two conventions is a common source of off-by-one-sense bugs.

## Self-check

### Questions

1. (Easy) Compute the two's complement negation of $0001_2$ (4-bit), and verify by adding it to $0001_2$ that the 4-bit (mod-16) result is $0000_2$.
2. (Easy) For $A=0110_2, B=0110_2$ (both equal), what does the zero-detect equality comparator (Section 3) output, and why?
3. (Medium) Using Section 2's subtractor construction, state exactly which two things change in the adder's wiring (compared to plain addition) to perform subtraction, and why both are necessary (referencing Section 1's proof).
4. (Medium) For 4-bit unsigned $A=0010_2$ ($2$), $B=0111_2$ ($7$), compute $A-B$ via the subtractor (Section 2), determine $C_4$, and use Section 4's rule to state whether $A<B$ — check this matches the true unsigned comparison.
5. (Medium) In the 4-operation ALU of Section 5, what opcode(s) would you add, and how would you wire the extra MUX input, to add a fifth operation "bitwise NOT of $A$" (ignoring $B$ entirely)? State the new opcode width needed.
6. (Hard) Prove that $\overline{\overline{A}} = A$ (double bitwise complement returns the original value) using only Boolean algebra's laws (6.004 Topic 01), and use this fact to argue that negating a two's complement number twice (via Section 1's "invert and add one," applied twice) returns the original number — you'll need to handle the "add one" part carefully, not just the bitwise complement part.
7. (Hard) A buggy ALU implementation wires the "sub" control bit into the $B$-input inverters correctly (Section 2) but leaves $C_{in}^{(0)}$ permanently tied to $0$ regardless of the opcode. For $A=0100_2$ ($4$), $B=0001_2$ ($1$), compute what this buggy circuit produces for the "subtraction" opcode, compare to the true value of $A-B$, and explain the discrepancy in terms of Section 1's derivation.
8. (Hard) Extend Section 4's unsigned less-than argument to derive a rule for unsigned "$A>B$" purely in terms of $C_n$ and the zero-detect equality signal from Section 3 (i.e., express $A>B$ as a Boolean combination of $C_n$ and $\text{Equal}$, both computed by circuitry already derived in this note), and verify it against Example 2's numeric case ($A=3,B=5$, where $A>B$ is false).

### Answers

1. $\overline{0001_2} = 1110_2$. Add $1$: $1110_2+1=1111_2$. So negation of $0001_2$ is $1111_2$. Check: $0001_2+1111_2 = 1+15=16=10000_2$ (5 bits); truncated to 4 bits (mod 16, dropping the carry), result $=0000_2$ — confirms $1111_2$ is the additive inverse of $0001_2$ mod 16.
2. $A\oplus B = 0110\oplus0110 = 0000_2$ (every bit position agrees, XOR gives 0 everywhere per Topic 06's XOR truth table). The $n$-input NOR of an all-zero pattern is $1$ (NOR of all-0 inputs $=\overline{0+0+0+0}=\overline0=1$). So $\text{Equal}=1$, correctly signaling $A=B$.
3. Two changes: (1) each bit of $B$ is bitwise-complemented before entering the adder (realized via $n$ XOR gates gated by the sub control bit, or plain inverters if subtraction is hardwired), and (2) $C_{in}^{(0)}$ is set to $1$ instead of $0$. Both are necessary because Section 1's proof shows $-B = \overline B + 1$ specifically — complementing $B$'s bits alone only produces $\overline B$ (missing the $+1$), and adding $1$ alone to uncomplemented $B$ doesn't produce $-B$ at all; the two together are what the proof derives as jointly necessary and sufficient.
4. $\overline B = \overline{0111_2}=1000_2$. $A+\overline B+1 = 0010_2+1000_2+1 = 2+8+1=11=1011_2$ (fits in 4 bits, no overflow beyond 4 bits since $11<16$). So $A-B$ result bits $=1011_2$, and since the true unbounded sum $11 < 16=2^4$, no overflow occurred, meaning $C_4=0$. Per Section 4, $C_4=0\iff A<B$ — and indeed $2<7$ is true, matching the true unsigned comparison.
5. Add a new 3-bit opcode (since 4 operations needed 2 bits for $2^2=4$ codes, and a 5th operation requires $\lceil\log_2 5\rceil=3$ bits to distinguish 5+ cases, per the encoding argument implicit in Topic 05's decoder/MUX select-line sizing) — e.g. $\text{Op}=100$ for NOT-$A$. Wire an additional $n$-bit bitwise inverter (Topic 04, $n$ single-input NOT gates on $A$'s bits, ignoring $B$ entirely) as a fifth candidate result, and widen the output MUX from a 4:1 to at least a 5:1 (in practice implemented as an 8:1 MUX with 3 select bits, since $3$ bits naturally support up to 8 codes, with the unused 3 codes' MUX inputs left as don't-cares/tied off).
6. By definition, $\overline{X}$ flips $X$: if $X=0$, $\overline X=1$; if $X=1$, $\overline X=0$. Applying complement twice: if $X=0$: $\overline X=1$, then $\overline{\overline X}=\overline 1=0=X$. If $X=1$: $\overline X=0$, then $\overline{\overline X}=\overline0=1=X$. Both cases give $\overline{\overline X}=X$ — by perfect induction (only 2 cases for 1 variable), proven for a single bit, and therefore bitwise for every bit of an $n$-bit pattern $A$: $\overline{\overline A}=A$. For the full "negate twice returns the original" claim: negation is $N(A) = \overline A+1$. Compute $N(N(A)) = \overline{N(A)}+1 = \overline{\overline A+1}+1$. This requires reasoning about complementing a *sum*, not just a bit pattern directly — but by Section 1's own proof (applied to $N(A)$ in place of $A$), $N(N(A))$ is, by definition, whatever value added to $N(A)$ gives $0 \pmod{2^n}$; since $N(A)$ was itself defined as the value that added to $A$ gives $0\pmod{2^n}$ (i.e. $A+N(A)\equiv0$), this same equation read the other way says $N(A) + A \equiv 0$, i.e. $A$ is *also* a valid "value that added to $N(A)$ gives $0$" — and since that defining value is unique mod $2^n$ (only one $n$-bit pattern added to a given $N(A)$ can give exactly $0\pmod{2^n}$), $N(N(A))=A$. So double negation returns the original value, confirmed via the additive-inverse definition from Section 1 rather than by directly re-expanding the bitwise-complement-plus-one formula (which the double-complement fact alone, from the first part of this answer, isn't quite sufficient to finish without this additional "uniqueness of additive inverse" argument).
7. With $C_{in}^{(0)}$ stuck at $0$: the circuit computes $A+\overline B+0 = A+\overline B$ instead of $A+\overline B+1$. $\overline B = \overline{0001_2}=1110_2=14$. Buggy result: $0100_2+1110_2 = 4+14=18=10010_2$ (5 bits), truncated to 4 bits: $0010_2=2$, with a carry out. True value: $A-B=4-1=3=0011_2$. The buggy circuit produces $2$ instead of the correct $3$ — off by exactly $1$, precisely the missing "$+1$" from Section 1's derivation ($\overline B$ alone, without the $+1$, computes $\overline B = 2^n-1-B$, not $-B=2^n-B$; the buggy adder therefore computes $A+(2^n-1-B) = A-B-1+2^n \equiv A-B-1\pmod{2^n}$, one less than the correct $A-B$, matching the observed discrepancy of exactly $1$).
8. $A>B$ (unsigned) holds exactly when $A\ne B$ AND NOT($A<B$), i.e. $A\ge B$ and $A\ne B$. From Section 4, $A<B \iff C_n=0$ for the subtractor computing $A-B$, so $A\ge B \iff C_n=1$. Combined with $A\ne B \iff \overline{\text{Equal}}$ (Section 3's Equal signal complemented): $A>B = C_n \cdot \overline{\text{Equal}}$. Check against Example 2 ($A=3,B=5$): that example's subtractor computed $C_4=0$ (already found in Example 2 for the $A-B$ case) and $A\ne B$ so $\text{Equal}=0,\overline{\text{Equal}}=1$; $A>B = C_4\cdot\overline{\text{Equal}} = 0\cdot1=0$ — correctly reports $A>B$ is false, matching the true relation $3>5=\text{false}$.

## Summary / cheat sheet

**Two's complement negation:** $-A = \overline{A}+1 \pmod{2^n}$ (proof: $A+\overline A = 2^n-1$ bitwise, so $A+\overline A+1=2^n\equiv0$).

**Subtraction via adder reuse:** $A-B = A+\overline B+1$; wire $B$ through invertors (or XOR with a "sub" control bit) and set $C_{in}^{(0)}=1$.

**Equality comparator:** $\text{Equal} = \overline{(A_0\oplus B_0)+\cdots+(A_{n-1}\oplus B_{n-1})}$ (zero-detect NOR of bitwise XOR).

**Unsigned comparisons via the subtractor's $C_n$:** $A<B \iff C_n=0$; $A\ge B\iff C_n=1$; $A>B = C_n\cdot\overline{\text{Equal}}$.

**ALU structure:** compute every candidate operation in parallel (dedicated hardware each), select the exposed result with an opcode-driven MUX (Topic 05) — no operation-specific circuit is "turned off."

## Used later in
(none yet)
