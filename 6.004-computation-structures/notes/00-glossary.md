# 6.004 Glossary

Running list of terms/symbols introduced in 6.004 notes, in order of first appearance.

## From [[6.004-computation-structures/notes/01-boolean-algebra-logic-functions]]

- **Bit (Boolean value)** — an element of $\{0,1\}$; the abstract, voltage-free version of a digital-abstraction logic level.
- **Boolean variable** — a symbol standing for an unspecified bit.
- **Logic function (Boolean function)** — a function $f:\{0,1\}^n\to\{0,1\}$.
- **Truth table** — explicit tabulation of a logic function's output over all $2^n$ input combinations.
- **AND ($X\cdot Y$)** — outputs 1 iff both inputs are 1.
- **OR ($X+Y$)** — outputs 1 iff at least one input is 1.
- **NOT ($\bar X$ / $X'$)** — flips the bit.
- **Literal** — a variable or its complement.
- **Boolean expression** — built from variables, $0$/$1$, AND/OR/NOT; precedence NOT $>$ AND $>$ OR.
- **Logically equivalent expressions** — identical truth tables (same function), written with $=$.
- **Perfect induction** — proof by checking all $2^n$ truth-table rows; valid because the domain is finite.
- **Boolean algebra (Huntington's postulates)** — $\{0,1\}$ with AND/OR/NOT satisfying closure, identity, commutativity, distributivity (both directions), and complement.
- **Duality principle** — any valid law stays valid under simultaneous $+\leftrightarrow\cdot$, $0\leftrightarrow1$, because the postulates come in mirror pairs.
- **Dominance/null law** — $x+1=1$, $x\cdot0=0$.
- **Idempotent law** — $x+x=x$, $x\cdot x=x$.
- **Absorption law** — $x+xy=x$, $x(x+y)=x$.
- **De Morgan's laws** — $\overline{X+Y}=\bar X\bar Y$; $\overline{XY}=\bar X+\bar Y$.
