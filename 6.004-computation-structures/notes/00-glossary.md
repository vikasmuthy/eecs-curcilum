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

## From [[6.004-computation-structures/notes/02-canonical-forms-simplification]]

- **Minterm ($m_i$)** — a product term with every variable exactly once, uncomplemented iff its bit in index $i$ is 1; $1$ at exactly one row.
- **Maxterm ($M_i$)** — a sum term with every variable exactly once, uncomplemented iff its bit in index $i$ is 0 (opposite of minterm convention); $0$ at exactly one row; $M_i=\overline{m_i}$.
- **Canonical SOP (minterm expansion)** — $F=\Sigma m(\text{indices where }F{=}1)$; direct translation of a truth table into an OR of minterms.
- **Canonical POS (maxterm expansion)** — $F=\Pi M(\text{indices where }F{=}0)$; direct translation into an AND of maxterms.
- **(General) sum-of-products / product-of-sums** — OR of AND terms / AND of OR terms, not necessarily full minterms/maxterms.
- **Karnaugh map (K-map)** — grid of truth-table rows in Gray-code order so grid-adjacent cells (with wraparound) differ in exactly one variable.
- **Gray code** — ordering of binary numbers where consecutive entries differ in exactly one bit.
- **Implicant** — a product term that is 1 only where $F$ is 1 (never covers a 0-row).
- **Prime implicant** — an implicant that can't be enlarged (no literal droppable) while remaining valid; largest valid K-map group containing a given cell.
- **Essential prime implicant** — the only prime implicant covering some particular minterm; must appear in every minimal SOP.
- **Don't-care ($d$/$\times$)** — an input row with unspecified required output; assignable to 0 or 1 freely during minimization.
