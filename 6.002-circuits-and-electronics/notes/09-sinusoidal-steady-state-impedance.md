---
title: "Sinusoidal Steady State, Impedance"
course: "6.002"
topic_number: 09
prerequisites: ["6.002-circuits-and-electronics/notes/02-resistive-networks-node-mesh-analysis", "6.002-circuits-and-electronics/notes/03-dependent-sources-superposition-thevenin-norton", "6.002-circuits-and-electronics/notes/06-capacitors-inductors-energy-storage", "6.002-circuits-and-electronics/notes/07-first-order-transients-rc-rl", "6.002-circuits-and-electronics/notes/08-second-order-transients-rlc-damping"]
status: in progress
---

# Sinusoidal Steady State, Impedance

## Why this matters

Every earlier topic in this course dealt with circuits driven by DC sources or by sources that switch on/off (producing transients that die away). But most of the electrical world — the power grid, radio, audio, communication links — runs on signals that oscillate forever: $v(t) = V\cos(\omega t + \phi)$. If we tried to analyze such a circuit using the raw differential-equation methods from transient analysis (writing KVL/KCL, getting a differential equation, solving for the transient plus a particular solution), we would drown in algebra every time we added an element. This topic introduces **impedance**: a trick that converts capacitors and inductors into "complex-valued resistors" once a circuit has settled into sinusoidal steady state. With impedance, every resistive-network technique already known ([[6.002-circuits-and-electronics/notes/02-resistive-networks-node-mesh-analysis]]'s node analysis and series/parallel combination, [[6.002-circuits-and-electronics/notes/03-dependent-sources-superposition-thevenin-norton]]'s Thevenin/Norton equivalents, and voltage/current dividers) applies unchanged to AC circuits — just using complex numbers instead of real ones. Skip this topic and every later topic (op-amp frequency response, Bode plots) becomes unreachable, because they are all phrased directly in terms of impedance.

## Builds on

- [[6.002-circuits-and-electronics/notes/06-capacitors-inductors-energy-storage]] — the defining relations $i = C\,dv/dt$ for a capacitor and $v = L\,di/dt$ for an inductor.
- [[6.002-circuits-and-electronics/notes/07-first-order-transients-rc-rl]] — the natural response / forced response split for a differential-equation circuit, and the notion that the natural response decays away leaving a steady state.
- [[6.002-circuits-and-electronics/notes/08-second-order-transients-rlc-damping]] — the fact that solving a circuit with energy-storage elements means solving a differential equation, here for the second-order (series/parallel $RLC$) case. A short recap of both prior results is included inline below for continuity.
- [[6.002-circuits-and-electronics/notes/02-resistive-networks-node-mesh-analysis]] — node-voltage analysis (assign a voltage variable to each non-reference node, write KCL there, solve the resulting linear system) and the series/parallel resistor-combination and voltage/current-divider formulas, all originally derived for real-valued resistances.
- [[6.002-circuits-and-electronics/notes/03-dependent-sources-superposition-thevenin-norton]] — the Thevenin equivalent (any linear two-terminal network reduces to a single voltage source in series with a single resistance) and Norton equivalent (current source in parallel with a resistance), both originally derived for real-valued resistances.

## Core definitions

