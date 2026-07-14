---
title: "Canonical Forms & Simplification (SOP/POS, K-Maps, Don't-Cares)"
course: "6.004"
topic_number: 02
prerequisites: ["6.004 Topic 01 — Boolean Algebra & Logic Functions"]
status: done
---

# Canonical Forms & Simplification (SOP/POS, K-Maps, Don't-Cares)

## Why this matters

Topic 01's last Common Pitfall flagged an open problem: given a Boolean function, algebraic manipulation can simplify an expression, but different sequences of manipulation can land on different-looking, equally-short results, and nothing said *where to start* or *when you're actually done*. In real design, you don't start with an expression at all — you start with a specification (a truth table: "here is what the circuit must output for every input combination"), and you need two things algebra alone doesn't hand you: (1) a mechanical, unambiguous way to turn that specification into *some* correct expression, and (2) a systematic procedure that's guaranteed to find a minimal (or near-minimal) expression, not just "an expression I stopped simplifying."

This note supplies both. **Canonical SOP/POS forms** solve problem (1): a direct, unique translation from any truth table to a Boolean expression. **Karnaugh maps (K-maps)** solve problem (2) for small numbers of variables: a visual arrangement of the truth table that turns "find a valid algebraic simplification" into "look for adjacent groups of 1s," with a guarantee that the result is minimal by a fixed cost measure (fewest literals). **Don't-cares** extend both tools to specifications that are only partially defined. Skip this topic and Topic 03 onward (turning expressions into actual CMOS gates) has no principled way to start from a design spec, and no way to know whether a circuit is unnecessarily large.

## Builds on

- [[6.004-computation-structures/notes/01-boolean-algebra-logic-functions]] — Boolean algebra's postulates and derived laws (identity, distributive both directions, complement, absorption), truth tables, perfect induction, and De Morgan's laws. This note reuses all of these directly, and in particular reuses the exact algebraic pattern from Topic 01's Self-check Question 4 ($XY+X\bar Y=X$) as the mechanical engine behind every K-map grouping below.

## Core definitions

