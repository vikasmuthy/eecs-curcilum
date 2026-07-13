---
title: "Nonlinear Elements, Digital Abstraction, MOSFET Model"
course: "6.002"
topic_number: 04
prerequisites: ["01-lumped-circuit-abstraction-kvl-kcl", "02-resistive-networks-node-mesh-analysis", "03-dependent-sources-superposition-thevenin-norton"]
status: in progress
---

# Nonlinear Elements, Digital Abstraction, MOSFET Model

## Why this matters

Every circuit analyzed so far (topics 01-03) used elements — resistors, independent sources, dependent sources — whose current-voltage (i-v) relationship is a straight line or a simple linear combination. Real switching devices, and every digital logic gate ever built, are made from **nonlinear** elements: their i-v curve is not a line, and that curvature is exactly what makes them useful. A resistor cannot amplify a signal or act as a switch; a transistor can, because its behavior changes character (from "off" to "on," or from "sensitive to input" to "insensitive to input") over different operating regions.

This note does three things that unlock the rest of 6.002:

1. Extends KVL/KCL analysis to elements with nonlinear i-v curves, via the graphical/algebraic *load-line* method.
2. Introduces the **digital abstraction** — the idea that a continuous voltage signal can be reliably interpreted as one of two discrete logic values (0 or 1), *provided* the circuit elements satisfy certain gain and noise-margin conditions. This is the conceptual bridge between analog circuits and the boolean logic of 6.004.
3. Introduces the **MOSFET** (Metal-Oxide-Semiconductor Field-Effect Transistor) and its simplified switch-level and piecewise-linear models, which are the building block used to *implement* the digital abstraction in real hardware (and which reappear in topic 05 as the amplifying element).

Skip this topic and nothing after it makes sense: amplifiers (topic 05) are MOSFETs biased into a specific operating region; digital gates (6.004) are MOSFETs used in a different region; and the entire discipline of treating a wire's voltage as "a 1 or a 0" — which every computer depends on — rests on the noise-margin argument developed here.

## Builds on

- [[6.002-circuits-and-electronics/notes/01-lumped-circuit-abstraction-kvl-kcl]] — KVL (sum of voltage drops around any closed loop = 0) and KCL (sum of currents into any node = 0), and the lumped-element assumption that lets us describe any two-terminal device purely by its i-v relationship.
- [[6.002-circuits-and-electronics/notes/02-resistive-networks-node-mesh-analysis]] — Ohm's law $v = iR$, node analysis, and the idea of solving a circuit by combining device equations with KVL/KCL constraints.
- [[6.002-circuits-and-electronics/notes/03-dependent-sources-superposition-thevenin-norton]] — Thevenin equivalent circuits (any linear one-port reduces to a series combination of a voltage source $v_{TH}$ and resistance $R_{TH}$), used below to replace the linear part of a circuit before solving against a nonlinear element.

No knowledge of semiconductor physics (that belongs to 6.012, not yet written) is assumed. Everything about *why* a MOSFET behaves as it does is recapped from scratch in this note, at the level of a terminal model — a set of equations relating terminal voltages to terminal current — not a physical device-physics derivation.

## Core definitions

- **Nonlinear element** — a circuit element whose i-v relationship is not of the form $i = v/R$ (a single straight line through characteristics consistent with a constant resistance). Its i-v curve can bend, saturate, or have multiple slopes.
- **Operating point (Q-point)** — the specific $(v, i)$ pair at which a device actually sits once it is embedded in a particular circuit; the intersection of the device's i-v curve with the constraint imposed by the rest of the circuit.
- **Load line** — the straight line, in the $i$-$v$ plane of a nonlinear device, representing the constraint imposed on that device by the linear rest-of-circuit (typically obtained by reducing the rest of the circuit to its Thevenin equivalent). The load line's equation comes from KVL applied to the loop containing the device and its Thevenin source/resistance.
- **Digital abstraction** — the engineering convention of dividing the continuous range of voltages into two disjoint bands, one interpreted as logic "0" and one as logic "1," with a forbidden band in between that a valid signal should never rest in (except transiently while switching).
- **Gain** — the ratio of a circuit's output signal change to its input signal change (output/input); a gate with gain greater than 1 in magnitude in its transition region actively restores a degraded ("noisy") input toward the clean output levels rather than merely passing the degradation through, which is what lets errors from stage to stage not accumulate as signals pass through many gates in cascade.
- **Logic levels** $V_{OL}, V_{OH}, V_{IL}, V_{IH}$ — four threshold voltages that define the digital abstraction for a given gate family:
  - $V_{OL}$ = maximum voltage a gate is guaranteed to *output* when driving a logic 0.
  - $V_{OH}$ = minimum voltage a gate is guaranteed to *output* when driving a logic 1.
  - $V_{IL}$ = maximum voltage a gate is guaranteed to still *interpret as* a logic 0 at its input.
  - $V_{IH}$ = minimum voltage a gate is guaranteed to still *interpret as* a logic 1 at its input.
