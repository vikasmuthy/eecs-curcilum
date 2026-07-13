---
title: "Lumped Circuit Abstraction, KVL/KCL"
course: "6.002"
topic_number: 01
prerequisites: ["8.02 (charge, current, potential difference) — recapped inline below"]
status: in progress
---

# Lumped Circuit Abstraction, KVL/KCL

## Why this matters

Maxwell's equations describe electromagnetic fields exactly, but solving them directly for every wire and component in a real circuit (a radio, a laptop motherboard, an amplifier) is intractable for engineering purposes. The **lumped circuit abstraction** is the foundational simplification that makes circuit design possible: it replaces continuous, spatially-distributed electromagnetic behavior with a small set of discrete "lumps" (elements) connected by idealized wires, each lump characterized by only two numbers at each instant — the current through it and the voltage across it. Once this abstraction is accepted, two bookkeeping laws — **Kirchhoff's Voltage Law (KVL)** and **Kirchhoff's Current Law (KCL)** — fall out directly from charge conservation and the properties of the electric field, and together with element laws (like Ohm's law) they are *sufficient* to solve any lumped circuit. Every later topic in 6.002 (node analysis, Thevenin equivalents, transistor models, amplifiers, transients) is built by writing KVL/KCL equations for more complicated collections of lumped elements. If you skip this topic, nothing downstream has a rigorous foundation — you'd be pattern-matching circuit-solving recipes without knowing why they work or when they stop working.

## Builds on

- **8.02 (charge, current, potential difference)** — recapped self-contained below, since 8.02 notes do not yet exist in this vault. We need: what electric charge and current are, and what potential difference (voltage) means.

## Core definitions

- **Charge ($q$)** — a fundamental property of matter, measured in coulombs (C), that is the source of electric fields and the quantity that electric forces act on. Charge is conserved: it cannot be created or destroyed, only moved or rearranged.
- **Current ($i$)** — the rate of flow of charge past a point, $i = dq/dt$, measured in amperes (A), where 1 A = 1 C/s. Current has a reference direction (an arrow you draw on a circuit diagram); a positive numeric value means positive charge flows in the direction of the arrow (or equivalently negative charge flows opposite it).
- **Electric potential ($V$)** — the potential energy per unit charge at a point in space due to the electric field, measured in volts (V), where 1 V = 1 J/C. Only *differences* in potential are physically meaningful (potential is defined up to an additive constant).
- **Voltage / potential difference ($v$)** — the difference in electric potential between two points, $v_{AB} = V_A - V_B$, measured in volts. It is the work per unit charge required to move a charge from point $B$ to point $A$ against the electric field (or the work done by the field per unit charge moving from $A$ to $B$).
- **Lumped element** — an idealized circuit component (resistor, source, capacitor, etc.) that is fully characterized, at every instant of time, by exactly two numbers: the current flowing through its terminals and the voltage across its terminals. All of the element's internal spatial/field structure is compressed ("lumped") into this current-voltage relationship.
- **Lumped circuit abstraction** — the modeling assumption that an entire circuit can be represented as a finite set of lumped elements connected by ideal (resistanceless, zero-time-delay) wires called **nodes** and **branches**, valid under conditions given in the Derivation section below.
- **Node** — a point in a circuit diagram where two or more element terminals are connected together by ideal wire; every point on a connected set of ideal wires is, electrically, the same node (same potential).
- **Branch** — a single lumped element together with the two nodes it connects to; a branch has one current (flowing through it) and one voltage (across it).
- **Loop** — any closed path through a circuit diagram that starts and ends at the same node, traversing a sequence of branches without repeating a node (except the start/end).
- **Kirchhoff's Voltage Law (KVL)** — the sum of branch voltages around any closed loop, taken with consistent sign convention, is zero at every instant of time.
- **Kirchhoff's Current Law (KCL)** — the sum of currents leaving (or entering) any node is zero at every instant of time.
- **Reference direction / sign convention** — an arbitrarily chosen arrow (for current) or $+/-$ pair (for voltage) drawn on a diagram before solving; the actual physical direction is deduced from the sign of the solved numeric value, and once chosen for KVL/KCL bookkeeping, must be used consistently.

## Intuition

Think of a circuit as plumbing for charge. Water (charge) flows through pipes (wires and elements); at any junction (node) where several pipes meet, whatever water flows in must flow out — water doesn't pile up or vanish at a junction if the pipe segment holds no water reservoir. That's KCL.

