---
title: "Resistive Networks, Node/Mesh Analysis"
course: "6.002"
topic_number: 02
prerequisites: ["6.002 Topic 01 — Lumped circuit abstraction, KVL/KCL"]
status: not started
---

# Resistive Networks, Node/Mesh Analysis

## Why this matters

A circuit with more than two or three resistors and sources quickly becomes too tangled to solve by ad-hoc series/parallel combination and repeated substitution of Ohm's law. Node analysis and mesh analysis are the two systematic bookkeeping procedures that turn "stare at the circuit and guess a solution method" into "write down N linear equations in N unknowns and solve." Every later 6.002 topic — dependent sources, Thevenin/Norton equivalents, amplifier analysis, transient RC/RL circuits — reduces, at some intermediate step, to solving a resistive (or resistive-plus-one-reactive-element) network. If you cannot mechanically set up and solve node/mesh equations without error, every subsequent topic becomes unreliable, because you'll be doing this same bookkeeping under more time pressure and with more elements in play. Skipping this topic means falling back to intuition-only circuit solving, which does not scale past two-resistor circuits.

## Builds on

- [[6.002-circuits-and-electronics/notes/01-lumped-circuit-abstraction-kvl-kcl]] — Kirchhoff's Voltage Law (KVL) and Kirchhoff's Current Law (KCL), the lumped-element abstraction, and the definitions of **node**, **branch**, and **loop**. This note reuses those definitions directly and builds node/mesh analysis on top of KVL/KCL rather than re-deriving them.

This note relies on the following results from Topic 01 without re-deriving them: **KCL** (sum of currents leaving any node is zero, from charge conservation), **KVL** (sum of voltage drops around any closed loop is zero, from the conservative electric field in the lumped abstraction), **Ohm's law** ($v = iR$ for a resistor), and the definitions of **node**, **branch**, and **loop**. See [[6.002-circuits-and-electronics/notes/01-lumped-circuit-abstraction-kvl-kcl]] for the full derivations and the self-check confirming mastery of these before proceeding.

## Core definitions

New terms introduced in this note (node and branch are already defined in [[6.002-circuits-and-electronics/notes/01-lumped-circuit-abstraction-kvl-kcl]] and are used here unchanged):

- **Reference node (ground)** — one node in the circuit chosen, arbitrarily, to have a defined voltage of $0\text{ V}$. All other node voltages are measured relative to it. The choice of reference node does not affect any current or voltage *difference* in the circuit — it's bookkeeping, not physics.
- **Node voltage** $v_n$ — the voltage of node $n$ relative to the reference node. There is one node voltage unknown per non-reference node.
- **Mesh** — a loop (as defined in Topic 01) that does not enclose any other loop (a "window pane" of the circuit as drawn). Meshes only exist as a clean concept for *planar* circuits (circuits that can be drawn on a flat page with no branch crossing another branch).
- **Mesh current** $i_m$ — a fictitious current assumed to circulate around mesh $m$ (by convention, clockwise). The actual current through any branch shared by two meshes is the (signed) sum of the mesh currents that pass through it.
- **Node analysis (nodal analysis)** — a systematic method that takes node voltages as the unknowns, writes one KCL equation per non-reference node, and solves the resulting linear system.
- **Mesh analysis (loop analysis)** — a systematic method that takes mesh currents as the unknowns, writes one KVL equation per independent mesh, and solves the resulting linear system.
- **Independent source** — a voltage or current source whose value is fixed (does not depend on any other voltage/current in the circuit). This note only uses independent sources; dependent sources are introduced in the next topic.
- **Supernode** — a region formed by merging two non-reference nodes joined by a floating (non-reference-connected) voltage source, used to write a single combined KCL equation when the current through that source is not directly known.

## Intuition

Every circuit-solving problem has some number of unknown voltages and currents, and you need exactly that many independent equations to pin them all down. KCL and KVL are the only two structural facts you get for free from the wiring diagram — Ohm's law is the only fact you get for free from each individual resistor. The problem with solving circuits "by inspection" (combining series/parallel resistors, substituting one relation into another) is that it does not generalize: the moment the network is not reducible by series/parallel tricks (e.g., a bridge-like circuit), ad-hoc methods run out of tricks.