- **Sinusoidal steady state (SSS)** — the condition of a linear circuit, driven by a sinusoidal source of a single fixed frequency $\omega$, after all transient ("natural response") terms have decayed to zero, leaving only a response that oscillates at the same frequency $\omega$ as the drive, forever. ("Steady state" here means *periodic* steady state, not constant — every voltage and current is still time-varying, but its pattern repeats indefinitely.)
- **Phasor** — a complex number that represents a real sinusoid of known angular frequency $\omega$ by encoding only its amplitude and phase. For a signal $x(t) = X\cos(\omega t + \phi)$, the phasor is $\hat{X} = X e^{j\phi}$ (a complex number; $X$ is the real, nonnegative amplitude and $\phi$ the phase, both independent of $t$). The angular frequency $\omega$ is *not* stored in the phasor — it is a separate, shared piece of information that applies to the whole circuit.
- **Complex exponential representation** — the identity $\cos(\omega t + \phi) = \mathrm{Re}\{e^{j(\omega t + \phi)}\}$, where $j = \sqrt{-1}$ (electrical engineering uses $j$, not $i$, to avoid clashing with current $i$). This lets us replace a real sinusoid $x(t)$ with the complex signal $\hat{X} e^{j\omega t}$ and recover $x(t)$ at the end by taking the real part.
- **Impedance** $Z$ — the complex-valued ratio of the phasor voltage across a two-terminal element to the phasor current through it, in sinusoidal steady state: $Z = \hat{V}/\hat{I}$. Units: ohms ($\Omega$), same as resistance. $Z$ depends on $\omega$ but not on time.
- **Admittance** $Y$ — the reciprocal of impedance, $Y = 1/Z = \hat{I}/\hat{V}$, units siemens (S). Used occasionally for parallel combinations, analogous to conductance $G = 1/R$.
- **Reactance** $X$ — the imaginary part of impedance, $Z = R_{\text{eff}} + jX$. Reactance stores/returns energy (like $L, C$) rather than dissipating it (like $R$).
- **Angular frequency** $\omega$ — rate of oscillation in radians per second, related to ordinary frequency $f$ (hertz) by $\omega = 2\pi f$, and to period $T$ (seconds) by $\omega = 2\pi/T$.
- **RMS (root-mean-square) value** $X_{\text{RMS}}$ — for a sinusoid $x(t) = X\cos(\omega t+\phi)$ with peak amplitude $X$, the RMS value is $X_{\text{RMS}} = X/\sqrt2$: the DC-equivalent magnitude that would dissipate the same average power in a resistor ($X_{\text{RMS}} \equiv \sqrt{\langle x(t)^2\rangle}$, the square root of the time-average of $x(t)^2$ over one period). Every quantity in this note ($V$, $I$, phasor magnitudes) is expressed in peak amplitude unless explicitly stated otherwise.

## Intuition