Now think of voltage as elevation in a gravity analogy: potential is like height, and voltage between two points is like the height difference you'd measure with two points on a hiking trail. If you walk a loop on a mountain — up some slopes, down others — and return to your starting point, your net change in elevation is zero, no matter which path you took, because elevation only depends on position, not on the path taken to get there. That's KVL: potential is a function of *position* (which node you're at), not of the path used to travel between nodes, so walking any loop and summing up "elevation changes" (voltage drops and rises) must return you to zero net change.

The "lumping" step is the modeling leap: real components have physical extent, and electromagnetic effects inside them (charge distributions, changing fields) are, in general, complicated and spatially spread out. The lumped abstraction says: as long as the physical size of the circuit and the speed of change of signals are such that electromagnetic effects propagate across the whole circuit much faster than anything of interest changes, we can ignore all the internal spatial detail and pretend each element is a point, fully described by just its terminal current and voltage. This throws away information (which is fine, because for typical benchtop and chip-scale circuits at typical operating frequencies, the ignored information doesn't matter to the accuracy engineers need) but it also means the lumped model *breaks down* for circuits large enough, or signals fast enough, that this assumption fails (e.g., transmission lines, RF circuits at GHz frequencies, or antennas — deliberately outside 6.002's scope).

## Derivation / formalism

### Step 0: Recap of 8.02 background

Charge $q$ is measured in coulombs. Current is the flow rate of charge through a cross-section: $i(t) = \frac{dq}{dt}$. Electric potential $V$ at a point is potential energy per unit charge; a positive test charge released at high potential and free to move will accelerate toward lower potential (fields point from high to low potential, forces push positive charge "downhill" in potential). Potential difference (voltage) between two points $A$ and $B$ is $v_{AB} = V_A - V_B$, and it is path-independent: it depends only on the two endpoint positions, not on the path used to compute it, because the electrostatic field is conservative (this follows from $\oint \vec{E}\cdot d\vec{l} = 0$ for any closed path, a direct consequence of Maxwell's equations in the static, or quasi-static, limit).

### Step 1: Conditions for the lumped circuit abstraction to hold

The lumped abstraction is valid when two conditions hold:

1. **No net charge accumulates inside any circuit element or wire over time**, except inside elements explicitly designed to store charge (capacitors), whose charge-storing behavior is itself captured in their element law. Practically, this means we treat wires and non-capacitive elements as unable to accumulate net charge — everything that flows in flows back out.
2. **The signal timescales of interest are slow compared to the time for electromagnetic effects to propagate across the physical size of the circuit.** If a circuit has physical dimension $d$ and signals change on a timescale $\tau$, we need $d \ll c\tau$ (where $c$ is the speed of propagation of the field, at most the speed of light). Under this condition, at any instant, the whole circuit "agrees" on potential values consistently (no retardation effects), and we may treat voltage and current as functions of time alone, evaluated instantaneously and uniformly throughout each lumped element, without worrying about spatial variation within an element or propagation delay between elements.

6.002 restricts attention to circuits and signals for which condition 2 holds (this is why 6.002 doesn't need transmission-line theory); we take it as a standing assumption from here on.

### Step 2: Derive KCL from charge conservation

Consider any node $N$ in a lumped circuit, where branches $1, 2, \ldots, k$ connect to it, carrying currents $i_1, i_2, \ldots, i_k$, all defined with reference directions pointing *away from* the node (out of the node into each branch).

Draw a closed surface $S$ enclosing node $N$ and a negligibly short length of each of its attached wires, so that $S$ crosses each of the $k$ branches exactly once and otherwise crosses no other conductors. Charge conservation, applied to the volume enclosed by $S$, says:

$$\frac{dq_{enc}}{dt} = -\left(\text{net current flowing out through } S\right)$$

where $q_{enc}$ is the total charge inside $S$. This is just a statement that charge cannot be created or destroyed: any decrease in charge enclosed by $S$ must be accounted for by charge leaving through $S$, and vice versa. The net current flowing out through $S$ is exactly the sum of the currents in the $k$ branches (using the away-from-node reference directions):

$$\frac{dq_{enc}}{dt} = -\sum_{j=1}^{k} i_j$$

By the lumped abstraction condition 1 above, node $N$ (being a piece of ideal wire, not a charge-storage element) cannot accumulate net charge: $q_{enc}$ stays at (essentially) zero at all times, so $\frac{dq_{enc}}{dt} = 0$. Substituting:

$$0 = -\sum_{j=1}^{k} i_j \quad\Longrightarrow\quad \sum_{j=1}^{k} i_j = 0$$

This is **KCL**: the sum of currents leaving any node, using consistent away-from-node reference directions, is zero at every instant $t$. (Equivalently, sum of currents entering a node is zero, or "current in = current out.")

### Step 3: Derive KVL from path-independence of potential

Consider any closed loop in a lumped circuit passing through nodes $N_0, N_1, N_2, \ldots, N_{m-1}, N_0$ (returning to the start), traversing branches with voltages $v_1$ (from $N_0$ to $N_1$), $v_2$ (from $N_1$ to $N_2$), …, $v_m$ (from $N_{m-1}$ back to $N_0$), where each $v_j$ is defined as the potential at the node the branch is *leaving* minus the potential at the node the branch is *arriving at* (this is a "voltage drop" sign convention in the direction of loop traversal). Let $V_0, V_1, \ldots, V_{m-1}$ be the potentials of the respective nodes at the current instant $t$ (well-defined, instantaneous values, valid because of the lumped abstraction's timescale condition). Then by definition of voltage as a potential difference:

$$v_1 = V_0 - V_1,\quad v_2 = V_1 - V_2,\quad \ldots,\quad v_m = V_{m-1} - V_0$$

Sum all $m$ equations:

$$\sum_{j=1}^{m} v_j = (V_0 - V_1) + (V_1 - V_2) + \cdots + (V_{m-1} - V_0)$$

Every $V_i$ term appears exactly once with a $+$ sign and once with a $-$ sign (this is a telescoping sum), so every term cancels:

$$\sum_{j=1}^{m} v_j = 0$$

This is **KVL**: the sum of branch voltages around any closed loop, with signs assigned by consistent traversal direction, is zero at every instant $t$. The only physics used is that potential is a well-defined, single-valued function of node location at each instant (path-independence, itself a consequence of the electrostatic/quasi-static field being conservative, i.e., $\oint \vec E \cdot d\vec l = 0$) — no assumption about what's inside each branch (resistor, source, capacitor, anything) was needed. That is why KVL and KCL apply to *any* lumped circuit topology regardless of what elements are used: they come purely from charge conservation and field conservativeness, not from any particular element's behavior.

### Step 4: KVL/KCL alone are not enough — element laws close the system

KVL and KCL are purely topological/structural constraints: they only encode "how the circuit is wired," not "what each element does." For a circuit with $b$ branches, KCL gives (number of nodes $- 1$) independent equations and KVL gives enough independent loop equations to total $b$ independent equations when combined with KCL — but the unknowns are $2b$ (a current and a voltage per branch, i.e. $b$ currents + $b$ voltages), so KVL+KCL supply only $b$ equations for $2b$ unknowns. The remaining $b$ equations come from each branch's **element law** (also called the *constitutive relation*), e.g. Ohm's law $v = iR$ for a resistor, which relates that particular branch's own voltage and current to each other. Only with element laws added does the system become fully determined (exactly enough independent equations to solve for all $2b$ unknowns) — this is why solving a circuit always means: (a) write KCL at nodes, (b) write KVL around loops, (c) write each element's law, then (d) solve the resulting simultaneous equations.

## Worked examples

### Example 1 — Single loop, KVL + Ohm's law

**Setup.** A circuit has one loop: an ideal voltage source of $V_s = 9\ \text{V}$ in series with two resistors, $R_1 = 100\ \Omega$ and $R_2 = 200\ \Omega$. Label the top node (positive terminal of the source) as node $A$, the node between $R_1$ and $R_2$ as node $B$, and the bottom node (negative terminal of the source, and the return to $R_2$) as node $C$, with $C$ also the reference ("ground," $V_C = 0\ \text{V}$). Define loop current $i$ flowing clockwise: out of the source's $+$ terminal, through $R_1$ from $A$ to $B$, through $R_2$ from $B$ to $C$, and back into the source's $-$ terminal.

**Step 1 — KVL around the single loop**, traversing clockwise starting at $C$, summing voltage *drops* in the traversal direction:

- Rise of $V_s$ across the source (from $-$ to $+$ is a rise, so as a drop it's $-V_s$)
- Drop of $v_{R_1} = iR_1$ across $R_1$ (current flows from $A$ to $B$, so potential drops in that direction, by Ohm's law with the current-reference-aligned sign convention)
- Drop of $v_{R_2} = iR_2$ across $R_2$

KVL: $-V_s + iR_1 + iR_2 = 0$.

**Step 2 — Solve for $i$:**

$$i(R_1 + R_2) = V_s \implies i = \frac{V_s}{R_1+R_2} = \frac{9\ \text{V}}{100\ \Omega + 200\ \Omega} = \frac{9\ \text{V}}{300\ \Omega} = 0.03\ \text{A} = 30\ \text{mA}$$

**Step 3 — Find node voltages** using $V_C = 0\ \text{V}$ as reference:

$$V_A = V_C + V_s = 0 + 9 = 9\ \text{V}$$
$$v_{R_1} = i R_1 = (0.03\ \text{A})(100\ \Omega) = 3\ \text{V} \implies V_B = V_A - v_{R_1} = 9 - 3 = 6\ \text{V}$$
$$v_{R_2} = i R_2 = (0.03\ \text{A})(200\ \Omega) = 6\ \text{V} \implies V_C = V_B - v_{R_2} = 6 - 6 = 0\ \text{V} \checkmark$$

The check $V_C = 0\ \text{V}$ matches our reference choice, confirming consistency (this is exactly KVL closing the loop).

**Step 4 — Verify KCL.** At node $B$, only $R_1$ and $R_2$ connect (current in from $R_1$ = current out to $R_2$, both equal to $i = 30\ \text{mA}$, single loop, no branching) — trivially satisfied since it's a series circuit with one current path.

### Example 2 — Two nodes, KCL with three branches

**Setup.** Node $A$ (at unknown potential $V_A$) connects to ground (node $C$, $V_C = 0\ \text{V}$, the reference) through three parallel branches:
- Branch 1: an ideal current source injecting $i_s = 50\ \text{mA}$ *into* node $A$ (fixed, independent of voltage).
- Branch 2: resistor $R_1 = 1\ \text{k}\Omega$ from $A$ to $C$.
- Branch 3: resistor $R_2 = 4\ \text{k}\Omega$ from $A$ to $C$.

Define $i_1$ as the current flowing from $A$ to $C$ through $R_1$, and $i_2$ as the current flowing from $A$ to $C$ through $R_2$ (both reference directions pointing away from node $A$, into their branches).

**Step 1 — KCL at node $A$**, currents leaving $A$ sum to zero; the current source's $i_s$ enters $A$, so as a "leaving" current it is $-i_s$:

$$-i_s + i_1 + i_2 = 0 \implies i_1 + i_2 = i_s$$

**Step 2 — Element laws (Ohm's law)** relate each resistor's current to the shared node voltage $V_A$ (since both resistors connect the same two nodes $A$ and $C$, they have the same voltage across them, $V_A - V_C = V_A$):

$$i_1 = \frac{V_A - V_C}{R_1} = \frac{V_A}{R_1}, \qquad i_2 = \frac{V_A - V_C}{R_2} = \frac{V_A}{R_2}$$

**Step 3 — Substitute into KCL and solve for $V_A$:**

$$\frac{V_A}{R_1} + \frac{V_A}{R_2} = i_s \implies V_A\left(\frac{1}{R_1}+\frac{1}{R_2}\right) = i_s \implies V_A = \frac{i_s}{\frac{1}{R_1}+\frac{1}{R_2}}$$

Numerically, $\frac{1}{R_1} = \frac{1}{1000\ \Omega} = 0.001\ \text{S}$ (siemens, unit of conductance $=1/\Omega$), and $\frac{1}{R_2} = \frac{1}{4000\ \Omega} = 0.00025\ \text{S}$. Sum: $0.00125\ \text{S}$.

$$V_A = \frac{0.050\ \text{A}}{0.00125\ \text{S}} = 40\ \text{V}$$

**Step 4 — Back out branch currents:**

$$i_1 = \frac{V_A}{R_1} = \frac{40\ \text{V}}{1000\ \Omega} = 0.040\ \text{A} = 40\ \text{mA}$$
$$i_2 = \frac{V_A}{R_2} = \frac{40\ \text{V}}{4000\ \Omega} = 0.010\ \text{A} = 10\ \text{mA}$$

**Step 5 — Check KCL:** $i_1 + i_2 = 40\ \text{mA} + 10\ \text{mA} = 50\ \text{mA} = i_s$. ✓ Matches the source current exactly, confirming the solution.

## Common pitfalls

- **Inconsistent sign convention within one loop or node.** KVL/KCL only work if you fix a reference direction (arrow) *before* solving and apply it uniformly; mixing "drop" and "rise" conventions mid-loop, or switching a current's assumed direction partway through a node's KCL sum, produces sign errors that look like valid algebra but give wrong answers.
- **Forgetting that a negative solved value is not an error.** If you assume current $i$ flows left-to-right and solve $i = -20\ \text{mA}$, that means the physical current actually flows right-to-left at $20\ \text{mA}$ — the reference direction was just a bookkeeping choice, not a prediction.
- **Confusing potential ($V$, defined at a single node relative to a reference) with voltage ($v$, defined as a difference between two nodes).** Writing "$V_{R_1}$" as if a resistor has one absolute potential is meaningless; only $v_{R_1} = V_A - V_B$ (the difference across its two terminals) is meaningful.
- **Applying KVL/KCL to a circuit where the lumped abstraction doesn't hold** — e.g., signals fast enough, or circuits physically large enough, that propagation delay across the circuit isn't negligible (transmission lines, antennas, GHz-and-above on-chip interconnect). 6.002's version of KVL/KCL implicitly assumes the lumped abstraction's timescale condition; outside that regime you need distributed (transmission-line) models instead.
- **Double-counting or skipping a branch in a KCL sum** at a node with many connections — always recount every branch physically touching the node, not just the ones that "look important."
- **Treating a wire with zero resistance as if it could have a voltage drop.** An ideal wire (or a "node") has, by definition of the lumped abstraction, the same potential everywhere along it; if your equations imply a wire carries a nonzero voltage, you've mislabeled which points are the same node.
- **Forgetting element laws.** Students sometimes try to solve for all branch currents and voltages using KVL and KCL alone; as shown in Step 4 of the derivation, this is structurally impossible (not enough equations) without also writing each element's own current-voltage relationship.

## Self-check

### Questions

1. State KCL and KVL in your own words, including what physical law each ultimately comes from.
2. Why is a "node" in the lumped-circuit sense always at a single, well-defined potential, even though physically it's a piece of wire with some finite length?
3. A single loop has a $12\ \text{V}$ ideal source in series with resistors $R_1 = 3\ \Omega$ and $R_2 = 9\ \Omega$. Find the loop current and the voltage across $R_2$.
4. At a node, three currents are defined with arrows pointing *into* the node: $i_1 = 2\ \text{A}$, $i_2 = -1\ \text{A}$, and $i_3$ is unknown. Using KCL, find $i_3$.
5. Explain why KVL and KCL, by themselves, cannot fully solve a circuit with resistors in it — what additional ingredient is required, and why?
6. A circuit has two ideal current sources feeding into the same node, one injecting $3\ \text{A}$ and one *extracting* $1\ \text{A}$ from that node, alongside a single resistor $R = 2\ \Omega$ from that node to ground. Find the node voltage relative to ground.
7. Explain, physically, one concrete scenario (mentioned in this note) where the lumped circuit abstraction breaks down, and identify which of the two validity conditions (from Step 1 of the Derivation) fails.
8. Two resistors $R_1 = 6\ \Omega$ and $R_2 = 3\ \Omega$ are connected in parallel between node $A$ and ground. A third resistor $R_3 = 4\ \Omega$ connects node $A$ to a $10\ \text{V}$ ideal source (i.e., in series between the source and node $A$). Find $V_A$ and the current through $R_3$. (Hint: combine KCL at node $A$ with Ohm's law on all three resistors; the current through $R_3$ equals the sum of currents through $R_1$ and $R_2$ by KCL.)

### Answers

1. KCL: the sum of currents leaving (or entering) any node is zero at every instant; it follows from conservation of charge (charge can't accumulate at an ideal node that isn't a charge-storage element). KVL: the sum of voltage drops around any closed loop is zero at every instant; it follows from potential being a single-valued, path-independent function of position (a consequence of the electrostatic/quasi-static field being conservative).
2. Because the lumped abstraction assumes the wire is ideal (zero resistance, and signal timescales slow relative to propagation delay across the circuit), so there is no mechanism for a potential difference to exist between two points connected by that wire — Ohm's law on the wire itself, $v = iR_{wire}$ with $R_{wire}=0$, forces $v=0$ regardless of current, hence every point on it is at the same potential by definition.
3. KVL: $-12 + iR_1 + iR_2 = 0 \Rightarrow i = \frac{12}{3+9} = \frac{12}{12} = 1\ \text{A}$. Voltage across $R_2$: $v_{R_2} = iR_2 = (1\ \text{A})(9\ \Omega) = 9\ \text{V}$.
4. KCL (currents into node sum to zero, using the "into" convention as given, treated as equivalent to "sum in = 0" when all are defined as "into"): actually with all three defined as *into* the node, conservation requires sum of currents into the node = 0 only if nothing else connects; here $i_1 + i_2 + i_3 = 0 \Rightarrow 2 + (-1) + i_3 = 0 \Rightarrow i_3 = -1\ \text{A}$. (The negative sign means $i_3$'s actual physical direction is out of the node, at $1\ \text{A}$.)
5. KVL and KCL are purely structural/topological — they encode only how branches are connected (loops and nodes), not what any particular element does. For $b$ branches there are $2b$ unknowns (a voltage and current per branch) but KVL+KCL together supply only $b$ independent equations. The missing $b$ equations must come from each branch's element law (e.g. Ohm's law $v=iR$ for a resistor), which relates that specific branch's own voltage and current based on the physics of that element.
6. KCL at the node: net current in $= 3\ \text{A} - 1\ \text{A} = 2\ \text{A}$, which must all flow through $R$ to ground: $2\ \text{A} = V_{node}/R \Rightarrow V_{node} = (2\ \text{A})(2\ \Omega) = 4\ \text{V}$.
7. Example: a GHz-frequency signal on a long on-chip interconnect, or an antenna. The failing condition is condition 2 (timescale/size condition, $d \ll c\tau$): the physical size of the conductor is no longer negligible compared to the distance the field's influence can propagate within one signal period, so different points along the "wire" no longer agree on potential at the same instant, and treating it as a single node/branch with instantaneous, spatially-uniform voltage is no longer accurate. (Condition 1, no charge accumulation, is the one that fails for capacitors specifically, but that's an intentional, modeled exception, not an abstraction breakdown — antennas/transmission lines are the genuine breakdown case.)
8. Let $i_3$ be the current through $R_3$ from the source into node $A$, and $i_1, i_2$ the currents from $A$ to ground through $R_1, R_2$. KVL from source through $R_3$ to node $A$: $v_{R_3} = 10 - V_A$, so $i_3 = \frac{10-V_A}{R_3} = \frac{10-V_A}{4}$. KCL at $A$: $i_3 = i_1+i_2 = \frac{V_A}{R_1}+\frac{V_A}{R_2} = \frac{V_A}{6}+\frac{V_A}{3} = V_A\left(\frac{1}{6}+\frac{2}{6}\right)=\frac{V_A}{2}$. Set equal: $\frac{10-V_A}{4} = \frac{V_A}{2} \Rightarrow 10 - V_A = 2V_A \Rightarrow 10 = 3V_A \Rightarrow V_A = \frac{10}{3} \approx 3.33\ \text{V}$. Then $i_3 = \frac{V_A}{2} = \frac{10/3}{2} = \frac{10}{6} = \frac{5}{3} \approx 1.67\ \text{A}$.

## Summary / cheat sheet

- **Lumped element**: fully described at each instant by one current (through it) + one voltage (across it). Valid when (1) no unmodeled charge accumulation anywhere in wires/non-capacitive elements, and (2) circuit physical size $d \ll c\tau$ for signal timescale $\tau$ (electromagnetic propagation across the circuit is effectively instantaneous compared to signal changes).
- **KCL**: $\displaystyle\sum_{\text{branches at a node}} i_j = 0$ (with consistent away-from-node, or consistent into-node, sign convention). Comes from **charge conservation** at a node that cannot store net charge.
- **KVL**: $\displaystyle\sum_{\text{branches around a loop}} v_j = 0$ (with consistent traversal-direction sign convention). Comes from potential being **single-valued and path-independent** (conservative field).
- **Element law** (e.g. Ohm's law $v = iR$) supplies the equations KVL/KCL cannot: for $b$ branches, KVL+KCL give $b$ equations, element laws give the other $b$, for $2b$ unknowns total ($b$ currents + $b$ voltages).
- **Recipe to solve any lumped circuit**: (1) label all node potentials and branch current reference directions; (2) write KCL at all nodes but one (last one is redundant given the others); (3) write KVL around enough independent loops, or work directly with node potentials (equivalent, and used more in later notes); (4) write each element's law; (5) solve the simultaneous equations.
- **Sign discipline**: pick reference directions *before* solving; a negative result flips the physical direction, it is not an error.
