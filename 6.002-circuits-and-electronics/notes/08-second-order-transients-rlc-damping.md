---
title: "Second-Order Transients (RLC), Damping Regimes"
course: "6.002"
topic_number: 08
prerequisites: ["First-order transients (RC, RL)", "Capacitors, inductors, energy storage", "second-order linear ODEs and characteristic roots (18.03)"]
status: not started
---

# Second-Order Transients (RLC), Damping Regimes

## Why this matters

Every circuit with both a capacitor and an inductor — a filter, a resonant tank, the parasitic inductance-plus-capacitance of a long wire driven by a switch — responds to a sudden change (a switch closing, a supply turning on) with a *second-order* transient, not the simple exponential decay/rise you saw for single-capacitor or single-inductor circuits. That second-order response can ring (oscillate while decaying), overshoot, or crawl slowly to its final value depending on the component values — and predicting which of those behaviors you get, and how fast, is essential before you build anything with an inductor and a capacitor together (tuned circuits, power-supply filters, even the parasitic ringing on a fast digital signal line). If you skip this topic, you cannot predict overshoot/ringing in a real circuit, size a damping resistor to avoid it, or understand where the sinusoidal-steady-state analysis in the next topic comes from (steady state is what's left over after this transient dies out).

## Builds on

- [[6.002-circuits-and-electronics/notes/07-first-order-transients-rc-rl]] — the general method: write one differential equation for the circuit, solve for a natural response plus a particular (forced) response, use initial conditions to fix constants. Second order reuses this method with one more order of difficulty.
- [[6.002-circuits-and-electronics/notes/06-capacitors-inductors-energy-storage]] — the defining element laws $i_C = C\,\dfrac{dv_C}{dt}$ and $v_L = L\,\dfrac{di_L}{dt}$, and the two facts that make transient analysis work: capacitor voltage cannot jump instantaneously (a jump would require infinite current) and inductor current cannot jump instantaneously (a jump would require infinite voltage).
- Second-order linear ODEs with constant coefficients and their characteristic-root method (normally covered in 18.03, not yet written in this vault) — recapped self-containedly below, since it is the entire mathematical engine of this note.

## Core definitions

- **Second-order circuit** — a circuit whose describing differential equation, after eliminating all variables but one, is second order in time (contains a second derivative). This happens whenever a circuit has exactly two independent energy-storage elements (one inductor and one capacitor, or two capacitors/two inductors not reducible to one by series/parallel combination). Here we focus on the series and parallel RLC circuit, which each have one $L$ and one $C$.
- **Characteristic equation** — the algebraic equation obtained by assuming a trial solution $x(t) = e^{st}$ in the homogeneous (source-free) differential equation and demanding it satisfy the equation; its roots $s$ determine the shape of the natural response.
- **Natural frequency $\omega_0$** — the angular frequency (rad/s) at which the circuit would oscillate if there were *no* resistance at all: $\omega_0 = 1/\sqrt{LC}$.
- **Damping factor $\alpha$** (also called the neper frequency) — the rate constant (units 1/s) that sets how fast energy is dissipated by the resistor: for series RLC, $\alpha = R/(2L)$; for parallel RLC, $\alpha = 1/(2RC)$.
- **Damping ratio $\zeta$** — the dimensionless ratio $\zeta = \alpha/\omega_0$, which alone determines whether the response is overdamped, critically damped, or underdamped, independent of the specific $R$, $L$, $C$ values.
- **Overdamped** — $\alpha > \omega_0$ (equivalently $\zeta > 1$): the characteristic equation has two distinct real negative roots; the natural response is a sum of two decaying exponentials, no oscillation.
- **Critically damped** — $\alpha = \omega_0$ (equivalently $\zeta = 1$): the characteristic equation has one repeated real negative root; the natural response is $(A + Bt)e^{st}$ — the fastest possible return to steady state without any oscillation.
- **Underdamped** — $\alpha < \omega_0$ (equivalently $\zeta < 1$): the characteristic equation has a complex-conjugate pair of roots; the natural response is a decaying sinusoid (ringing).
- **Damped natural frequency $\omega_d$** — the actual angular frequency of the ringing seen in the underdamped case, $\omega_d = \sqrt{\omega_0^2 - \alpha^2}$ (real only when underdamped).
- **Natural response** — the part of the solution driven only by the circuit's stored energy and topology (solves the ODE with the source set to zero); it decays away as $t \to \infty$ for any circuit with positive $R$.
- **Forced (particular) response** — the part of the solution that matches the shape of the input (e.g., a constant, if the source is a DC step); it is whatever the circuit settles to once the natural response has died out.

## Intuition

A capacitor stores energy in an electric field; an inductor stores energy in a magnetic field. Put them in the same loop with no resistor, and any energy you inject sloshes back and forth between the two forever, at a fixed rate set by $L$ and $C$ — that rate is $\omega_0 = 1/\sqrt{LC}$, exactly analogous to a frictionless mass-spring system oscillating forever at $\sqrt{k/m}$. This LC oscillation is the "conservative" skeleton of the second-order response.

Now add a resistor. It drains energy out of the loop every cycle, exactly like friction or air drag on the mass-spring system. Three qualitatively different things can happen depending on how strong that drag is relative to the oscillation rate:

- **Weak drag (underdamped):** the sloshing survives for several cycles before dying out — you see decaying oscillation, i.e., ringing. Think of a bell: it rings a few times before going silent.
- **Strong drag (overdamped):** the drag is so dominant that the energy just leaks out monotonically, with no back-and-forth at all — like a mass dropped into thick honey. It approaches its final value slowly, from one side, no overshoot.
- **Just-right drag (critically damped):** the boundary case — the fastest possible monotonic approach to the final value with zero oscillation. Any less resistance and you'd start to see a tiny overshoot; any more and you'd needlessly slow down the approach.

The single number that tells you which regime you're in is the damping ratio $\zeta = \alpha/\omega_0$: compare "how fast energy leaks out" ($\alpha$) to "how fast the LC pair would oscillate on its own" ($\omega_0$). $\zeta < 1$ rings, $\zeta = 1$ is the crossover, $\zeta > 1$ doesn't ring.

## Derivation / formalism

We derive the series RLC step response in full; the parallel RLC case is the dual (same math with $R \to 1/R$, $L \leftrightarrow C$ roles swapped) and is summarized at the end.

### Setting up the circuit

Consider a series loop: a DC voltage source $V_s$ (suddenly switched on at $t=0$, i.e., a step), a resistor $R$, an inductor $L$, and a capacitor $C$, all in series, forming one loop. Let $i(t)$ be the loop current (same in all three series elements) and let $v(t) \equiv v_C(t)$ be the capacitor voltage. Before $t=0$, assume the circuit is at rest: $i(0^-) = 0$ and $v(0^-) = 0$ (capacitor uncharged, no current flowing) — these are the initial conditions we'll use later; other initial conditions are handled the same way, just with different constants.

### Step 1 — Write KVL around the loop

For $t>0$, Kirchhoff's Voltage Law around the loop (sum of voltage drops around the loop equals the source):
$$V_s = v_R(t) + v_L(t) + v_C(t)$$

Using the resistor law $v_R = iR$ and the inductor law $v_L = L\,\dfrac{di}{dt}$:
$$V_s = iR + L\frac{di}{dt} + v_C(t) \tag{1}$$

### Step 2 — Eliminate $i$ in favor of $v_C$

The capacitor law gives $i = C\,\dfrac{dv_C}{dt}$. Substitute this into (1), writing $v \equiv v_C$ for brevity:
$$V_s = RC\frac{dv}{dt} + L\frac{d}{dt}\left(C\frac{dv}{dt}\right) + v$$
$$V_s = LC\frac{d^2v}{dt^2} + RC\frac{dv}{dt} + v \tag{2}$$

This is a single second-order linear ODE in the one unknown $v(t)$ — exactly the "eliminate all variables but one" step promised in Core definitions. Divide through by $LC$ to put it in standard form:
$$\frac{d^2v}{dt^2} + \frac{R}{L}\frac{dv}{dt} + \frac{1}{LC}v = \frac{V_s}{LC} \tag{3}$$

### Step 3 — Split into natural + forced response

Because (3) is linear with constant coefficients, the general solution is a sum:
$$v(t) = v_n(t) + v_p(t)$$
where $v_n$ solves the *homogeneous* equation (right side set to 0) and $v_p$ is *any one* solution that matches the constant forcing on the right.

**Forced (particular) response.** Since the forcing term $V_s/(LC)$ is a constant, try a constant $v_p = K$. Then $dv_p/dt = 0$ and $d^2v_p/dt^2 = 0$, so (3) becomes $\dfrac{1}{LC}K = \dfrac{V_s}{LC}$, giving $K = V_s$. This matches physical intuition: at DC (final) steady state, no current flows through the capacitor branch's series inductor once everything settles (in true DC steady state $di/dt \to 0$ and $i \to$ constant; for a series RLC loop driven by a DC source with no other path, the final current must be zero because a nonzero constant current would keep charging the capacitor forever, which is inconsistent with steady state — so all of $V_s$ appears across $C$). Hence:
$$v_p(t) = V_s$$

**Natural (homogeneous) response.** Set the right side of (3) to zero:
$$\frac{d^2v_n}{dt^2} + \frac{R}{L}\frac{dv_n}{dt} + \frac{1}{LC}v_n = 0 \tag{4}$$

Recap of the method for solving this (the 18.03 result used here, stated self-containedly): for a linear, constant-coefficient homogeneous ODE, try a trial solution $v_n(t) = e^{st}$ for some constant $s$ (possibly complex) to be determined. Then $\dfrac{dv_n}{dt} = s e^{st}$ and $\dfrac{d^2v_n}{dt^2} = s^2 e^{st}$. Substituting into (4):
$$s^2 e^{st} + \frac{R}{L}s\,e^{st} + \frac{1}{LC}e^{st} = 0$$

Since $e^{st} \ne 0$ for any finite $t$, divide it out:
$$s^2 + \frac{R}{L}s + \frac{1}{LC} = 0 \tag{5}$$

This is the **characteristic equation**. It is a plain quadratic in $s$, so by the quadratic formula:
$$s = \frac{-\dfrac{R}{L} \pm \sqrt{\left(\dfrac{R}{L}\right)^2 - \dfrac{4}{LC}}}{2} = -\frac{R}{2L} \pm \sqrt{\left(\frac{R}{2L}\right)^2 - \frac{1}{LC}} \tag{6}$$

Define $\alpha \equiv \dfrac{R}{2L}$ (damping factor, units 1/s — check: $R$ in $\Omega = V/A$, $L$ in $H = V\!\cdot\! s/A$, so $R/L$ has units $(V/A)/(V\!\cdot\! s/A) = 1/s$, correct) and $\omega_0 \equiv \dfrac{1}{\sqrt{LC}}$ (units: $L$ in $H$, $C$ in $F = A\!\cdot\! s/V$, so $LC$ has units $s^2$, and $1/\sqrt{LC}$ has units $1/s$, correct — an angular frequency). Then (6) becomes:
$$s = -\alpha \pm \sqrt{\alpha^2 - \omega_0^2} \tag{7}$$

Two roots, call them $s_1$ and $s_2$, given by the $+$ and $-$ signs. The general natural response is a linear combination of the two exponential modes (whenever $s_1 \ne s_2$):
$$v_n(t) = A_1 e^{s_1 t} + A_2 e^{s_2 t}$$
with $A_1, A_2$ constants fixed later by initial conditions. The sign of the quantity under the square root in (7), i.e., the sign of $\alpha^2 - \omega_0^2$, is what splits the three damping regimes:

**Case 1: Overdamped, $\alpha > \omega_0$.** Then $\alpha^2 - \omega_0^2 > 0$, so $\sqrt{\alpha^2-\omega_0^2}$ is a real number smaller than $\alpha$ (since $\omega_0^2>0$). Both roots
$$s_{1,2} = -\alpha \pm \sqrt{\alpha^2-\omega_0^2}$$
are real and negative (the $-$ root is clearly negative; the $+$ root is $-\alpha + \sqrt{\alpha^2-\omega_0^2}$, and since $\sqrt{\alpha^2-\omega_0^2} < \sqrt{\alpha^2} = \alpha$, this is still negative). Two real negative roots $\Rightarrow$ two decaying exponentials, no oscillation:
$$v_n(t) = A_1 e^{s_1 t} + A_2 e^{s_2 t}, \qquad s_1, s_2 \text{ real, negative, distinct}$$

**Case 2: Critically damped, $\alpha = \omega_0$.** Then $\alpha^2 - \omega_0^2 = 0$, so both roots collapse to the single repeated value $s = -\alpha$. A repeated root of a linear ODE's characteristic equation does *not* just give $e^{st}$ once — there is a second, independent solution $t\,e^{st}$. Check this by direct substitution into the homogeneous ODE (4), $\dfrac{d^2v_n}{dt^2}+2\alpha\dfrac{dv_n}{dt}+\alpha^2 v_n=0$, with $v_n = t e^{st}$ and $s=-\alpha$:

First and second derivatives of $v_n = t e^{st}$ (product rule):
$$\frac{dv_n}{dt} = e^{st} + st\,e^{st} = (1+st)e^{st}$$
$$\frac{d^2v_n}{dt^2} = s\,e^{st} + s(1+st)e^{st} = (2s+s^2t)e^{st}$$
Substitute these into the left side of (4):
$$(2s+s^2t)e^{st} + 2\alpha(1+st)e^{st} + \alpha^2 t\,e^{st}$$
Factor out $e^{st}$ and collect the constant and $t$-terms separately:
$$\big[2s + 2\alpha\big]e^{st} + \big[s^2 + 2\alpha s + \alpha^2\big]t\,e^{st}$$
Now set $s=-\alpha$. The constant-term bracket becomes $2(-\alpha)+2\alpha = 0$. The $t$-term bracket becomes $(-\alpha)^2+2\alpha(-\alpha)+\alpha^2 = \alpha^2-2\alpha^2+\alpha^2 = 0$. Both brackets vanish, so the whole expression is $0\cdot e^{st}+0\cdot t\,e^{st}=0$: $v_n=t\,e^{-\alpha t}$ satisfies (4) identically, confirming it is a genuine second, independent solution. So the natural response has the form:
$$v_n(t) = (A_1 + A_2 t)\,e^{-\alpha t}$$

**Case 3: Underdamped, $\alpha < \omega_0$.** Then $\alpha^2 - \omega_0^2 < 0$, so the square root is imaginary. Write $\sqrt{\alpha^2-\omega_0^2} = \sqrt{-(\omega_0^2-\alpha^2)} = j\sqrt{\omega_0^2-\alpha^2} \equiv j\omega_d$, defining the **damped natural frequency** $\omega_d \equiv \sqrt{\omega_0^2-\alpha^2}$ (real and positive since $\omega_0 > \alpha$ here), and $j \equiv \sqrt{-1}$. The roots are a complex-conjugate pair:
$$s_{1,2} = -\alpha \pm j\omega_d$$

The raw solution $A_1 e^{(-\alpha+j\omega_d)t} + A_2 e^{(-\alpha-j\omega_d)t}$ can be rewritten using Euler's identity, $e^{j\theta} = \cos\theta + j\sin\theta$ (a standard fact about complex exponentials): factor out $e^{-\alpha t}$ and combine the two complex exponential terms into a real sinusoid — for a physically real voltage $v_n(t)$, this combination always reduces to the real form:
$$v_n(t) = e^{-\alpha t}\big(B_1 \cos\omega_d t + B_2 \sin\omega_d t\big)$$
with $B_1, B_2$ real constants fixed by initial conditions. This is a sinusoid of angular frequency $\omega_d$, decaying inside an envelope $e^{-\alpha t}$ — the ringing behavior described in Intuition.

### Step 4 — Apply initial conditions to fix the constants

The full solution is $v(t) = v_p(t) + v_n(t) = V_s + v_n(t)$, with $v_n(t)$ from whichever case applies. We need two initial conditions because this is a second-order equation (two arbitrary constants to fix). The physically available ones are:

1. **Capacitor voltage continuity:** $v(0^+) = v(0^-)$, since $v_C$ cannot jump (Core definitions). If the capacitor starts uncharged, $v(0^+) = 0$.
2. **Inductor current continuity**, converted into a statement about $dv/dt$: since $i = C\,dv/dt$, and $i(0^+) = i(0^-)$ (inductor current cannot jump), we get $\dfrac{dv}{dt}\Big|_{t=0^+} = \dfrac{i(0^-)}{C}$. If the inductor starts with zero current, $\dfrac{dv}{dt}\Big|_{0^+} = 0$.

Worked numeric applications of this last step are in the Worked Examples below (this is where the three cases actually diverge into concrete formulas).

### The parallel RLC case (stated by duality, not re-derived)

For a parallel RLC circuit (a current source, or a capacitor with initial charge, driving $R$, $L$, $C$ all in parallel, with $v(t)$ the shared node voltage), KCL in place of KVL gives the dual equation
$$C\frac{dv}{dt} + \frac{v}{R} + i_L = I_s, \qquad \text{with } i_L\text{ related by } v = L\frac{di_L}{dt}$$
which reduces (eliminating $v$ in favor of $i_L$, by the same elimination method as Step 2) to
$$\frac{d^2 i_L}{dt^2} + \frac{1}{RC}\frac{di_L}{dt} + \frac{1}{LC}i_L = \frac{I_s}{LC}$$
This has exactly the same structure as (3), with the roles of "damping term coefficient" changed: comparing to (3)'s $\dfrac{R}{L}$ term, here the damping term is $\dfrac{1}{RC}$. So for parallel RLC:
$$\alpha_{\text{parallel}} = \frac{1}{2RC}, \qquad \omega_0 = \frac{1}{\sqrt{LC}} \ \text{(same formula — depends only on } L, C\text{)}$$
and the same three damping cases (over/critical/under) apply with this $\alpha$. Notice the role of $R$ flips: in series RLC, *larger* $R$ means *more* damping ($\alpha=R/2L$ grows with $R$); in parallel RLC, *larger* $R$ means *less* damping ($\alpha = 1/2RC$ shrinks as $R$ grows). This is worth memorizing as a pitfall (see below).

## Worked examples

### Example 1 — Series RLC step response, underdamped

A series loop has $R = 100\ \Omega$, $L = 0.1\ \text{H}$, $C = 0.1\ \mu\text{F} = 1\times10^{-7}\ \text{F}$, and a DC source $V_s = 10\ \text{V}$ switched on at $t=0$. Before the switch closes, the capacitor is uncharged and no current flows: $v_C(0^-) = 0$, $i_L(0^-) = 0$. Find $v_C(t)$ for $t > 0$ and classify the damping.

**Step 1 — compute $\omega_0$ and $\alpha$.**
$$\omega_0 = \frac{1}{\sqrt{LC}} = \frac{1}{\sqrt{(0.1\ \text{H})(1\times10^{-7}\ \text{F})}} = \frac{1}{\sqrt{1\times10^{-8}}} = \frac{1}{1\times10^{-4}} = 1\times10^{4}\ \text{rad/s}$$
$$\alpha = \frac{R}{2L} = \frac{100\ \Omega}{2(0.1\ \text{H})} = \frac{100}{0.2} = 500\ \text{s}^{-1}$$

**Step 2 — classify.** $\alpha = 500\ \text{s}^{-1} < \omega_0 = 10{,}000\ \text{rad/s}$, so $\zeta = \alpha/\omega_0 = 500/10000 = 0.05 \ll 1$: strongly **underdamped**. Compute the damped frequency:
$$\omega_d = \sqrt{\omega_0^2 - \alpha^2} = \sqrt{(10^4)^2 - (500)^2} = \sqrt{10^8 - 2.5\times10^5} = \sqrt{9.975\times10^7} \approx 9987\ \text{rad/s}$$
(very close to $\omega_0$ since damping is weak, as expected).

**Step 3 — write the general form and apply initial conditions.**
$$v(t) = V_s + e^{-\alpha t}\big(B_1\cos\omega_d t + B_2 \sin \omega_d t\big) = 10 + e^{-500t}\big(B_1 \cos(9987t) + B_2\sin(9987t)\big)$$

Initial condition 1 — voltage continuity: $v(0^+) = 0$.
$$0 = 10 + e^{0}\big(B_1 \cos 0 + B_2 \sin 0\big) = 10 + B_1 \implies B_1 = -10$$

Initial condition 2 — current continuity, giving $\dfrac{dv}{dt}\Big|_{0^+} = \dfrac{i_L(0^-)}{C} = \dfrac{0}{C} = 0$. Differentiate $v(t)$:
$$\frac{dv}{dt} = -\alpha e^{-\alpha t}(B_1\cos\omega_d t + B_2 \sin\omega_d t) + e^{-\alpha t}(-B_1\omega_d \sin \omega_d t + B_2 \omega_d \cos \omega_d t)$$
At $t=0$ ($\cos 0 = 1$, $\sin 0 = 0$, $e^0=1$):
$$\frac{dv}{dt}\Big|_{0} = -\alpha B_1 + B_2\omega_d$$
Set equal to 0 and solve for $B_2$:
$$0 = -\alpha B_1 + B_2 \omega_d \implies B_2 = \frac{\alpha B_1}{\omega_d} = \frac{(500)(-10)}{9987} = \frac{-5000}{9987} \approx -0.5007$$

**Step 4 — final answer.**
$$v_C(t) = 10 - e^{-500t}\big(10\cos(9987t) + 0.5007\sin(9987t)\big)\ \text{V}, \quad t>0$$

Sanity checks: at $t=0$, $v_C(0) = 10 - (10\cos 0 + 0.5007\sin 0) = 10 - 10 = 0$ V, matching the initial condition. As $t \to \infty$, $e^{-500t}\to 0$, so $v_C \to 10$ V $= V_s$, matching the expected DC final value. The voltage rings around 10 V with a decaying envelope of time constant $1/\alpha = 1/500\ \text{s} = 2\ \text{ms}$, oscillating at $\omega_d \approx 9987$ rad/s (period $T = 2\pi/\omega_d \approx 0.629\ \text{ms}$) — so you'd see roughly $2\ \text{ms}/0.629\ \text{ms} \approx 3$ visible rings before the transient is negligible.

### Example 2 — Series RLC, critically damped design + overdamped comparison

A series RLC circuit has $L = 1\ \text{mH} = 1\times10^{-3}\ \text{H}$ and $C = 1\ \mu\text{F} = 1\times10^{-6}\ \text{F}$, driven by a 5 V step, starting from rest ($v_C(0^-)=0$, $i_L(0^-)=0$). (a) Find the value of $R$ that makes the circuit exactly critically damped. (b) Write $v_C(t)$ for that $R$. (c) If instead $R$ is doubled from the critical value, is the circuit over- or underdamped?

**Part (a) — find $R$ for critical damping.**
Critical damping means $\alpha = \omega_0$, i.e., $\dfrac{R}{2L} = \dfrac{1}{\sqrt{LC}}$. Solve for $R$:
$$R = \frac{2L}{\sqrt{LC}} = 2\sqrt{\frac{L^2}{LC}} = 2\sqrt{\frac{L}{C}}$$
Plug in numbers:
$$R = 2\sqrt{\frac{1\times10^{-3}\ \text{H}}{1\times10^{-6}\ \text{F}}} = 2\sqrt{1000} = 2(31.62) \approx 63.25\ \Omega$$
(Units check: $L/C$ has units $\text{H/F} = (V\!\cdot\!s/A)/(A\!\cdot\!s/V) = V^2\!\cdot\!s^2/(A^2\!\cdot\!s^2)\cdot(1)$... more directly, $\sqrt{L/C}$ is a standard quantity with units of ohms, since $\omega_0 L = 1/(\omega_0 C)$ at resonance gives $L/C$ having units of $\Omega^2$.)

**Part (b) — solve for $v_C(t)$ at $R = 63.25\ \Omega$.**
Here $\alpha = \omega_0 = \dfrac{1}{\sqrt{LC}} = \dfrac{1}{\sqrt{(10^{-3})(10^{-6})}} = \dfrac{1}{\sqrt{10^{-9}}} = \dfrac{1}{3.162\times10^{-5}} \approx 3.162\times10^{4}\ \text{s}^{-1}$.

General critically-damped form: $v(t) = V_s + (A_1 + A_2 t)e^{-\alpha t} = 5 + (A_1+A_2 t)e^{-\alpha t}$.

Initial condition 1: $v(0^+) = 0 \implies 5 + A_1 = 0 \implies A_1 = -5$.

Initial condition 2: differentiate: $\dfrac{dv}{dt} = A_2 e^{-\alpha t} - \alpha(A_1+A_2 t)e^{-\alpha t}$. At $t=0$: $\dfrac{dv}{dt}\Big|_0 = A_2 - \alpha A_1$. This must equal $i_L(0^-)/C = 0$:
$$0 = A_2 - \alpha A_1 \implies A_2 = \alpha A_1 = (3.162\times10^4)(-5) = -1.581\times10^5$$

Final answer:
$$v_C(t) = 5 - \big(5 + 1.581\times10^{5}\,t\big)e^{-3.162\times10^{4}\,t}\ \text{V}, \quad t>0$$
Check at $t=0$: $5 - (5+0)(1) = 0$ V. Correct. As $t\to\infty$, the exponential kills the linear-in-$t$ term (exponential decay always beats polynomial growth), so $v_C \to 5$ V. Correct, and no oscillation appears anywhere in this expression (no sine/cosine) — consistent with critical damping.

**Part (c) — doubling $R$.**
New $R = 2(63.25\ \Omega) = 126.5\ \Omega$, so new $\alpha = R/2L = 126.5/(2\times10^{-3}) = 6.325\times10^4\ \text{s}^{-1}$, while $\omega_0$ is unchanged at $3.162\times10^4\ \text{rad/s}$ (it depends only on $L,C$, not $R$). Since the new $\alpha \approx 6.325\times10^4 > \omega_0 \approx 3.162\times10^4$, the circuit is now **overdamped**. This matches the general rule for series RLC: increasing $R$ increases damping, pushing you from critical into overdamped, never into underdamped.

## Common pitfalls

- **Forgetting that $R$'s effect on damping flips sign between series and parallel RLC.** In series RLC, more $R$ means more damping ($\alpha = R/2L$). In parallel RLC, more $R$ means *less* damping ($\alpha = 1/2RC$) — a large parallel resistor barely drains the tank, so the circuit rings longer. Mixing these up gives exactly the wrong design intuition.
- **Using the wrong initial condition for $dv/dt$.** The two initial conditions are *capacitor voltage* and *inductor current* (both physically continuous), not "voltage and its derivative" chosen arbitrarily. You must convert the inductor-current condition into a $dv/dt$ condition via $i = C\,dv/dt$ (or the dual, converting a capacitor-current condition into $di/dt$ for the parallel case) — forgetting the factor of $C$ (or $L$) here is one of the most common algebra slips.
- **Assuming the repeated-root case is just $Ae^{st}$.** At critical damping the two "would-be" independent exponential solutions coincide ($s_1=s_2$), so you need the second, linearly independent solution $t\,e^{st}$ as well, giving $(A_1+A_2t)e^{st}$. Dropping the $t$ term leaves you unable to satisfy both initial conditions.
- **Confusing $\omega_0$ (undamped natural frequency) with $\omega_d$ (damped natural frequency).** They are equal only in the limit $\alpha \to 0$ (no resistance at all). In any real underdamped circuit $\omega_d < \omega_0$ always, since $\omega_d = \sqrt{\omega_0^2-\alpha^2}$.
- **Forgetting the forced/particular response.** The natural response by itself decays to zero; it is not the whole answer. The full response is natural + forced, and the forced response is what the circuit settles to (e.g. $V_s$ for a DC step in the example above). Reporting only the decaying part gives a $v(t)$ that wrongly goes to 0 instead of $V_s$.
- **Sign errors under the square root in the characteristic equation.** Whether $\alpha^2-\omega_0^2$ is positive, zero, or negative is the entire branch point of the problem (real roots vs. repeated root vs. complex roots) — compute $\alpha$ and $\omega_0$ separately and compare them explicitly before deciding which case's formula to use, rather than guessing.
- **Plugging in $\zeta$ or $\alpha,\omega_0$ from the wrong topology.** A quick sanity check: $\omega_0=1/\sqrt{LC}$ never involves $R$ at all — if your $\omega_0$ formula has an $R$ in it, you've made an error.

## Self-check

### Questions

1. A series RLC circuit has $\alpha = 200\ \text{s}^{-1}$ and $\omega_0 = 500\ \text{rad/s}$. Is it overdamped, critically damped, or underdamped?
2. Write down the characteristic equation for a series RLC circuit in terms of $R$, $L$, $C$, and $s$, starting from the KVL loop equation. (You may state the final ODE and characteristic equation without re-deriving every substitution, but show at least the key elimination step.)
3. For a series RLC circuit, if you double $R$ while keeping $L$ and $C$ fixed, what happens to $\alpha$? What happens to $\omega_0$?
4. A parallel RLC circuit has $L = 10\ \text{mH}$, $C = 4\ \mu\text{F}$. What value of $R$ gives critical damping?
5. A series RLC circuit is underdamped with $\alpha = 1000\ \text{s}^{-1}$ and $\omega_d = 4000\ \text{rad/s}$. What is $\omega_0$?
6. Explain in one or two sentences, using the energy-storage picture (not equations), why an LC circuit with zero resistance oscillates forever while adding any positive $R$ eventually kills the oscillation.
7. A series RLC circuit has $R=50\ \Omega$, $L=2\ \text{mH}$, $C=0.5\ \mu\text{F}$, driven by a 12 V step from rest ($v_C(0^-)=0$, $i_L(0^-)=0$). Determine the damping regime and compute $\alpha$ and $\omega_0$ (do not need to solve the full $v(t)$).
8. For the critically-damped case, the natural response is $(A_1+A_2t)e^{st}$. Show (by direct substitution into the homogeneous ODE $\dfrac{d^2v_n}{dt^2}+2\alpha\dfrac{dv_n}{dt}+\alpha^2 v_n=0$, using $s=-\alpha$) that $v_n = t e^{-\alpha t}$ really does satisfy the equation.

### Answers

1. $\zeta = \alpha/\omega_0 = 200/500 = 0.4 < 1$, so the circuit is **underdamped**.
2. KVL: $V_s = iR + L\,di/dt + v_C$. Substitute $i = C\,dv_C/dt$ (key elimination step): $V_s = RC\,dv_C/dt + LC\,d^2v_C/dt^2 + v_C$, i.e. $LC\,\ddot v_C + RC\,\dot v_C + v_C = V_s$. Trying $v_n=e^{st}$ in the homogeneous version gives the characteristic equation $LCs^2+RCs+1=0$, or equivalently (dividing by $LC$) $s^2+(R/L)s+1/(LC)=0$.
3. $\alpha = R/2L$ **doubles** (since it's directly proportional to $R$, with $L$ fixed). $\omega_0 = 1/\sqrt{LC}$ is **unchanged** — it depends only on $L$ and $C$, not on $R$ at all.
4. Parallel RLC critical damping: $\alpha=\omega_0 \Rightarrow \dfrac{1}{2RC}=\dfrac{1}{\sqrt{LC}} \Rightarrow R = \dfrac{\sqrt{LC}}{2C} = \dfrac{1}{2}\sqrt{\dfrac{L}{C}}$. Plug in: $\sqrt{L/C} = \sqrt{(0.01\ \text{H})/(4\times10^{-6}\ \text{F})} = \sqrt{2500} = 50$. So $R = 0.5 \times 50 = 25\ \Omega$.
5. $\omega_d = \sqrt{\omega_0^2-\alpha^2} \Rightarrow \omega_0 = \sqrt{\omega_d^2+\alpha^2} = \sqrt{4000^2+1000^2} = \sqrt{16{,}000{,}000+1{,}000{,}000} = \sqrt{17{,}000{,}000} \approx 4123\ \text{rad/s}$.
6. With zero resistance nothing removes energy from the loop, so the fixed total stored energy just keeps converting back and forth between the capacitor's electric field and the inductor's magnetic field forever, at the rate set by $L$ and $C$ — a lossless exchange, like a frictionless pendulum. Any positive $R$ dissipates some of that energy as heat every cycle, so the total energy available to slosh back and forth shrinks every period until essentially none is left, which is why the oscillation (if any) always decays and eventually the circuit sits at whatever the source dictates.
7. $\omega_0 = 1/\sqrt{LC} = 1/\sqrt{(2\times10^{-3})(0.5\times10^{-6})} = 1/\sqrt{1\times10^{-9}} = 1/(3.162\times10^{-5}) \approx 3.162\times10^{4}\ \text{rad/s}$. $\alpha = R/2L = 50/(2\times2\times10^{-3}) = 50/0.004 = 1.25\times10^{4}\ \text{s}^{-1}$. Since $\alpha \approx 1.25\times10^4 < \omega_0 \approx 3.162\times10^4$, the circuit is **underdamped**.
8. Let $v_n = te^{-\alpha t}$. First derivative (product rule): $\dot v_n = e^{-\alpha t} + t(-\alpha)e^{-\alpha t} = e^{-\alpha t}(1-\alpha t)$. Second derivative: $\ddot v_n = -\alpha e^{-\alpha t}(1-\alpha t) + e^{-\alpha t}(-\alpha) = e^{-\alpha t}\big[-\alpha(1-\alpha t) - \alpha\big] = e^{-\alpha t}\big[-2\alpha+\alpha^2 t\big]$. Now substitute into $\ddot v_n + 2\alpha \dot v_n + \alpha^2 v_n$:
$$e^{-\alpha t}(-2\alpha+\alpha^2 t) + 2\alpha\, e^{-\alpha t}(1-\alpha t) + \alpha^2\, t\,e^{-\alpha t}$$
Factor out $e^{-\alpha t}$ and collect the bracket:
$$(-2\alpha+\alpha^2 t) + (2\alpha - 2\alpha^2 t) + \alpha^2 t = -2\alpha+2\alpha + \alpha^2 t - 2\alpha^2 t + \alpha^2 t = 0$$
All terms cancel to 0, confirming $v_n=te^{-\alpha t}$ satisfies the homogeneous equation, as claimed.

## Summary / cheat sheet

**Series RLC** (loop of $V_s$, $R$, $L$, $C$; unknown $v_C$):
$$LC\ddot v_C + RC\dot v_C + v_C = V_s, \qquad \alpha=\frac{R}{2L}, \quad \omega_0=\frac{1}{\sqrt{LC}}$$

**Parallel RLC** (node with $I_s$, $R$, $L$, $C$; unknown $i_L$):
$$LC\ddot i_L + \frac{L}{R}\dot i_L + i_L = I_s, \qquad \alpha=\frac{1}{2RC}, \quad \omega_0=\frac{1}{\sqrt{LC}}$$

**Characteristic roots:** $s_{1,2} = -\alpha \pm \sqrt{\alpha^2-\omega_0^2}$. Damping ratio $\zeta = \alpha/\omega_0$.

| Regime | Condition | Roots | Natural response form |
|---|---|---|---|
| Overdamped | $\alpha>\omega_0$ ($\zeta>1$) | 2 distinct real negative | $A_1e^{s_1t}+A_2e^{s_2t}$ |
| Critically damped | $\alpha=\omega_0$ ($\zeta=1$) | 1 repeated real negative, $s=-\alpha$ | $(A_1+A_2t)e^{-\alpha t}$ |
| Underdamped | $\alpha<\omega_0$ ($\zeta<1$) | complex pair $-\alpha\pm j\omega_d$ | $e^{-\alpha t}(B_1\cos\omega_d t+B_2\sin\omega_d t)$ |

$$\omega_d = \sqrt{\omega_0^2-\alpha^2}\ \ (\text{underdamped only})$$

**Full response** $= $ forced (particular, matches source shape) $+$ natural (decays to 0). Fix the two constants using: (1) $v_C(0^+)=v_C(0^-)$ (capacitor voltage continuous), (2) $i_L(0^+)=i_L(0^-)$, converted to a slope condition via $i=C\,dv/dt$ (series case) or $v=L\,di/dt$ (parallel case).

**Key sanity checks:** $\omega_0$ never contains $R$. In series RLC, bigger $R$ → more damping; in parallel RLC, bigger $R$ → less damping. Exponential decay always beats any polynomial-in-$t$ prefactor, so every case settles to the forced response as $t\to\infty$ provided $\alpha>0$.

## Used later in
(none yet — filled in once a later note cites this one)