Here is the core recap of what a capacitor and inductor do, self-contained, before we build on it. A **capacitor** stores charge on two plates separated by an insulator; its defining law is $i_C(t) = C \dfrac{dv_C(t)}{dt}$ — the current into it is proportional to *how fast* its voltage is changing, not to the voltage itself. An **inductor** stores energy in a magnetic field created by current flowing through a coil; its defining law is $v_L(t) = L \dfrac{di_L(t)}{dt}$ — the voltage across it is proportional to how fast its current is changing. Because both laws involve derivatives, any circuit containing $L$ or $C$ obeys a differential equation, not just an algebraic (Ohm's-law-style) equation. In earlier transient-analysis topics, solving that differential equation for a step or pulse input gave a response with two parts: a **natural response** (an exponential, e.g., $e^{-t/RC}$, that depends only on the circuit and decays to zero) plus a **forced/particular response** that tracks the input. Once the natural response has died out, only the forced response is left — that is the *steady state*.

Now specialize the input to a sinusoid, $v_s(t) = V_s\cos(\omega t)$, and ask what the forced response looks like once steady state is reached. Physically: a resistor's current instantly tracks its voltage (no memory), so driving a resistor with a cosine gives back a cosine at the same frequency, same phase, scaled by $1/R$. A capacitor's current is proportional to the *derivative* of a sinusoid, and the derivative of a cosine is a (scaled, negated) sine — so the capacitor's current is a sinusoid of the *same frequency* $\omega$, but shifted in phase and scaled in amplitude. The same holds for an inductor's voltage relative to its current. This is the key physical fact that makes impedance possible: **linear circuits driven by a sinusoid, in steady state, respond with a sinusoid of the exact same frequency** — only the amplitude and phase differ. Since frequency never changes, we don't need to carry $\omega t$ around symbolically through every calculation; we only need to track *how* each element scales amplitude and shifts phase. A phasor is exactly that bookkeeping device: a complex number holding amplitude and phase, with the common $e^{j\omega t}$ factor stripped off because it is identical (and hence cancels) everywhere in the circuit.

Once every voltage and current is replaced by its phasor, the derivative operator $d/dt$ turns into "multiply by $j\omega$" (shown rigorously below), so the differential equations describing $L$ and $C$ collapse into plain algebraic Ohm's-law-like relations, $\hat V = Z\hat I$, with $Z$ complex. This means every DC-circuit trick — series/parallel resistor combination formulas, voltage dividers, node-voltage KCL equations, Thevenin equivalents — carries over verbatim to sinusoidal AC circuits, just replacing real resistances with complex impedances and real voltages/currents with complex phasors.

## Derivation / formalism

### Step 1: Represent a real sinusoid as the real part of a complex exponential

Euler's formula states $e^{j\theta} = \cos\theta + j\sin\theta$. Therefore, for any real amplitude $X\ge 0$, phase $\phi$, and angular frequency $\omega$,
$$
X\cos(\omega t+\phi) = \mathrm{Re}\left\{X e^{j(\omega t+\phi)}\right\} = \mathrm{Re}\left\{\left(Xe^{j\phi}\right)e^{j\omega t}\right\}.
$$
Define the **phasor** $\hat X \triangleq Xe^{j\phi}$ (a complex constant, no $t$ dependence). Then any real sinusoidal signal can be written
$$
x(t) = \mathrm{Re}\{\hat X e^{j\omega t}\}.
$$
We will work with the *complex* signal $\underline{x}(t) \triangleq \hat X e^{j\omega t}$ and only take the real part at the very end. This is legal because the circuit equations we are about to use (KVL, KCL, the $C$ and $L$ laws) are all *linear* with *real* coefficients, and the real part of a sum/derivative of complex signals equals the sum/derivative of the real parts — i.e., $\mathrm{Re}\{\cdot\}$ commutes with every operation a linear circuit performs. So if the complex signals satisfy the circuit's equations, their real parts (the actual physical signals) do too.

### Step 2: Differentiating a phasor signal = multiplying by $j\omega$

Take the complex signal $\underline{x}(t) = \hat X e^{j\omega t}$ and differentiate with respect to time:
$$
\frac{d}{dt}\underline{x}(t) = \frac{d}{dt}\left(\hat X e^{j\omega t}\right) = \hat X \cdot j\omega \cdot e^{j\omega t} = (j\omega \hat X)\,e^{j\omega t}.
$$
This is again of the form (some phasor) $\times\, e^{j\omega t}$ — i.e., differentiating in time corresponds exactly to multiplying the phasor by $j\omega$. This single fact is what turns differential equations into algebraic ones.

### Step 3: Impedance of a resistor

Ohm's law: $v_R(t) = R\, i_R(t)$ for all $t$. Substitute $v_R(t) = \mathrm{Re}\{\hat V e^{j\omega t}\}$ and $i_R(t) = \mathrm{Re}\{\hat I e^{j\omega t}\}$. Since this must hold with the real part stripped too (by the linearity argument of Step 1), the complex signals satisfy $\hat V e^{j\omega t} = R\,\hat I e^{j\omega t}$, so
$$
\hat V = R\,\hat I \quad\Longrightarrow\quad Z_R = \frac{\hat V}{\hat I} = R.
$$
A resistor's impedance is just its resistance — real-valued, no phase shift, independent of $\omega$.

### Step 4: Impedance of a capacitor

Capacitor law: $i_C(t) = C\dfrac{dv_C(t)}{dt}$. Write $v_C(t) = \mathrm{Re}\{\hat V e^{j\omega t}\}$; then by Step 2, $i_C(t) = \mathrm{Re}\{C\cdot j\omega \hat V\, e^{j\omega t}\}$, so the current phasor is $\hat I = j\omega C\,\hat V$. Solve for the impedance:
$$
Z_C = \frac{\hat V}{\hat I} = \frac{\hat V}{j\omega C\,\hat V} = \frac{1}{j\omega C}.
$$
Rewrite using $1/j = -j$ (since $j\cdot(-j) = -j^2 = 1$):
$$
Z_C = \frac{-j}{\omega C} = \frac{1}{\omega C}\angle{-90^\circ}.
$$
So a capacitor's impedance has magnitude $1/(\omega C)$ (shrinking as frequency rises — capacitors "pass" high frequencies more easily) and a phase of exactly $-90^\circ$: the capacitor's current phasor leads its voltage phasor by $90^\circ$ (equivalently, voltage lags current by $90^\circ$).

### Step 5: Impedance of an inductor

Inductor law: $v_L(t) = L\dfrac{di_L(t)}{dt}$. By the same steps, with $i_L(t) = \mathrm{Re}\{\hat I e^{j\omega t}\}$, the voltage phasor is $\hat V = j\omega L\,\hat I$, so
$$
Z_L = \frac{\hat V}{\hat I} = j\omega L = \omega L\angle{+90^\circ}.
$$
An inductor's impedance grows with frequency, and its voltage phasor leads its current phasor by $90^\circ$.

### Step 6: KVL/KCL survive unchanged under the phasor transform

KVL says the sum of voltage drops around any loop is zero at every instant $t$: $\sum_k v_k(t) = 0$. Writing each $v_k(t) = \mathrm{Re}\{\hat V_k e^{j\omega t}\}$, linearity of $\mathrm{Re}\{\cdot\}$ and of the sum gives $\mathrm{Re}\{(\sum_k \hat V_k)e^{j\omega t}\}=0$ for all $t$, which forces $\sum_k \hat V_k = 0$ (a sum of the form $\mathrm{Re}\{Ae^{j\omega t}\}$ is zero for all $t$ only if the complex constant $A=0$, because $e^{j\omega t}$ sweeps through all phases). So **KVL holds directly on phasors**: $\sum_k \hat V_k = 0$. By an identical argument, **KCL holds directly on phasors**: $\sum_k \hat I_k = 0$ at any node.

### Step 7: Consequence — all resistive-network algebra carries over

Because (a) each element's phasor voltage and phasor current are related by $\hat V = Z\hat I$ (Ohm's-law form, Steps 3–5) and (b) KVL/KCL hold on phasors exactly as they did on DC voltages/currents (Step 6), every algebraic manipulation valid for resistor circuits is valid for impedance circuits, with $R\to Z$ (complex) and $v,i \to \hat V,\hat I$ (complex). In particular:

