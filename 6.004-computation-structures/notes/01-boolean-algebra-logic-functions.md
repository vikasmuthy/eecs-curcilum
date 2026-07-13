---
title: "Boolean Algebra & Logic Functions"
course: "6.004"
topic_number: 01
prerequisites: ["6.002 Topic 04 — Nonlinear Elements, Digital Abstraction, MOSFET Model"]
status: done
---

# Boolean Algebra & Logic Functions

## Why this matters

6.002 Topic 04 established the **digital abstraction**: under the static discipline, a wire's continuous, physical voltage can be reliably read as exactly one of two symbols, 0 or 1, with the noise-margin argument guaranteeing that reading is safe against realistic disturbances. That result is the last time this curriculum needs to think about voltage at all to reason about *what a digital circuit computes*. Once a wire's value is guaranteed to be a clean 0 or 1, the question "what does this network of gates compute?" becomes a purely symbolic question about functions on the two-element set $\{0,1\}$ — no calculus, no circuit theory, just algebra.

This note builds that algebra: what the primitive operations (AND, OR, NOT) mean as functions on $\{0,1\}$, and the algebraic laws that let you prove two circuits compute the *same* function without checking every input by hand. Skip this and every later topic breaks down to brute-force truth-table checking, which stops being practical the moment a circuit has more than four or five inputs ($2^n$ rows) — Topic 02's simplification techniques, Topic 04's gate implementations, and every FSM design later in the course all lean on being able to manipulate Boolean expressions algebraically instead of exhaustively.

## Builds on

- [[6.002-circuits-and-electronics/notes/04-nonlinear-elements-digital-abstraction-mosfet-model]] — the digital abstraction and its static-discipline guarantee, recapped inline below so you don't have to context-switch back to that note.

This note relies on one result from 6.002: the **digital abstraction** (6.002 Topic 04) is the engineering convention of dividing a continuous range of wire voltages into two disjoint bands — one read as logic "0," one read as logic "1" — with a static-discipline guarantee ($V_{OL}\le V_{IL}$, $V_{OH}\ge V_{IH}$) that a driving gate's worst-case output voltage is always safely inside the receiving gate's input tolerance for that symbol. That guarantee is exactly what licenses everything in this note: because a wire's value is *guaranteed* to be read as one of two symbols and nothing else, we can drop the voltage entirely and treat that wire as carrying a value from the two-element set $\{0,1\}$, full stop.

No 6.042 material is assumed; although 6.042 (not yet written) covers propositional logic formally and it overlaps with this note's subject matter, everything below is derived from scratch and does not depend on it.

## Core definitions