- **Minterm** — for a function of $n$ variables, a product (AND) term containing all $n$ variables exactly once each, either uncomplemented or complemented; a minterm evaluates to $1$ for exactly one of the $2^n$ input combinations and $0$ for all others.
- **Minterm index $m_i$** — the minterm corresponding to input combination $i$, where $i$ is the input row read as a binary number (e.g. for variables $A,B,C$, row $A{=}1,B{=}0,C{=}1$ is index $5$ ($101_2$), so $m_5 = A\bar B C$). A variable appears uncomplemented in $m_i$ if its bit in $i$'s binary representation is $1$, complemented if that bit is $0$.
- **Maxterm $M_i$** — for a function of $n$ variables, a sum (OR) term containing all $n$ variables exactly once each; a maxterm evaluates to $0$ for exactly one input combination (row $i$) and $1$ for all others. A variable appears uncomplemented in $M_i$ if its bit in $i$ is $0$, complemented if that bit is $1$ — the *opposite* convention from minterms, so that $M_i=0$ exactly at row $i$.
- **Canonical sum-of-products (canonical SOP / minterm expansion)** — the OR of every minterm $m_i$ for which the function's truth table has output $1$ at row $i$. Written $F=\Sigma m(i_1,i_2,\dots)$, listing the indices where $F=1$.
- **Canonical product-of-sums (canonical POS / maxterm expansion)** — the AND of every maxterm $M_i$ for which the function's truth table has output $0$ at row $i$. Written $F=\Pi M(i_1,i_2,\dots)$, listing the indices where $F=0$.
- **(General, non-canonical) sum-of-products expression** — an OR of AND terms where the AND terms are not required to each contain every variable (i.e., simplified products, possibly with fewer literals than a full minterm).
- **(General, non-canonical) product-of-sums expression** — an AND of OR terms, similarly not required to be full maxterms.
- **Karnaugh map (K-map)** — a rectangular grid holding one cell per row of a truth table, with rows/columns ordered so that any two grid-adjacent cells (including wraparound from one edge to the opposite edge) correspond to input combinations differing in exactly one variable.
- **Gray code** — an ordering of binary numbers in which consecutive entries differ in exactly one bit (e.g. for 2 bits: $00,01,11,10$); this is exactly the ordering K-map rows/columns use to guarantee the adjacency property above.
- **Implicant** — any product term (of one or more literals) that is $1$ only for input rows where the function is $1$ (i.e., never "covers" a row where $F=0$).
- **Prime implicant** — an implicant that cannot be enlarged (have a literal dropped) while remaining an implicant; on a K-map, the largest valid rectangular group of $1$s/don't-cares containing a given cell.
- **Essential prime implicant** — a prime implicant that is the *only* prime implicant covering some particular minterm; every minimal SOP expression must include every essential prime implicant.
- **Don't-care condition** (written $d$ or $\times$ in a truth table/K-map) — an input row for which the function's required output is unspecified (that input combination either cannot occur, or its output genuinely doesn't affect correctness); may be treated as either $0$ or $1$ during simplification, whichever produces a larger/cheaper grouping.

## Intuition

**Canonical forms are a direct translation, not a simplification.** A canonical SOP is built by a purely mechanical rule: for every row where the truth table says $1$, write down the one minterm that is $1$ exactly there, and OR them all together. Nothing is simplified yet — a canonical SOP for an $n$-variable function typically has as many terms as there are $1$s in the truth table, each with $n$ literals, which is usually the *least* efficient way to write the function. Its value is that it's unambiguous and mechanical: given any truth table, you always know exactly how to write down a correct expression, with zero judgment calls. Everything after this is about shrinking that starting point.

**K-maps turn algebraic pairing into a visual pattern-matching game.** The single algebraic move that shrinks a canonical SOP is one you already proved in Topic 01, Self-check Question 4: $XY+X\bar Y = X$. Any two minterms that agree on every variable except one can be merged this way, dropping that one variable. A K-map's entire purpose is to arrange the truth table so that "these two rows agree on every variable but one" becomes "these two cells are next to each other" — literally, spatial adjacency substitutes for algebraic bookkeeping. That's why the rows/columns must be ordered in Gray code rather than plain binary count order: plain counting jumps by more than one bit between some consecutive rows (e.g. $01\to10$ flips two bits), which would break the "adjacent cells differ in one variable" property the whole method depends on.

**Grouping bigger blocks eliminates more variables, by repeating the same trick.** Merging 2 adjacent cells (a "pair") uses the $XY+X\bar Y=X$ move once, dropping 1 variable. Merging 4 cells (a "quad") is two such merges in a row, dropping 2 variables; merging 8 cells drops 3; in general, a group of $2^k$ cells (arranged so every combination of some $k$ variables appears while the rest stay fixed) drops exactly those $k$ variables, leaving a term with $n-k$ literals. Bigger valid groups are strictly better (fewer literals), which is why the minimization procedure is "find the largest groups you can."

**Don't-cares are free design latitude, not a hack.** If a specification never constrains what a circuit should output for some input (because that input can't physically occur, or because nothing downstream depends on the answer there), then *by definition* no correctness requirement is violated no matter what that cell outputs. Treating a don't-care cell as a $1$ specifically when doing so lets a group grow bigger (fewer literals, cheaper circuit) is not "cheating the spec" — the spec placed no constraint there in the first place, so every choice is equally correct; you're only choosing based on cost, which is exactly the freedom "don't-care" was defined to grant.

## Derivation / formalism

### 1. Why the canonical SOP reproduces the truth table exactly

Claim: for a function $F$ with truth table rows $0,\dots,2^n-1$, the expression $F=\sum_{i:\,F(i)=1} m_i$ (OR of the minterms for every $1$-row) computes the same function as the original table.

**Proof.** By definition, $m_i$ evaluates to $1$ at exactly one input combination — row $i$ — and $0$ at every other row. Consider the OR of a set $S=\{i: F(i)=1\}$ of such minterms, evaluated at some arbitrary row $j$:
- If $j\in S$ (i.e. $F(j)=1$ per the table): then $m_j$ is one of the terms being ORed, and $m_j=1$ at row $j$ (by the minterm's own definition). Since OR is $1$ if *any* term is $1$ (Topic 01's OR truth table), the whole sum is $1$ at row $j$ — matching $F(j)=1$.
- If $j\notin S$ (i.e. $F(j)=0$ per the table): every minterm $m_i$ in the sum has $i\ne j$, so by the minterm's defining property (evaluates to $0$ everywhere except row $i$), each such $m_i=0$ at row $j$. OR of all-$0$ terms is $0$ (again, the OR truth table), matching $F(j)=0$.