- **Series impedances add:** $Z_{\text{series}} = Z_1+Z_2+\dots$ (same KVL argument as series resistors).
- **Parallel impedances combine as reciprocal-sum:** $\dfrac{1}{Z_{\text{parallel}}} = \dfrac{1}{Z_1}+\dfrac{1}{Z_2}+\dots$ (same KCL argument as parallel resistors).
- **Voltage/current dividers, node-voltage analysis ([[6.002-circuits-and-electronics/notes/02-resistive-networks-node-mesh-analysis]]), Thevenin/Norton equivalents ([[6.002-circuits-and-electronics/notes/03-dependent-sources-superposition-thevenin-norton]])** — all derived only from KVL, KCL, and $v=Zi$, so all transfer verbatim with complex arithmetic.

### Step 8: Recovering the real time-domain answer

Once phasors $\hat V,\hat I$ for a desired quantity are found (by ordinary complex algebra), convert back: if $\hat X = X\angle\phi$ (i.e. $X = |\hat X|$, $\phi = \angle \hat X$), then the actual physical, real-valued sinusoid is
$$
x(t) = X\cos(\omega t+\phi).
$$
This is just undoing Step 1.

## Worked examples

### Example 1 — Series $RC$ voltage divider in sinusoidal steady state

A resistor $R = 1000\,\Omega$ and a capacitor $C = 1\,\mu\text{F} = 1\times10^{-6}\,\text{F}$ are in series, driven by a source $v_s(t) = 5\cos(2000t)\,\text{V}$ (so $\omega = 2000\,\text{rad/s}$). Find the steady-state voltage $v_C(t)$ across the capacitor.

