---
title: "Amplifiers: large-signal, small-signal analysis"
course: "6.002"
topic_number: 05
prerequisites: ["04-nonlinear-elements-digital-abstraction-mosfet-model", "02-resistive-networks-node-mesh-analysis", "03-dependent-sources-superposition-thevenin-norton"]
status: in progress
---

# Amplifiers: large-signal, small-signal analysis

## Why this matters

Digital gates (the previous topic) only care about two voltage levels: "low enough" and "high enough" to represent 0 and 1. But the same nonlinear device — the MOSFET — can also be biased to sit in a region where a *small* change in input voltage produces a *large, faithfully scaled* change in output voltage. That is an amplifier. Amplifiers are the bridge between the messy, small, continuous signals the physical world produces (a microphone diaphragm moving fractions of a millimeter, a sensor outputting microvolts) and signals large enough to drive a speaker, a display, or the digital logic built in earlier topics.

This topic sits here because it needs exactly one thing from Topic 04 — the MOSFET's $i_D$ vs. $v_{GS}$, $v_{DS}$ relationship — and nothing else. Everything else (resistive networks, KVL/KCL, Thevenin equivalents) came even earlier. If you skip this topic, later topics build op-amps and frequency response on the assumption that you already know how to (1) pick an operating point on a nonlinear device's characteristic curve, and (2) linearize around that point to get a simple linear circuit you can solve with the tools from Topics 1–3. Skipping straight to op-amps without this would leave "gain," "bias point," and "small-signal model" as unexplained black boxes.

## Builds on

- [[6.002-circuits-and-electronics/notes/04-nonlinear-elements-digital-abstraction-mosfet-model]] — MOSFET large-signal (saturation-region) model, $i_D=\frac{K}{2}(v_{GS}-V_T)^2$; recapped in full below since the zero-assumed-knowledge rule requires this note be self-contained.
- [[6.002-circuits-and-electronics/notes/02-resistive-networks-node-mesh-analysis]] — KVL/KCL and resistive network analysis: solving linear resistor circuits with sources — used throughout to solve both the bias circuit and the small-signal circuit.
- [[6.002-circuits-and-electronics/notes/03-dependent-sources-superposition-thevenin-norton]] — Thevenin/Norton equivalents and dependent sources: the small-signal MOSFET model introduced below *is* a dependent-source model, and we use Thevenin reduction to simplify gain calculations.

## Core definitions

Before anything else, here is the MOSFET model this note depends on, recapped self-containedly (it is the one result borrowed from Topic 04).

- **MOSFET (n-channel, enhancement mode)** — a three-terminal device (gate G, source S, drain D) that acts as a voltage-controlled switch/current-source. Current $i_D$ flows from drain to source, controlled by the gate-to-source voltage $v_{GS}$ and drain-to-source voltage $v_{DS}$. No current flows into the gate terminal (idealized model: $i_G = 0$).
- **Threshold voltage $V_T$** — the value of $v_{GS}$ below which the device carries (ideally) zero drain current, no matter what $v_{DS}$ is. For $v_{GS} < V_T$: $i_D = 0$ (device is "off," cutoff region).
- **Saturation (active) region** — the operating region used for amplification, valid when $v_{GS} \ge V_T$ and $v_{DS} \ge v_{GS} - V_T$. In this region the large-signal model used in this course is
$$i_D = \frac{K}{2}(v_{GS} - V_T)^2$$
  where $K$ is a device constant (units: A/V$^2$) set by fabrication (channel width/length, oxide thickness, mobility) — the same $K$, with the same $\frac12$ convention, as [[6.002-circuits-and-electronics/notes/04-nonlinear-elements-digital-abstraction-mosfet-model]]'s saturation-region formula. Note: this expression depends on $v_{GS}$ only, *not* on $v_{DS}$ — this is the simplified ("ideal saturation") MOSFET model used throughout 6.002's amplifier treatment. (A more refined model adds a mild $v_{DS}$ dependence; that refinement is not needed here.)