- **Static discipline** — the requirement that a valid gate satisfy $V_{OL} \le V_{IL}$ and $V_{OH} \ge V_{IH}$, so that one gate's worst-case output is still safely inside the next gate's input tolerance.
- **Noise margin** — the "slack" between a gate's guaranteed output level and the next gate's required input level: $NM_L = V_{IL} - V_{OL}$ (low-side) and $NM_H = V_{OH} - V_{IH}$ (high-side). Positive noise margins mean the circuit tolerates some added noise/distortion on the wire between gates without misinterpreting the bit.
- **MOSFET** — a four-terminal (often treated as three-terminal, with the fourth grounded to the source or ignored) semiconductor device with terminals **gate (G)**, **drain (D)**, **source (S)**, and **body/substrate (B)**. In this note we use the **n-channel enhancement-mode MOSFET (NMOS)**, the type used in the simplified switch model.
- **Threshold voltage** $V_T$ — the gate-to-source voltage $v_{GS}$ below which the MOSFET conducts (approximately) no drain current; the switch-on threshold.
- **Switch-level (SR) MOSFET model** — the crudest useful model: the MOSFET is an ideal open switch between D and S when $v_{GS} < V_T$, and an ideal closed switch with a fixed "on" resistance $R_{ON}$ when $v_{GS} \ge V_T$. G draws no current in this model (the gate is treated as an ideal, infinite-impedance control input).
- **Piecewise-linear (incremental / regions-of-operation) MOSFET model** — a refinement used later (topic 05) with three regions: cutoff ($v_{GS} < V_T$, $i_D = 0$), triode/linear ($v_{GS} \ge V_T$ and $v_{DS} < v_{GS} - V_T$), and saturation ($v_{GS} \ge V_T$ and $v_{DS} \ge v_{GS} - V_T$). This note derives and uses the equations for all three regions but only the cutoff/switch behavior and static-discipline consequences are used for digital-abstraction purposes; the saturation region's use as an amplifier is deferred to topic 05.

## Intuition

**Nonlinear i-v curves and the load line.** A resistor's i-v curve is a straight line through the origin — double the voltage, double the current, always, everywhere. A nonlinear device's curve can look like almost anything: flat (no current no matter the voltage, i.e. "off"), a step (jumps from no current to a lot of current past some threshold voltage, i.e. "on/off switch"), or something in between with curvature (like a diode, covered in a later topic). Because KVL and KCL are always true regardless of whether the elements are linear, we can still write down the constraint the rest of the circuit places on our nonlinear element — that constraint is always a straight line (the load line) *if the rest of the circuit is linear*, because Thevenin's theorem (topic 03) says any linear one-port looks like a source in series with a resistor, and a source-plus-resistor constraint is a line in the $i$-$v$ plane. The actual operating point is wherever the device's own (possibly curved) i-v curve crosses that line. This is just "solve two equations in two unknowns," but visualized graphically it becomes "find the intersection of two curves."

**Digital abstraction.** A wire's voltage is, physically, a continuous quantity — it could be any value, drift with temperature, pick up noise, etc. Digital logic wants to treat that same wire as carrying exactly one bit: 0 or 1, nothing in between. The trick that makes this legitimate is *not* pretending the voltage is exactly 0 V or exactly 5 V; it's agreeing on a band of voltages near 0 that everyone calls "0" and a band near the supply voltage that everyone calls "1," with enough of a gap between the bands (and enough gain in the devices used to build gates) that even after picking up some noise or distortion, a signal that started in the "0" band is still read as "0" by the next gate, and likewise for "1." The static discipline conditions are exactly the algebraic statement of "the bands don't overlap and there's margin left over."

