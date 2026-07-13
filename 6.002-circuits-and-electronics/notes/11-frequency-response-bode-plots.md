---
title: "Frequency Response and Bode Plots (Introduction)"
course: "6.002"
topic_number: 11
prerequisites: ["09-sinusoidal-steady-state-impedance", "10-op-amps-ideal-model-feedback-circuits"]
status: in progress
---

# Frequency Response and Bode Plots (Introduction)

## Why this matters

Every real circuit treats different sinusoidal frequencies differently — a microphone preamp needs to pass 20 Hz–20 kHz but reject 60 Hz hum and MHz radio noise; an op-amp that looks "ideal" at DC turns into a sluggish, phase-shifting mess above some cutoff frequency. To predict this behavior you need a compact way to describe how a circuit's output-to-input ratio (its **transfer function**) changes with frequency, across a range that might span six or more decades (1 Hz to 1 MHz, say). Plotting that ratio on ordinary linear axes is useless — the interesting features get squeezed into a corner of the page. The **Bode plot** — log frequency axis, log-magnitude (dB) axis, phase on a separate log-frequency axis — is the standard tool that makes this tractable, and it lets you sketch the whole picture using straight-line approximations instead of grinding through algebra at every frequency. This note builds the first, and most important, piece of that toolkit: the single-pole (first-order) low-pass and high-pass responses, and how to read/draw their Bode plots.

If you skip this, later 6.002 topics (diode/rectifier ripple filtering) and the entire feedback-stability story in 6.302 become opaque, because "gain rolls off at 20 dB/decade" and "the pole is at 1/RC" are the basic vocabulary of that discussion.

## Builds on

- [[6.002-circuits-and-electronics/notes/09-sinusoidal-steady-state-impedance]] — impedance $Z(j\omega)$ of resistors, capacitors, inductors, and the rule that in steady state with a sinusoidal source you can replace time-domain KVL/KCL with phasor (complex-number) KVL/KCL using these impedances. Recapped self-contained below.
- [[6.002-circuits-and-electronics/notes/10-op-amps-ideal-model-feedback-circuits]] — ideal op-amp model (infinite gain, infinite input impedance, zero output impedance, and the two "golden rules" for a negative-feedback op-amp circuit). Recapped self-contained below.

### Recap: impedance and phasors (from Topic 09)

When every independent source in a linear circuit is a sinusoid at a single angular frequency $\omega$ (units: rad/s, related to ordinary frequency by $\omega = 2\pi f$), and we wait long enough for any startup transient to die out, every voltage and current in the circuit is also a sinusoid at that same frequency, differing only in amplitude and phase. It is convenient to represent a real sinusoid $v(t) = V_m \cos(\omega t + \phi)$ by a complex number (**phasor**) $\hat V = V_m e^{j\phi}$, with the understanding that the actual time-domain signal is recovered as $v(t) = \mathrm{Re}\{\hat V e^{j\omega t}\}$. Here $j = \sqrt{-1}$ (electrical engineering uses $j$, not $i$, because $i$ already means current).

Under this representation, each circuit element's voltage-current relationship becomes a simple ratio, called its **impedance** $Z$ (units: ohms, $\Omega$), defined by $\hat V = Z\,\hat I$:

- Resistor: $Z_R = R$ (real, frequency-independent).
- Capacitor: $Z_C = \dfrac{1}{j\omega C}$ (purely imaginary, magnitude shrinks as $\omega$ grows — a capacitor looks like a short circuit at high frequency and an open circuit at DC).
- Inductor: $Z_L = j\omega L$ (purely imaginary, magnitude grows with $\omega$ — opposite behavior from a capacitor).

Because these are complex numbers obeying the same algebra as resistances, Kirchhoff's voltage law (sum of voltage drops around a loop is zero) and Kirchhoff's current law (sum of currents into a node is zero) apply directly to phasors and impedances, so series/parallel combination rules, voltage dividers, and node equations all carry over unchanged from resistive DC analysis — you just use complex arithmetic instead of real arithmetic.

### Recap: ideal op-amp model (from Topic 10)