**Step A — phasor of the source.** $v_s(t) = 5\cos(2000t + 0^\circ)$, so $\hat V_s = 5\angle 0^\circ = 5\,\text{V}$ (a real number here since the phase is zero).

**Step B — impedances.**
$$
Z_R = R = 1000\,\Omega.
$$
$$
Z_C = \frac{1}{j\omega C} = \frac{1}{j(2000\,\text{rad/s})(1\times10^{-6}\,\text{F})} = \frac{1}{j(2\times10^{-3})}\,\Omega = \frac{1}{0.002}\cdot\frac{1}{j}\,\Omega = 500\cdot(-j)\,\Omega = -j500\,\Omega.
$$
(Units check: $\text{rad/s}\times\text{F} = \text{rad/s} \times \text{s/}\Omega = \text{rad}/\Omega$; since radians are dimensionless here, this is $1/\Omega$, so $1/(\omega C)$ is in $\Omega$. Good.)

**Step C — voltage divider (same formula as resistive divider, with $Z$ in place of $R$):**
$$
\hat V_C = \hat V_s\cdot\frac{Z_C}{Z_R+Z_C} = 5\,\text{V}\cdot\frac{-j500}{1000-j500}.
$$
Compute the fraction. Multiply numerator and denominator by the conjugate of the denominator, $1000+j500$:
$$
\frac{-j500}{1000-j500}\cdot\frac{1000+j500}{1000+j500} = \frac{-j500(1000+j500)}{1000^2+500^2} = \frac{-j500{,}000 - j^2 250{,}000}{1{,}000{,}000+250{,}000}.
$$
Since $j^2=-1$, $-j^2\cdot250{,}000 = +250{,}000$. So numerator $= 250{,}000 - j500{,}000$, denominator $=1{,}250{,}000$:
$$
\frac{250{,}000-j500{,}000}{1{,}250{,}000} = 0.2-j0.4.
$$
Convert $0.2-j0.4$ to magnitude/angle: magnitude $=\sqrt{0.2^2+0.4^2}=\sqrt{0.04+0.16}=\sqrt{0.20}=0.4472$. Angle $=\arctan\!\left(\dfrac{-0.4}{0.2}\right)=\arctan(-2) = -63.43^\circ$ (in the fourth quadrant, since real part positive, imaginary part negative — consistent with $\arctan$ directly).

So $\hat V_C = 5\,\text{V}\times 0.4472\angle{-63.43^\circ} = 2.236\,\text{V}\angle{-63.43^\circ}$.

**Step D — back to time domain:**
$$
v_C(t) = 2.236\,\text{V}\cos\!\big(2000t - 63.43^\circ\big).
$$
Sanity checks: (1) magnitude $2.236\,\text{V} < 5\,\text{V}$ — the capacitor gets only part of the source voltage, as expected for a divider. (2) The phase is negative — the capacitor voltage lags the source, consistent with a capacitor's voltage lagging current, and current here is roughly in phase with the source when $R$ dominates less. (3) At $\omega=0$ (DC), $Z_C\to\infty$ and the divider ratio $\to 1$, matching the known DC behavior of a charged capacitor eventually equaling the source voltage — a useful limiting-case check on the divider formula itself.

### Example 2 — Parallel $RL$ combination: equivalent impedance and phase angle

A resistor $R = 200\,\Omega$ is in parallel with an inductor $L = 0.1\,\text{H}$, at $\omega = 1000\,\text{rad/s}$. Find the equivalent impedance $Z_{eq}$ and state whether the combination is net inductive or resistive-dominated.

**Step A — individual impedances.**
$$
Z_R = 200\,\Omega, \qquad Z_L = j\omega L = j(1000)(0.1) = j100\,\Omega.
$$