- **Triode (linear) region** — the region $v_{GS} \ge V_T$ and $0 \le v_{DS} < v_{GS}-V_T$, where $i_D$ depends on both $v_{GS}$ and $v_{DS}$. Amplifiers in this note are always biased in saturation, so the triode-region formula is not needed; it is mentioned only so you can recognize when a design has *left* the amplifying region by mistake.
- **Operating point (bias point, quiescent point), $(V_{GS}, V_{DS}, I_D)$** — the DC (no-signal) voltages and current at which the transistor sits when only constant (DC) sources are applied. Capital letters with capital subscripts denote these fixed DC quantities throughout this note.
- **Large-signal analysis** — solving the full nonlinear circuit (using $i_D = \frac{K}{2}(v_{GS}-V_T)^2$ directly) to find the operating point, or to find the output for an input of arbitrary size. "Large-signal" because the algebra keeps the full nonlinear equation — no approximation.
- **Small-signal (incremental) analysis** — a linear approximation of the circuit's behavior for *small* deviations away from the operating point. We write every voltage/current as (DC operating value) + (small time-varying deviation), throw away every term that is quadratic or higher order in the small deviation, and are left with a linear circuit relating just the deviations. Lowercase letters with lowercase subscripts (e.g. $v_{gs}$, $i_d$) denote these small-signal deviations throughout this note.
- **Total instantaneous quantity** — the actual physical voltage/current at any instant, equal to operating point + small-signal deviation, e.g. $v_{GS}^{total}(t) = V_{GS} + v_{gs}(t)$. This note uses the convention: capital letter/capital subscript = DC bias value, lowercase letter/lowercase subscript = small-signal deviation, capital letter/lowercase subscript is NOT used (avoids the classic notation trap described in Common pitfalls).
- **Transconductance $g_m$** — the small-signal gain parameter of the MOSFET, defined as
$$g_m \equiv \left.\frac{\partial i_D}{\partial v_{GS}}\right|_{\text{operating point}}$$
  units: amperes/volt (siemens). It is the slope of the $i_D$–$v_{GS}$ curve *at* the bias point — the local "gain" of the device converting a gate-voltage wiggle into a drain-current wiggle.
- **Small-signal (linearized) MOSFET model** — the circuit equivalent, valid only for small deviations around a fixed operating point in saturation: an open circuit from gate to source (still $i_g=0$), and a dependent current source from drain to source of value $g_m v_{gs}$, where $v_{gs}$ is the small-signal gate-source voltage. Derived below.
- **Amplifier** — a circuit built around a device biased in its "active" region (here, MOSFET saturation) such that a small-signal input voltage produces a small-signal output voltage that is a scaled (and typically inverted) version of the input, with scale factor (gain) larger than 1 in magnitude.
- **Voltage gain $A_v$** — ratio of small-signal output voltage to small-signal input voltage, $A_v \equiv v_{out}/v_{in}$ (dimensionless, since both are volts). For the common-source amplifier derived below, $A_v$ is negative (inverting).
- **Load line** — the graph, on the $i_D$–$v_{DS}$ plane, of the linear relationship imposed by the external resistor network (from KVL around the drain-source loop). The operating point is the intersection of the load line with the device's $i_D$–$v_{GS}$($v_{DS}$-fixed) characteristic.

## Intuition

A MOSFET in saturation behaves like a current source whose output current is controlled by $v_{GS}$, but the control law $i_D = \frac{K}{2}(v_{GS}-V_T)^2$ is a parabola — nonlinear. If you feed it a signal that swings $v_{GS}$ over a wide range, the output current waveform gets visibly distorted (a sine in, not-quite-a-sine-out), because you're riding all over the curved parabola.

But zoom in on any *one point* of a smooth curve, and it looks like a straight line — that's just what "smooth" means. If we force the signal to be small enough that $v_{GS}$ only ever wiggles a tiny amount around some fixed DC value $V_{GS}$, then the little arc of parabola we're riding on is well approximated by its tangent line at that point. The slope of that tangent line is $g_m$. This is exactly the same idea as approximating $\sin\theta \approx \theta$ for small $\theta$, or approximating any smooth function by its first-order Taylor expansion.

