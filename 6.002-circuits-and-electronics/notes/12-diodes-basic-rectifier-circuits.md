---
title: "Diodes, Basic Rectifier Circuits"
course: "6.002"
topic_number: 12
prerequisites: ["6.002 Topic 01 (lumped circuit abstraction, KVL/KCL)", "6.002 Topic 04 (nonlinear elements, piecewise-linear modeling)"]
status: not started
---

# Diodes, Basic Rectifier Circuits

## Why this matters
Every piece of electronics that runs off a wall outlet needs to convert AC (alternating current, which reverses direction periodically) into DC (direct current, which flows one way) somewhere inside it. The device that makes this conversion possible is the **diode** — a two-terminal element that (approximately) lets current flow in one direction and blocks it in the other. Diodes are the simplest nonlinear circuit element you will meet after resistors, and they are the natural next step after Topic 04's general treatment of nonlinear elements: everything you learned there about piecewise-linear (PWL) modeling and graphical/algebraic solution of nonlinear circuits gets applied here to one specific, extremely common device. If you skip this topic, you cannot analyze power supplies, signal detectors, voltage clamps, or protection circuits — and you will not have the conceptual bridge between "abstract nonlinear element" (Topic 04) and "real nonlinear devices with useful applications" (this topic and everything downstream, including the amplifier and MOSFET topics that reuse the same PWL technique).

## Builds on
- [[6.002-circuits-and-electronics/notes/01-lumped-circuit-abstraction]] — KVL (the sum of voltage drops around any closed loop is zero) and KCL (the sum of currents into any node is zero) are used to write the loop/node equations for every circuit in this note.
- [[6.002-circuits-and-electronics/notes/04-nonlinear-elements]] — the general method of solving a circuit containing one nonlinear element by (a) writing the element's own current-voltage relationship, (b) writing the "load line" equation imposed by the rest of the circuit, and (c) intersecting the two. This note also reuses the piecewise-linear (PWL) modeling idea: approximate a curved i–v relationship by straight-line segments so that ordinary linear-circuit algebra (Ohm's law, KVL, KCL) can be used on each segment.

Because Topic 01 and Topic 04 do not exist as finished notes yet, the two paragraphs below recap everything from them that this note actually needs, so you do not have to look anything up.

**Recap of KVL/KCL (Topic 01).** A *lumped circuit* is one built from idealized two-terminal (or multi-terminal) elements connected by perfect wires, small enough that signal propagation delay across the circuit is negligible. Two laws govern every lumped circuit: **Kirchhoff's Voltage Law (KVL)** — going around any closed loop and adding up the voltage drops (with sign) in a consistent direction, the total is zero; and **Kirchhoff's Current Law (KCL)** — the sum of currents flowing into any node is zero (equivalently, current in equals current out). These two laws, plus each element's own current-voltage ($i$–$v$) relationship, are enough to solve any lumped circuit.

**Recap of nonlinear-element analysis (Topic 04).** A *linear* element (like a resistor) has an $i$–$v$ relationship that is a straight line through some fixed intercept, e.g. $v = iR$. A *nonlinear* element has a curved or kinked $i$–$v$ relationship. To analyze a circuit containing one nonlinear element embedded in an otherwise linear network, replace the linear part of the network (everything except the nonlinear element) with its Thevenin equivalent: a single voltage source $v_{TH}$ in series with a single resistance $R_{TH}$, as seen from the two terminals where the nonlinear element attaches. Writing KVL around that loop gives a straight-line relationship between the terminal current $i$ and terminal voltage $v$ of the nonlinear element:
$$v = v_{TH} - iR_{TH}$$
This is called the **load line**. The operating point of the circuit is the single $(i, v)$ pair that satisfies both the load line and the nonlinear element's own $i$–$v$ curve — i.e., the intersection of the two curves in the $i$–$v$ plane. When the nonlinear element's curve is itself approximated by straight-line segments (PWL modeling), the intersection can be found algebraically: guess which segment the operating point lies on, solve the resulting linear system, and check that the answer is consistent with the assumed segment (e.g., the diode is "on" and the solved current is indeed $\geq 0$).