**Step B — parallel combination formula (same as parallel resistors, complex-valued):**
$$
Z_{eq} = \frac{Z_R\,Z_L}{Z_R+Z_L} = \frac{(200)(j100)}{200+j100} = \frac{j20{,}000}{200+j100}.
$$
Multiply numerator and denominator by the conjugate $200 - j100$:
$$
\frac{j20{,}000(200-j100)}{(200+j100)(200-j100)} = \frac{j20{,}000\cdot200 - j20{,}000\cdot j100}{200^2+100^2}.
$$
Numerator: $j4{,}000{,}000 - j^2\,2{,}000{,}000 = j4{,}000{,}000 + 2{,}000{,}000$ (since $-j^2=+1$). Denominator: $40{,}000+10{,}000=50{,}000$.
$$
Z_{eq} = \frac{2{,}000{,}000 + j4{,}000{,}000}{50{,}000} = 40 + j80\ \ \Omega.
$$

**Step C — magnitude and phase.**
$$
|Z_{eq}| = \sqrt{40^2+80^2} = \sqrt{1600+6400} = \sqrt{8000} \approx 89.44\,\Omega.
$$
$$
\angle Z_{eq} = \arctan\!\left(\frac{80}{40}\right) = \arctan(2) \approx 63.43^\circ.
$$

**Interpretation.** $Z_{eq} = 40+j80\,\Omega$ has a positive imaginary (reactive) part, meaning the combination is **net inductive**: current phasor lags voltage phasor by about $63.4^\circ$. This makes physical sense — even though the resistor alone is purely real, at this frequency the inductor's impedance magnitude ($100\,\Omega$) is comparable to $R$ ($200\,\Omega$), so it meaningfully shunts current and pulls the combined behavior toward the inductive (lagging) side rather than being negligible.

## Common pitfalls

- **Mixing up $j$ and $i$.** In circuits, $i$ always means current; the imaginary unit is written $j=\sqrt{-1}$. Never write $i=\sqrt{-1}$ in this course — it collides with current notation.
- **Forgetting that impedance depends on $\omega$.** $Z_C = 1/(j\omega C)$ and $Z_L = j\omega L$ are not fixed numbers; changing the drive frequency changes every impedance in the circuit and hence the whole answer. A single circuit has different phasor answers at different frequencies — always recompute $Z_C,Z_L$ for the specific $\omega$ given.
- **Dropping the $j$ or its sign when computing $1/(j\omega C)$.** Remember $1/j=-j$. A common error is writing $Z_C = j/( \omega C)$ (wrong sign) instead of $-j/(\omega C)$.
- **Using peak amplitude where RMS was intended, or vice versa, in power calculations.** This note phrases everything with peak amplitude $X$ in $\hat X = Xe^{j\phi}$; if a source is instead specified by RMS value, convert with $X_{\text{peak}} = \sqrt2\, X_{\text{RMS}}$ before forming the phasor. Power formulas commonly used elsewhere ($P=\tfrac12 VI\cos\theta$ with peak values, or $P=V_{\text{rms}}I_{\text{rms}}\cos\theta$) are easy to conflate — check which convention a given problem uses.
- **Treating phasors as if they carry time dependence.** A phasor $\hat X$ is a single fixed complex number for a given signal; it is *not* a function of $t$. The $e^{j\omega t}$ factor is implicit and shared by the whole circuit — do not write $\hat X(t)$.
- **Applying phasor/impedance analysis to non-sinusoidal or transient signals.** Impedance is valid only once the *sinusoidal steady state* has been reached, for a single fixed frequency. It cannot be used directly for a step input, a switch-closing transient, or a signal with multiple frequency components without doing a separate phasor calculation per frequency (superposition) — and even then, only after each frequency's own transient has died out.
- **Sign/direction errors in defining $\hat V$ and $\hat I$ for an element.** Impedance $Z=\hat V/\hat I$ assumes the "passive sign convention" — current phasor $\hat I$ defined flowing into the terminal marked $+$ for $\hat V$. Flip the assumed current direction and you flip the sign of the computed $Z$'s reactive part.
- **Forgetting to convert back to a real cosine at the end.** Phasor algebra gives a complex number; the physically meaningful answer is $x(t)=X\cos(\omega t+\phi)$ where $X=|\hat X|,\ \phi=\angle \hat X$. Reporting the phasor itself as "the voltage" without this last conversion step is incomplete.