Node analysis flips the problem around: instead of solving for every branch current individually, it observes that if you know the voltage at *every node*, then Ohm's law immediately tells you the current through every resistor (from the node-voltage difference across it), and KCL then becomes one linear equation per node relating those node voltages. Since there are usually far fewer nodes than branches, this is a smaller system to solve than "list every branch current as an unknown and write KCL and KVL everywhere."

Mesh analysis flips it the other way: instead of tracking every node voltage, it assumes a fictitious circulating current in each mesh. Any real branch current is then some combination of one or two mesh currents (one if the branch borders only one mesh from the outside of the network, two if it's shared between two meshes). KVL around each mesh, combined with Ohm's law, gives one equation per mesh. This works only for planar circuits, because "which meshes share this branch" is only well-defined when the circuit can be drawn flat without crossings.

Which one you should use is a practical choice: node analysis is preferred when there are few nodes (or many voltage sources, which simplify the equations), mesh analysis when there are few meshes (or many current sources). Both methods, done correctly, must give the same final branch currents and voltages — they're just two different choices of unknowns for the same physics.

## Derivation / formalism

### Setup and general procedure — Node analysis

Consider a resistive circuit with $N+1$ nodes total. Pick one node as the reference (ground, $v = 0$). This leaves $N$ unknown node voltages $v_1, v_2, \dots, v_N$.

**Step 1 — Assign node voltages.** Label every non-reference node with an unknown voltage $v_k$, measured relative to the reference node.

**Step 2 — Express every branch current via Ohm's law in terms of node voltages.** For a resistor of resistance $R$ connecting node $a$ (voltage $v_a$) to node $b$ (voltage $v_b$), the current flowing from node $a$ to node $b$ through that resistor is
$$i_{a \to b} = \frac{v_a - v_b}{R}$$
This follows directly from Ohm's law $v = iR$: the voltage drop across the resistor in the direction $a \to b$ is $v_a - v_b$, so the current in that direction is that drop divided by $R$.

**Step 3 — Write one KCL equation per non-reference node.** At each non-reference node $k$, sum the currents leaving the node through every branch and set the sum equal to zero (equivalently: sum of currents leaving = sum of currents entering, or "leaving = 0" if you fold sources in as signed terms):
$$\sum_{\text{branches at } k} \left(\text{current leaving node } k \text{ through that branch}\right) = 0$$
For resistor branches, substitute the Ohm's-law expression from Step 2. For a branch that is an independent current source of value $I_s$ pushing current *into* node $k$, that branch contributes a constant $-I_s$ to the "current leaving" sum (i.e. it supplies $I_s$ into the node, so it's $-I_s$ leaving).

**Step 4 — Handle voltage sources.** An independent voltage source directly sets the voltage *difference* between the two nodes it connects. If one terminal of the source is the reference node, the other node's voltage is simply fixed at the source value — no KCL equation is needed for the source's own value, but that node's unknown is now known and substituted into the *other* KCL equations. If the voltage source connects two non-reference nodes (a "floating" source), it constrains the relation between the two node voltages but doesn't by itself tell you the current through it; the standard technique is the **supernode**: combine the two nodes into one region, write a single KCL equation for the combined region (summing all currents leaving the region through resistors, treating the internal source current as unknown but self-cancelling since it leaves one node and enters the other inside the region), and add the constraint equation $v_a - v_b = V_s$ separately.

