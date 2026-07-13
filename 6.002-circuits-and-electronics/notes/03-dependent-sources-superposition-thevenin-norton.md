---
title: "Dependent Sources, Superposition, Thevenin/Norton"
course: "6.002"
topic_number: 03
prerequisites: ["6.002 Topic 02: Resistive networks, node/mesh analysis", "6.002 Topic 01: Lumped circuit abstraction, KVL/KCL"]
status: in progress
---

# Dependent Sources, Superposition, Thevenin/Norton

## Why this matters

So far (Topic 02) every source in a circuit has been *independent*: a battery supplies a fixed voltage, a current source supplies a fixed current, and neither cares what else is happening in the circuit. Real amplifying devices (transistors, op-amps) cannot be modeled this way — a transistor's output current depends on some *other* voltage or current elsewhere in the circuit. To analyze any circuit containing an amplifying element, we need **dependent sources**: sources whose value is a multiple of some other voltage or current in the circuit. This note extends node/mesh analysis to handle them.

Once dependent sources are in play, two more tools become essential. **Superposition** lets us analyze a circuit with several independent sources by solving one source at a time and adding the results — this only works because resistors and dependent sources are *linear*, and it will be the justification for small-signal analysis of amplifiers later in the course. **Thevenin/Norton equivalents** let us replace an arbitrarily complicated linear one-port (two-terminal) network with a single source and a single resistor, which is indispensable for figuring out how a complicated source circuit (e.g., an amplifier's output stage) will behave once you connect an arbitrary load to it, without redoing the whole internal analysis every time the load changes.

Skip this topic and you cannot analyze any circuit with a transistor or op-amp in it (Topics 04, 05, 10 all depend on it), and you're stuck re-solving entire networks from scratch every time a load resistor changes.

## Builds on

- [[6.002-circuits-and-electronics/notes/02-resistive-networks-node-mesh-analysis]] — Kirchhoff's Current Law (KCL: currents into a node sum to zero), Kirchhoff's Voltage Law (KVL: voltage drops around any closed loop sum to zero), Ohm's law $v = iR$, and the node-analysis procedure (pick a reference/ground node, write one KCL equation per remaining node in terms of unknown node voltages, solve the resulting linear system). Because Topic 02 has not been written yet at the time of this note, the exact statement of that procedure is recapped self-containedly below.
- [[6.002-circuits-and-electronics/notes/01-lumped-circuit-abstraction-kvl-kcl]] — the lumped circuit abstraction (elements connected by ideal wires, no fields "leaking" between elements) that justifies applying KVL/KCL at all.

### Self-contained recap of node analysis (from Topic 02)

A circuit is a set of **nodes** (junction points) connected by two-terminal **elements** (resistors, sources). Pick one node as the **reference node** (ground), defined to be at $0\text{ V}$. Every other node $k$ gets an unknown **node voltage** $v_k$, meaning the voltage of that node measured relative to ground. For a resistor of resistance $R$ connecting node $a$ to node $b$, the current flowing from $a$ to $b$ through it is, by Ohm's law,
$$i_{a\to b} = \frac{v_a - v_b}{R}.$$
**KCL** states that at every node, the sum of currents leaving the node through all connected elements equals zero (equivalently, current in = current out — charge cannot pile up at a node in the steady, lumped picture). Writing one such KCL equation per non-reference node, in terms of the unknown node voltages, gives exactly as many independent linear equations as unknowns; solving that linear system gives every node voltage, from which every branch current follows via Ohm's law.

## Core definitions

- **Independent source** — a voltage or current source whose value is fixed (a constant, or a fixed function of time) and does not depend on any other voltage or current in the circuit. Drawn as a circle with $+/-$ (voltage) or an arrow (current) inside.
- **Dependent source** — a voltage or current source whose value is a *scalar multiple* of some other voltage or current measured elsewhere in the same circuit. Drawn as a diamond (to visually distinguish it from independent sources). There are four kinds:
  - **VCVS** (voltage-controlled voltage source): output voltage $v_o = \mu\, v_c$, where $v_c$ is a controlling voltage elsewhere and $\mu$ (dimensionless) is the gain.
  - **VCCS** (voltage-controlled current source): output current $i_o = g_m v_c$, where $g_m$ (units: siemens, S, i.e. A/V) is the **transconductance**.
  - **CCVS** (current-controlled voltage source): $v_o = r_m i_c$, where $r_m$ (units: ohms, Ω) is a transresistance.
  - **CCCS** (current-controlled current source): $i_o = \beta i_c$, where $\beta$ (dimensionless) is a current gain.
- **Controlling variable** — the voltage $v_c$ or current $i_c$ elsewhere in the circuit that a dependent source's value is locked to. It is *not* an independent unknown supplied from outside; it is itself determined by solving the circuit.
- **Linear circuit** — a circuit built only from elements whose defining $v$–$i$ relationship is linear (resistors: $v=iR$; independent sources: fixed; dependent sources: output is a linear — here, proportional — function of a controlling variable). Linear circuits obey superposition (proved below).
- **Superposition** — the response (any node voltage or branch current) of a linear circuit containing multiple independent sources equals the sum of the responses to each independent source acting alone, with every *other* independent source **zeroed**: a zeroed independent voltage source is replaced by a short circuit ($0\text{ V}$, i.e., a plain wire), and a zeroed independent current source is replaced by an open circuit (removed, $0\text{ A}$ branch). Dependent sources are **never** zeroed during superposition — they stay in the circuit for every step, because they are not "sources" of new energy, they're bookkeeping devices tied to the linear structure of the network.
- **One-port network** — any circuit with exactly two exposed terminals, to be connected to the rest of the world (a "load") only through those two terminals.
- **Thevenin equivalent** — for a linear one-port made of resistors and independent/dependent sources, the whole thing is externally indistinguishable from a single independent voltage source $v_{th}$ (the **Thevenin voltage**, equal to the open-circuit voltage across the two terminals) in series with a single resistor $R_{th}$ (the **Thevenin resistance**, the resistance seen at the terminals with all independent sources zeroed).
- **Norton equivalent** — the dual representation: the same one-port is externally indistinguishable from a single independent current source $i_{no}$ (the **Norton current**, equal to the short-circuit current out of the two terminals) in parallel with a resistance $R_{no}$. It turns out $R_{no} = R_{th}$ and $i_{no} = v_{th}/R_{th}$ (proved below), so Thevenin and Norton equivalents are two views of the same underlying pair of numbers $(v_{th}, R_{th})$.
- **Open-circuit voltage** $v_{oc}$ — the voltage across a one-port's terminals when nothing is connected to them (terminal current forced to zero).
- **Short-circuit current** $i_{sc}$ — the current that flows out of one terminal and into the other when the two terminals are connected directly by a wire (terminal voltage forced to zero).

## Intuition

A dependent source is a circuit's way of saying "whatever is happening over *there* gets copied — scaled by some fixed factor — over *here*." It has no mind of its own: if you disconnect the part of the circuit that produces its controlling variable, or that variable happens to be zero, the dependent source outputs exactly zero. That is the single fact that separates it from an independent source, and it's exactly why you must never zero it during superposition — zeroing it would be double-counting the "zeroing" that already happens naturally when its controlling variable becomes zero as part of the algebra.

Superposition itself is just "solve the (still complicated) linear circuit by exploiting linearity of addition." Because every element's law is of the form (output) $=$ (linear combination of other variables), the whole system is a big linear system $A\mathbf{x} = \mathbf{b}$, where $\mathbf{b}$ collects the *independent* source values. Linear systems obey $A(\mathbf{x}_1+\mathbf{x}_2) = A\mathbf{x}_1 + A\mathbf{x}_2$, so the solution for a combined right-hand side $\mathbf{b}_1+\mathbf{b}_2$ is the sum of the solutions for $\mathbf{b}_1$ alone and $\mathbf{b}_2$ alone. Physically: turn on one battery at a time, note what every node voltage does, repeat for each independent source, add up the columns.

Thevenin/Norton equivalence is the statement that, viewed from just two terminals, *no experiment you can perform from outside* can tell the difference between the real (possibly huge) linear network and a trivially simple one-source-one-resistor stand-in. Only two numbers describe everything an outside load could ever see: how hard the network pushes with nothing attached ($v_{oc}$), and how stiffly its output voltage sags as you draw more current ($R_{th}$, the slope of the $v$–$i$ line at the terminals). That is exactly the information a straight line needs — an intercept and a slope — which is why a single source plus a single resistor suffices.

## Derivation / formalism

### 1. Handling dependent sources in node analysis

The node-analysis procedure from Topic 02 is unchanged in structure; dependent sources just add one extra bookkeeping step. Procedure:

1. Choose a reference (ground) node; assign unknown node voltages $v_1,\dots,v_n$ to the rest.
2. For every dependent source, express its value symbolically in terms of node voltages (or currents derived from them) — e.g. if a VCCS has controlling voltage $v_c = v_2 - v_3$ across a known element, write its output current as $g_m(v_2-v_3)$ directly; do **not** introduce it as a new unknown.
3. Write one KCL equation per non-reference node exactly as before, now including these dependent-source terms as functions of the node voltages already in the system.
4. Solve the resulting linear system for $v_1,\dots,v_n$. Because step 2 already substituted the dependent source's value in terms of existing unknowns, no new unknowns and no new equations were introduced — the system size is identical to what it would be if the dependent source were absent (as long as the controlling variable is a node voltage or reducible to one).

This is why dependent sources don't change *how many* equations you write; they change *what's inside* the equations.

### 2. Proof that dependent sources must not be zeroed under superposition

Consider a linear circuit whose independent sources take values $s_1, s_2, \dots, s_m$ (each a voltage or current). Node analysis produces a linear system
$$G\,\mathbf{v} = \mathbf{b}(s_1,\dots,s_m),$$
where $G$ is a conductance-like matrix built only from resistor conductances and dependent-source gains ($\mu, g_m, r_m, \beta$) — it does **not** involve the independent source values at all, because dependent sources are proportional to circuit variables, not fixed numbers — and $\mathbf{b}$ is linear in $s_1,\dots,s_m$ (independent sources enter only as fixed injected currents/voltages on the right-hand side). Because $\mathbf{b}$ is linear in the $s_i$, we can write $\mathbf{b}(s_1,\dots,s_m) = \sum_{i=1}^m \mathbf{b}^{(i)}$, where $\mathbf{b}^{(i)}$ is the contribution with only source $s_i$ nonzero (all other independent sources set to zero — i.e., voltage sources shorted, current sources opened). Then
$$\mathbf{v} = G^{-1}\mathbf{b} = G^{-1}\sum_i \mathbf{b}^{(i)} = \sum_i G^{-1}\mathbf{b}^{(i)} = \sum_i \mathbf{v}^{(i)},$$
where $\mathbf{v}^{(i)} = G^{-1}\mathbf{b}^{(i)}$ is exactly the solution you'd get by solving the circuit with only source $i$ active. This is superposition. Crucially, $G$ itself was left completely untouched throughout — every dependent source's gain is baked into $G$, present in *every* term $\mathbf{v}^{(i)}$, exactly because $G$ never depended on which independent source was "on." Zeroing a dependent source would mean deleting a row/column contribution from $G$ itself, which is not what superposition does and would give the wrong answer (except in the degenerate case where the dependent source's controlling variable happens to be zero anyway in a particular sub-problem, which is a consequence of the correct math, not a rule you impose).

### 3. Deriving the Thevenin equivalent

Claim: any one-port made only of linear resistors and independent/dependent sources has a terminal relationship of the form
$$v = v_{th} - R_{th}\,i \qquad (\star)$$
where $v$ is the terminal voltage, $i$ is the current drawn out of the $+$ terminal into an external load, and $R_{th}, v_{th}$ are constants (independent of the load). The minus sign is required by the stated convention: drawing more load current ($i$ larger) must make the terminal voltage sag, not rise — a real source's output voltage drops as it's asked to supply more current into a load.

*Proof sketch via superposition.* Attach an external test current source to the two terminals, oriented so it pulls a current $i$ out of the $+$ terminal (this is just a stand-in for "whatever load is eventually connected" — for the purposes of finding the equivalent, we probe the port ourselves). Equivalently, this is an independent current source of value $i$ *injected into* the port from outside, i.e. it removes current $i$ from the node at rate $i$. The one-port now has two categories of sources driving it: (a) all the original independent sources inside the network, and (b) this one external test current source. By superposition (Part 2), the terminal voltage is
$$v = \underbrace{v\big|_{\text{internal sources on}, i=0}}_{\text{contribution from internal sources alone}} \;+\; \underbrace{v\big|_{\text{internal sources off}, i}}_{\text{contribution from the test source alone}}.$$
The first term, by definition, is the open-circuit voltage $v_{oc}$ (test current forced to $0$ means the port is open). The second term: with every internal independent source zeroed, the one-port becomes a network of pure resistors and dependent sources with no independent driving — such a network, seen from any pair of terminals, behaves as a single equivalent resistance $R_{eq}$ (dependent sources with zeroed controlling loops that pass through them still just scale existing variables; the network remains linear and source-free, so by Ohm's law its port behaves as $v = -R_{eq} i$ for the externally applied load current $i$ — the minus sign because pulling current $i$ *out of* the $+$ terminal, through a passive resistive network, drops the potential at that terminal, exactly as a resistor's voltage falls in the direction current leaves it). Call this resistance $R_{th}$. So
$$v = v_{oc} - R_{th}\, i,$$
which is $(\star)$ with $v_{th} \equiv v_{oc}$. $\blacksquare$

*Recipe to compute the two numbers:*
- $v_{th} = v_{oc}$: remove the load, compute the voltage across the open terminals directly (ordinary node/mesh analysis with all real sources active).
- $R_{th}$: zero every **independent** source (short independent voltage sources, open independent current sources) — leave dependent sources alone — then compute the equivalent resistance looking into the two terminals. If there are no dependent sources, this is often doable by resistor combination rules (series/parallel). If there *are* dependent sources whose controlling variable lives inside the same one-port, resistor-combination rules can fail (the "resistance" is a coupled effect), and instead you apply a test source method: inject a known test current $i_t$ into the terminals (with all independent sources still zeroed), solve for the resulting terminal voltage $v_t$, and compute $R_{th} = v_t/i_t$.

Equivalently, $R_{th}$ can always be computed as $R_{th} = \dfrac{v_{oc}}{i_{sc}}$, where $i_{sc}$ is the short-circuit current, measured flowing out of the one-port's $+$ terminal (the same direction as $i$ in $(\star)$) — this is the same direction convention used throughout, no redefinition needed. Setting $v=0$ (short the terminals) in $(\star)$ gives
$$0 = v_{th} - R_{th}\,i_{sc} \;\Longrightarrow\; i_{sc} = \frac{v_{th}}{R_{th}} \;\Longrightarrow\; R_{th} = \frac{v_{oc}}{i_{sc}},$$
using $v_{th}\equiv v_{oc}$. Both $v_{oc}$ and $i_{sc}$ come out positive for a source that can actually deliver current in the direction implied by its own $+$ terminal, so this ratio is a positive resistance, exactly as expected — no sign fix-up or opposite-direction convention is needed anywhere in this derivation. This is the standard "$v_{oc}$ over $i_{sc}$" rule, and it is fully equivalent to the zeroed-independent-sources resistance calculation whenever $i_{sc} \ne 0$; the zeroed-source method is preferred whenever it's easier, and is the *only* method that still works if $v_{oc}=0$ (which would make $v_{oc}/i_{sc}$ the indeterminate $0/0$).

### 4. Deriving the Norton equivalent from the Thevenin one

Take $(\star)$: $v = v_{th} - R_{th} i$. Solve for $i$:
$$i = \frac{v_{th}-v}{R_{th}} = \frac{v_{th}}{R_{th}} - \frac{v}{R_{th}}.$$
Define $i_{no} \equiv v_{th}/R_{th}$ and $R_{no}\equiv R_{th}$. Then
$$i = i_{no} - \frac{v}{R_{no}} \quad\Longleftrightarrow\quad i_{no} = i + \frac{v}{R_{no}}.$$
This is exactly the standard KCL-at-the-terminal form: the current $i_{no}$ supplied by a Norton source splits between the load (drawing $i$) and the parallel resistor $R_{no}$ (drawing $v/R_{no}$), i.e. $i_{no} = i + v/R_{no}$, with no sign fix-up needed — the direction convention for $i$ was carried through consistently from $(\star)$. This confirms: a Norton source of value $i_{no} = v_{th}/R_{th}$ in parallel with $R_{no}=R_{th}$ reproduces the identical terminal law $(\star)$. So Thevenin and Norton parameters carry the same information, related by
$$R_{no} = R_{th}, \qquad i_{no} = \frac{v_{th}}{R_{th}}.$$

## Worked examples

### Example 1 — Superposition with two independent sources, no dependent sources yet

Circuit: a $12\text{ V}$ independent voltage source $V_1$ in series with resistor $R_1 = 4\,\Omega$, feeding node $A$; a $2\text{ A}$ independent current source $I_1$ injecting current directly into node $A$; and a resistor $R_2 = 6\,\Omega$ from node $A$ to ground. Find the node voltage $v_A$.

*Direct node analysis (check answer).* KCL at node $A$ (currents leaving $A$ sum to zero): current leaving through $R_2$ to ground is $v_A/R_2$; current entering from the $V_1$–$R_1$ branch is $(V_1 - v_A)/R_1$ (so current *leaving* along that branch toward the source is $-(V_1-v_A)/R_1$); the current source injects $2\text{ A}$ into $A$, i.e. $-2\text{ A}$ "leaving." Sum of leaving currents $=0$:
$$\frac{v_A}{6} - \frac{12 - v_A}{4} - 2 = 0.$$
Multiply through by $12$ (LCM of $6,4$):
$$2v_A - 3(12-v_A) - 24 = 0 \;\Rightarrow\; 2v_A - 36 + 3v_A - 24 = 0 \;\Rightarrow\; 5v_A = 60 \;\Rightarrow\; v_A = 12\text{ V}.$$

*Superposition check.*
- Step A — only $V_1=12\text{ V}$ active, $I_1$ opened (removed): circuit is just $V_1$ through $R_1=4\,\Omega$ into $R_2=6\,\Omega$ to ground, a series loop, so this is a voltage divider: $v_A^{(1)} = V_1 \cdot \dfrac{R_2}{R_1+R_2} = 12\cdot\dfrac{6}{10} = 7.2\text{ V}$.
- Step B — only $I_1 = 2\text{ A}$ active, $V_1$ shorted (replaced by a wire): current source drives node $A$ through the parallel combination of $R_1=4\,\Omega$ (now just a resistor to the shorted, i.e. grounded, source terminal) and $R_2=6\,\Omega$ to ground. Parallel resistance $R_1\parallel R_2 = \dfrac{4\cdot6}{4+6}=\dfrac{24}{10}=2.4\,\Omega$. So $v_A^{(2)} = I_1\cdot(R_1\parallel R_2) = 2\times 2.4 = 4.8\text{ V}$.
- Sum: $v_A = v_A^{(1)}+v_A^{(2)} = 7.2\text{ V} + 4.8\text{ V} = 12.0\text{ V}$. Matches the direct calculation exactly.

### Example 2 — Thevenin equivalent of a network containing a VCCS

Circuit: independent source $V_s = 10\text{ V}$ in series with $R_1=2\,\Omega$ connects to node $1$. From node $1$, resistor $R_2=8\,\Omega$ goes to ground. Also from node $1$, a VCCS with $g_m = 0.5\text{ S}$ pulls a current $g_m v_1$ out of node $1$ to ground (i.e., it is a current source *from* node 1 *to* ground of value $g_m v_1$, controlled by $v_1$ itself, the node-1 voltage). The two output terminals of the one-port are node $1$ and ground. Find $v_{th}$ and $R_{th}$.

*Step 1 — find $v_{th} = v_{oc}$.* Open-circuit means no external load current is drawn from node 1 beyond what's already in the network (the network already has $R_2$ and the VCCS attached at node 1 — those stay; "open circuit at the port" just means we don't attach anything extra). KCL at node 1 (currents leaving node 1 sum to zero): current leaving through $R_2$ is $v_1/8$; current leaving through the VCCS branch is $g_m v_1 = 0.5v_1$; current entering from the source branch is $(V_s-v_1)/R_1 = (10-v_1)/2$, so current leaving along that branch is $-(10-v_1)/2$.
$$\frac{v_1}{8} + 0.5v_1 - \frac{10-v_1}{2} = 0.$$
Multiply by 8:
$$v_1 + 4v_1 - 4(10-v_1) = 0 \;\Rightarrow\; v_1 + 4v_1 - 40 + 4v_1 = 0 \;\Rightarrow\; 9v_1 = 40 \;\Rightarrow\; v_1 = \frac{40}{9}\text{ V} \approx 4.444\text{ V}.$$
So $v_{th} = \dfrac{40}{9}\text{ V} \approx 4.44\text{ V}$.

*Step 2 — find $R_{th}$.* Zero the independent source $V_s$ (short it: node connecting to $R_1$'s far end is now grounded), leave the VCCS alone, and inject a test current $i_t$ into node 1 from outside, then find $v_1$ in terms of $i_t$. With $V_s$ shorted, $R_1=2\,\Omega$ now simply runs from node 1 to ground (a second resistor to ground). KCL at node 1, currents leaving $=$ current entering $=i_t$:
$$\frac{v_1}{R_1} + \frac{v_1}{R_2} + g_m v_1 = i_t \;\Rightarrow\; v_1\Big(\frac{1}{2}+\frac{1}{8}+0.5\Big) = i_t.$$
Compute the coefficient: $\frac12 = 0.5$, $\frac18=0.125$, plus $g_m=0.5$: total $=0.5+0.125+0.5 = 1.125\text{ S}$.
$$v_1 = \frac{i_t}{1.125} \;\Rightarrow\; R_{th} = \frac{v_1}{i_t} = \frac{1}{1.125}\,\Omega = 0.8\overline{8}\,\Omega \approx 0.889\,\Omega.$$

So the whole network is externally equivalent to a $\dfrac{40}{9}\text{ V}\approx 4.44\text{ V}$ source in series with $\dfrac{8}{9}\,\Omega\approx 0.889\,\Omega$. (Sanity check on the fractions: $1/1.125 = 1/(9/8) = 8/9$, consistent.)

*Norton form:* $R_{no}=R_{th}=8/9\,\Omega$, $i_{no}=v_{th}/R_{th} = (40/9)/(8/9) = 40/8 = 5\text{ A}$.

## Common pitfalls

- **Zeroing a dependent source during superposition.** Only *independent* sources get zeroed at each superposition step. A dependent source stays fully in the circuit (with its gain/formula intact) at every step, because its value automatically becomes whatever the (now partially zeroed) controlling variable dictates — you don't force it to zero by hand.
- **Treating a dependent source's controlling variable as an independent unknown.** In node analysis, never introduce a separate unknown for a dependent source's output; substitute its algebraic definition (e.g. $g_m v_1$) directly into the KCL equation using node voltages you already have.
- **Using resistor series/parallel shortcuts for $R_{th}$ when dependent sources are present.** If a dependent source's controlling loop is *inside* the one-port whose $R_{th}$ you're finding, plain series/parallel resistor combination will silently give a wrong answer, because it ignores the extra current the dependent source injects. Use the "zero independent sources, inject test current $i_t$, solve for $v_t/i_t$" method instead (Example 2).
- **Forgetting to also zero independent sources when computing $R_{th}$.** $R_{th}$ is a property of the network with no independent sources active — leaving one on while measuring "resistance" gives a meaningless number (a live source isn't a resistor).
- **Sign/direction confusion for $i_{sc}$.** The relation $R_{th}=v_{oc}/i_{sc}$ requires $i_{sc}$ measured in the direction consistent with $v_{oc}$'s reference polarity (current flowing out of the $+$-labeled terminal through the short). Get the direction backwards and you'll get $R_{th}$ with the wrong sign, or nonsensical negative resistance.
- **Applying $R_{th}=v_{oc}/i_{sc}$ when $v_{oc}=0$.** This gives $0/0$; some networks (symmetric ones, or ones with only dependent sources and no independent ones) genuinely have $v_{oc}=0$. Always fall back on the zeroed-source/test-current method in that case.
- **Forgetting units on $g_m$, $r_m$, $\beta$, $\mu$.** $\mu$ and $\beta$ are dimensionless (V/V or A/A); $g_m$ has units of siemens (A/V); $r_m$ has units of ohms (V/A). Plugging a $g_m$ value into a formula expecting $\mu$ (or vice versa) is a dimension error that node-equation bookkeeping will not automatically catch.
- **Assuming Thevenin/Norton equivalents predict *internal* behavior.** The equivalent circuit reproduces the $v$–$i$ relationship *at the two exposed terminals only*. Internal branch currents/voltages, or power dissipated by the network's own internal resistors, are generally different from what you'd compute from the equivalent circuit's own resistor.

## Self-check

### Questions

1. A one-port has $v_{oc} = 9\text{ V}$ and $i_{sc} = 3\text{ A}$ (short-circuit current flowing out of the $+$ terminal, i.e. in the direction that makes $R_{th}=v_{oc}/i_{sc}$ directly). Find $R_{th}$ and the Norton current $i_{no}$.
2. A CCVS has $r_m = 100\,\Omega$ and controlling current $i_c = 5\text{ mA}$. What voltage does it output? Give units.
3. True or false, with one sentence of justification: "During superposition, you should replace every dependent source with an open or short circuit just like independent sources, since it's also technically a source."
4. A linear one-port contains two independent sources, $V_1$ and $I_1$, and no dependent sources. Using superposition, the contribution of $V_1$ alone to the open-circuit terminal voltage is $3\text{ V}$, and the contribution of $I_1$ alone is $-1\text{ V}$. What is $v_{oc}$ for the full circuit?
5. A one-port's internal network has $R_{th}=5\,\Omega$ and $v_{th}=20\text{ V}$. A load resistor $R_L = 15\,\Omega$ is attached to the terminals. Find the current through $R_L$ and the voltage across it.
6. For the network in Example 2 (Worked Example 2 above), suppose $g_m$ were instead $0\text{ S}$ (i.e., no VCCS, just $R_1, R_2, V_s$). Recompute $v_{th}$ and $R_{th}$ using ordinary voltage-divider / resistor-combination reasoning, and confirm it's consistent with what the general method would give.
7. Explain, using the node-analysis system $G\mathbf v = \mathbf b$ from the derivation, why a circuit containing *only* dependent sources and resistors (no independent sources at all) must have every node voltage equal to zero (assuming the system has a unique solution).
8. A one-port has a dependent source whose controlling variable is a current $i_c$ that happens to flow through a branch *external* to the one-port itself (i.e., $i_c$ is set by whatever load is attached, not by anything internal). Does the standard Thevenin-equivalent derivation in this note still apply unmodified? Why or why not?

### Answers

1. $R_{th} = v_{oc}/i_{sc} = 9\text{ V}/3\text{ A} = 3\,\Omega$. $i_{no}=i_{sc}=3\text{ A}$ directly (by definition, the Norton current is exactly the short-circuit current), which also checks against $i_{no}=v_{th}/R_{th}=9/3=3\text{ A}$. Consistent.
2. $v_o = r_m i_c = 100\,\Omega \times 5\text{ mA} = 100\times 0.005\text{ A} = 0.5\text{ V}$.
3. False. Dependent sources are never zeroed in superposition; only independent sources are. A dependent source's value is tied to another circuit variable and is left as that algebraic expression at every superposition step — it is not an independent driving term and zeroing it would corrupt the matrix $G$ that must stay fixed across all superposition steps (see Derivation Part 2).
4. By superposition, $v_{oc} = 3\text{ V} + (-1\text{ V}) = 2\text{ V}$.
5. By the Thevenin terminal law $v = v_{th} - R_{th}i$ applied with a resistive load $R_L$: the current out of the $+$ terminal and into the load is the same physical current $i$ flowing around this single series loop ($v_{th}$, $R_{th}$, $R_L$), and $v = i R_L$ (Ohm's law across the load). Substituting: $iR_L = v_{th}-R_{th}i \Rightarrow i(R_{th}+R_L) = v_{th} \Rightarrow i = v_{th}/(R_{th}+R_L) = 20/(5+15) = 20/20 = 1\text{ A}$. Voltage across $R_L$: $v_{R_L} = i\times R_L = 1\text{ A}\times 15\,\Omega = 15\text{ V}$. (Check: voltage across $R_{th}$ is $1\text{ A}\times5\,\Omega=5\text{ V}$, and $5\text{ V}+15\text{ V}=20\text{ V}=v_{th}$. Consistent.)
6. With $g_m=0$, node 1 is just a plain voltage divider from $V_s=10\text{ V}$ through $R_1=2\,\Omega$ into $R_2=8\,\Omega$ to ground: $v_{th}=v_{oc}=V_s\cdot\dfrac{R_2}{R_1+R_2}=10\cdot\dfrac{8}{10}=8\text{ V}$. $R_{th}$ with $V_s$ shorted is just $R_1\parallel R_2 = \dfrac{2\times8}{2+8}=\dfrac{16}{10}=1.6\,\Omega$. Check against the general node-equation method: KCL with $V_s$ shorted and test current $i_t$ injected: $v_1(1/R_1+1/R_2)=i_t \Rightarrow v_1(0.5+0.125)=i_t\Rightarrow v_1/i_t = 1/0.625=1.6\,\Omega$. Matches.
7. If there are no independent sources, every entry of $\mathbf b$ is zero (recall $\mathbf b$ is built only from independent source values), so the system is $G\mathbf v = \mathbf 0$. If $G$ is invertible (system has a unique solution), the only solution is $\mathbf v = G^{-1}\mathbf 0 = \mathbf 0$ — every node voltage is exactly zero. Physically: with no independent driving term anywhere, there is nothing to establish any voltage difference; a purely resistive/dependent-source network with zero input produces zero output (this is exactly why dependent sources cannot "create" a response on their own — they only scale/redirect a response that some independent source elsewhere already created).
8. No — not unmodified. The whole derivation assumed the one-port's internal behavior, encoded in $G$, does not depend on what's attached externally, so that $R_{th}$ is a fixed number characterizing the port alone. If the dependent source's controlling variable is itself set by the external load (e.g. a current that only exists once you attach a specific load), then $G$ effectively changes depending on the load, and the port no longer has a fixed, load-independent Thevenin/Norton equivalent — the "resistance seen at the terminals" would depend on what you're measuring it with, which contradicts the definition of $R_{th}$. In practice, well-posed Thevenin/Norton problems always have all dependent-source controlling variables strictly internal to the one-port being reduced.

## Summary / cheat sheet

- **Dependent sources**: VCVS $v_o=\mu v_c$ (μ dimensionless), VCCS $i_o=g_m v_c$ ($g_m$ in S), CCVS $v_o=r_m i_c$ ($r_m$ in Ω), CCCS $i_o=\beta i_c$ (β dimensionless). Controlling variable is itself a circuit unknown, never introduce it as a separate free variable in node analysis — substitute directly.
- **Node analysis with dependent sources**: identical procedure to Topic 02; just write the dependent source's algebraic value (in terms of existing node voltages/currents) directly into the KCL equations. No new unknowns, no new equations.
- **Superposition** (linear circuits only): response $=\sum_i$ (response to independent source $i$ alone, all *other independent* sources zeroed — voltage sources shorted, current sources opened). **Never zero dependent sources.**
- **Thevenin equivalent**: any linear resistive one-port $\Leftrightarrow$ $v_{th}$ in series with $R_{th}$, where terminal law is $v=v_{th}-R_{th}i$ ($i$ = current out of the $+$ terminal into the load).
  - $v_{th}=v_{oc}$: open-circuit terminal voltage, all real (independent) sources active.
  - $R_{th}$: resistance at terminals with all independent sources zeroed (dependent sources stay). Use resistor combination if no internal dependent-source coupling to the port; otherwise inject test current $i_t$, solve for $v_t$, $R_{th}=v_t/i_t$.
  - Shortcut when valid ($v_{oc}\ne0$): $R_{th}=v_{oc}/i_{sc}$.
- **Norton equivalent**: $i_{no}$ in parallel with $R_{no}$, where $R_{no}=R_{th}$ and $i_{no}=v_{th}/R_{th}=i_{sc}$.
- Thevenin/Norton equivalence only guarantees matching **external, terminal** behavior — not internal voltages/currents/power of the original network.

## Used later in
(filled in once a later note actually cites this one)