## Self-check

### Questions

1. What is the impedance of a $10\,\mu\text{F}$ capacitor at $\omega = 5000\,\text{rad/s}$? Give the answer as a complex number.
2. A resistor and an inductor are in series. As $\omega\to 0$, does the impedance of the inductor's branch dominate or vanish relative to the resistor? Explain physically.
3. Convert the phasor $\hat V = 10\angle{30^\circ}\,\text{V}$ (with $\omega = 100\,\text{rad/s}$) back into a time-domain sinusoid $v(t)$.
4. Two impedances $Z_1 = 50\,\Omega$ and $Z_2 = j50\,\Omega$ are in series, driven by $\hat V_s = 100\angle 0^\circ\,\text{V}$. Find the phasor current $\hat I$.
5. For the same circuit as Question 4, find the phasor voltage across $Z_2$ using the voltage-divider formula, and state whether it leads or lags $\hat V_s$.
6. Explain, using the derivation in Step 2, why $Z_L$ must be purely imaginary for an ideal inductor (no resistance) — i.e., why can't an ideal inductor's impedance have a nonzero real part?
7. A series $RLC$ circuit has $R=100\,\Omega$, $L=0.05\,\text{H}$, $C=2\,\mu\text{F}$. At what angular frequency $\omega$ does the *net reactance* ($X_L - |X_C|$, i.e. $\mathrm{Im}\{Z_L+Z_C\}$) equal zero? (This frequency is called resonance — you are not expected to have seen the term before; just solve the algebra.)
8. Why does phasor analysis fail to correctly describe a circuit immediately after a switch closes (i.e., during the transient), even though the drive is a pure sinusoid for $t>0$?

### Answers

1. $Z_C = \dfrac{1}{j\omega C} = \dfrac{1}{j(5000)(10\times10^{-6})} = \dfrac{1}{j(0.05)} = \dfrac{-j}{0.05} = -j20\,\Omega.$
2. As $\omega\to0$, $Z_L=j\omega L\to 0$: the inductor's impedance vanishes relative to the resistor. Physically, at DC (zero frequency) current isn't changing, so $v_L=L\,di/dt\to0$ — an inductor looks like a short circuit (zero-ohm wire) once steady DC is reached, so essentially all the resistor's voltage divider weight remains with $R$.
3. $v(t) = 10\cos(100t+30^\circ)\ \text{V}.$ (Amplitude = magnitude of phasor, phase = angle of phasor, frequency $\omega$ carried separately as given.)
4. $Z_{\text{total}} = Z_1+Z_2 = 50+j50\,\Omega$. Magnitude $=\sqrt{50^2+50^2}=50\sqrt2\approx70.71\,\Omega$; angle $=\arctan(50/50)=45^\circ$. So $Z_{\text{total}}=70.71\angle45^\circ\,\Omega$. Then $\hat I = \hat V_s/Z_{\text{total}} = \dfrac{100\angle0^\circ}{70.71\angle45^\circ} = 1.414\angle{-45^\circ}\,\text{A}.$
5. $\hat V_{Z_2} = \hat V_s\cdot\dfrac{Z_2}{Z_1+Z_2} = 100\angle0^\circ\cdot\dfrac{j50}{50+j50}$. Note $j50 = 50\angle90^\circ$ and $50+j50=70.71\angle45^\circ$ (from Q4). So $\hat V_{Z_2}=100\cdot\dfrac{50\angle90^\circ}{70.71\angle45^\circ} = 100\cdot0.7071\angle(90^\circ-45^\circ) = 70.71\angle45^\circ\,\text{V}$. This has a positive phase angle relative to $\hat V_s$ (which is at $0^\circ$), so $\hat V_{Z_2}$ **leads** $\hat V_s$ by $45^\circ$.
6. From Step 2/Step 5, $\hat V = j\omega L\,\hat I$, so $Z_L=\hat V/\hat I = j\omega L$ — a real number $\omega L$ multiplied by $j$, which is purely imaginary for any real $\omega,L>0$. There is no algebraic step that could introduce a real part unless the inductor itself had series resistance (a separate physical element, not part of the ideal $v=L\,di/dt$ law). The derivative operation ($d/dt\to j\omega$) intrinsically injects a factor of $j$; an ideal inductor's defining law has no term proportional to $i$ itself (only to $di/dt$), so no real (in-phase) component can appear.
7. Net reactance zero means $\mathrm{Im}\{Z_L\} + \mathrm{Im}\{Z_C\} = 0$: $\omega L - \dfrac{1}{\omega C}=0 \Rightarrow \omega^2 = \dfrac{1}{LC} \Rightarrow \omega=\dfrac{1}{\sqrt{LC}}$. Plugging in: $LC = (0.05)(2\times10^{-6}) = 1\times10^{-7}$, so $\omega = 1/\sqrt{1\times10^{-7}} = 1/(3.162\times10^{-4}) \approx 3162\,\text{rad/s}.$
8. Phasor/impedance analysis assumes the circuit is already in *sinusoidal steady state* — i.e., that the natural-response (transient) term from solving the underlying differential equation has already decayed to (essentially) zero, leaving only the forced sinusoidal term. Immediately after a switch closes, the circuit's voltages/currents are a *superposition* of that not-yet-decayed transient term (an exponential depending on initial conditions) plus the eventual steady-state sinusoid. Phasors only capture the second piece; they say nothing about the exponential transient, so using them alone during that window gives an incomplete (and generally wrong) answer until enough time (several time constants) has passed for the transient to die out.