So the whole "large-signal vs. small-signal" story is:
1. **Set up a DC bias circuit** (resistors + DC sources only) so the transistor sits at some quiescent point $(V_{GS}, V_{DS}, I_D)$ comfortably inside saturation, away from the edges (cutoff, triode) — this uses the full nonlinear equation once, to solve for numbers. This is the "large-signal" step.
2. **Perturb slightly**: add a small time-varying signal on top of the DC input.
3. **Linearize**: replace the transistor by its small-signal model (a dependent current source $g_m v_{gs}$) valid only near that one bias point, and solve the resulting *linear* circuit for the small-signal output — ordinary resistor-network algebra from Topics 1–3.
4. **Superpose**: total output = DC bias output + small-signal output. Because steps 1 and 3 are algebraically independent (DC-only circuit vs. deviations-only circuit), you get to solve two much easier problems instead of one hard nonlinear one.

The catch, and the entire point of "small-signal": this linear approximation is only valid as long as the signal stays small enough that the parabola still looks straight over that range. Push the input swing too large and you leave the linear region (distortion), or push the transistor entirely out of saturation into cutoff or triode (clipping) — the small-signal model then gives wrong answers because it doesn't know those boundaries exist.

## Derivation / formalism

### Step 0: the circuit

The standard circuit analyzed here is the **common-source amplifier**:

- A DC supply $V_{DD}$ connects through a drain resistor $R_D$ to the MOSFET drain.
- The MOSFET source is grounded.
- The gate is driven by a DC bias voltage $V_{GG}$ (through some bias network — for this derivation we just treat $V_{GG}$ as an ideal DC voltage source at the gate, in series with the input signal) plus a small-signal input $v_{in}(t)$: $v_{GS}^{total}(t) = V_{GG} + v_{in}(t)$.
- The output is taken at the drain: $v_{out}^{total}(t) = v_{DS}^{total}(t)$.

KVL around the drain loop (supply → $R_D$ → drain-source of MOSFET → ground):
$$V_{DD} = i_D R_D + v_{DS}$$
i.e.
$$v_{DS} = V_{DD} - i_D R_D \qquad (\star)$$
This is the load line: a straight line in the $i_D$–$v_{DS}$ plane with $v_{DS}$-intercept $V_{DD}$ and slope $-1/R_D$.

### Step 1: large-signal (DC) operating point

With no signal ($v_{in}=0$), $v_{GS} = V_{GG}$ is constant, so (assuming saturation, to be checked after) the device equation gives the DC drain current:
$$I_D = \frac{K}{2}(V_{GG}-V_T)^2$$
Substitute into $(\star)$ with DC quantities:
$$V_{DS} = V_{DD} - I_D R_D = V_{DD} - \frac{K}{2}(V_{GG}-V_T)^2 R_D$$
This gives the full operating point $(V_{GG}, V_{DS}, I_D)$. **Check saturation**: require $V_{DS} \ge V_{GG} - V_T$ (and $V_{GG}\ge V_T$). If this fails, the bias choice is wrong (device is actually in triode) and $(\star)$'s solution using the saturation formula is invalid — go back and choose different $R_D$, $V_{GG}$, or $V_{DD}$.

### Step 2: introduce the small signal and expand

Now let $v_{in}(t)$ be small and nonzero. Every quantity becomes DC value + deviation:
$$v_{GS}(t) = V_{GG} + v_{in}(t), \qquad i_D(t) = I_D + i_d(t), \qquad v_{DS}(t) = V_{DS} + v_{out}(t)$$
where $v_{in}, i_d, v_{out}$ are the (as yet unknown, presumed small) deviations, and $v_{out}\equiv v_{ds}$ by definition of "output = drain voltage deviation."

Substitute the total $v_{GS}$ into the device equation (still valid — the device equation holds at every instant, not just at DC):
$$I_D + i_d = \frac{K}{2}\big[(V_{GG}+v_{in}) - V_T\big]^2 = \frac{K}{2}\big[(V_{GG}-V_T) + v_{in}\big]^2$$

Expand the square:
$$= \frac{K}{2}\Big[(V_{GG}-V_T)^2 + 2(V_{GG}-V_T)v_{in} + v_{in}^2\Big]$$
$$= \underbrace{\frac{K}{2}(V_{GG}-V_T)^2}_{=I_D} + \underbrace{K(V_{GG}-V_T)}_{\text{call this } g_m} v_{in} + \frac{K}{2} v_{in}^2$$

So:
$$I_D + i_d = I_D + g_m v_{in} + \frac{K}{2} v_{in}^2$$