An **operational amplifier (op-amp)** is a five-terminal active device (two inputs — labeled $+$ and $-$ — one output, and two power-supply pins we usually don't draw) whose output tries to drive the voltage difference between its two inputs to zero, by producing $v_{out} = A(v_+ - v_-)$ for a very large open-loop gain $A$. The **ideal op-amp model** takes three limits that are excellent approximations for real op-amps over the frequency range of interest in an intro course:

1. $A \to \infty$ (infinite open-loop gain).
2. Input impedance $\to \infty$ (no current flows into either input terminal).
3. Output impedance $\to 0$ (the output behaves as an ideal voltage source, unaffected by loading).

When an ideal op-amp is wired with **negative feedback** (some path from output back to the $-$ input) in a stable configuration, these limits produce two "golden rules" used to analyze the circuit:

- **Golden Rule 1:** No current flows into either input terminal (from limit 2).
- **Golden Rule 2:** The two input terminals sit at the same voltage, $v_+ = v_-$ (this follows because if $A$ is infinite but $v_{out}$ stays finite, then $v_+ - v_- = v_{out}/A \to 0$).

The standard **inverting amplifier**: input resistor $R_1$ from the source to the $-$ input, feedback resistor $R_2$ from the $-$ input to the output, $+$ input grounded. Golden Rule 2 forces $v_- = v_+ = 0$. Golden Rule 1 means the same current $i$ flows through $R_1$ and $R_2$ (no current is siphoned off into the input pin), so $i = v_{in}/R_1 = -v_{out}/R_2$, giving the closed-loop gain $v_{out}/v_{in} = -R_2/R_1$. This note will re-derive a *frequency-dependent* version of this same circuit topology (Example 2) once $R_2$ is replaced by a capacitor.

## Core definitions

- **Transfer function $H(j\omega)$** — for a linear circuit driven by a sinusoidal input phasor $\hat V_{in}$ at angular frequency $\omega$, producing steady-state output phasor $\hat V_{out}$, the transfer function is the complex ratio $H(j\omega) = \hat V_{out}/\hat V_{in}$. It depends on $\omega$ but not on the amplitude or phase of the specific input (linear circuit ⇒ ratio is independent of input size).
- **Magnitude response $|H(j\omega)|$** — the factor by which the input sinusoid's amplitude is scaled at frequency $\omega$. If $|H(j\omega)| = 0.5$, an input of amplitude 1 V produces an output of amplitude 0.5 V at that frequency.
- **Phase response $\angle H(j\omega)$** — the phase shift (in radians or degrees) added to the input sinusoid at frequency $\omega$.
- **Decibel (dB)** — a logarithmic unit for the magnitude response, defined as $|H|_{dB} = 20\log_{10}|H(j\omega)|$. (The factor 20, not 10, is because dB for power ratios uses $10\log_{10}$, and power $\propto$ voltage$^2$, so converting a voltage ratio to an equivalent "power-like" log scale doubles the multiplier: $10\log_{10}(|H|^2) = 20\log_{10}|H|$.)
- **Decade** — a frequency interval where the frequency multiplies by 10 (e.g., 10 Hz to 100 Hz is one decade).
- **Octave** — a frequency interval where the frequency doubles (e.g., 100 Hz to 200 Hz).
- **Pole (of a first-order $RC$ or $RL$ circuit)** — the angular frequency $\omega_0$ at which the circuit's magnitude response transitions from roughly flat to roughly falling; for a single-time-constant circuit, $\omega_0 = 1/\tau$ where $\tau$ is the circuit's time constant (units: rad/s). Also called the **corner frequency**, **break frequency**, or **cutoff frequency** ($f_0 = \omega_0/2\pi$ in Hz).
- **Bode plot** — a pair of plots of a transfer function versus frequency, both using a logarithmic frequency axis: (1) magnitude in dB vs. $\log \omega$, (2) phase (in degrees) vs. $\log \omega$.
- **Bode magnitude asymptote (straight-line approximation)** — the piecewise-linear approximation to a Bode magnitude plot, built from straight segments that are exact at $\omega \to 0$ and $\omega \to \infty$ and meet at the corner frequency; the true curve deviates from this approximation by at most about 3 dB right at the corner, and the error is negligible more than a decade away from the corner.
- **Low-pass filter** — a circuit whose $|H(j\omega)|$ is largest at low $\omega$ and falls off at high $\omega$.
- **High-pass filter** — a circuit whose $|H(j\omega)|$ is largest at high $\omega$ and falls off at low $\omega$.
- **Roll-off rate** — the slope of the Bode magnitude asymptote past the corner frequency, conventionally stated in dB/decade (a single-pole circuit rolls off at 20 dB/decade, equivalently 6 dB/octave).

## Intuition

A capacitor's impedance $1/(j\omega C)$ is huge at low frequency and tiny at high frequency; an inductor's impedance $j\omega L$ is the mirror image. So any circuit built from resistors plus one capacitor (or one inductor) is really just a frequency-dependent voltage (or current) divider: at very low frequency, the capacitor acts like an open circuit (infinite impedance) and the divider ratio settles to one value; at very high frequency the capacitor acts like a short circuit (zero impedance) and the divider ratio settles to a different value (often zero). In between, the impedance is comparable to the resistors, and the output amplitude and phase transition smoothly from one regime to the other. The "corner frequency" is simply the frequency at which the capacitor's impedance magnitude equals the relevant resistance — that's the crossover point between "capacitor looks like a wire" and "capacitor looks like a break in the wire."

The reason we plot on log-log (frequency) / log (magnitude, i.e., dB) axes rather than linear axes is that the response of these simple circuits, when plotted this way, becomes almost exactly two straight lines meeting at a corner — a flat line and a sloped line — which is trivial to sketch from just two numbers (the DC/HF gain and the corner frequency) without evaluating the transfer function anywhere else. That straight-line sketch is called the **Bode plot asymptote**, and it's accurate to within about 3 dB everywhere, which is more than good enough for design intuition.

## Derivation / formalism

### Setting up the RC low-pass transfer function

Take the standard series $RC$ low-pass filter: a sinusoidal source $v_{in}(t)$ drives a resistor $R$ in series with a capacitor $C$, and the output $v_{out}(t)$ is taken across the capacitor (to ground). We want $H(j\omega) = \hat V_{out}/\hat V_{in}$.

Using phasors and impedances from the Topic 09 recap, this is a two-impedance voltage divider: $Z_R = R$ in series with $Z_C = 1/(j\omega C)$, output taken across $Z_C$. The voltage-divider formula (same derivation as the resistive case, just with complex impedances) gives

$$\hat V_{out} = \hat V_{in}\cdot \frac{Z_C}{Z_R + Z_C}$$

Substituting the impedances:

$$H(j\omega) = \frac{\hat V_{out}}{\hat V_{in}} = \frac{\dfrac{1}{j\omega C}}{R + \dfrac{1}{j\omega C}}$$

Multiply numerator and denominator by $j\omega C$ (legal since $j\omega C \neq 0$ for $\omega \neq 0$):

$$H(j\omega) = \frac{1}{j\omega RC + 1} = \frac{1}{1 + j\omega RC}$$

Define $\tau \equiv RC$ (units: seconds — check: $\Omega \cdot F = \Omega \cdot \mathrm{s}/\Omega = \mathrm{s}$, using $F = \mathrm{s}/\Omega$ from $Z_C = 1/(j\omega C)$ having units of ohms) and $\omega_0 \equiv 1/\tau = 1/(RC)$ (units: rad/s). Then

$$H(j\omega) = \frac{1}{1 + j\omega/\omega_0}$$

This is the canonical single-pole low-pass transfer function.

### Magnitude and phase

Write $j\omega/\omega_0 = x$ where $x$ is a real number for real $\omega$. The magnitude of a quotient of complex numbers is the quotient of magnitudes: $|H| = |1|/|1+jx|$. The magnitude of a complex number $a+jb$ is $\sqrt{a^2+b^2}$, so $|1+jx| = \sqrt{1^2+x^2} = \sqrt{1+x^2}$. Thus

$$|H(j\omega)| = \frac{1}{\sqrt{1+(\omega/\omega_0)^2}}$$

The phase of a quotient is the difference of phases: $\angle H = \angle 1 - \angle(1+jx) = 0 - \arctan(x/1) = -\arctan(\omega/\omega_0)$, using the standard rule that the phase of $a+jb$ (with $a>0$) is $\arctan(b/a)$.

$$\angle H(j\omega) = -\arctan(\omega/\omega_0)$$

### Behavior at the extremes (why the Bode plot is two straight lines)

**Low frequency, $\omega \ll \omega_0$:** $(\omega/\omega_0)^2 \ll 1$, so $|H| \approx 1/\sqrt{1+0} = 1$. In dB: $20\log_{10}(1) = 0$ dB. This is a flat horizontal line at 0 dB.

**High frequency, $\omega \gg \omega_0$:** $(\omega/\omega_0)^2 \gg 1$, so the "1+" is negligible and $|H| \approx 1/\sqrt{(\omega/\omega_0)^2} = \omega_0/\omega$. In dB:

$$|H|_{dB} \approx 20\log_{10}(\omega_0/\omega) = 20\log_{10}\omega_0 - 20\log_{10}\omega$$

This is linear in $\log_{10}\omega$ with slope $-20$ dB per unit increase of $\log_{10}\omega$ — and one unit increase of $\log_{10}\omega$ is exactly one decade. So the high-frequency asymptote is a straight line of slope $-20$ dB/decade on the dB-vs-$\log\omega$ axes, and it crosses 0 dB exactly at $\omega=\omega_0$ (since at $\omega=\omega_0$, $|H|_{dB}\approx 20\log_{10}(1) = 0$ dB by this same approximate formula).

**At the corner, $\omega = \omega_0$ (exact, not approximate):** $|H| = 1/\sqrt{1+1} = 1/\sqrt2 \approx 0.707$. In dB: $20\log_{10}(1/\sqrt2) = -10\log_{10}2 \approx -3.01$ dB. This confirms the two asymptotes (0 dB flat line, and the $-20$dB/decade line through $\omega_0$) both predict 0 dB at $\omega_0$, while the true curve is 3 dB below that — the "3 dB down point" is the standard way engineers describe the corner frequency.

**Phase asymptotes:** $\angle H = -\arctan(\omega/\omega_0)$ ranges from $0°$ (at $\omega \to 0$) through $-45°$ exactly at $\omega=\omega_0$ (since $\arctan(1)=45°$) to $-90°$ (at $\omega \to \infty$). The common straight-line phase approximation is: flat at $0°$ for $\omega < 0.1\,\omega_0$, a straight line from $0°$ to $-90°$ across the two decades from $0.1\,\omega_0$ to $10\,\omega_0$ (passing through $-45°$ at $\omega_0$), flat at $-90°$ for $\omega > 10\,\omega_0$.

### The high-pass case

If instead the output is taken across the resistor $R$ (same series $RC$ circuit), then by the same voltage-divider logic,

$$H_{HP}(j\omega) = \frac{Z_R}{Z_R+Z_C} = \frac{R}{R+\frac{1}{j\omega C}} = \frac{j\omega RC}{1+j\omega RC} = \frac{j\omega/\omega_0}{1+j\omega/\omega_0}$$

with the same $\omega_0 = 1/(RC)$. At low frequency ($\omega \ll \omega_0$) the numerator dominates the smallness: $|H_{HP}| \approx \omega/\omega_0$, which is $0$ dB at $\omega=\omega_0$ and falls at $+20$ dB/decade going down in frequency (equivalently, rises at 20 dB/decade as $\omega$ increases toward $\omega_0$). At high frequency ($\omega \gg \omega_0$), $|H_{HP}| \to 1$ (0 dB, flat). So the high-pass Bode magnitude plot is the mirror image of the low-pass one about the corner frequency: rising at 20 dB/decade below $\omega_0$, flat at 0 dB above it. This is the general pattern: swapping which element you take the output across turns a low-pass into a high-pass with the *same* corner frequency $\omega_0 = 1/(RC)$, because the corner frequency is a property of the $R$-$C$ combination, not of where you happen to measure the output.

### General rule for sketching any Bode magnitude plot with multiple poles/zeros

A transfer function built from multiple $RC$-type stages (or, more generally, expressed as a product/ratio of factors like $(1+j\omega/\omega_0)$) has a magnitude in dB equal to the *sum* of the dB contributions of each factor, because $20\log_{10}|H_1 H_2| = 20\log_{10}|H_1| + 20\log_{10}|H_2|$ (log of a product is the sum of logs) and $20\log_{10}|1/H_1| = -20\log_{10}|H_1|$ (a factor in the denominator, i.e. a "pole," contributes a *negative* slope break; a factor in the numerator, a "zero," contributes a *positive* slope break). So the general sketching procedure is: (1) identify every pole and zero frequency; (2) starting from the low-frequency asymptote value, add a $-20$ dB/decade slope-break at each pole frequency and a $+20$ dB/decade slope-break at each zero frequency, in increasing order of frequency; (3) the phase plot is built the same way by summing each factor's phase contribution. This note only develops the single-pole building block in detail; multi-pole sketching is the natural extension used in later frequency-response work.

## Worked examples

### Example 1: Corner frequency and gain at a specified frequency for an RC low-pass filter

A series $RC$ low-pass filter (output across the capacitor) has $R = 1\ \mathrm{k\Omega} = 1000\ \Omega$ and $C = 100\ \mathrm{nF} = 100\times10^{-9}\ \mathrm{F}$.

**(a) Find the corner frequency in rad/s and Hz.**

$$\tau = RC = (1000\ \Omega)(100\times10^{-9}\ \mathrm{F}) = 1\times10^{-4}\ \mathrm{s}$$

$$\omega_0 = \frac{1}{\tau} = \frac{1}{1\times10^{-4}\ \mathrm{s}} = 1\times10^{4}\ \mathrm{rad/s} = 10{,}000\ \mathrm{rad/s}$$

$$f_0 = \frac{\omega_0}{2\pi} = \frac{10{,}000}{2\pi}\ \mathrm{Hz} \approx 1591.5\ \mathrm{Hz}$$

**(b) Find $|H|$ in dB and the phase at $f = 15{,}915\ \mathrm{Hz}$ (i.e., exactly one decade above $f_0$, since $10\times1591.5 = 15{,}915$).**

At one decade above the corner, $\omega/\omega_0 = 10$. Exact magnitude:

$$|H| = \frac{1}{\sqrt{1+10^2}} = \frac{1}{\sqrt{101}} \approx \frac{1}{10.05} \approx 0.0995$$

$$|H|_{dB} = 20\log_{10}(0.0995) \approx 20\times(-1.002) \approx -20.04\ \mathrm{dB}$$

This matches the straight-line asymptote prediction almost exactly: the asymptote says $-20$ dB one decade past the corner (since it's flat at 0 dB up to $\omega_0$ and falls at $-20$dB/decade after), and the true value is $-20.04$ dB — an error of only 0.04 dB, confirming the asymptote is essentially exact more than a decade from the corner.

Phase: $\angle H = -\arctan(10) \approx -84.29°$ (close to the $-90°$ high-frequency asymptote, as expected one decade past the corner).

**(c) Find $|H|$ in dB at $f = f_0$ exactly.**

$$|H| = \frac{1}{\sqrt{1+1^2}} = \frac{1}{\sqrt2} \approx 0.7071 \implies |H|_{dB} = 20\log_{10}(0.7071) \approx -3.01\ \mathrm{dB}$$

confirming the "3 dB down" rule at the corner.

### Example 2: Frequency-dependent gain of an op-amp inverting integrator-like circuit (RC feedback), and the frequency at which its gain magnitude equals a target value

Take the inverting-amplifier topology recapped above (Builds on section), but replace the feedback resistor $R_2$ with a resistor $R_2 = 10\ \mathrm{k\Omega}$ in parallel with a capacitor $C_f = 1\ \mathrm{nF}$, feeding back from the output to the $-$ input; the input resistor stays $R_1 = 1\ \mathrm{k\Omega}$, and the $+$ input is grounded. This is a real, commonly used circuit: an inverting amplifier whose gain rolls off at high frequency (a "lossy integrator").

**Step 1 — impedance of the feedback network.** $R_2$ and $C_f$ are in parallel, so by the parallel-impedance rule (same form as parallel resistors, using impedances):

$$Z_f = \frac{Z_{R_2}\,Z_{C_f}}{Z_{R_2}+Z_{C_f}} = \frac{R_2 \cdot \frac{1}{j\omega C_f}}{R_2+\frac{1}{j\omega C_f}}$$

Multiply numerator and denominator by $j\omega C_f$:

$$Z_f = \frac{R_2}{1+j\omega R_2 C_f}$$

**Step 2 — apply the golden rules exactly as in the DC recap, but with impedances instead of resistances.** Golden Rule 2 still forces $v_-=v_+=0$ (grounded $+$ input, ideal op-amp). Golden Rule 1 still forces the same current phasor $\hat I$ through $R_1$ and through $Z_f$ (no current into the input pin), so

$$\hat I = \frac{\hat V_{in}-0}{R_1} = \frac{0-\hat V_{out}}{Z_f}$$

Solve for the closed-loop transfer function:

$$H(j\omega) = \frac{\hat V_{out}}{\hat V_{in}} = -\frac{Z_f}{R_1} = -\frac{R_2}{R_1\left(1+j\omega R_2 C_f\right)}$$

**Step 3 — identify the DC gain and corner frequency.** At $\omega=0$: $H(0) = -R_2/R_1 = -10{,}000/1000 = -10$ (i.e., $-10$, or $20.0$ dB in magnitude, with a $180°$ phase inversion from the minus sign — this matches the plain resistive inverting amplifier result from the recap, as it must, since at DC the capacitor is an open circuit and contributes nothing). The corner frequency comes from the same $(1+j\omega \cdot \text{time constant})$ form as before, with time constant $\tau = R_2 C_f$:

$$\tau = R_2 C_f = (10{,}000\ \Omega)(1\times10^{-9}\ \mathrm{F}) = 1\times10^{-5}\ \mathrm{s}$$

$$\omega_0 = \frac{1}{\tau} = 1\times10^{5}\ \mathrm{rad/s} = 100{,}000\ \mathrm{rad/s}, \qquad f_0 = \frac{\omega_0}{2\pi} \approx 15{,}915\ \mathrm{Hz}$$

So $|H(j\omega)| = \dfrac{R_2/R_1}{\sqrt{1+(\omega/\omega_0)^2}} = \dfrac{10}{\sqrt{1+(\omega/\omega_0)^2}}$: flat at a magnitude of 10 (20 dB) below $\approx 15.9\ \mathrm{kHz}$, then rolling off at $-20$ dB/decade above it — exactly the low-pass shape from Example 1, just scaled up by the DC gain of 10 and shifted to a different corner frequency, and inverted in sign.

**Step 4 — find the frequency at which $|H| = 1$ (unity gain, 0 dB) using the asymptote.** Using the high-frequency asymptote $|H| \approx (R_2/R_1)\cdot(\omega_0/\omega)$, set this equal to 1:

$$1 = 10\cdot\frac{\omega_0}{\omega} \implies \omega = 10\,\omega_0 = 10\times10^{5} = 1\times10^{6}\ \mathrm{rad/s}$$

$$f = \frac{10^6}{2\pi}\ \mathrm{Hz} \approx 159{,}155\ \mathrm{Hz} \approx 159.2\ \mathrm{kHz}$$

Check with the exact formula at this frequency ($\omega/\omega_0 = 10$): $|H| = 10/\sqrt{1+100} = 10/\sqrt{101} \approx 10/10.05 \approx 0.995$ — within half a percent of 1, confirming the asymptote-based estimate of the unity-gain frequency is accurate.

## Common pitfalls

- **Forgetting the factor of 20 (using $10\log_{10}$ instead of $20\log_{10}$) for voltage/current ratios.** The $10\log_{10}$ form is for *power* ratios; since power scales as amplitude squared, converting an amplitude ratio requires the extra factor of 2, giving $20\log_{10}$.
- **Confusing $\omega$ (rad/s) with $f$ (Hz).** All the transfer-function algebra above is in terms of $\omega$; always convert with $\omega = 2\pi f$ before/after comparing to a frequency spec given in Hz. A very common numeric slip is plugging a Hz value directly into a formula that expects rad/s (or vice versa), which is off by a factor of $2\pi \approx 6.28$.
- **Assuming the asymptote is exact at the corner frequency.** It isn't — the true curve is 3 dB below the two asymptotes' intersection at $\omega=\omega_0$ (for a pole) and phase is exactly $-45°$, not $0°$ or $-90°$, right at the corner. The asymptote is only exact in the limits $\omega\to0$ and $\omega\to\infty$.
- **Sign/direction errors on high-pass vs. low-pass slopes.** A pole always contributes $-20$ dB/decade starting at its corner frequency and moving upward in frequency; it is easy to accidentally apply the roll-off in the wrong direction (e.g., treating the high-pass rise as happening above $\omega_0$ instead of below it).
- **Forgetting that a negative (inverting) gain still has the same $|H|$ and dB value as a positive gain of the same magnitude.** The minus sign in Example 2's $H(j\omega) = -R_2/(R_1(1+j\omega R_2C_f))$ shows up as a $180°$ phase shift, not as a change in the magnitude or dB plot.
- **Applying the ideal-op-amp golden rules without checking the frequency is within the op-amp's own bandwidth.** This note assumes an ideal (infinite-bandwidth) op-amp so that the *external* $R$-$C$ network alone sets the frequency response; a real op-amp has its own internal corner frequency (its "gain-bandwidth product") which, at high enough frequency, will dominate and invalidate the ideal model — that complication is outside this intro note's scope.
- **Mixing up decade and octave.** A decade is $\times10$ in frequency; an octave is $\times2$. The 20 dB/decade single-pole roll-off is equivalently 6.02 dB/octave ($20\log_{10}2 \approx 6.02$), not 20 dB/octave — conflating the two overstates the roll-off by a factor of $\log_{10}10/\log_{10}2 \approx 3.32$.

## Self-check

### Questions

1. What is the corner (angular) frequency $\omega_0$ of a series $RC$ low-pass filter with $R = 2\ \mathrm{k\Omega}$ and $C = 50\ \mathrm{nF}$?
2. By how many dB does the magnitude of a single-pole low-pass response fall between the corner frequency and one decade above it (using the straight-line asymptote)?
3. For the $RC$ low-pass filter of Question 1, what is $|H(j\omega)|$ in dB, exactly (not the asymptote), at $\omega = \omega_0$?
4. A single-pole high-pass filter has corner frequency $f_0 = 500\ \mathrm{Hz}$. Sketch (in words) the magnitude Bode plot: what is the slope below $f_0$, what is the slope above $f_0$, and what is $|H|_{dB}$ far above $f_0$?
5. An inverting op-amp circuit has $R_1 = 2\ \mathrm{k\Omega}$, and a feedback network consisting of $R_2 = 20\ \mathrm{k\Omega}$ in parallel with $C_f = 2\ \mathrm{nF}$. Find the DC gain (magnitude and sign) and the corner frequency $f_0$ in Hz.
6. For the circuit in Question 5, estimate (using the high-frequency asymptote) the frequency at which $|H|$ drops to 1 (unity gain).
7. Explain, using impedances, why a series $RC$ circuit with output taken across the resistor is high-pass, while output taken across the capacitor is low-pass, even though both have exactly the same corner frequency $\omega_0 = 1/(RC)$.
8. A signal at $f = 50\ \mathrm{Hz}$ passes through a single-pole low-pass filter with $f_0 = 5\ \mathrm{kHz}$. Using the low-frequency asymptote, roughly how many dB of attenuation (if any) should you expect, and why?

### Answers

1. $\tau = RC = (2000\ \Omega)(50\times10^{-9}\ \mathrm{F}) = 1\times10^{-4}\ \mathrm{s}$, so $\omega_0 = 1/\tau = 1\times10^{4}\ \mathrm{rad/s} = 10{,}000\ \mathrm{rad/s}$.
2. $-20$ dB (the defining slope of a single real pole, by the derivation above: flat at 0 dB up to $\omega_0$, then falling at $-20$ dB/decade).
3. $|H(\omega_0)| = 1/\sqrt{1+1} = 1/\sqrt2$, so $|H|_{dB} = 20\log_{10}(1/\sqrt2) \approx -3.01\ \mathrm{dB}$ (independent of the specific $R,C$ values — this "3 dB down" result is universal for any single real pole at its own corner frequency).
4. Below $f_0=500\ \mathrm{Hz}$: rising at $+20$ dB/decade as frequency increases toward $f_0$ (equivalently, falling at $-20$ dB/decade as frequency decreases away from $f_0$). Above $f_0$: flat. Far above $f_0$, $|H|_{dB} \to 0$ dB (unity gain, since a high-pass passes high frequencies unattenuated in this simple single-stage form).
5. DC gain $= -R_2/R_1 = -20{,}000/2000 = -10$ (magnitude 10, i.e. 20 dB, inverting/negative sign). Time constant $\tau = R_2C_f = (20{,}000)(2\times10^{-9}) = 4\times10^{-5}\ \mathrm{s}$, so $\omega_0 = 1/\tau = 25{,}000\ \mathrm{rad/s}$, and $f_0 = \omega_0/(2\pi) \approx 3978.9\ \mathrm{Hz} \approx 3.98\ \mathrm{kHz}$.
6. Set $10\cdot(\omega_0/\omega) = 1 \Rightarrow \omega = 10\,\omega_0 = 250{,}000\ \mathrm{rad/s}$, so $f = \omega/(2\pi) \approx 39{,}789\ \mathrm{Hz} \approx 39.8\ \mathrm{kHz}$ (one decade above $f_0$, since the DC gain is 10 = 20 dB and it takes exactly one decade of $-20$dB/decade roll-off to bring 20 dB down to 0 dB).
7. At $\omega \to 0$, $Z_C = 1/(j\omega C) \to \infty$ (capacitor is an open circuit) and $Z_R=R$ stays finite, so nearly all the source voltage drops across the capacitor (output there $\approx v_{in}$, i.e. low-pass) while almost none drops across $R$ (output there $\approx 0$, i.e. high-pass, blocking low frequencies). At $\omega\to\infty$, $Z_C\to0$ (short circuit), so the capacitor's voltage $\to 0$ (low-pass output goes to zero at high frequency, as expected) while the resistor's voltage $\to v_{in}$ (high-pass output passes high frequency essentially unattenuated). Both share the same $\omega_0=1/(RC)$ because that's the frequency at which $|Z_C|=R$, the crossover point of the same underlying divider regardless of which element's voltage you read out.
8. $f=50\ \mathrm{Hz}$ is two decades below $f_0 = 5000\ \mathrm{Hz}$ ($5000/50=100=10^2$), and $50\ \mathrm{Hz} \ll f_0$ means we're deep in the flat low-frequency asymptote region ($|H|\approx 1$, i.e. 0 dB). So essentially no attenuation (0 dB) is expected — the filter has not yet started rolling off that far below its corner.

## Summary / cheat sheet

- Transfer function: $H(j\omega) = \hat V_{out}/\hat V_{in}$, a complex function of $\omega$ for a linear circuit in sinusoidal steady state.
- $\mathrm{dB} = 20\log_{10}|H(j\omega)|$ (voltage/current ratio — factor of 20, not 10).
- Single-pole low-pass: $H(j\omega) = \dfrac{1}{1+j\omega/\omega_0}$, $\omega_0 = 1/\tau = 1/(RC)$ (series $RC$, output across $C$). Flat at 0 dB below $\omega_0$, falls at $-20$ dB/decade above it, exactly $-3$ dB and $-45°$ at $\omega_0$, $\to -90°$ phase at high frequency.
- Single-pole high-pass: $H(j\omega) = \dfrac{j\omega/\omega_0}{1+j\omega/\omega_0}$, same $\omega_0$ (output across $R$ in the same series $RC$). Rises at $+20$ dB/decade below $\omega_0$, flat at 0 dB above it, $+45°$ phase at $\omega_0$ (measuring toward $0°$ from a $90°$ low-frequency phase lead — mirror image of the low-pass phase curve).
- Corner/break/cutoff frequency $\omega_0=1/\tau$; "3 dB down point" is the standard name for this frequency on a magnitude Bode plot.
- 20 dB/decade $=$ 6.02 dB/octave for a single real pole or zero.
- Multi-pole/zero transfer functions: dB contributions of each factor simply add (log of product = sum of logs); each pole breaks the slope down by 20 dB/decade at its corner, each zero breaks it up by 20 dB/decade at its corner, in increasing-frequency order.
- Op-amp with $R$-$C$ feedback network: same golden rules as the DC inverting amp, but replace resistances with impedances; DC gain set by the resistive part, roll-off/corner set by the added capacitor's time constant with the relevant resistor.
- Always convert between $f$ (Hz) and $\omega$ (rad/s) via $\omega = 2\pi f$ before combining with a spec given in the other unit.

## Used later in
(none yet)