**MOSFET as a voltage-controlled switch.** Physically (deferred to 6.012 for the "why"), an NMOS transistor has a channel between its drain and source terminals whose conductivity is controlled by the voltage on the gate terminal, relative to the source. Below a threshold gate voltage, the channel does not form and no current flows D-to-S no matter what voltage is put across D and S (within reason) — this is "off." Above threshold, a channel forms and current can flow D-to-S, with the amount of current depending on both $v_{GS}$ (how far above threshold, i.e. how "open" the switch is) and $v_{DS}$ (how hard current is being pushed through it). The crucial feature exploited by digital logic: the gate terminal, in this idealized model, draws *no* current — controlling the switch costs no current from whatever circuit is driving the gate. That is what makes a MOSFET a good building block for cascaded logic gates: each gate's output only has to supply current to charge/discharge wires and drive the *next* gate's channel resistance, not to feed a continuous current into the next gate's control terminal.

## Derivation / formalism

### 1. Solving a circuit containing one nonlinear element: the load-line method

Consider a circuit consisting of a single nonlinear two-terminal element (call its terminal voltage $v$ and terminal current $i$, with $i$ defined flowing into the $+$-marked terminal) embedded in an otherwise-linear network.

**Step 1 — Isolate the nonlinear element.** Treat the nonlinear element as the "load," and treat everything else in the circuit (all resistors and independent/dependent sources) as a linear one-port "source" network attached to its two terminals.

**Step 2 — Reduce the linear part to its Thevenin equivalent.** By the Thevenin theorem (topic 03), the linear one-port is equivalent to a single voltage source $v_{TH}$ in series with a single resistance $R_{TH}$, both computed by the standard Thevenin procedure (find $v_{TH}$ as the open-circuit voltage at the two terminals with the nonlinear element removed; find $R_{TH}$ by zeroing independent sources and measuring the two-terminal resistance, or via $R_{TH} = v_{TH}/i_{SC}$ using the short-circuit current $i_{SC}$).

**Step 3 — Write KVL around the resulting loop.** The loop now contains: the Thevenin source $v_{TH}$, the Thevenin resistance $R_{TH}$, and the nonlinear element with terminal voltage $v$ and current $i$. Going around the loop and summing voltage drops to zero:
$$v_{TH} = i R_{TH} + v$$

Solve for $i$ in terms of $v$:
$$i = \frac{v_{TH} - v}{R_{TH}} = -\frac{1}{R_{TH}} v + \frac{v_{TH}}{R_{TH}}$$

This is the **load line**: a straight line in the $(v, i)$ plane with slope $-1/R_{TH}$ and $i$-intercept $v_{TH}/R_{TH}$ (the value of $i$ when $v=0$) and $v$-intercept $v_{TH}$ (the value of $v$ when $i=0$, i.e. the open-circuit condition).

**Step 4 — Impose the device equation.** The nonlinear element itself obeys some function $i = f(v)$ (its i-v characteristic, given as a graph, a table, or a formula depending on the device). The actual operating point $(V_Q, I_Q)$ must satisfy *both* the load-line equation and the device equation simultaneously:
$$I_Q = \frac{v_{TH} - V_Q}{R_{TH}} \quad \text{and} \quad I_Q = f(V_Q)$$

Graphically this is the intersection of the device's i-v curve with the load line. Algebraically, substitute $f(V_Q)$ for $I_Q$ in the first equation and solve for $V_Q$:
$$f(V_Q) = \frac{v_{TH} - V_Q}{R_{TH}}$$
This is one equation in one unknown $V_Q$ (though it may be transcendental or piecewise, depending on $f$); once solved, $I_Q = f(V_Q)$ gives the current.

### 2. The switch-level MOSFET model, from terminal behavior

We now specialize the "nonlinear element" to an NMOS transistor, using only its terminal (input-output) behavior — no device physics required. Terminals: gate $G$, drain $D$, source $S$. Define $v_{GS} = v_G - v_S$ (gate-to-source voltage, the *control* variable) and $v_{DS} = v_D - v_S$ (drain-to-source voltage), and $i_D$ = current flowing into the drain terminal (by convention, current flows in at D and out at S for an NMOS operated normally).