## Core definitions
- **Diode** — a two-terminal nonlinear circuit element with two terminals called the **anode** (+, arrow side) and **cathode** (−, bar side). Circuit symbol: a triangle pointing from anode to cathode with a bar at the cathode. Current is defined positive flowing from anode to cathode through the device (in the direction the triangle "arrow" points).
- **Forward bias** — the condition where the anode is at higher potential than the cathode ($v_D > 0$ using the convention below), i.e. the diode is being pushed to conduct in its easy direction.
- **Reverse bias** — the condition where the cathode is at higher potential than the anode ($v_D < 0$), i.e. the diode is being pushed to conduct backward, which it strongly resists.
- **Diode convention (associated variables)** — throughout this note, $v_D$ is the voltage measured from anode to cathode (anode terminal minus cathode terminal), and $i_D$ is the current entering the anode terminal and flowing through the device toward the cathode. This is the "associated variables" sign convention: positive $i_D$ flows in the direction of the arrow when $v_D$ is measured with + at the anode.
- **Ideal diode model** — the simplest PWL approximation: $i_D = 0$ for $v_D < 0$ (reverse bias, "off," acts as an open circuit) and $v_D = 0$ for $i_D > 0$ (forward bias, "on," acts as a $0\,\Omega$ wire — a short circuit). Formally, this is a switch that is open when reverse-biased and closed (with zero voltage drop) when forward-biased.
- **Piecewise-linear diode model with threshold ($V_{on}$) and on-resistance ($r_D$)** — a refinement of the ideal model that better matches real diodes: $i_D = 0$ for $v_D < V_{on}$, and for $v_D \geq V_{on}$ the diode conducts along a line $v_D = V_{on} + i_D r_D$, i.e. it behaves like a $V_{on}$ voltage source in series with resistance $r_D$ whenever it's on. $V_{on}$ (also called the **cut-in** or **threshold voltage**, typically about $0.6$–$0.7\,\text{V}$ for silicon diodes) and $r_D$ (the diode's small effective series resistance once conducting, often a few ohms to tens of ohms) are the two parameters of this model.
- **Rectifier** — a circuit that uses one or more diodes to convert an AC (alternating, periodically reversing) input voltage into a DC-like (single-polarity) output. A **half-wave rectifier** passes only one half of each AC cycle (e.g., only the positive half) and blocks the other half, producing an output that is zero (or near zero) during the blocked half-cycle.
- **Conduction interval** — the portion of the input cycle during which the diode is forward-biased and conducting (i.e., the diode is "on").
- **Cut-off interval** — the portion of the input cycle during which the diode is reverse-biased and not conducting (i.e., the diode is "off," carrying no current).
- **Peak inverse voltage (PIV)** — the maximum magnitude of reverse voltage that appears across the diode during the cut-off interval; a design/rating parameter (the diode must be able to withstand this without breaking down).

## Intuition
Think of an ideal diode as a one-way valve for current, like a check valve in a pipe: water (current) can flow through it in one direction with no resistance at all, but it completely refuses to flow backward, no matter how hard you push. A real diode is almost like that valve, except it takes a small amount of forward "push" (the threshold voltage $V_{on}$, roughly $0.6$–$0.7\,\text{V}$ for silicon) before it opens up, and even once open it offers a little bit of resistance ($r_D$) to the flow, rather than being a perfectly free path.

This one-way behavior is exactly what you need to turn an AC voltage — a sine wave that swings positive and negative every cycle — into something that only ever pushes current in one direction. If you put a diode in series with a resistor across an AC source, then whenever the source polarity makes the diode's anode more positive than its cathode, the diode "opens the valve" and current flows through the resistor, appearing as a positive voltage across it. When the source reverses, the diode "closes the valve," no current flows, and the resistor's voltage sits at (approximately) zero. The load resistor's voltage waveform then looks like the AC input with every other half-cycle chopped off — a half-wave rectified waveform, which is the first step of the process (repeated with more diodes and a smoothing capacitor in later, more elaborate power-supply designs) that produces the steady DC voltage inside virtually every electronic device.

The reason this whole analysis is tractable with the tools you already have is that a diode's curved real-world $i$–$v$ characteristic is being replaced by straight-line segments (the PWL models above). On each segment the diode behaves like a familiar linear element (an open circuit, a wire, or a resistor in series with a fixed voltage source), so ordinary KVL/KCL algebra applies — you just have to figure out, from the sign of the source voltage at each instant, which segment you're on.