- **Bit (Boolean value)** — an element of the two-element set $\{0,1\}$; the abstract, voltage-free version of a digital-abstraction logic level.
- **Boolean variable** — a symbol (e.g. $A$, $B$, $X$) standing for an unspecified bit.
- **Logic function (Boolean function) of $n$ variables** — a function $f:\{0,1\}^n \to \{0,1\}$: it assigns exactly one output bit to every possible combination of $n$ input bits.
- **Truth table** — an explicit table listing a logic function's output for every one of its $2^n$ input combinations (rows), for $n$ input variables.
- **AND ($X\cdot Y$, or $XY$)** — the logic function of two variables defined by: output is $1$ if and only if both $X=1$ and $Y=1$; otherwise $0$.
- **OR ($X+Y$)** — the logic function of two variables defined by: output is $1$ if and only if at least one of $X=1$ or $Y=1$; otherwise $0$.
- **NOT ($\bar X$, also written $X'$)** — the logic function of one variable defined by: output is $1$ if $X=0$, and $0$ if $X=1$ (it flips the bit).
- **Literal** — a variable or its complement (e.g. $A$ and $\bar A$ are both literals).
- **Boolean expression** — a symbol string built from variables, the constants $0$ and $1$, and the operators AND/OR/NOT, combined with the precedence NOT $>$ AND $>$ OR (highest-binding first) unless parentheses override it — e.g. $A\bar B + C$ means $(A\cdot\bar B)+C$, not $A\cdot(\bar B+C)$.
- **Logically equivalent expressions** — two Boolean expressions that produce identical output for every input combination, i.e. the same truth table (they compute the same logic function), written with $=$.
- **Perfect induction** — the proof method of establishing that two Boolean expressions are equivalent by checking *all* $2^n$ rows of their truth tables and confirming they match on every row. Valid because $\{0,1\}^n$ is finite — "checking every case" really is a complete proof here, unlike in most of ordinary algebra where the domain is infinite.
- **Boolean algebra (as an algebraic system)** — the set $\{0,1\}$ together with the operators AND, OR, NOT, satisfying the postulates below (Huntington's postulates), which license proving equivalences by symbolic manipulation instead of perfect induction.
- **Duality principle** — every valid Boolean law remains valid if you simultaneously swap every $+ \leftrightarrow \cdot$ and every $0 \leftrightarrow 1$ throughout it. (Justified below, once the postulates are stated: the postulates themselves come in dual pairs.)

## Intuition

Ordinary algebra (over the real numbers) has an infinite domain, so you can never check "is this identity true" by trying every possible value of $x$ — you're stuck proving it structurally. Boolean algebra is the opposite extreme: its domain, $\{0,1\}$, is so small that for a claim about $n$ variables, there are only $2^n$ possible input combinations, period. That means every Boolean claim *can* be settled by brute-force checking (perfect induction). So why bother with algebraic laws at all, if you can always just check the table?

Two reasons. First, $2^n$ grows fast — by $n=10$ variables that's 1024 rows, and real digital designs routinely have far more inputs than that; nobody hand-checks a thousand-row table. Second, and more important for engineering: the *algebraic* laws are what let you take a Boolean expression that came out of a design process looking complicated, and mechanically simplify it into one with fewer literals — which, once you connect expressions to gates (a later topic), directly means fewer physical gates, less silicon area, and less propagation delay. Perfect induction can tell you *whether* two expressions match; only algebraic manipulation tells you *how to get from one to a smaller one*.

One more thing worth flagging before the notation trips you up: AND and OR are written with the arithmetic symbols $\cdot$ and $+$ on purpose, because they behave *similarly* to multiplication and addition in many ways (both are commutative, associative, and one distributes over the other) — but $\{0,1\}$ under these operations is **not** ordinary arithmetic. $1+1=1$ here, not $2$ — there is no symbol "$2$" in this algebra at all. Every time a Boolean law looks like it "shouldn't" be true because it clashes with real-number intuition (e.g. OR distributing over AND, derived below), that clash is a sign you're leaking arithmetic intuition into a different algebraic system — trust the postulates and perfect induction, not the resemblance to arithmetic.

## Derivation / formalism

### 1. The postulates (Huntington's postulates for a Boolean algebra)

Take $B=\{0,1\}$ with operators $+$ (OR) and $\cdot$ (AND) and unary $\ \overline{\ \cdot\ }\ $ (NOT). The following are taken as given (postulates, not derived from anything more basic — they are simply asserted to hold, and verified to match the truth-table definitions above by direct inspection):

1. **Closure** — for all $x,y\in B$: $x+y \in B$ and $x\cdot y \in B$. (True by the truth-table definitions: AND/OR of two bits is always a bit.)
2. **Identity elements** — there exist $0,1\in B$ such that for all $x\in B$: $x+0=x$ and $x\cdot 1 = x$.
3. **Commutativity** — for all $x,y\in B$: $x+y=y+x$ and $x\cdot y = y\cdot x$.
4. **Distributivity** — for all $x,y,z\in B$: $x\cdot(y+z) = x\cdot y + x\cdot z$ (AND over OR), **and** $x+(y\cdot z) = (x+y)\cdot(x+z)$ (OR over AND — this second one has no counterpart in ordinary arithmetic; verified by perfect induction in Worked Example 2 below).
5. **Complement** — for every $x\in B$ there exists $\bar x \in B$ such that $x+\bar x = 1$ and $x\cdot \bar x = 0$.

Notice postulates 2, 3, 4, 5 each come as a **pair** of statements, one with $+/0$ and one with $\cdot/1$ swapped in. That pairing is exactly why the duality principle (stated in Core definitions) holds: any proof built only from these postulates can be "mirrored" by swapping $+\leftrightarrow\cdot$ and $0\leftrightarrow1$ in every line, and the mirrored proof is equally valid because the postulates it rests on are themselves mirror pairs.

### 2. Deriving further laws from the postulates

The postulates alone are enough to prove every other Boolean identity. Two derivations, worked in full:

**Dominance/null law for OR: $x+1=1$.**
$$x+1 = (x+1)\cdot 1 \qquad \text{[identity: } y\cdot1=y \text{, applied with } y=(x+1)\text{]}$$
$$= (x+1)\cdot(x+\bar x) \qquad \text{[complement: } 1 = x+\bar x \text{]}$$
$$= x + (1\cdot\bar x) \qquad \text{[distributive, OR-over-AND: } (x+1)(x+\bar x) = x+(1\cdot\bar x)\text{]}$$
$$= x + \bar x \qquad \text{[identity: } 1\cdot\bar x = \bar x \text{]}$$
$$= 1 \qquad \text{[complement]}$$

By the duality principle, swapping $+\leftrightarrow\cdot$ and $0\leftrightarrow 1$ throughout immediately gives the dual law $x\cdot 0 = 0$ (dominance for AND) with no separate proof needed.

**Idempotent law: $x+x=x$.**
$$x+x = (x+x)\cdot 1 \qquad \text{[identity]}$$
$$= (x+x)\cdot(x+\bar x) \qquad \text{[complement: } 1=x+\bar x\text{]}$$
$$= x + (x\cdot\bar x) \qquad \text{[distributive, OR-over-AND]}$$
$$= x + 0 \qquad \text{[complement: } x\bar x = 0\text{]}$$
$$= x \qquad \text{[identity]}$$

By duality, $x\cdot x = x$ follows immediately (and is also proved directly in Self-check Question 3).

**Absorption law: $x + xy = x$.**
$$x+xy = x\cdot 1 + x\cdot y \qquad \text{[identity: } x = x\cdot1\text{, applied to the first term]}$$
$$= x\cdot(1+y) \qquad \text{[distributive, AND-over-OR, reversed: } x\cdot1+x\cdot y = x(1+y)\text{]}$$
$$= x\cdot 1 \qquad \text{[dominance for OR, proved above: } 1+y=1\text{]}$$
$$= x \qquad \text{[identity]}$$

### 3. De Morgan's laws, proved by perfect induction

$$\overline{X+Y} = \bar X \cdot \bar Y \qquad \text{and} \qquad \overline{X\cdot Y} = \bar X + \bar Y$$

Unlike the laws above, De Morgan's laws are proved directly by perfect induction rather than from the postulates in one line, since they involve the complement operator applied to a compound expression — checking all $2^2=4$ rows is simplest and fully rigorous here (the domain is finite, so this is a complete proof, not a spot-check):

| $X$ | $Y$ | $X+Y$ | $\overline{X+Y}$ | $\bar X$ | $\bar Y$ | $\bar X\cdot\bar Y$ |
|---|---|---|---|---|---|---|
| 0 | 0 | 0 | 1 | 1 | 1 | 1 |
| 0 | 1 | 1 | 0 | 1 | 0 | 0 |
| 1 | 0 | 1 | 0 | 0 | 1 | 0 |
| 1 | 1 | 1 | 0 | 0 | 0 | 0 |

Columns $\overline{X+Y}$ and $\bar X\cdot\bar Y$ match on all 4 rows, so the two expressions compute the same function — the law is proved. (The second law, $\overline{XY}=\bar X+\bar Y$, is left as Self-check Question 7, which asks you to disprove the *wrong* pairing $\overline{XY}=\bar X\bar Y$ instead, using the same method.) General De Morgan's for more than two variables (e.g. $\overline{A+B+C}$) follows by applying the two-variable law repeatedly — worked in Self-check Question 5.

## Worked examples

### Example 1 — Absorption law, both proof methods side by side

Claim: $x + xy = x$ for all $x,y \in \{0,1\}$.

**Algebraic proof:** given in full in Derivation §2 above (four steps: identity, distributive, dominance, identity).

**Perfect-induction check (all 4 rows):**

| $x$ | $y$ | $xy$ | $x+xy$ |
|---|---|---|---|
| 0 | 0 | 0 | 0 |
| 0 | 1 | 0 | 0 |
| 1 | 0 | 0 | 1 |
| 1 | 1 | 1 | 1 |

Column $x+xy$ exactly matches column $x$ on every row ($0,0,1,1$ vs. $x$ values $0,0,1,1$), confirming the algebraic proof. This is the general pattern for verifying a claimed simplification: the algebraic proof shows *why* it's true and generalizes to any number of variables; the truth table is a fast, mechanical double-check for small cases.

### Example 2 — Simplifying $F = AB + A\bar B + \bar A B$ down to $A+B$

**Step 1 — Group the first two terms and factor:**
$$F = AB + A\bar B + \bar A B = A(B+\bar B) + \bar A B \qquad \text{[distributive, AND-over-OR, reversed]}$$

**Step 2 — Apply the complement law $B+\bar B = 1$:**
$$F = A\cdot 1 + \bar A B = A + \bar A B \qquad \text{[identity: } A\cdot1=A\text{]}$$

**Step 3 — Apply the identity $A = A\cdot 1$ to expand for a second factoring:**
$$F = A\cdot 1 + \bar A \cdot B$$
Now use distributivity in the OR-over-AND direction on the pair $(A+\bar A)$ and $(A+B)$ — first confirm this is the right target by expanding $(A+\bar A)(A+B)$ using AND-over-OR distributivity twice:
$$(A+\bar A)(A+B) = (A+\bar A)A + (A+\bar A)B = (A\cdot A + \bar A\cdot A) + (A\cdot B+\bar A\cdot B)$$
Using $A\cdot A = A$ (idempotent) and $\bar A\cdot A = 0$ (complement) on the first parenthesis, and leaving the second alone:
$$= (A + 0) + (AB+\bar A B) = A + AB + \bar A B$$
Using absorption ($A+AB=A$) on the first two terms:
$$= A + \bar A B$$
This confirms $(A+\bar A)(A+B) = A+\bar A B$ — exactly the expression from Step 2. So:
$$F = A + \bar A B = (A+\bar A)(A+B) = 1\cdot(A+B) = A+B \qquad \text{[complement, then identity]}$$

**Verification by perfect induction (all 4 rows of $A,B$):**

| $A$ | $B$ | $AB$ | $A\bar B$ | $\bar A B$ | $F=AB+A\bar B+\bar A B$ | $A+B$ |
|---|---|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| 0 | 1 | 0 | 0 | 1 | 1 | 1 |
| 1 | 0 | 0 | 1 | 0 | 1 | 1 |
| 1 | 1 | 1 | 0 | 0 | 1 | 1 |

$F$ matches $A+B$ on all four rows: confirmed. Notice how much shorter perfect induction was here than the algebraic derivation — that's expected for only two variables; the algebraic method's payoff grows precisely when $n$ gets too large to tabulate, or when you need to understand *why* a simplification exists rather than merely that it does.

## Common pitfalls

- **Reading $+$ as arithmetic addition.** $1+1=1$ in Boolean algebra, not $2$ — there is no "$2$" in $\{0,1\}$. Every time an identity looks "wrong" by arithmetic intuition, that's a sign you're accidentally importing real-number arithmetic into an algebra that only shares notation with it, not behavior.
- **Forgetting OR distributes over AND too.** Ordinary arithmetic only has one distributive law (multiplication over addition); Boolean algebra has both directions as separate postulates. Assuming $x+(yz)=(x+y)(x+z)$ "can't be true because addition doesn't distribute over multiplication in normal math" is a direct instance of leaking arithmetic intuition (see Self-check Question 8).
- **Complement-scope ambiguity.** $\overline{A+B}$ (NOT of the whole sum) and $\bar A + B$ (NOT of $A$ alone, ORed with $B$) are different expressions with different truth tables — the overline's scope is exactly what's directly underneath it; missing or misreading a bar's width is a common transcription error.
- **Misapplying De Morgan's by complementing the operator but not the literals (or vice versa).** $\overline{AB}$ is $\bar A + \bar B$ — both the operator flips (AND$\to$OR) *and* every literal underneath gets complemented. Writing $\overline{AB}=\bar A\bar B$ (flipping only the operator) or $\overline{AB}=A+B$ (flipping only the literals) are both common wrong answers.
- **Treating a spot-check (a few rows) as a proof.** Perfect induction requires *all* $2^n$ rows. Checking 2 out of 4 rows and declaring an identity "verified" is not a proof — the remaining rows might disagree (this is exactly the mistake Self-check Question 7 is designed to catch).
- **Assuming there is one unique "most simplified" form of an expression.** Different sequences of algebraic manipulation can lead to different-looking but equally short final expressions; "the" minimal form is only well-defined once you fix a cost metric (e.g. fewest literals, fewest gates) — the systematic tool for finding a guaranteed-minimal form (Karnaugh maps) is the subject of the next topic.

## Self-check

### Questions

1. (Easy) Write out the full truth table for $\bar A + B$ (4 rows).
2. (Easy) Evaluate the Boolean expression $1\cdot 0 + \overline{1}$ (i.e., reduce it to a single bit, showing which postulate/definition you used at each step).
3. (Medium) Prove $x\cdot x = x$ (idempotent law for AND) directly from the postulates, mirroring the OR-idempotent proof in Derivation §2 (don't just invoke duality — write out the actual steps).
4. (Medium) Simplify $XY+X\bar Y$ using the algebraic laws (show each step and which law justifies it).
5. (Medium) Using the two-variable De Morgan's law twice, derive the three-variable version: express $\overline{A+B+C}$ purely in terms of $\bar A,\bar B,\bar C$.
6. (Hard) Prove the dual absorption law $x\cdot(x+y)=x$, either (a) by direct algebraic manipulation from the postulates, or (b) by applying the duality principle to the absorption law proved in Derivation §2 — do both and confirm they agree.
7. (Hard) Show, using perfect induction, that $\overline{X\cdot Y} \ne \bar X\cdot\bar Y$ in general — find at least one row of the truth table where they disagree, and state what the correct De Morgan's pairing for $\overline{XY}$ actually is.
8. (Hard) A friend says the law $A+BC=(A+B)(A+C)$ "can't be a real algebraic law, because in ordinary arithmetic addition never distributes over multiplication like that." Settle the question with a perfect-induction table (all 8 rows of $A,B,C$), and explain in one or two sentences why the resemblance to arithmetic notation is misleading here.

### Answers

1. $A=0,B=0$: $\bar A=1$, so $\bar A+B=1$. $A=0,B=1$: $\bar A+B=1+1=1$. $A=1,B=0$: $\bar A=0$, so $\bar A+B=0+0=0$. $A=1,B=1$: $\bar A+B=0+1=1$. Table: $(0,0)\to1$, $(0,1)\to1$, $(1,0)\to0$, $(1,1)\to1$.
2. $1\cdot 0 = 0$ [AND truth-table definition: output is 1 only if both inputs are 1; here one input is 0]. $\overline{1}=0$ [NOT truth-table definition]. So the expression becomes $0+0$. $0+0=0$ [identity postulate, $x+0=x$ with $x=0$, or directly from the OR truth table]. Final value: $0$.
3. $x\cdot x = x\cdot x + 0$ [identity: $y=y+0$, with $y=x\cdot x$] $= x\cdot x + x\bar x$ [complement: $0=x\bar x$] $= x\cdot(x+\bar x)$ [distributive, AND-over-OR, reversed] $= x\cdot 1$ [complement: $x+\bar x=1$] $= x$ [identity]. Matches the OR-idempotent proof line-for-line with $+\leftrightarrow\cdot$ and $0\leftrightarrow1$ swapped, confirming duality.
4. $XY+X\bar Y = X(Y+\bar Y)$ [distributive, AND-over-OR, reversed] $=X\cdot 1$ [complement: $Y+\bar Y=1$] $=X$ [identity].
5. $\overline{A+B+C} = \overline{(A+B)+C} = \overline{A+B}\cdot\bar C$ [two-variable De Morgan's, treating $(A+B)$ as one variable and $C$ as the other] $= (\bar A\cdot\bar B)\cdot \bar C$ [two-variable De Morgan's again, applied to $\overline{A+B}$] $=\bar A\bar B\bar C$.
6. (a) Direct: $x(x+y) = x\cdot x + x\cdot y$ [distributive, AND-over-OR] $= x+xy$ [idempotent: $x\cdot x=x$] $=x$ [absorption, proved in Derivation §2]. (b) Duality: the absorption law proved in the note is $x+xy=x$; swapping $+\leftrightarrow\cdot$ throughout gives $x\cdot(x+y)=x$ — wait, carefully: swapping every $+$ and $\cdot$ in "$x+xy=x$" gives "$x\cdot(x+y)=x$" (the juxtaposition $xy$, meaning $x\cdot y$, becomes $x+y$, and the outer $+$ becomes $\cdot$). This is exactly the target law. Both methods agree: $x(x+y)=x$.
7. Take $X=0, Y=1$: $XY = 0\cdot1=0$, so $\overline{XY}=\overline{0}=1$. Meanwhile $\bar X\cdot\bar Y = \overline{0}\cdot\overline{1} = 1\cdot 0 = 0$. Since $1\ne 0$, the two expressions disagree at this row, so $\overline{XY}\ne\bar X\bar Y$ in general. The correct pairing (Derivation §3) is $\overline{XY}=\bar X+\bar Y$; checking the same row: $\bar X+\bar Y = 1+0=1$, which does match $\overline{XY}=1$.
8. Full table:

   | $A$ | $B$ | $C$ | $BC$ | $A+BC$ | $A+B$ | $A+C$ | $(A+B)(A+C)$ |
   |---|---|---|---|---|---|---|---|
   | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
   | 0 | 0 | 1 | 0 | 0 | 0 | 1 | 0 |
   | 0 | 1 | 0 | 0 | 0 | 1 | 0 | 0 |
   | 0 | 1 | 1 | 1 | 1 | 1 | 1 | 1 |
   | 1 | 0 | 0 | 0 | 1 | 1 | 1 | 1 |
   | 1 | 0 | 1 | 0 | 1 | 1 | 1 | 1 |
   | 1 | 1 | 0 | 0 | 1 | 1 | 1 | 1 |
   | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 |

   Columns $A+BC$ and $(A+B)(A+C)$ match on all 8 rows, so the law holds — it is in fact postulate 4 (distributivity) itself, not a derived theorem, so this table is a confirmation of a postulate rather than a proof of a new fact. The friend's intuition fails because $+$ and $\cdot$ on $\{0,1\}$ are far more symmetric with each other than real-number addition and multiplication are (that symmetry is exactly the content of the duality principle); Boolean algebra's notation borrows arithmetic symbols, but the two systems are different algebraic structures, and only perfect induction (or the postulates) — not arithmetic experience — settles what's true here.

## Summary / cheat sheet

**Primitive operators (truth-table definitions):** $X\cdot Y=1$ iff both are 1; $X+Y=1$ iff at least one is 1; $\bar X$ flips $X$.

**Huntington's postulates:** identity ($x+0=x$, $x\cdot1=x$), commutativity, distributivity **both directions** ($x(y+z)=xy+xz$ **and** $x+yz=(x+y)(x+z)$), complement ($x+\bar x=1$, $x\bar x=0$).

**Derived laws (proved in this note):** dominance ($x+1=1$, $x\cdot0=0$), idempotent ($x+x=x$, $x\cdot x=x$), absorption ($x+xy=x$, $x(x+y)=x$).

**De Morgan's laws (proved by perfect induction):** $\overline{X+Y}=\bar X\bar Y$; $\overline{XY}=\bar X+\bar Y$. Generalizes to $n$ variables by repeated application.

**Duality principle:** any valid law stays valid under simultaneous $+\leftrightarrow\cdot$, $0\leftrightarrow1$ — because the postulates come in such mirror pairs.

**Two proof methods:** perfect induction (check all $2^n$ truth-table rows — always valid, but doesn't scale) vs. algebraic manipulation from the postulates (scales to any $n$, and shows *why* a law holds, not just *that* it holds).

## Used later in
(none yet)