**Switch-level model, by definition:**
$$
i_D =
\begin{cases}
0 & \text{if } v_{GS} < V_T \quad (\text{"cutoff," switch open}) \\
v_{DS}/R_{ON} & \text{if } v_{GS} \ge V_T \quad (\text{"on," switch closed with resistance } R_{ON})
\end{cases}
$$
and, in this simplified model, the gate current $i_G = 0$ always (the gate terminal is treated as a perfect, current-free control input — no channel connects G to D or S).

This is exactly a nonlinear i-v element (for the $D$-$S$ port) whose i-v curve is a function of a third variable, $v_{GS}$: for $v_{GS} < V_T$ the $i_D$-$v_{DS}$ curve is the horizontal line $i_D = 0$ (open switch, infinite resistance); for $v_{GS} \ge V_T$ it is a straight line of slope $1/R_{ON}$ through the origin (closed switch with resistance $R_{ON}$). Solving a circuit with a MOSFET therefore means: (a) first determine $v_{GS}$ from the rest of the circuit to decide *which* i-v curve applies (which region the device is in), then (b) apply the load-line method of Section 1 using that curve.

### 3. The piecewise-linear (three-region) MOSFET model

The switch-level model is a special case (with $R_{ON} \to$ a constant) of a richer model that also captures how $i_D$ depends on both $v_{GS}$ and $v_{DS}$ once the switch is "on." State it as the standard three regions (this is the model as commonly used in 6.002-level analysis, itself a simplification of the full physical device equations from 6.012):

- **Cutoff:** $v_{GS} < V_T \implies i_D = 0$.
- **Triode / linear region:** $v_{GS} \ge V_T$ and $0 \le v_{DS} < v_{GS}-V_T$:
$$i_D = K\left[(v_{GS}-V_T)v_{DS} - \frac{v_{DS}^2}{2}\right]$$
- **Saturation region:** $v_{GS} \ge V_T$ and $v_{DS} \ge v_{GS}-V_T$:
$$i_D = \frac{K}{2}(v_{GS}-V_T)^2$$