The $I_D$ terms cancel (that's the DC equation from Step 1, already solved). What's left:
$$i_d = g_m v_{in} + \frac{K}{2} v_{in}^2$$

**This is the linearization step.** If $v_{in}$ is small enough that $|v_{in}| \ll |V_{GG}-V_T|$, the last term $\frac{K}{2}v_{in}^2$ is second-order small (product of two small things) compared to the first term $g_m v_{in}$ (product of one small thing and one fixed number), so we drop it:
$$i_d \approx g_m v_{in}, \qquad \text{where } g_m = K(V_{GG}-V_T)$$

Check against the definition given earlier, $g_m \equiv \partial i_D/\partial v_{GS}$ evaluated at the bias point: differentiating $i_D = \frac{K}{2}(v_{GS}-V_T)^2$ gives $\partial i_D/\partial v_{GS} = K(v_{GS}-V_T)$, evaluated at $v_{GS}=V_{GG}$ gives exactly $K(V_{GG}-V_T)$. Matches — the Taylor-expansion argument and the calculus definition of $g_m$ are the same statement.

**This is exactly why the small-signal MOSFET model is "gate-source open circuit, drain-source current source $g_m v_{gs}$"**: we just showed the *only* linear relationship the small-signal deviations obey is $i_d = g_m v_{gs}$ (here $v_{gs}=v_{in}$ since source is grounded), with no current into the gate ($i_g=0$ always, large-signal model already assumed this).

### Step 3: solve the linear small-signal circuit

Take $(\star)$ (drain loop KVL) and substitute totals:
$$V_{DS}+v_{out} = V_{DD} - (I_D+i_d)R_D = V_{DD} - I_D R_D - i_d R_D$$
The DC parts ($V_{DS} = V_{DD}-I_D R_D$, from Step 1) cancel again, leaving the small-signal drain loop equation:
$$v_{out} = -i_d R_D$$
Substitute $i_d = g_m v_{in}$ from Step 2:
$$\boxed{v_{out} = -g_m R_D\, v_{in}}$$

So the small-signal voltage gain is
$$A_v \equiv \frac{v_{out}}{v_{in}} = -g_m R_D$$
Negative sign: this amplifier is **inverting** — a positive wiggle in gate voltage pulls more current through $R_D$, dropping more voltage across $R_D$ and leaving *less* voltage $v_{DS}$ at the drain. Larger $g_m$ (more sensitive device, or biased with $V_{GG}$ further above $V_T$) or larger $R_D$ (more voltage drop per amp of current wiggle) both mean bigger gain magnitude.

### Step 4: superpose to get the actual total output

$$v_{DS}^{total}(t) = V_{DS} + v_{out}(t) = \underbrace{V_{DD}-I_D R_D}_{\text{DC bias part (Step 1)}} \;\underbrace{-\, g_m R_D\, v_{in}(t)}_{\text{amplified signal (Step 3)}}$$

This is the complete answer: a DC level (the bias point) riding a scaled, inverted copy of the input signal on top.

### Validity condition (when is this whole procedure allowed?)

Two separate conditions must both hold, and it's important to keep them straight:
1. **Bias point in saturation, with margin**: $V_{GS}=V_{GG} \ge V_T$ and $V_{DS} \ge V_{GG}-V_T$, checked with strict inequality and margin (not just barely satisfied), since the signal will wiggle $v_{DS}$ and $v_{GS}$ around these values and we need the device to *stay* in saturation for the whole swing.
2. **Signal small enough for linearization**: the total swing of $v_{GS}(t) = V_{GG}+v_{in}(t)$ must stay close enough to $V_{GG}$ that the dropped term $\frac{K}{2}v_{in}^2$ is genuinely negligible next to $g_m v_{in}$, i.e. $|v_{in}| \ll |V_{GG}-V_T|$ ("small signal" is quantitative, not just a name).

## Worked examples

### Example 1 — finding the operating point and small-signal gain

**Given:** An n-channel MOSFET with $K = 2\ \text{mA/V}^2$ and $V_T = 1\ \text{V}$, used in the common-source amplifier of the derivation above with $V_{DD} = 10\ \text{V}$, $R_D = 4\ \text{k}\Omega$, and DC gate bias $V_{GG} = 3\ \text{V}$.

**Find:** the operating point $(V_{GS}, I_D, V_{DS})$, verify saturation, find $g_m$, and find the small-signal voltage gain $A_v$.

**Solution.**

Step 1 — DC drain current:
$$I_D = \frac{K}{2}(V_{GG}-V_T)^2 = \frac{2\ \text{mA/V}^2}{2}(3\ \text{V} - 1\ \text{V})^2 = (1\ \text{mA/V}^2)(2\ \text{V})^2 = 4\ \text{mA}$$

Step 2 — DC drain voltage, from the load line $(\star)$:
$$V_{DS} = V_{DD} - I_D R_D = 10\ \text{V} - (4\ \text{mA})(4\ \text{k}\Omega) = 10\ \text{V} - 16\ \text{V} = -6\ \text{V}$$

That's negative, which is impossible for this circuit (drain tied to $V_{DD}$ through a resistor with current flowing from drain to source cannot make $v_{DS}$ negative here) — this is a red flag that the saturation assumption fails and $R_D$ is too large for this bias/supply combination. Check the saturation condition directly: we need $V_{DS} \ge V_{GS}-V_T = 3-1=2\ \text{V}$, but solving self-consistently the algebra just gave $V_{DS}=-6\ \text{V} < 2\ \text{V}$ — contradiction, confirming the device cannot be in saturation with this $R_D$.

**Redo with a design that works.** Take $R_D = 1\ \text{k}\Omega$ instead (everything else the same):
$$V_{DS} = V_{DD} - I_D R_D = 10\ \text{V} - (4\ \text{mA})(1\ \text{k}\Omega) = 10\ \text{V} - 4\ \text{V} = 6\ \text{V}$$
Check saturation: need $V_{DS}\ge V_{GS}-V_T = 2\ \text{V}$. Indeed $6\ \text{V} \ge 2\ \text{V}$ — saturation confirmed, with healthy margin (4 V of headroom), so the small-signal swing can move $V_{DS}$ around without leaving saturation.

Operating point: $(V_{GS}, I_D, V_{DS}) = (3\ \text{V},\ 4\ \text{mA},\ 6\ \text{V})$.

Step 3 — transconductance:
$$g_m = K(V_{GG}-V_T) = (2\ \text{mA/V}^2)(2\ \text{V}) = 4\ \text{mA/V} = 4\times10^{-3}\ \text{S}$$

Step 4 — small-signal gain:
$$A_v = -g_m R_D = -(4\times10^{-3}\ \text{S})(1\times10^3\ \Omega) = -4$$

So a small input wiggle is inverted and amplified by a factor of 4 in magnitude, e.g. $v_{in}(t) = 0.1\sin(\omega t)\ \text{V}$ produces $v_{out}(t) = -0.4\sin(\omega t)\ \text{V}$, riding on the 6 V DC bias: $v_{DS}^{total}(t) = 6\ \text{V} - 0.4\sin(\omega t)\ \text{V}$.

Sanity check on "small": here $|v_{in}|_{max}=0.1\ \text{V}$ vs. $V_{GG}-V_T = 2\ \text{V}$ — the signal is 5% of the bias overdrive, comfortably small, so dropping the $\frac{K}{2}v_{in}^2$ term was justified. (Quantitatively, the neglected term contributes $\frac{K}{2} v_{in}^2 = \left(\frac{2\text{mA/V}^2}{2}\right)(0.1\text{V})^2=0.01\ \text{mA}$ to $i_d$ at the peak, versus the kept term $g_m v_{in} = (4\text{mA/V})(0.1\text{V})=0.4\ \text{mA}$ — a 2.5% correction, small as claimed.)

### Example 2 — designing $R_D$ for a target gain, then checking the signal-swing limit

**Given:** Same transistor ($K=2\ \text{mA/V}^2$, $V_T=1\ \text{V}$), same $V_{DD}=10\ \text{V}$, but now $V_{GG}=2\ \text{V}$. Design $R_D$ for a small-signal gain magnitude of exactly $|A_v| = 5$.

**Solution.**

DC current: $I_D = \frac{K}{2}(V_{GG}-V_T)^2 = \frac{2\ \text{mA/V}^2}{2}(2-1)^2\ \text{V}^2 = 1\ \text{mA}$.

Transconductance: $g_m = K(V_{GG}-V_T) = (2\ \text{mA/V}^2)(1\ \text{V}) = 2\ \text{mA/V}$.

Required $R_D$ from $|A_v| = g_m R_D$:
$$R_D = \frac{|A_v|}{g_m} = \frac{5}{2\times10^{-3}\ \text{S}} = 2500\ \Omega = 2.5\ \text{k}\Omega$$

Check DC operating point: $V_{DS} = V_{DD}-I_D R_D = 10\ \text{V} - (1\ \text{mA})(2.5\ \text{k}\Omega) = 10\ \text{V}-2.5\ \text{V}=7.5\ \text{V}$.

Saturation check: need $V_{DS}\ge V_{GS}-V_T = 1\ \text{V}$. Indeed $7.5\ \text{V}\ge 1\ \text{V}$ — satisfied, large margin.

**Now find the maximum allowed signal swing before saturation is violated on the low side of $v_{DS}$** (this is the practical limit on "how big can $v_{in}$ get before the small-signal model breaks down because the device leaves saturation," as opposed to the separate quadratic-term-negligible criterion). Output swings as $v_{out}=-g_m R_D v_{in} = -5v_{in}$, so total $v_{DS}^{total} = 7.5\ \text{V} - 5v_{in}(t)$. Saturation requires $v_{DS}^{total} \ge v_{GS}^{total}-V_T \approx (V_{GG}-V_T) = 1\ \text{V}$ (using the DC overdrive as the boundary since $v_{in}$ is assumed small compared to it — self-consistent with the small-signal assumption). Setting the worst case equal to the boundary:
$$7.5 - 5 v_{in,max} = 1 \ \Rightarrow\ v_{in,max} = \frac{6.5}{5} = 1.3\ \text{V}$$
But this violates the small-signal assumption itself ($|v_{in}|\ll V_{GG}-V_T=1\ \text{V}$ requires $v_{in}$ well under 1 V, and 1.3 V is bigger than the overdrive itself) — so in practice the *linearization* validity condition, not just the saturation-boundary condition, is what caps the usable signal amplitude; a designer would keep $|v_{in}|$ to a few tens of millivolts here, far below 1.3 V, to keep distortion low.

## Common pitfalls

- **Mixing capital and lowercase subscripts inconsistently.** The convention in this note is strict: $V_{GS}, I_D, V_{DS}$ (all caps) = fixed DC operating-point numbers; $v_{gs}, i_d, v_{ds}$ (all lowercase) = small-signal deviations only; $v_{GS}(t)$ etc. (mixed case) = total instantaneous time-varying quantity = sum of the two. Writing "$V_{gs}$" or "$v_{GS}$" when you mean one of the other two is one of the most common algebra-breaking errors in this topic — it silently mixes a linear (small-signal) equation with a nonlinear (large-signal) one.
- **Forgetting to check saturation after solving the bias point.** The formula $I_D = \frac{K}{2}(V_{GS}-V_T)^2$ is only valid *if* the device is actually in saturation. Solving for $I_D$ and $V_{DS}$ using this formula and never checking $V_{DS}\ge V_{GS}-V_T$ can silently produce a self-inconsistent (physically impossible) answer, as in Example 1's first attempt.
- **Using the small-signal gain formula outside the region where the small-signal approximation is valid.** $A_v=-g_mR_D$ describes the response to *small* wiggles about one specific bias point. Feeding in a large input swing and expecting $v_{out}=A_v v_{in}$ to still hold is a common misuse — the real (large-signal) relationship is the full nonlinear $i_D=\frac{K}{2}(v_{GS}-V_T)^2$, and $A_v$ is only its local, linearized slope.
- **Recomputing $g_m$ from the wrong operating point.** $g_m$ depends on where you linearize ($g_m = K(V_{GS}-V_T)$, evaluated at the bias point). If you change the DC bias ($V_{GG}$, $R_D$, or supply), $g_m$ changes too — it is not a fixed device constant like $K$ or $V_T$; it's a property of the *operating point*, not just of the device.
- **Dropping the DC part entirely and reporting only the small-signal output as "the" output voltage.** The physically real, measurable voltage at the drain is always $V_{DS}+v_{out}(t)$, not $v_{out}(t)$ alone. Reporting a bare $-4$ V·(signal shape) with no DC offset is a common but incomplete answer.
- **Sign confusion on the inverting gain.** Because $A_v = -g_mR_D$ is negative, an input that rises makes the output *fall*. Forgetting the minus sign (e.g., saying "gain of 4" instead of "gain of $-4$," or drawing the output in phase with the input) is a frequent error, and matters when amplifier stages are cascaded (each inverting stage flips phase again).
- **Confusing the load-line equation's slope with the small-signal gain.** The load line $(\star)$ has slope $-1/R_D$ in the $i_D$–$v_{DS}$ plane; the small-signal voltage gain $-g_mR_D$ is a different quantity (a ratio of two *voltages*, gate and drain, not the slope of the drain $I$–$V$ curve). Both involve $R_D$ but are not the same object — do not substitute one formula for the other.

## Self-check

### Questions

1. Define, in one sentence each, the difference between "large-signal analysis" and "small-signal analysis."
2. A MOSFET has $K = 4\ \text{mA/V}^2$, $V_T=0.8\ \text{V}$, and is biased at $V_{GS}=2.0\ \text{V}$ in saturation. Find $I_D$ and $g_m$.
3. Why must the saturation condition $V_{DS}\ge V_{GS}-V_T$ be checked *after* solving the bias-point equations, rather than assumed?
4. In the common-source amplifier, why is the voltage gain $A_v=-g_mR_D$ negative? Explain physically (not just "there's a minus sign in the formula").
5. For the transistor in Question 2, biased with $R_D = 2\ \text{k}\Omega$ and $V_{DD}=9\ \text{V}$: find $V_{DS}$, confirm saturation, and find $A_v$.
6. Two designers build the same common-source amplifier but designer B doubles $R_D$ compared to designer A (keeping $V_{GG}$, $V_{DD}$, and the transistor the same). Assuming both stay in saturation, how do their gains $A_v$ compare? What changes about their DC operating points?
7. A student computes $i_d = g_m v_{in} + \frac{K}{2}v_{in}^2$ and keeps *both* terms, arguing "more accuracy is always better." Explain what's inconsistent about keeping the quadratic term while still calling the result "the small-signal model."
8. Suppose in Example 2's circuit ($V_{GG}=2\ \text{V}$, $V_T=1\ \text{V}$, $g_m=2\ \text{mA/V}$, $R_D=2.5\ \text{k}\Omega$, $V_{DS}=7.5\ \text{V}$) the input signal is $v_{in}(t) = 0.05\sin(\omega t)$ V. Write the full total output voltage $v_{DS}^{total}(t)$, including both DC and signal parts, with correct sign.

### Answers

1. Large-signal analysis solves the circuit using the full nonlinear device equation (e.g. $i_D=\frac{K}{2}(v_{GS}-V_T)^2$) exactly, valid for any size of signal; small-signal analysis linearizes the device's behavior around one fixed DC operating point (using the tangent-line slope $g_m$) and is valid only for signal deviations small enough that the linear approximation remains accurate.
2. $I_D = \frac{K}{2}(V_{GS}-V_T)^2 = \frac{4\ \text{mA/V}^2}{2}(2.0-0.8)^2\text{V}^2 = (2\ \text{mA/V}^2)(1.44\ \text{V}^2)=2.88\ \text{mA}$. $g_m = K(V_{GS}-V_T) = (4\ \text{mA/V}^2)(1.2\ \text{V}) = 4.8\ \text{mA/V}$.
3. The formula $I_D=\frac{K}{2}(V_{GS}-V_T)^2$ used to solve for the bias point is itself only valid *if* the device turns out to be in saturation. Solving the bias equations doesn't automatically guarantee this — the algebra will happily produce a numerical answer even if the real device would actually be in triode or cutoff instead, so the saturation inequality must be checked against the solved numbers afterward to confirm the formula used was the right one.
4. Physically: a small positive increase in gate voltage $v_{gs}$ increases the channel current $i_d = g_m v_{gs}$ (more current pulled through the channel). That extra current flows through $R_D$, which (by Ohm's law/KVL on the drain loop) increases the voltage *dropped across* $R_D$ — and since the drain voltage is the supply voltage minus the drop across $R_D$, a bigger drop means a *smaller* drain voltage. So gate up → drain down: inversion is a direct consequence of the drain node being "downstream" of a current that increases with gate voltage, flowing through a fixed resistor to a fixed supply.
5. $I_D=2.88\ \text{mA}$ (from Q2). $V_{DS}=V_{DD}-I_DR_D = 9\ \text{V}-(2.88\ \text{mA})(2\ \text{k}\Omega)=9-5.76=3.24\ \text{V}$. Saturation check: need $V_{DS}\ge V_{GS}-V_T=1.2\ \text{V}$; $3.24\ \text{V}\ge1.2\ \text{V}$ — confirmed. $A_v=-g_mR_D=-(4.8\ \text{mA/V})(2\ \text{k}\Omega)=-(4.8\times10^{-3})(2\times10^3)=-9.6$.
6. $A_v=-g_mR_D$ and $g_m$ is unchanged (same transistor, same $V_{GG}$ ⇒ same operating-point overdrive), so designer B's gain magnitude is exactly double designer A's ($|A_{v,B}|=2|A_{v,A}|$), still negative/inverting. DC operating point: $I_D$ is identical for both (depends only on $K,V_T,V_{GG}$, not $R_D$), but $V_{DS}=V_{DD}-I_DR_D$ is lower for designer B (larger $R_D$ drops more DC voltage) — so B sits closer to the edge of saturation and has less headroom for signal swing before clipping.
7. The small-signal model is defined by *discarding* the second-order term $\frac{K}{2}v_{in}^2$ specifically because keeping it makes the $i_d$–$v_{in}$ relationship nonlinear again (quadratic), which defeats the entire purpose of linearizing — you'd no longer have a linear circuit solvable by ordinary resistor-network (KVL/KCL) algebra, and "gain" $A_v$ would no longer be a well-defined constant (it would depend on $v_{in}$'s own size). Keeping both terms is just large-signal analysis with an extra approximation step in the middle — not a small-signal model at all.
8. $v_{out}(t) = -g_mR_D v_{in}(t) = -(2\ \text{mA/V})(2.5\ \text{k}\Omega)(0.05\sin\omega t\ \text{V}) = -5\times0.05\sin\omega t\ \text{V} = -0.25\sin(\omega t)\ \text{V}$. Total: $v_{DS}^{total}(t) = V_{DS}+v_{out}(t) = 7.5\ \text{V} - 0.25\sin(\omega t)\ \text{V}$.

## Summary / cheat sheet

- MOSFET saturation (large-signal): $i_D = \frac{K}{2}(v_{GS}-V_T)^2$, valid only for $v_{GS}\ge V_T$ and $v_{DS}\ge v_{GS}-V_T$; $i_G=0$ always.
- Notation: CAPS/CAPS = DC bias point ($V_{GS}, I_D, V_{DS}$); lowercase/lowercase = small-signal deviation ($v_{gs}, i_d, v_{ds}$); mixed case = total instantaneous ($v_{GS}(t)=V_{GS}+v_{gs}(t)$).
- Bias-point procedure: solve $I_D=\frac{K}{2}(V_{GS}-V_T)^2$ and load line $V_{DS}=V_{DD}-I_DR_D$ simultaneously (device eq. + KVL); then **check** $V_{DS}\ge V_{GS}-V_T$.
- Transconductance: $g_m = \dfrac{\partial i_D}{\partial v_{GS}}\Big|_{\text{bias pt}} = K(V_{GS}-V_T)$ — units A/V; depends on the bias point, not just the device.
- Small-signal MOSFET model: gate–source open circuit ($i_g=0$); drain–source dependent current source of value $g_m v_{gs}$.
- Common-source amplifier small-signal gain: $A_v = v_{out}/v_{in} = -g_mR_D$ (inverting).
- Total output: $v_{DS}^{total}(t) = \underbrace{(V_{DD}-I_DR_D)}_{\text{DC bias}} \;\underbrace{-\,g_mR_D\,v_{in}(t)}_{\text{amplified signal}}$.
- Validity: (a) bias point must stay in saturation for full signal swing; (b) $|v_{in}| \ll |V_{GS}-V_T|$ (overdrive voltage) for the linear (dropped-quadratic-term) approximation to hold.

## Used later in
(none yet)