## Derivation / formalism

### 1. Setting up the diode's own $i$–$v$ relationship (ideal model)

By definition of the ideal diode model:
$$
i_D = 0 \quad \text{when } v_D < 0
$$
$$
v_D = 0 \quad \text{when } i_D > 0
$$
Graphically, in the $i_D$–$v_D$ plane this is the union of the negative $v_D$ axis (for $v_D < 0$, $i_D = 0$) and the positive $i_D$ axis (for $i_D > 0$, $v_D = 0$) — an "L" shaped (rotated) curve with the corner at the origin.

### 2. The series diode–resistor circuit driven by a source

Consider the circuit: an independent voltage source $v_S(t)$, in series with a diode $D$ (anode toward the source's + terminal) and a load resistor $R$, forming a single loop, with the diode's terminals labeled $v_D$ (anode to cathode) and $i_D$ (into the anode), and the resistor's voltage $v_R$ across it with current $i_R$ flowing in the same loop direction as $i_D$.

Because this is a single loop (series circuit), KCL immediately gives:
$$
i_D = i_R \equiv i
$$
(the same current flows through every element in the loop).

Apply KVL around the loop, summing voltage drops in the direction of positive current flow:
$$
v_S(t) - v_D - v_R = 0
$$
$$
\Rightarrow v_S(t) = v_D + v_R
$$
Using $v_R = iR$ (Ohm's law for the resistor):
$$
v_S(t) = v_D + iR \qquad (\star)
$$
Solving $(\star)$ for $v_D$ as a function of $i$ gives the **load line**:
$$
v_D = v_S(t) - iR
$$
This is a straight line in the $i_D$–$v_D$ plane (for a fixed instant of time $t$, since $v_S(t)$ is just a number at that instant): it has $v_D$-intercept $v_S(t)$ (at $i = 0$) and slope $-R$.

### 3. Finding the operating point: case analysis

The circuit's actual operating point $(i, v_D)$ at each instant is the intersection of the load line from Step 2 with the diode's own curve from Step 1. Because the diode's curve is two different line segments, solve by cases and check consistency (the standard PWL method from Topic 04):

**Case A — assume diode OFF ($i = 0$):**
From the diode's OFF branch, $i_D = 0$. Substitute into $(\star)$:
$$
v_S(t) = v_D + (0)R = v_D
$$
So $v_D = v_S(t)$. This is only consistent with "diode OFF" if the OFF-branch condition $v_D < 0$ holds, i.e. if
$$
v_S(t) < 0.
$$
If this holds, the assumption is self-consistent: $i = 0$, $v_D = v_S(t)$, and (since $i=0$) $v_R = iR = 0$.

**Case B — assume diode ON ($v_D = 0$):**
From the diode's ON branch, $v_D = 0$. Substitute into $(\star)$:
$$
v_S(t) = 0 + iR \;\Rightarrow\; i = \frac{v_S(t)}{R}
$$
This is only consistent with "diode ON" if the ON-branch condition $i_D > 0$ holds, i.e. if
$$
v_S(t) > 0.
$$
If this holds, the assumption is self-consistent: $v_D = 0$, $i = v_S(t)/R$, and $v_R = iR = v_S(t)$.

### 4. Assembling the piecewise result (ideal-diode half-wave rectifier)

Combining Cases A and B, for every instant $t$:
$$
v_R(t) =
\begin{cases}
v_S(t), & v_S(t) > 0 \quad \text{(diode ON)}\\[4pt]
0, & v_S(t) < 0 \quad \text{(diode OFF)}
\end{cases}
$$
and
$$
i(t) =
\begin{cases}
v_S(t)/R, & v_S(t) > 0\\[4pt]
0, & v_S(t) < 0
\end{cases}
$$
This is the defining behavior of a **half-wave rectifier**: the output $v_R(t)$ reproduces the source waveform exactly during the positive half-cycle and is clamped to zero during the negative half-cycle. If $v_S(t) = V_m \sin(\omega t)$ (a sinusoid of peak amplitude $V_m$ and angular frequency $\omega$), then $v_S(t) > 0$ for $0 < \omega t < \pi$ (within each $2\pi$ cycle) — the conduction interval — and $v_S(t) < 0$ for $\pi < \omega t < 2\pi$ — the cut-off interval.

### 5. Redoing Steps 2–4 with the more realistic PWL model ($V_{on}$, $r_D$)

Now let the diode obey $i_D = 0$ for $v_D < V_{on}$, and $v_D = V_{on} + i_D r_D$ for $i_D \geq 0$ (the ON branch). Repeat the same case analysis.

**Case A — OFF ($i=0$):** as before, $v_D = v_S(t)$, consistent provided $v_S(t) < V_on$... more precisely provided $v_S(t) < V_{on}$ (since the OFF-branch condition is now $v_D < V_{on}$, not $v_D<0$).

**Case B — ON:** substitute the ON-branch relation $v_D = V_{on} + iR_D$ into $(\star)$:
$$
v_S(t) = (V_{on} + i r_D) + iR
$$
$$
v_S(t) - V_{on} = i(r_D + R)
$$
$$
i = \frac{v_S(t) - V_{on}}{R + r_D}
$$
consistent provided $i > 0$, i.e. provided $v_S(t) > V_{on}$.

So with the more realistic model, the output resistor voltage is:
$$
v_R(t) = iR =
\begin{cases}
\dfrac{\left(v_S(t) - V_{on}\right)R}{R + r_D}, & v_S(t) > V_{on}\\[8pt]
0, & v_S(t) \le V_{on}
\end{cases}
$$
The only qualitative changes from the ideal case are: (1) the diode does not turn on until the source exceeds the threshold $V_{on}$, so the conduction interval shrinks slightly and starts a bit later/ends a bit earlier in each cycle, and (2) once on, the output is slightly reduced and slightly nonlinear in $v_S$ because of the $R/(R+r_D)$ scaling and the $-V_{on}$ offset, rather than following $v_S(t)$ exactly.

### 6. Peak inverse voltage (PIV)

During the cut-off interval of the ideal-diode circuit above, $i = 0$ so $v_R = 0$, and KVL gives $v_D = v_S(t) - v_R = v_S(t)$. The magnitude of $v_D$ is largest in the reverse direction when $v_S(t)$ is at its most negative value, $-V_m$. Hence:
$$
\text{PIV} = V_m
$$
(for this simple series diode–resistor circuit with a sinusoidal source of peak amplitude $V_m$). Any real diode used here must be rated to withstand at least this much reverse voltage without breaking down.

## Worked examples

### Example 1 — Ideal-diode half-wave rectifier, numeric waveform values

**Setup.** A source $v_S(t) = 10\sin(\omega t)\,\text{V}$ (peak amplitude $V_m = 10\,\text{V}$) drives a series loop consisting of an ideal diode and a load resistor $R = 100\,\Omega$ (anode connected to the source's positive reference terminal). Find $v_R$, $i$, and $v_D$ at three instants: (a) $\omega t = 90^\circ$ (source at its positive peak), (b) $\omega t = 270^\circ$ (source at its negative peak), (c) $\omega t = 30^\circ$.

**(a) $\omega t = 90^\circ$:** $v_S = 10\sin(90^\circ) = 10\,\text{V} > 0$, so by Step 3 Case B, the diode is ON.
$$
i = \frac{v_S}{R} = \frac{10\,\text{V}}{100\,\Omega} = 0.1\,\text{A} = 100\,\text{mA}
$$
$$
v_R = iR = (0.1\,\text{A})(100\,\Omega) = 10\,\text{V}, \qquad v_D = 0\,\text{V}
$$
Check via KVL: $v_D + v_R = 0 + 10 = 10\,\text{V} = v_S$. ✓.

**(b) $\omega t = 270^\circ$:** $v_S = 10\sin(270^\circ) = -10\,\text{V} < 0$, so by Step 3 Case A, the diode is OFF.
$$
i = 0\,\text{A}, \qquad v_R = iR = 0\,\text{V}, \qquad v_D = v_S = -10\,\text{V}
$$
Check: $v_D + v_R = -10 + 0 = -10\,\text{V} = v_S$. ✓. This instant is also where the reverse voltage across the diode is at its most negative, so it sets the PIV for this circuit: $\text{PIV} = 10\,\text{V}$, matching $V_m$ from Step 6.

**(c) $\omega t = 30^\circ$:** $v_S = 10\sin(30^\circ) = 10(0.5) = 5\,\text{V} > 0$, so the diode is ON.
$$
i = \frac{5\,\text{V}}{100\,\Omega} = 0.05\,\text{A} = 50\,\text{mA}, \qquad v_R = (0.05\,\text{A})(100\,\Omega) = 5\,\text{V}, \qquad v_D = 0\,\text{V}
$$
Check: $0 + 5 = 5\,\text{V} = v_S$. ✓.

### Example 2 — Realistic PWL diode model ($V_{on}, r_D$), find operating point and output at one instant

**Setup.** The same source and resistor as Example 1 ($v_S(t) = 10\sin(\omega t)\,\text{V}$, $R = 100\,\Omega$), but now the diode is modeled with threshold $V_{on} = 0.7\,\text{V}$ and on-resistance $r_D = 10\,\Omega$. Find $i$, $v_D$, and $v_R$ at $\omega t = 90^\circ$ (source at its peak, $v_S = 10\,\text{V}$). Also find the smallest $v_S$ for which the diode conducts at all under this model.

**Step 1 — check which case applies.** Guess the diode is ON (reasonable, since $v_S = 10\,\text{V}$ is clearly bigger than a $0.7\,\text{V}$ threshold). From Step 5 Case B:
$$
i = \frac{v_S(t) - V_{on}}{R + r_D} = \frac{10\,\text{V} - 0.7\,\text{V}}{100\,\Omega + 10\,\Omega} = \frac{9.3\,\text{V}}{110\,\Omega}
$$
$$
i = 0.0845\overline{45}\,\text{A} \approx 84.5\,\text{mA}
$$
Check consistency: $i > 0$ ✓, so the ON assumption holds.

**Step 2 — back out $v_D$ and $v_R$.**
$$
v_D = V_{on} + i\,r_D = 0.7\,\text{V} + (0.0845\,\text{A})(10\,\Omega) = 0.7\,\text{V} + 0.845\,\text{V} = 1.545\,\text{V}
$$
$$
v_R = iR = (0.0845\,\text{A})(100\,\Omega) = 8.45\,\text{V}
$$
**Check via KVL:** $v_D + v_R = 1.545\,\text{V} + 8.45\,\text{V} = 9.995\,\text{V} \approx 10\,\text{V} = v_S$ (the small residual is rounding of the repeating decimal $0.084545...$; using the exact fraction $i = 93/1100\,\text{A}$ gives $v_D + v_R$ exactly $10\,\text{V}$). ✓.

Compare to Example 1(a): the ideal model predicted $v_R = 10\,\text{V}$ exactly at this instant; the more realistic model predicts $v_R = 8.45\,\text{V}$ — about $1.55\,\text{V}$ lower, consistent with the qualitative prediction in Step 5 that the realistic output is reduced relative to the ideal case.

**Step 3 — turn-on threshold for the source.** The diode conducts (Case B is consistent) exactly when $v_S(t) > V_{on}$, i.e. when $v_S(t) > 0.7\,\text{V}$. So for this source, conduction requires $10\sin(\omega t) > 0.7\,\text{V}$, i.e. $\sin(\omega t) > 0.07$, i.e. roughly $\omega t$ between about $4.0^\circ$ and $176.0^\circ$ (using $\arcsin(0.07) \approx 4.0^\circ$) — a slightly narrower conduction interval than the ideal model's full $0^\circ$ to $180^\circ$.

## Common pitfalls
- **Mixing up the diode's sign convention.** Forgetting which terminal is the anode and which is the cathode (or flipping the assumed direction of $i_D$) silently flips every inequality in the case analysis (Step 3/5), making a circuit that should be conducting appear cut off, and vice versa. Always fix the polarity of $v_D$ (anode minus cathode) and the direction of $i_D$ (into the anode) before writing any equation.
- **Assuming a case without checking consistency.** The whole method in Step 3 requires you to *verify* that the assumed branch (ON or OFF) is actually consistent with the solved values (e.g., that a solved $i$ under the "ON" assumption really does come out positive). Skipping the check and just picking whichever case "looks right" leads to wrong answers whenever the source is near its zero crossing or when there are multiple diodes with different turn-on conditions.
- **Forgetting the threshold voltage entirely when a real (non-ideal) diode is specified.** Using $v_D = 0$ instead of $v_D = V_{on} + i r_D$ when the problem gives you $V_{on}$ and $r_D$ throws away exactly the effect the problem is testing, and produces an output that's systematically too high by roughly $V_{on}$.
- **Treating the load line's slope as $+R$ instead of $-R$.** The load line $v_D = v_S(t) - iR$ has slope $-R$ in the $i$–$v_D$ plane; writing it as $v_D = v_S(t) + iR$ (a common sign slip when rearranging KVL) gives an intersection that doesn't correspond to any physically consistent operating point.
- **Confusing peak amplitude $V_m$ with RMS or average value when computing PIV or output levels.** PIV in this circuit equals the *peak* source amplitude $V_m$, not the source's RMS value (which is $V_m/\sqrt{2}$ for a sinusoid) — using RMS instead under-rates the diode.
- **Assuming the output is a clean DC voltage after only one diode and no capacitor.** A single-diode half-wave rectifier (as analyzed here) produces a pulsating, half-wave-shaped output, not smooth DC; producing smooth DC additionally requires a smoothing/filter capacitor and possibly a full-wave (multi-diode) configuration, which are refinements beyond the scope of this note.
- **Ignoring $r_D$'s effect on the conduction boundary.** Some students correctly add $V_{on}$ but forget that $r_D$ also changes the *slope* of the ON-branch relation, so the current formula in Step 5 is $i = \dfrac{v_S(t)-V_{on}}{R+r_D}$, not $i = \dfrac{v_S(t) - V_{on}}{R}$ (which would ignore $r_D$ altogether).

## Self-check

### Questions
1. In the associated-variables convention used in this note, which terminal is $v_D$ measured with + at, and which direction does positive $i_D$ flow?
2. For an ideal diode, write the two-branch $i_D$–$v_D$ relationship (the two equations, each with its condition).
3. A series loop has an ideal diode, a $50\,\Omega$ resistor, and a source $v_S(t) = 6\cos(\omega t)\,\text{V}$. At $\omega t = 0$, is the diode ON or OFF, and what is $i$ at that instant?
4. Using the same circuit as Question 3, what is the peak inverse voltage (PIV) across the diode?
5. A diode with $V_{on} = 0.6\,\text{V}$ and $r_D = 5\,\Omega$ is in series with $R = 45\,\Omega$ and a source $v_S(t) = 12\sin(\omega t)\,\text{V}$. At the instant when $v_S = 4\,\text{V}$, find $i$, $v_D$, and $v_R$, showing the consistency check.
6. Still using Question 5's circuit, over what range of $v_S(t)$ values (in volts) is the diode OFF?
7. Suppose in Question 5's circuit someone forgets $r_D$ (sets $r_D = 0$) but keeps $V_{on}=0.6\,\text{V}$. Recompute $i$ at $v_S = 4\,\text{V}$ under this (wrong) simplification and compare numerically to the correct answer from Question 5 — by how many mA do they differ?
8. A series diode–resistor circuit is driven by $v_S(t) = 5\sin(\omega t)\,\text{V}$ with $R = 20\,\Omega$, using the ideal diode model, but this time the diode is installed backward (cathode toward the source's + terminal, anode toward the resistor). Write $v_R(t)$ as a piecewise function of $v_S(t)$ for this reversed installation.

### Answers
1. $v_D$ is measured with + at the anode (anode terminal minus cathode terminal); positive $i_D$ flows into the anode and through the device toward the cathode (the direction the triangle in the diode symbol points).
2. $i_D = 0$ for $v_D < 0$ (OFF branch); $v_D = 0$ for $i_D > 0$ (ON branch).
3. At $\omega t = 0$: $v_S = 6\cos(0) = 6\,\text{V} > 0$, so by the Case B analysis (Step 3) the diode is ON. Then $i = v_S/R = 6\,\text{V}/50\,\Omega = 0.12\,\text{A} = 120\,\text{mA}$.
4. PIV equals the peak amplitude of the source, $V_m = 6\,\text{V}$ (occurs when $v_S(t) = -6\,\text{V}$, at which point the diode is OFF, $i=0$, $v_R=0$, and KVL gives $v_D = v_S = -6\,\text{V}$, magnitude $6\,\text{V}$).
5. Guess ON: $i = \dfrac{v_S - V_{on}}{R+r_D} = \dfrac{4\,\text{V}-0.6\,\text{V}}{45\,\Omega+5\,\Omega} = \dfrac{3.4\,\text{V}}{50\,\Omega} = 0.068\,\text{A} = 68\,\text{mA}$. Since $i>0$, ON assumption is consistent. $v_D = V_{on}+ir_D = 0.6\,\text{V} + (0.068\,\text{A})(5\,\Omega) = 0.6+0.34 = 0.94\,\text{V}$. $v_R = iR = (0.068\,\text{A})(45\,\Omega) = 3.06\,\text{V}$. Check: $v_D+v_R = 0.94+3.06 = 4.00\,\text{V} = v_S$. ✓
6. The diode is OFF whenever $v_S(t) \le V_{on} = 0.6\,\text{V}$, i.e. for all $v_S(t) \in [-12\,\text{V}, 0.6\,\text{V}]$ (recalling $v_S(t)=12\sin(\omega t)$ ranges over $[-12\,\text{V},12\,\text{V}]$).
7. With $r_D$ wrongly set to $0$: $i = \dfrac{v_S - V_{on}}{R} = \dfrac{4-0.6}{45} = \dfrac{3.4}{45} = 0.0756\,\text{A} = 75.6\,\text{mA}$, versus the correct $68\,\text{mA}$ from Question 5 — the wrong simplification overestimates $i$ by about $75.6-68 = 7.6\,\text{mA}$.
8. With the diode reversed, its anode is now on the resistor side and cathode on the source side, so (redefining $v_D$ consistently with the new physical anode/cathode locations) the diode conducts exactly when the *resistor* side is more positive than the source side reaching the device in the forward direction — which, tracing through the same KVL loop, means the diode is ON precisely when $v_S(t) < 0$ and OFF when $v_S(t) > 0$ (the mirror image of the original circuit). So $v_R(t) = v_S(t)$ for $v_S(t) < 0$, and $v_R(t) = 0$ for $v_S(t) > 0$ — i.e., this circuit passes the *negative* half-cycle and blocks the positive one, the complementary half-wave rectifier to the one derived in Step 4.

## Summary / cheat sheet
- **Diode convention:** $v_D$ = anode − cathode; $i_D$ positive into anode, out cathode.
- **Ideal diode model:** OFF ($i_D=0$) when $v_D<0$; ON ($v_D=0$) when $i_D>0$. Acts as open circuit (OFF) or $0\,\Omega$ wire (ON).
- **Realistic PWL model:** OFF ($i_D=0$) when $v_D<V_{on}$; ON ($v_D = V_{on}+i_Dr_D$) when $i_D\ge0$. Typical silicon $V_{on}\approx 0.6$–$0.7\,\text{V}$.
- **Method for any series diode + linear network:** reduce the linear part to Thevenin form $v_{TH}, R_{TH}$; write load line $v_D = v_{TH}-iR_{TH}$; guess ON or OFF branch of the diode; solve; check the branch assumption is self-consistent (ON needs $i>0$; OFF needs $v_D$ below threshold).
- **Ideal-diode series diode–resistor rectifier, source $v_S(t)$, load $R$:**
$$v_R(t)=v_S(t),\ i=v_S(t)/R \quad\text{when } v_S(t)>0 \ (\text{ON})$$
$$v_R(t)=0,\ i=0 \quad\text{when } v_S(t)<0 \ (\text{OFF})$$
- **Realistic-model version, load $R$, diode params $V_{on}, r_D$:**
$$i=\frac{v_S(t)-V_{on}}{R+r_D},\quad v_R=iR \quad\text{when } v_S(t)>V_{on}$$
$$i=0,\ v_R=0 \quad\text{when } v_S(t)\le V_{on}$$
- **Peak inverse voltage (PIV)** for a sinusoidal source of peak amplitude $V_m$ in this simple series circuit: $\text{PIV}=V_m$.
- A single diode + resistor gives **half-wave rectification**: output tracks the source during conduction, sits at (near) zero during cutoff — not yet smooth DC (that needs filtering/multi-diode circuits, covered later).

## Used later in
(none yet — this is currently the last topic in the 6.002 sequence)