Here $K$ (units A/V$^2$) is a device/process-dependent constant (sometimes written $K = k_n' \frac{W}{L}$ using channel width $W$ and length $L$; the origin of $K$ is device physics and is *not* derived here — we take $K$ and $V_T$ as given device parameters, exactly the way we take $R$ as a given parameter for a resistor). Two facts used repeatedly below:

- **Continuity check at the triode/saturation boundary.** At the boundary $v_{DS} = v_{GS}-V_T$, both formulas must agree (a physical device's current can't jump discontinuously as $v_{DS}$ crosses this line). Substitute $v_{DS} = v_{GS}-V_T$ into the triode formula:
$$i_D = K\left[(v_{GS}-V_T)(v_{GS}-V_T) - \frac{(v_{GS}-V_T)^2}{2}\right] = K(v_{GS}-V_T)^2\left[1-\frac12\right] = \frac{K}{2}(v_{GS}-V_T)^2$$
which matches the saturation formula exactly. This confirms the model is self-consistent (continuous) at the boundary.
- **Cutoff is the $v_{GS} \to V_T$ limit of either region.** As $v_{GS} \to V_T^+$, saturation current $\frac{K}{2}(v_{GS}-V_T)^2 \to 0$, matching cutoff's $i_D=0$. Again continuous.

For this note's purposes (digital abstraction), only **cutoff** and the fact that in triode with small $v_{DS}$ the device behaves approximately resistively (used to justify $R_{ON}$ in the switch model — set $v_{DS}\to 0$ in the triode formula: $i_D \approx K(v_{GS}-V_T)v_{DS}$, i.e. $i_D/v_{DS} \approx K(v_{GS}-V_T)$, a conductance that depends on $v_{GS}$, so $R_{ON} \approx \frac{1}{K(v_{GS}-V_T)}$ for a fixed "on" gate drive) are needed. Saturation-region use as an amplifying element is developed in topic 05.

### 4. Static discipline as an algebraic condition

Consider two identical inverting logic gates in cascade, gate 1 driving gate 2. Gate 1, when its input is at logic 1, is specified to produce an output voltage no higher than... (for an inverting gate outputting logic 0) $V_{OL}$; when its input is at logic 0, an output no lower than $V_{OH}$. Gate 2 is specified to correctly interpret any input voltage at or below $V_{IL}$ as logic 0, and any input at or above $V_{IH}$ as logic 1. For the cascade to work correctly under all guaranteed conditions:

$$V_{OL} \le V_{IL} \qquad \text{and} \qquad V_{OH} \ge V_{IH}$$

If either inequality failed — say $V_{OL} > V_{IL}$ — then gate 1's worst-case "0" output ($V_{OL}$) could sit *above* gate 2's threshold for accepting a "0" ($V_{IL}$), meaning gate 2 might read a legitimately-driven "0" as a "1." The inequalities are exactly the no-misread condition. Noise margins quantify the slack:
$$NM_L = V_{IL}-V_{OL} \ge 0, \qquad NM_H = V_{OH}-V_{IH} \ge 0$$
Larger noise margins mean more tolerance for voltage drop, coupling, or other analog imperfections on the wire between gates before a bit is misread.

## Worked examples

### Example 1 — Load line and operating point for a switch-level MOSFET

A circuit has a $5\ \text{V}$ ideal voltage source in series with a $2\ \text{k}\Omega$ resistor $R_D$, driving the drain of an NMOS transistor whose source is grounded ($v_S = 0\ \text{V}$). The gate is driven to $v_{GS} = 3\ \text{V}$. The transistor's switch-level parameters are $V_T = 1\ \text{V}$ and $R_{ON} = 500\ \Omega$. Find the drain current $i_D$ and the drain-source voltage $v_{DS}$.

**Step 1 — Determine the region.** $v_{GS} = 3\ \text{V} \ge V_T = 1\ \text{V}$, so the transistor is "on": $i_D = v_{DS}/R_{ON}$.

**Step 2 — Find the Thevenin equivalent seen by the transistor's D-S port.** Looking from the D-S terminals back into the rest of the circuit: the $5\ \text{V}$ source and $2\ \text{k}\Omega = 2000\ \Omega$ resistor are already a series voltage-source/resistor combination — that *is* the Thevenin equivalent directly (no further reduction needed): $v_{TH} = 5\ \text{V}$, $R_{TH} = 2000\ \Omega$.

**Step 3 — Write the load line.** Let $v = v_{DS}$, $i = i_D$ (current flows from the $5\ \text{V}$ source, through $R_D$, into the drain, and out the source to ground — consistent sign convention with $i_D$ flowing into the drain). KVL around the loop:
$$5\ \text{V} = i_D (2000\ \Omega) + v_{DS}$$
$$i_D = \frac{5 - v_{DS}}{2000}$$
(units: volts divided by ohms = amps, throughout).

**Step 4 — Impose the device equation and solve.** Device equation (on-state): $i_D = v_{DS}/500$. Set equal:
$$\frac{v_{DS}}{500} = \frac{5-v_{DS}}{2000}$$
Multiply both sides by $2000$:
$$4\, v_{DS} = 5 - v_{DS}$$
$$5\, v_{DS} = 5$$
$$v_{DS} = 1\ \text{V}$$
Then:
$$i_D = \frac{v_{DS}}{500\ \Omega} = \frac{1\ \text{V}}{500\ \Omega} = 0.002\ \text{A} = 2\ \text{mA}$$

**Check:** plug back into the load-line equation: $i_D = (5-1)/2000 = 4/2000 = 0.002\ \text{A} = 2\ \text{mA}$. Matches. So $Q$-point is $(v_{DS}, i_D) = (1\ \text{V}, 2\ \text{mA})$.

### Example 2 — Checking the static discipline / noise margins for a gate family

A family of logic gates, when driven by a $5\ \text{V}$ supply, is specified with:
$$V_{OL} = 0.4\ \text{V}, \quad V_{OH} = 4.4\ \text{V}, \quad V_{IL} = 1.5\ \text{V}, \quad V_{IH} = 3.5\ \text{V}$$

(a) Verify the static discipline holds. (b) Compute both noise margins. (c) A downstream wire, due to resistive coupling from a neighboring signal, adds up to $0.9\ \text{V}$ of unwanted offset to a logic-0 signal being transmitted (worst case, offset is always in the "wrong" direction, i.e. it can push the voltage up). Determine whether the digital abstraction still holds for this wire.

**(a) Static discipline.** Check $V_{OL} \le V_{IL}$: $0.4\ \text{V} \le 1.5\ \text{V}$ — true. Check $V_{OH} \ge V_{IH}$: $4.4\ \text{V} \ge 3.5\ \text{V}$ — true. Both hold, so the static discipline is satisfied.

**(b) Noise margins.**
$$NM_L = V_{IL} - V_{OL} = 1.5\ \text{V} - 0.4\ \text{V} = 1.1\ \text{V}$$
$$NM_H = V_{OH} - V_{IH} = 4.4\ \text{V} - 3.5\ \text{V} = 0.9\ \text{V}$$

**(c) Effect of a 0.9 V offset on a logic-0 signal.** Driving gate guarantees output $\le V_{OL} = 0.4\ \text{V}$ for a logic 0. With up to $0.9\ \text{V}$ of added offset, the voltage arriving at the receiving gate could be as high as:
$$0.4\ \text{V} + 0.9\ \text{V} = 1.3\ \text{V}$$
The receiving gate is only guaranteed to still interpret a voltage as logic 0 if it is $\le V_{IL} = 1.5\ \text{V}$. Since $1.3\ \text{V} \le 1.5\ \text{V}$, the signal is still correctly read as logic 0 — the abstraction survives this offset, using $1.5 - 1.3 = 0.2\ \text{V}$ of the original $1.1\ \text{V}$ low-side noise margin ($0.9\ \text{V}$ of the margin was "used up" by the offset). Had the offset instead been, say, $1.2\ \text{V}$ (exceeding $NM_L = 1.1\ \text{V}$), the arriving voltage would be $0.4+1.2=1.6\ \text{V} > V_{IL}=1.5\ \text{V}$, and the receiving gate would no longer be guaranteed to read it as a 0 — the abstraction would break.

## Common pitfalls

- **Forgetting to reduce the rest of the circuit to its Thevenin equivalent before drawing the load line.** The load-line method as derived here requires the *linear* part of the circuit to be reduced to a single source + single resistor; skipping this and trying to write KVL directly around a multi-resistor network alongside the nonlinear device usually leads to needing to solve a larger, avoidable system.
- **Mixing up which region a MOSFET is in before applying an equation.** Each of cutoff/triode/saturation has its own formula; plugging $v_{GS}, v_{DS}$ into the triode formula when the device is actually in saturation (or vice versa) gives a wrong, and sometimes non-physical (e.g. negative or discontinuous), answer. Always check the region conditions ($v_{GS}$ vs. $V_T$, and $v_{DS}$ vs. $v_{GS}-V_T$) first.
- **Sign/direction confusion for $i_D$.** The convention here is $i_D$ flows into the drain and out the source for an NMOS in normal operation; getting this backward flips the sign of every downstream KVL equation.
- **Assuming the MOSFET gate always draws zero current in every model.** That is true in the switch-level and piecewise-linear models used in 6.002 (an idealization), but not exactly true in real devices (a small **leakage current** — unwanted current that leaks through insulation or reverse-biased junctions that are ideally supposed to block current entirely — flows into the gate) — don't over-generalize the idealization to "MOSFETs literally never draw gate current under any circumstance."
- **Confusing $V_{IL}/V_{IH}$ (input thresholds, properties of the *receiving* gate) with $V_{OL}/V_{OH}$ (output guarantees, properties of the *driving* gate).** All four are different circuit parameters; the static-discipline inequalities only make sense if you keep straight which pair belongs to which gate role.
- **Treating "positive noise margin" as sufficient for a working circuit under all conditions.** The static discipline as derived here is a worst-case, DC (steady-state) condition. Fast switching transients, capacitive coupling, and multiple noise sources adding up can still cause errors even with positive margins on paper — this note only develops the static (DC) discipline, not full noise analysis.
- **Forgetting units when computing $R_{ON}$ or $K$.** $R_{ON}$ is in ohms ($\Omega$) as usual; $K$ has units of $\text{A/V}^2$ (amps per volt squared) because it multiplies a voltage-squared term to give a current — dropping this unit and treating $K$ as dimensionless is a common algebra-tracking error.

## Self-check

### Questions

1. (Easy) Define, in one sentence each, $V_{OL}$ and $V_{IH}$, and state which one belongs to a driving gate vs. a receiving gate.
2. (Easy) An NMOS transistor has $V_T = 0.8\ \text{V}$. If $v_{GS} = 0.5\ \text{V}$, what region is it in, and what is $i_D$ in the switch-level model?
3. (Medium) A nonlinear device has i-v curve $i = 2v^2$ (for $v \ge 0$, in amps with $v$ in volts). It's connected to a Thevenin source with $v_{TH} = 6\ \text{V}$ and $R_{TH} = 1\ \Omega$. Write the load-line equation and set up (but do not necessarily solve) the equation for the operating point voltage $V_Q$.
4. (Medium) For the device/circuit in Question 3, solve for $V_Q$ (there are two mathematical roots of the resulting quadratic — determine which one is physically valid and why).
5. (Medium) A gate family has $V_{OL}=0.5\ \text{V}$, $V_{OH}=4.0\ \text{V}$, $V_{IL}=1.5\ \text{V}$, $V_{IH}=3.0\ \text{V}$. Compute both noise margins.
6. (Hard) Using the piecewise-linear model with $K = 4\ \text{mA/V}^2$ and $V_T = 1\ \text{V}$, an NMOS has $v_{GS}=3\ \text{V}$. At what value of $v_{DS}$ does the device transition from triode to saturation, and what is $i_D$ at that transition point (compute using both the triode and saturation formulas and confirm they agree)?
7. (Hard) In Example 1's circuit (5 V source, $R_D = 2\ \text{k}\Omega$, switch-level NMOS with $V_T=1\ \text{V}$, $R_{ON}=500\ \Omega$), suppose $v_{GS}$ is lowered to $0.5\ \text{V}$ instead of $3\ \text{V}$. Find the new operating point $(v_{DS}, i_D)$.
8. (Hard) Explain, using the noise-margin concept, why a gate family with $V_{OL} = V_{IL}$ exactly (zero low-side noise margin) is a poor engineering choice even though it technically satisfies the static discipline inequality $V_{OL}\le V_{IL}$.

### Answers

1. $V_{OL}$ = the maximum voltage a *driving* gate is guaranteed to output when its output is a logic 0. $V_{IH}$ = the minimum voltage a *receiving* gate is guaranteed to still interpret as a logic 1 at its input. $V_{OL}$ belongs to the driving gate's output specification; $V_{IH}$ belongs to the receiving gate's input specification.
2. Since $v_{GS} = 0.5\ \text{V} < V_T = 0.8\ \text{V}$, the transistor is in cutoff (off). In the switch-level model, $i_D = 0\ \text{A}$.
3. Load line from KVL: $v_{TH} = i R_{TH} + v \Rightarrow 6 = i(1) + v \Rightarrow i = 6-v$. Setting the device equation equal: $2v^2 = 6-v$, i.e. $2v^2 + v - 6 = 0$ is the equation to solve for $V_Q = v$.
4. Solve $2v^2+v-6=0$ using the quadratic formula, $v = \frac{-b\pm\sqrt{b^2-4ac}}{2a}$ with $a=2,b=1,c=-6$: $v = \frac{-1\pm\sqrt{1-4(2)(-6)}}{4} = \frac{-1\pm\sqrt{1+48}}{4} = \frac{-1\pm 7}{4}$. Roots: $v = \frac{6}{4}=1.5$ or $v=\frac{-8}{4}=-2$. Since the device equation $i=2v^2$ was specified for $v \ge 0$, and the physical setup (positive source driving the load) implies $v\ge 0$ operating region, the valid root is $V_Q = 1.5\ \text{V}$ (giving $I_Q = 2(1.5)^2 = 4.5\ \text{A}$, and checking the load line: $i=6-1.5=4.5\ \text{A}$ — consistent). $v=-2\ \text{V}$ is rejected as unphysical for this circuit.
5. $NM_L = V_{IL}-V_{OL} = 1.5-0.5 = 1.0\ \text{V}$. $NM_H = V_{OH}-V_{IH} = 4.0-3.0 = 1.0\ \text{V}$.
6. Transition (triode/saturation boundary) occurs at $v_{DS} = v_{GS}-V_T = 3-1 = 2\ \text{V}$. Using saturation formula: $i_D = \frac{K}{2}(v_{GS}-V_T)^2 = \frac{0.004}{2}(2)^2 = 0.002 \times 4 = 0.008\ \text{A} = 8\ \text{mA}$. Using triode formula at $v_{DS}=2\ \text{V}$: $i_D = K[(v_{GS}-V_T)v_{DS} - v_{DS}^2/2] = 0.004[(2)(2) - (4/2)] = 0.004[4-2] = 0.004(2) = 0.008\ \text{A} = 8\ \text{mA}$. Both formulas agree at $8\ \text{mA}$, confirming continuity at the boundary.
7. $v_{GS}=0.5\ \text{V} < V_T=1\ \text{V}$, so the transistor is in cutoff: $i_D = 0\ \text{A}$ (open switch, no current can flow through $R_D$ either since it's the same series loop). With $i_D=0$, the KVL loop $5 = i_D(2000)+v_{DS}$ gives $v_{DS} = 5\ \text{V} - 0 = 5\ \text{V}$ (all the source voltage appears across the open switch; none is dropped across $R_D$). Operating point: $(v_{DS}, i_D) = (5\ \text{V}, 0\ \text{A})$.
8. Zero noise margin means there is no tolerance whatsoever for any additional voltage drop, coupling, offset, or component-to-component variation between the driving gate's worst-case output and the receiving gate's threshold. Any infinitesimal amount of real-world noise, IR drop along a wire, or manufacturing variation that pushes the driven "0" signal even slightly above $V_{OL}=V_{IL}$ would cause a misread. Good engineering practice requires positive margin ($V_{OL} < V_{IL}$ strictly, with margin large compared to realistic noise sources) precisely because real circuits are never perfectly ideal; "technically satisfies the inequality with equality" provides no safety margin against any real-world imperfection.

## Summary / cheat sheet

**Load-line method** for a nonlinear element embedded in a linear circuit:
1. Reduce rest-of-circuit to Thevenin equivalent: $v_{TH}, R_{TH}$.
2. Load line (from KVL): $i = \dfrac{v_{TH}-v}{R_{TH}}$.
3. Operating point = intersection with device equation $i=f(v)$: solve $f(v) = \dfrac{v_{TH}-v}{R_{TH}}$ for $v$, then get $i$.

**Digital abstraction / static discipline:**
- Four levels: $V_{OL}$ (max guaranteed output-0), $V_{OH}$ (min guaranteed output-1), $V_{IL}$ (max input still read as 0), $V_{IH}$ (min input still read as 1).
- Static discipline: $V_{OL}\le V_{IL}$ and $V_{OH}\ge V_{IH}$.
- Noise margins: $NM_L = V_{IL}-V_{OL} \ge 0$, $NM_H = V_{OH}-V_{IH} \ge 0$. Larger = more robust.

**NMOS switch-level model** ($v_{GS}=v_G-v_S$, $v_{DS}=v_D-v_S$, $i_D$ into drain, $i_G=0$ always):
$$i_D = \begin{cases} 0 & v_{GS}<V_T \ \text{(cutoff, open switch)} \\ v_{DS}/R_{ON} & v_{GS}\ge V_T \ \text{(on, closed switch)}\end{cases}$$

**NMOS piecewise-linear (3-region) model:**
$$
i_D=\begin{cases}
0 & v_{GS}<V_T \ (\text{cutoff})\\[4pt]
K\!\left[(v_{GS}-V_T)v_{DS}-\dfrac{v_{DS}^2}{2}\right] & v_{GS}\ge V_T,\ 0\le v_{DS}<v_{GS}-V_T \ (\text{triode})\\[6pt]
\dfrac{K}{2}(v_{GS}-V_T)^2 & v_{GS}\ge V_T,\ v_{DS}\ge v_{GS}-V_T \ (\text{saturation})
\end{cases}
$$
Both region formulas agree at the boundary $v_{DS}=v_{GS}-V_T$ (continuity check). $K$ in A/V$^2$, $V_T$ in V, all voltages relative to source terminal.

## Used later in
- [[6.002-circuits-and-electronics/notes/05-amplifiers-large-small-signal-analysis]] — lists this note as a prerequisite, building on the MOSFET piecewise-linear model (specifically the saturation region) to develop amplifiers via large/small-signal analysis.