**Step 5 — Solve the resulting linear system.** You now have exactly $N$ independent linear equations (one KCL per non-reference node, with substitutions/supernode handling for voltage sources) in $N$ unknown node voltages. Solve by substitution, elimination, or matrix methods (Gaussian elimination / Cramer's rule).

**Step 6 — Back-substitute.** Once node voltages are known, every branch current follows immediately from Step 2's Ohm's-law expressions.

**Why this always produces enough independent equations:** a connected circuit with $N+1$ nodes and $B$ branches has, by the structure of KCL, exactly $N$ independent KCL equations (the KCL equation at the reference node is automatically implied by the other $N$, since total current into the whole circuit must be zero — this is a consequence of charge conservation applied to the entire circuit as one region). So writing KCL at all $N$ non-reference nodes gives exactly the right number of independent equations to solve for the $N$ unknown node voltages, provided every node voltage actually appears in at least one equation (true for any connected circuit).

### Setup and general procedure — Mesh analysis

Consider a *planar* resistive circuit with $B$ branches and $N+1$ nodes. The number of independent meshes is $M = B - N$ (this is a standard graph-theory fact: independent loops = branches − nodes + 1, and for a planar graph the mesh count equals this independent-loop count when meshes are taken as the "window panes" of the drawing).

**Step 1 — Identify meshes and assign mesh currents.** Draw the circuit planar (no branch crossings). Identify each window-pane mesh and assign it a circulating current $i_m$, by convention flowing clockwise.

**Step 2 — Express the actual current through each branch in terms of mesh currents.** A branch on the outer boundary of the circuit (bordering only one mesh) carries exactly that mesh's current, in the direction of that mesh's circulation. A branch shared between two adjacent meshes $m$ and $n$ carries the *difference* of the two mesh currents, e.g. $i_m - i_n$ in the direction associated with mesh $m$'s clockwise convention, because the two circulating currents pass through that shared branch in opposite physical directions.

**Step 3 — Write one KVL equation per mesh.** Traverse each mesh in the clockwise direction and sum voltage drops, setting the sum to zero. For a resistor $R$ carried by net current $i$ (found via Step 2) in the traversal direction, the drop in the traversal direction is $iR$ (Ohm's law again). For an independent voltage source in the loop, its contribution is $+V_s$ or $-V_s$ depending on whether the traversal direction moves from $-$ to $+$ terminal (a drop, so $+V_s$ is encountered as you go from $-$ to $+$... concretely: if traversal enters the $+$ terminal first, that source is a *rise* in the traversal direction, contributing $-V_s$ to the sum-of-drops equation; if it enters the $-$ terminal first, it's a drop, contributing $+V_s$). Be consistent: pick one sign rule and apply it uniformly.

**Step 4 — Handle current sources.** An independent current source directly fixes the difference between two mesh currents that share that branch (since the source dictates the net current through that branch regardless of the resistors around it): if current source $I_s$ is on the branch shared by meshes $m$ and $n$, then $i_m - i_n = \pm I_s$ (sign depending on source orientation vs. assumed mesh circulation directions). This replaces the KVL equation for one of the two meshes; the current source constraint is used instead. If the current source is on an outer-boundary branch (only one mesh touches it), it directly sets that mesh current: $i_m = \pm I_s$, and no KVL equation is needed for that mesh at all.

**Step 5 — Solve the resulting linear system** of $M$ equations in $M$ unknown mesh currents.

**Step 6 — Back-substitute** to get every branch current (via Step 2) and then every branch voltage (via Ohm's law).

### Equivalence of the two methods

Both methods are different bases for describing the same $B$-dimensional space of branch currents/voltages, subject to the same KCL and KVL constraints; each method just picks a minimal independent set of unknowns ($N$ node voltages, or $M = B-N$ mesh currents) from which every branch quantity is recoverable, and each produces exactly enough independent linear equations to solve for its unknowns. This is why both, applied correctly to the same circuit, always agree on the final branch currents and voltages.

## Worked examples

### Example 1 — Three-resistor node analysis with one current source

**Circuit:** Reference node (ground) at the bottom rail. Node 1 (voltage $v_1$) is connected to node 2 (voltage $v_2$) is connected to ground, forming the following branches:
- A current source of $I_s = 3\text{ A}$ injects current from ground into node 1.
- A resistor $R_1 = 2\ \Omega$ connects node 1 to ground.
- A resistor $R_2 = 4\ \Omega$ connects node 1 to node 2.
- A resistor $R_3 = 6\ \Omega$ connects node 2 to ground.

**Step 1 — Unknowns:** $v_1$, $v_2$ (ground $= 0\text{ V}$).

**Step 2/3 — KCL at node 1** (sum of currents *leaving* node 1 = 0; the current source injecting $3\text{ A}$ into node 1 counts as $-3\text{ A}$ leaving):
$$-3 + \frac{v_1 - 0}{R_1} + \frac{v_1 - v_2}{R_2} = 0$$
$$-3 + \frac{v_1}{2} + \frac{v_1 - v_2}{4} = 0$$

Multiply through by 4 to clear denominators:
$$-12 + 2v_1 + (v_1 - v_2) = 0$$
$$3v_1 - v_2 = 12 \quad \text{(Equation A)}$$

**KCL at node 2** (currents leaving node 2, through $R_2$ back toward node 1, and through $R_3$ to ground):
$$\frac{v_2 - v_1}{R_2} + \frac{v_2 - 0}{R_3} = 0$$
$$\frac{v_2 - v_1}{4} + \frac{v_2}{6} = 0$$

Multiply through by 12 (LCM of 4 and 6):
$$3(v_2 - v_1) + 2v_2 = 0$$
$$3v_2 - 3v_1 + 2v_2 = 0$$
$$-3v_1 + 5v_2 = 0 \quad \text{(Equation B)}$$

**Step 5 — Solve.** From Equation B: $v_2 = \dfrac{3v_1}{5}$.

Substitute into Equation A:
$$3v_1 - \frac{3v_1}{5} = 12$$
$$\frac{15v_1 - 3v_1}{5} = 12$$
$$\frac{12v_1}{5} = 12$$
$$v_1 = 5\text{ V}$$

Then $v_2 = \dfrac{3(5)}{5} = 3\text{ V}$.

**Step 6 — Back-substitute for branch currents:**
- Through $R_1$: $i_{R_1} = \dfrac{v_1}{R_1} = \dfrac{5\text{ V}}{2\ \Omega} = 2.5\text{ A}$ (flowing from node 1 to ground).
- Through $R_2$: $i_{R_2} = \dfrac{v_1 - v_2}{R_2} = \dfrac{5 - 3}{4} = \dfrac{2}{4} = 0.5\text{ A}$ (flowing from node 1 to node 2).
- Through $R_3$: $i_{R_3} = \dfrac{v_2}{R_3} = \dfrac{3\text{ V}}{6\ \Omega} = 0.5\text{ A}$ (flowing from node 2 to ground).

**Check (KCL at node 1):** current in from source $= 3\text{ A}$; current out $= i_{R_1} + i_{R_2} = 2.5 + 0.5 = 3\text{ A}$. Balances. **Check (node 2):** in from $R_2$ = $0.5\text{ A}$; out through $R_3 = 0.5\text{ A}$. Balances.

### Example 2 — Two-mesh circuit with one voltage source

**Circuit (planar, two meshes):** A $10\text{ V}$ independent voltage source (with $+$ terminal on the left) in series with $R_1 = 5\ \Omega$ forms the left branch of mesh 1. Mesh 1 and mesh 2 share a middle branch containing $R_2 = 10\ \Omega$. Mesh 2's outer branch contains $R_3 = 15\ \Omega$. Both mesh currents $i_1$ (mesh 1) and $i_2$ (mesh 2) are defined clockwise.

Concretely, going clockwise around mesh 1 starting just after the source's $-$ terminal: source ($10\text{ V}$, entering $+$ terminal first when traversing clockwise from bottom-left... to keep this concrete and unambiguous, we define it by the sign convention below), then $R_1$, then down through the shared branch $R_2$, back to start. Mesh 2, clockwise: up through shared branch $R_2$, then across through $R_3$, back to start.

**Step 2 — Branch currents in terms of mesh currents:**
- Through $R_1$ (outer branch of mesh 1 only): current $= i_1$.
- Through $R_2$ (shared branch): net current in mesh 1's clockwise sense $= i_1 - i_2$.
- Through $R_3$ (outer branch of mesh 2 only): current $= i_2$.

**Step 3 — KVL, mesh 1** (clockwise, sum of drops = 0). Traversing clockwise, the source is encountered such that we go from $-$ to $+$ (i.e., traversal direction is a *rise* through the source, so it contributes $-10\text{ V}$ to the sum-of-drops equation), then a drop $i_1 R_1$ across $R_1$, then a drop $(i_1 - i_2) R_2$ across the shared resistor:
$$-10 + i_1 R_1 + (i_1 - i_2) R_2 = 0$$
$$-10 + 5 i_1 + 10(i_1 - i_2) = 0$$
$$-10 + 5i_1 + 10 i_1 - 10 i_2 = 0$$
$$15 i_1 - 10 i_2 = 10 \quad \text{(Equation A)}$$

**KVL, mesh 2** (clockwise; the shared branch is traversed in the *opposite* sense relative to mesh 1, i.e. net current in mesh 2's clockwise sense is $i_2 - i_1$, and there is no source in this loop):
$$(i_2 - i_1) R_2 + i_2 R_3 = 0$$
$$10(i_2 - i_1) + 15 i_2 = 0$$
$$10 i_2 - 10 i_1 + 15 i_2 = 0$$
$$-10 i_1 + 25 i_2 = 0 \quad \text{(Equation B)}$$

**Step 5 — Solve.** From Equation B: $i_1 = \dfrac{25 i_2}{10} = 2.5 i_2$.

Substitute into Equation A:
$$15(2.5 i_2) - 10 i_2 = 10$$
$$37.5 i_2 - 10 i_2 = 10$$
$$27.5 i_2 = 10$$
$$i_2 = \frac{10}{27.5} = \frac{4}{11} \approx 0.3636\text{ A}$$

Then $i_1 = 2.5 \times \dfrac{4}{11} = \dfrac{10}{11} \approx 0.9091\text{ A}$.

**Step 6 — Branch currents:**
- Through $R_1$: $i_1 \approx 0.909\text{ A}$.
- Through $R_2$ (shared branch, mesh-1 sense): $i_1 - i_2 = \dfrac{10}{11} - \dfrac{4}{11} = \dfrac{6}{11} \approx 0.545\text{ A}$.
- Through $R_3$: $i_2 \approx 0.364\text{ A}$.

**Check via KVL around the outer loop** (source, $R_1$, $R_3$, skipping the shared branch entirely — valid since it's also a loop of the circuit): drop across $R_1$ is $5 \times 0.9091 = 4.545\text{ V}$; drop across $R_3$ is $15 \times 0.3636 = 5.455\text{ V}$; sum $= 4.545 + 5.455 = 10.0\text{ V}$, matching the $10\text{ V}$ source. Balances (within rounding).

## Common pitfalls

- **Forgetting to pick a reference node, or picking one inconsistently mid-problem.** Every node voltage is meaningless without a stated reference; if you switch reference nodes partway through you'll silently corrupt every equation written before the switch.
- **Sign errors in the "leaving = 0" KCL convention.** A current source injecting current *into* a node must be written as a *negative* term in a "sum of currents leaving = 0" equation (or, equivalently, moved to the other side as a positive term in a "leaving = injected" equation). Mixing "leaving" and "entering" conventions within the same equation is the single most common node-analysis error.
- **Applying Ohm's law with node voltages in the wrong order.** The current from node $a$ to node $b$ through a resistor is $(v_a - v_b)/R$, not $(v_b - v_a)/R$ — reversing this flips the sign of every downstream current.
- **Missing the supernode for a floating voltage source.** If a voltage source connects two non-reference nodes, you cannot write an ordinary KCL equation at either node individually (the current through the source is unknown and not given by Ohm's law). You must combine them into a supernode and separately enforce the voltage-difference constraint.
- **Assuming mesh analysis works on non-planar circuits.** If the circuit as drawn requires a branch to cross another branch without an actual node there, "which meshes share this branch" is not well-defined, and the mesh method (as taught here) does not directly apply; node analysis has no such restriction and always works.
- **Inconsistent mesh traversal direction for shared branches.** When a branch is shared by two meshes, the two mesh currents pass through it in *opposite* physical directions; forgetting the minus sign (writing $i_m + i_n$ instead of $i_m - i_n$) is a common error.
- **Forgetting that a current source constrains a mesh-current *difference*, not an absolute mesh current, when it's on a shared branch.** Only when the current source is on an *outer* (single-mesh) branch does it fix one mesh current outright.
- **Double-counting or skipping the reference-node KCL equation.** The reference node's own KCL equation is redundant (automatically satisfied given the other $N$), so don't write it as an "extra" independent equation — doing so just reproduces a linear combination of the others and adds no new information, though it also does no harm if you double check with it (as in the "Check" steps above).
- **Unit mix-ups.** Keep resistances in ohms ($\Omega$), currents in amperes (A), voltages in volts (V) throughout; a stray factor of $1000$ from confusing $\text{k}\Omega$ with $\Omega$ is common when problems are stated with prefixed units.

## Self-check

### Questions

1. In node analysis, why does a connected circuit with $N+1$ nodes need exactly $N$ independent KCL equations (not $N+1$)?
2. A resistor $R = 5\ \Omega$ connects node $a$ ($v_a = 8\text{ V}$) to node $b$ ($v_b = 3\text{ V}$). What is the current flowing from $a$ to $b$ through the resistor?
3. What is a "supernode," and when must you use one?
4. In mesh analysis, if mesh 1 (clockwise current $i_1$) and mesh 2 (clockwise current $i_2$) share a branch with resistor $R$, what is the voltage drop across that resistor in the direction of mesh 1's traversal, in terms of $i_1$, $i_2$, and $R$?
5. Why is mesh analysis, as presented here, restricted to planar circuits, while node analysis is not?
6. A circuit has a single node (besides ground) with a $2\text{ A}$ current source pushing current into it, and two resistors from that node to ground: $R_1 = 10\ \Omega$ and $R_2 = 10\ \Omega$ in parallel (both directly to ground). Find the node voltage and the current through each resistor.
7. A single-mesh circuit has a $12\text{ V}$ source in series with $R_1 = 3\ \Omega$ and $R_2 = 9\ \Omega$ (only one mesh, no shared branches). Using mesh analysis, find the mesh current and the voltage across $R_2$.
8. Suppose in Example 1 (three-resistor circuit) you had instead defined "sum of currents leaving = 0" but forgotten the minus sign on the current source term, writing $+3 + v_1/2 + (v_1-v_2)/4 = 0$ for node 1's equation while keeping node 2's equation the same. Solve this incorrect system for $v_1$ and explain, physically, why the answer must be wrong (i.e., what sign it gets and why that's implausible).

### Answers

1. Because the sum of *all* currents leaving *all* nodes in the circuit (including the reference node) must be zero by conservation of charge applied to the entire circuit as one region — every branch current appears once as "leaving" one endpoint and once as "entering" the other, so the $N+1$ KCL equations (one per node) always sum to $0=0$ and are therefore not all independent. Any $N$ of them are independent, and the $(N+1)$th is implied. Choosing to omit the reference node's equation is just a convenient choice.
2. $i_{a\to b} = (v_a - v_b)/R = (8 - 3)/5 = 5/5 = 1\text{ A}$.
3. A supernode is the combined region formed by merging two nodes that are directly connected by an independent voltage source (when neither of those two nodes is the reference node). You use it because the current through that voltage source isn't given directly by Ohm's law (a voltage source has no fixed resistance), so an ordinary single-node KCL equation can't be written for either node alone; instead you write one KCL equation for the combined region (in which the source's internal current cancels out, since it leaves one node and enters the other within the region) plus the separate constraint $v_a - v_b = V_s$.
4. The net current through the shared branch in mesh 1's clockwise sense is $i_1 - i_2$ (mesh 2's circulating current passes through that branch in the opposite physical direction relative to mesh 1's convention). By Ohm's law, the voltage drop in mesh 1's traversal direction is $(i_1 - i_2)R$.
5. Mesh analysis relies on being able to say unambiguously which two meshes (window-panes) border a given branch, so that "shared branch current = difference of the two bordering mesh currents" is well defined. This bordering relationship is only geometrically well-defined when the circuit is drawn with no branch crossing another branch (planar). Node analysis never refers to "which meshes border a branch" — it only uses KCL at nodes, which is well-defined regardless of how the circuit is drawn or whether it's planar.
6. KCL at the node (leaving = 0, current source term negative since it injects current in): $-2 + v/10 + v/10 = 0 \Rightarrow -2 + 2v/10 = 0 \Rightarrow 2v/10 = 2 \Rightarrow v = 10\text{ V}$. Current through each resistor: $i_{R_1} = i_{R_2} = 10\text{ V}/10\ \Omega = 1\text{ A}$ each (and $1+1=2\text{ A}$ matches the source, confirming KCL).
7. Single mesh, one equation: traversal direction is a rise through the source ($-12$) then drops $i R_1$ and $i R_2$: $-12 + i(3) + i(9) = 0 \Rightarrow 12i = 12 \Rightarrow i = 1\text{ A}$. Voltage across $R_2$: $v_{R_2} = iR_2 = 1 \times 9 = 9\text{ V}$.
8. With the sign error, node 1's equation becomes $3 + v_1/2 + (v_1-v_2)/4 = 0$, i.e. multiplying by 4: $12 + 2v_1 + v_1 - v_2 = 0 \Rightarrow 3v_1 - v_2 = -12$ (Equation A'). Node 2's equation is unchanged: $-3v_1 + 5v_2 = 0 \Rightarrow v_2 = 3v_1/5$ (Equation B). Substituting: $3v_1 - 3v_1/5 = -12 \Rightarrow 12v_1/5 = -12 \Rightarrow v_1 = -5\text{ V}$. This is implausible because a current source is *injecting* $3\text{ A}$ of current into a node that only has resistors to ground/other nodes as exits — physically, current flowing into a resistive network from a source should drive that node's voltage in the *positive* direction relative to ground (current flows from high to low potential through a resistor, so if current is flowing out of node 1 through $R_1$ and $R_2$, node 1 must be at higher potential than the nodes it flows into, not negative). Getting $v_1 = -5\text{ V}$ instead of the correct $+5\text{ V}$ is the direct fingerprint of the dropped minus sign on the current-source term — the magnitude is unaffected by this particular sign error, but the overall sign flips, revealing the mistake.

## Summary / cheat sheet

**Node analysis (unknowns: node voltages $v_k$, reference node $= 0\text{ V}$):**
1. Pick reference node.
2. Current from node $a$ to node $b$ through resistor $R$: $i_{a\to b} = (v_a - v_b)/R$.
3. At each non-reference node: $\sum(\text{currents leaving}) = 0$; independent current source injecting $I_s$ into node contributes $-I_s$ to that sum.
4. Voltage source to reference node: fixes that node's voltage directly. Floating voltage source between two non-reference nodes: use a **supernode** (one merged KCL equation + constraint $v_a - v_b = V_s$).
5. $N$ non-reference nodes $\Rightarrow$ exactly $N$ independent equations. Solve, then back-substitute for branch currents via Ohm's law.

**Mesh analysis (unknowns: clockwise mesh currents $i_m$; planar circuits only):**
1. Number of independent meshes $M = B - N$ (branches minus non-reference nodes).
2. Branch on outer boundary of one mesh only: carries that mesh's current. Branch shared by meshes $m,n$: carries $i_m - i_n$ in mesh $m$'s sense.
3. KVL per mesh, clockwise: $\sum(\text{drops}) = 0$; resistor drop in traversal direction $= (\text{net current through it in that sense}) \times R$; voltage source contributes $\pm V_s$ depending on traversal-vs-polarity.
4. Current source on outer branch: fixes that mesh current outright. Current source on shared branch: fixes the *difference* of the two mesh currents ($i_m - i_n = \pm I_s$), replacing one KVL equation.
5. Solve $M$ equations, back-substitute for branch currents/voltages.

**Both methods must agree** on final branch currents/voltages — they are different unknown-sets describing the same KVL/KCL/Ohm's-law-constrained system. Choose node analysis when there are few nodes or many voltage sources; choose mesh analysis when there are few meshes or many current sources (and the circuit is planar).

## Used later in
(none yet)