## Summary / cheat sheet

- **Phasor**: $x(t)=X\cos(\omega t+\phi) \;\longleftrightarrow\; \hat X = X\angle\phi = Xe^{j\phi}$ (complex number; $\omega$ tracked separately, shared by whole circuit).
- **Differentiation rule**: $d/dt \to \times\,(j\omega)$ on phasors.
- **Impedances**:
  - Resistor: $Z_R = R$ (real, no phase shift).
  - Capacitor: $Z_C = \dfrac{1}{j\omega C} = -j\dfrac{1}{\omega C}$ (voltage lags current by $90^\circ$; $|Z_C|$ shrinks as $\omega$ rises).
  - Inductor: $Z_L = j\omega L$ (voltage leads current by $90^\circ$; $|Z_L|$ grows as $\omega$ rises).
- **KVL/KCL** hold directly on phasors: $\sum \hat V_k = 0$ around a loop, $\sum \hat I_k = 0$ at a node.
- **All DC-circuit formulas transfer**, with $R\to Z$ (complex) everywhere: series $Z_{eq}=\sum Z_k$; parallel $1/Z_{eq}=\sum 1/Z_k$; voltage divider $\hat V_2 = \hat V_s\, Z_2/(Z_1+Z_2)$; current divider, node analysis ([[6.002-circuits-and-electronics/notes/02-resistive-networks-node-mesh-analysis]]), Thevenin/Norton ([[6.002-circuits-and-electronics/notes/03-dependent-sources-superposition-thevenin-norton]]) — all unchanged in form.
- **Workflow**: (1) convert every source to a phasor, (2) convert every element to its impedance at the given $\omega$, (3) solve the resulting complex-number circuit exactly like a resistive circuit, (4) convert the answer phasor back to $x(t) = X\cos(\omega t+\phi)$ with $X=|\hat X|,\ \phi=\angle\hat X$.
- **Validity condition**: only applies after transients have decayed — i.e., true sinusoidal *steady state*, single frequency $\omega$. For multiple frequencies, solve one phasor problem per frequency and superpose the resulting real time-domain signals (not the phasors directly, since phasors at different $\omega$ aren't comparable/addable).

## Used later in
(to be filled in once a later note cites this one)