Both cases match the original table at every row $j$, so by perfect induction the canonical SOP and the truth table define the identical function. $\blacksquare$

### 2. Why the canonical POS reproduces the truth table exactly (dual argument)

Claim: $F = \prod_{i:\,F(i)=0} M_i$ (AND of the maxterms for every $0$-row) also computes the same function.

**Proof.** By definition $M_i$ evaluates to $0$ at exactly row $i$ and $1$ everywhere else. Let $S=\{i:F(i)=0\}$. At an arbitrary row $j$:
- If $j\in S$ ($F(j)=0$): $M_j$ is one of the AND's factors, and $M_j=0$ at row $j$. AND is $0$ if *any* factor is $0$ (Topic 01's AND truth table), so the whole product is $0$ — matching.
- If $j\notin S$ ($F(j)=1$): every factor $M_i$ in the product has $i\ne j$, so each evaluates to $1$ at row $j$ (maxterms are $1$ everywhere except their own row). AND of all-$1$ factors is $1$ — matching.

Matches at every row, so by perfect induction the canonical POS is correct. $\blacksquare$ This is exactly the dual (in the sense of Topic 01's duality principle) of the SOP argument, with every $+\leftrightarrow\cdot$ and $0\leftrightarrow1$ swapped.

### 3. Converting between minterm and maxterm indices via De Morgan's

Applying De Morgan's laws (Topic 01) twice to $F$ relates the two canonical forms directly: $\overline{F} = \Sigma m(\text{indices where }F{=}0)$ (the complement function is $1$ exactly where $F$ was $0$), and taking the complement of that expression via De Morgan's converts every minterm (an AND of literals) into a maxterm (an OR of the complemented literals) with the *same index*. This is why the maxterm's complementation convention is the mirror image of the minterm's: $M_i = \overline{m_i}$ for every index $i$, which you can verify directly from the definitions above (every literal that's uncomplemented in $m_i$ becomes complemented in $M_i$, and vice versa — exactly what applying NOT to a product of literals produces via De Morgan's).

### 4. The pairing rule behind K-map grouping

Take two minterms that agree on every variable except one, say variable $Q$: $P\cdot Q$ and $P\cdot\bar Q$, where $P$ is the (identical) product of the other $n-1$ literals. Their sum:
$$PQ + P\bar Q = P(Q+\bar Q) \qquad \text{[distributive, AND-over-OR, reversed]}$$
$$= P\cdot 1 \qquad \text{[complement: } Q+\bar Q=1\text{]}$$
$$= P \qquad \text{[identity]}$$
This is line-for-line the identity from Topic 01 Self-check Question 4 ($XY+X\bar Y=X$), with $X=P$ and $Y=Q$. The variable that differed between the two minterms ($Q$) disappears entirely; every variable that was the same ($P$) survives. Repeating this on two adjacent pairs (each already missing the same one variable) merges them into a group of 4, dropping a second variable, by the same algebraic move applied to the new common factor. In general, a valid group of $2^k$ cells corresponds to $k$ applications of this pairing, and leaves a term with $n-k$ literals — the literals corresponding to whichever variables varied across the group.

### 5. The minimization procedure

1. Plot the truth table on a K-map (Gray-code row/column order), placing $1$, $0$, or don't-care ($d$) in each cell.
2. Find every **prime implicant**: for each $1$-cell (and optionally each don't-care cell), find the largest valid rectangular group (size a power of 2, containing only $1$s and don't-cares, wraparound allowed) that contains it.
3. Identify the **essential prime implicants**: any prime implicant that is the *only* one covering some particular required ($1$-valued, not don't-care) minterm — these must appear in the final expression.
4. If the essential prime implicants alone cover every required $1$-cell, the minimal SOP is their sum. If some required $1$-cells remain uncovered, add the fewest additional (non-essential) prime implicants needed to cover them (the "covering problem" — for the sizes in this course, solvable by inspection).
5. Don't-care cells never need to be covered themselves — they're only ever used to help some other group grow larger; a don't-care left uncovered by the final chosen groups is implicitly assigned $0$ in the resulting circuit, which is a fully valid choice since its value was unconstrained.

## Worked examples

### Example 1 — Canonical forms and K-map minimization for a 3-variable function

Let $F(A,B,C) = \Sigma m(0,1,2,3,5,7)$ (i.e. $F=1$ at rows $0,1,2,3,5,7$ and $F=0$ at rows $4,6$).

**Canonical SOP** (one minterm per listed index): $F = \bar A\bar B\bar C + \bar A\bar BC + \bar AB\bar C + \bar ABC + A\bar BC + ABC$.

**Algebraic simplification, using the pairing rule (Derivation §4) repeatedly:**
$$\bar A\bar B\bar C + \bar A\bar BC = \bar A\bar B(\bar C+C) = \bar A\bar B$$
$$\bar AB\bar C + \bar ABC = \bar AB(\bar C+C) = \bar AB$$
$$\bar A\bar B + \bar AB = \bar A(\bar B+B) = \bar A$$
$$A\bar BC + ABC = AC(\bar B+B) = AC$$
So $F = \bar A + AC$. One more simplification (verified by perfect induction below, since it's a slightly less obvious pattern than plain pairing): $\bar A+AC = (\bar A+A)(\bar A+C) = 1\cdot(\bar A+C) = \bar A + C$ (distributive OR-over-AND, then complement, then identity — the exact structure of Topic 01's Derivation §2 dominance proof, reused here).

**Minimal SOP: $F=\bar A+C$.** Notice $B$ dropped out entirely.

**Perfect-induction check** (all 8 rows):

| $A$ | $B$ | $C$ | Canonical $F$ | $\bar A+C$ |
|---|---|---|---|---|
| 0 | 0 | 0 | 1 | 1 |
| 0 | 0 | 1 | 1 | 1 |
| 0 | 1 | 0 | 1 | 1 |
| 0 | 1 | 1 | 1 | 1 |
| 1 | 0 | 0 | 0 | 0 |
| 1 | 0 | 1 | 1 | 1 |
| 1 | 1 | 0 | 0 | 0 |
| 1 | 1 | 1 | 1 | 1 |

Matches on all 8 rows.

**K-map (rows $A$, columns $BC$ in Gray order $00,01,11,10$):**

| | $BC{=}00$ | $BC{=}01$ | $BC{=}11$ | $BC{=}10$ |
|---|---|---|---|---|
| $A{=}0$ | 1 | 1 | 1 | 1 |
| $A{=}1$ | 0 | 1 | 1 | 0 |

Two prime implicants: the entire top row ($A{=}0$, all 4 cells) $=\bar A$ (drops $B,C$ — both vary across the group, only $A=0$ is common); the middle two columns ($BC{=}01$ and $BC{=}11$, both rows) $=C$ (both columns have $C{=}1$; $A$ and $B$ vary across the group). Row $A{=}0$ is the only group covering minterm $2$ ($\bar AB\bar C$), so it's essential; the $C$ group is the only one covering minterm $5$ ($A\bar BC$), so it's essential too. Together they cover $\{0,1,2,3,5,7\}$ — every required $1$ — matching the algebraic result $F=\bar A+C$ exactly.

**Canonical POS, as a consistency check:** $F=0$ at rows $4$ ($A\bar B\bar C$, giving maxterm $M_4=\bar A+B+C$) and $6$ ($AB\bar C$, giving $M_6 = \bar A+\bar B+C$). Canonical POS: $F=(\bar A+B+C)(\bar A+\bar B+C)$. Simplifying by factoring out the common literal $\bar A$ (distributive OR-over-AND, reversed: $(\bar A+P)(\bar A+Q) = \bar A + PQ$ with $P=B+C, Q=\bar B+C$): $F = \bar A + (B+C)(\bar B+C)$. Simplify the inner product by factoring out $C$: $(B+C)(\bar B+C) = C + B\bar B = C+0=C$. So $F = \bar A + C$ — matching the minimal SOP exactly, confirming the result from a completely independent starting point.

### Example 2 — 4-variable K-map with don't-cares

Let $F(A,B,C,D)$ be specified as: $F=1$ for minterms $\{0,2,4,6,8,10\}$, $F=0$ for every odd-indexed minterm ($1,3,5,7,9,11,13,15$), and don't-care ($d$) for $\{12,14\}$.

**K-map (rows $AB$, columns $CD$, both in Gray order $00,01,11,10$; index $=8A+4B+2C+D$):**

| | $CD{=}00$ | $CD{=}01$ | $CD{=}11$ | $CD{=}10$ |
|---|---|---|---|---|
| $AB{=}00$ | 1 ($m_0$) | 0 ($m_1$) | 0 ($m_3$) | 1 ($m_2$) |
| $AB{=}01$ | 1 ($m_4$) | 0 ($m_5$) | 0 ($m_7$) | 1 ($m_6$) |
| $AB{=}11$ | $d$ ($m_{12}$) | 0 ($m_{13}$) | 0 ($m_{15}$) | $d$ ($m_{14}$) |
| $AB{=}10$ | 1 ($m_8$) | 0 ($m_9$) | 0 ($m_{11}$) | 1 ($m_{10}$) |

Columns $CD{=}00$ and $CD{=}10$ are entirely $1$ or $d$ (reading down: $1,1,d,1$ in both columns) — both columns have $D{=}0$ in common, while $A$, $B$, and $C$ each take both values across the 8 cells in these two columns. Treating both don't-cares as $1$ makes this a single valid 8-cell group, dropping $A$, $B$, and $C$ entirely and leaving the single literal $\bar D$.

**Minimal SOP: $F = \bar D$.**

**Why the don't-cares matter here:** without using $m_{12}, m_{14}$ as $1$s, the required-$1$ set would be exactly $\{0,2,4,6,8,10\}$ with $\{12,14\}$ forbidden from being covered — the largest group would then have to stop at the $AB{=}11$ row, splitting into at least two smaller groups (e.g. $\bar B\bar D$ covering $\{0,2,8,10\}$, plus $\bar A B\bar D$ covering $\{4,6\}$ — 2 and 3 literals respectively, 5 literals total across two terms) instead of the single literal $\bar D$. This is the concrete payoff of don't-cares: the same correctness requirement, implemented with far fewer literals, because the specification never constrained rows $12,14$ in the first place.

**Partial perfect-induction check** (verifying $F=\bar D$ matches every *required* row — don't-care rows are skipped since no fixed value is required there):

| Row (index) | $D$ | Required $F$ | $\bar D$ |
|---|---|---|---|
| 0 | 0 | 1 | 1 |
| 2 | 0 | 1 | 1 |
| 4 | 0 | 1 | 1 |
| 6 | 0 | 1 | 1 |
| 8 | 0 | 1 | 1 |
| 10 | 0 | 1 | 1 |
| 1,3,5,7,9,11,13,15 | 1 | 0 | 0 |

Every required row matches; rows 12 and 14 (both $D{=}0$) get $\bar D=1$ from this implementation, which is a fully valid choice since those rows were unconstrained.

## Common pitfalls

- **Mixing up the minterm and maxterm complementation conventions.** Minterms use a variable uncomplemented when its bit is $1$; maxterms use a variable uncomplemented when its bit is $0$ (the opposite). Applying the minterm rule while building a POS (or vice versa) silently produces the complement of the intended term.
- **Building the POS from the rows where $F=1$ instead of $F=0$.** Canonical POS is the AND of maxterms for the *zero* rows — carrying over "list the rows where the function is 1" from SOP habit is a common transcription error.
- **Grouping a non-power-of-2 number of K-map cells.** Only groups of size $1,2,4,8,16,\dots$ correspond to valid eliminations, because each merge (Derivation §4) eliminates exactly one variable per doubling; a group of, say, 3 or 6 cells does not correspond to any single algebraic simplification.
- **Forgetting K-map wraparound.** The leftmost and rightmost columns are adjacent (as are the top and bottom rows), because Gray-code adjacency wraps around; groups that cross this "seam" are just as valid as any other, and missing them means missing a legitimate simplification.
- **Assuming "non-essential" means "never needed."** A non-essential prime implicant may still be required to cover a minterm that no essential prime implicant reaches (Self-check Question 6) — "non-essential" only means "not the *unique* option for some minterm," not "safe to discard unconditionally."
- **Treating a don't-care as truly nondeterministic in the final circuit.** "Don't-care" describes freedom during the *design* process (you may choose either value while minimizing); once you've picked specific groups and built the circuit, every input — including former don't-cares — produces one fixed, determined output. The nondeterminism is in your design choice, not in the finished hardware.

## Self-check

### Questions

1. (Easy) A 2-variable function $F(A,B)$ (the XOR function) is $1$ exactly when $A$ and $B$ differ. Write its canonical SOP, listing the minterm indices used.
2. (Easy) How many literals does each minterm term contain in the canonical SOP of a 5-variable function, and why?
3. (Medium) For the function in Question 1, find the canonical POS (list the maxterm(s) used and write the resulting expression).
4. (Medium) On a 3-variable K-map, a group consists of an entire row (all 4 cells for one fixed value of variable $A$). What single literal does this group correspond to, and why do the other two variables disappear?
5. (Medium) In your own words, explain why assigning a don't-care cell the value that makes a K-map group bigger never violates the function's specification.
6. (Hard) A K-map has a prime implicant that is not essential (every minterm it covers is also covered by some other prime implicant). Is it ever still necessary to include this non-essential prime implicant in the final minimized expression? Explain when yes and when no.
7. (Hard) Take Example 2's function, but now suppose minterms $12$ and $14$ are required to be $0$ (not don't-cares). Find the new minimal SOP, and explain using the K-map picture why the single-literal group $\bar D$ is no longer valid.
8. (Hard) Prove algebraically (not by appealing to the K-map picture) that $\bar A\bar B\bar C\bar D+\bar A\bar BC\bar D+\bar AB\bar C\bar D+\bar ABC\bar D = \bar A\bar D$, by applying the pairing rule from Derivation §4 twice.

### Answers

1. $F=1$ at $A{=}0,B{=}1$ (index $01_2=1$) and $A{=}1,B{=}0$ (index $10_2=2$). Canonical SOP: $F=\Sigma m(1,2) = \bar AB + A\bar B$.
2. Exactly 5 literals per minterm. A minterm, by definition, contains every variable of the function exactly once (complemented or not) — that's precisely what pins the term down to a single one of the $2^5=32$ rows, so for 5 variables every minterm has 5 literals.
3. $F=0$ at $A{=}0,B{=}0$ (index $0$) and $A{=}1,B{=}1$ (index $3$). Maxterm for row $0$ (bits $00$, both uncomplemented per the maxterm convention): $M_0=A+B$. Maxterm for row $3$ (bits $11$, both complemented): $M_3=\bar A+\bar B$. Canonical POS: $F=(A+B)(\bar A+\bar B)$.
4. The group corresponds to the single literal $A$ (or $\bar A$, depending on which row) — whichever value of $A$ that row fixes. The other two variables ($B$ and $C$) disappear because, within that one row, all $4$ combinations of $B,C$ are present (the row spans every column), so pairing eliminates $C$ first (merging cells that differ only in $C$), then eliminates $B$ (merging the two resulting $BC$-eliminated halves), exactly as in Derivation §4's repeated-pairing argument — leaving only the one variable ($A$) that never varied across the group.
5. A don't-care marks an input row the specification places no requirement on — either the input can't occur, or the correct output truly doesn't matter there. Since "correct" is only defined relative to the specification, and the specification says nothing about that row, no choice of output for that row can violate correctness; the only thing that choice affects is how large a K-map group can be built around it (i.e., circuit cost), never whether the circuit meets its spec.
6. Yes, sometimes it's still necessary: if, after including every essential prime implicant, some required ($1$-valued) minterm remains uncovered, you must add non-essential prime implicant(s) until every required minterm is covered by at least one chosen group (the covering step of the minimization procedure, Derivation §5 step 4). It is *not* necessary (and should be omitted) if every required minterm is already covered by the chosen essential prime implicants alone — including it then would only add literals/gates without being needed for correctness.
7. With $m_{12},m_{14}$ forced to $0$, the required-$1$ set is exactly $\{0,2,4,6,8,10\}$, and the $AB{=}11$ row is now entirely forbidden from any group. The two columns $CD{=}00,CD{=}10$ can no longer be grouped as a full 8-cell block (that block includes the now-forbidden $m_{12},m_{14}$ cells), so the single-literal group $\bar D$ is invalid — it would incorrectly force $F=1$ at rows $12,14$. Instead: group $AB\in\{00,10\}$ (both have $B{=}0$) across both $D{=}0$ columns, covering $\{0,2,8,10\}$, giving $\bar B\bar D$; separately group the $AB{=}01$ row's $D{=}0$ cells, covering $\{4,6\}$, giving $\bar A B\bar D$. Minimal SOP: $F=\bar B\bar D+\bar AB\bar D$ (5 literals total across two terms, versus the single literal $\bar D$ when $12,14$ were don't-cares) — a direct, concrete demonstration of what those don't-cares were worth.
8. $\bar A\bar B\bar C\bar D+\bar A\bar BC\bar D = \bar A\bar B\bar D(\bar C+C)$ [distributive, AND-over-OR, reversed, factoring out the common part $\bar A\bar B\bar D$] $=\bar A\bar B\bar D\cdot 1$ [complement] $=\bar A\bar B\bar D$ [identity]. Similarly $\bar AB\bar C\bar D+\bar ABC\bar D = \bar AB\bar D(\bar C+C) = \bar AB\bar D$. Now pair these two results: $\bar A\bar B\bar D+\bar AB\bar D = \bar A\bar D(\bar B+B)$ [distributive, factoring out $\bar A\bar D$] $=\bar A\bar D\cdot 1$ [complement] $=\bar A\bar D$ [identity]. Two applications of the pairing rule (first eliminating $C$, then eliminating $B$) reduce the original 4-term, 4-literal-each expression to $\bar A\bar D$.

## Summary / cheat sheet

**Canonical forms (direct from a truth table, not yet simplified):**
- SOP: $F=\Sigma m(\text{indices where }F{=}1)$ — OR of minterms, each variable uncomplemented iff its bit is 1.
- POS: $F=\Pi M(\text{indices where }F{=}0)$ — AND of maxterms, each variable uncomplemented iff its bit is 0 (opposite of minterm convention). $M_i=\overline{m_i}$.

**Pairing rule (the engine behind every simplification):** $PQ+P\bar Q=P$ (Topic 01's $XY+X\bar Y=X$, generalized to any common factor $P$). A group of $2^k$ adjacent K-map cells $=$ $k$ repeated applications $=$ a term with $n-k$ literals.

**K-map minimization procedure:** plot on Gray-code grid (wraparound adjacency) $\to$ find prime implicants (largest valid groups) $\to$ identify essential ones (uniquely cover some minterm; always include) $\to$ cover any remaining required minterms with additional prime implicants.

**Don't-cares ($d$):** unconstrained rows; assign $0$ or $1$ freely, whichever grows a group larger — never affects correctness, only cost.

## Used later in
- [[6.004-computation-structures/notes/03-cmos-switches-mosfet-abstraction]] — the SOP/POS canonical shapes reappear directly as switch-network topologies: a parallel-of-series switch network realizes an SOP expression, a series-of-parallel network realizes a POS expression.
- [[6.004-computation-structures/notes/05-multiplexers-decoders-encoders]] — a decoder's outputs are literally the minterms $m_i$ realized as physical gates (one per output), and a MUX's general formula sums minterms of the select lines, each multiplying a data input.
- [[6.004-computation-structures/notes/06-adders-basic-arithmetic]] — K-map minimization derives the full adder's sum and carry-out expressions from its 8-row truth table, including identifying all three carry-out prime implicants as essential.
- [[6.004-computation-structures/notes/12-fsm-design-procedure]] — K-map minimization and don't-cares are applied directly, unmodified, to each next-state and output bit of an encoded FSM transition table, with unused state code words supplying the don't-care rows.
