# 6.002 Glossary

Running list of terms/symbols introduced in 6.002 notes, in order of first appearance.

## From [[6.002-circuits-and-electronics/notes/01-lumped-circuit-abstraction-kvl-kcl]]

- **$q$ — charge** — fundamental, conserved property of matter (coulombs, C); source of electric fields.
- **$i$ — current** — rate of flow of charge, $i = dq/dt$ (amperes, A = C/s). Has a reference direction.
- **$V$ — electric potential** — potential energy per unit charge at a point (volts, V = J/C); defined up to an additive constant.
- **$v$ — voltage / potential difference** — $v_{AB} = V_A - V_B$ between two points (volts).
- **Lumped element** — component fully characterized at each instant by one terminal current and one terminal voltage.
- **Lumped circuit abstraction** — modeling assumption that a circuit is a finite set of lumped elements joined by ideal wires; valid when (1) no unmodeled charge accumulation, (2) circuit size $d \ll c\tau$ for signal timescale $\tau$.
- **Node** — point(s) of a circuit connected by ideal wire, all at one potential.
- **Branch** — one lumped element plus its two terminal nodes; has one current and one voltage.
- **Loop** — closed path through branches/nodes without repeating a node.
- **KVL — Kirchhoff's Voltage Law** — sum of branch voltages around any closed loop (consistent sign convention) is zero at every instant. Follows from potential being single-valued/path-independent.
- **KCL — Kirchhoff's Current Law** — sum of currents leaving (or entering) any node is zero at every instant. Follows from charge conservation.
- **Reference direction / sign convention** — arbitrarily chosen arrow (current) or $+/-$ pair (voltage) fixed before solving; sign of the solved value gives the true physical direction.
- **Element law / constitutive relation** — the $v$-$i$ relationship specific to one element type (e.g. Ohm's law $v = iR$); needed in addition to KVL/KCL to fully determine a circuit.

## From [[6.002-circuits-and-electronics/notes/02-resistive-networks-node-mesh-analysis]]

- **Reference node (ground)** — one node chosen, arbitrarily, to have voltage $0\text{ V}$; all other node voltages are measured relative to it.
- **Node voltage** $v_n$ — the voltage of node $n$ relative to the reference node; one unknown per non-reference node.
- **Mesh** — a loop (see Topic 01) that does not enclose any other loop (a "window pane" of the circuit as drawn); a clean concept only for planar circuits.
- **Mesh current** $i_m$ — a fictitious current assumed to circulate around mesh $m$ (by convention, clockwise); a branch shared by two meshes carries the signed sum of the mesh currents through it.
- **Node analysis (nodal analysis)** — systematic method taking node voltages as unknowns, writing one KCL equation per non-reference node, and solving the resulting linear system.
- **Mesh analysis (loop analysis)** — systematic method taking mesh currents as unknowns, writing one KVL equation per independent mesh, and solving the resulting linear system.
- **Independent source** — a voltage or current source whose value is fixed, independent of any other voltage/current in the circuit.
- **Supernode** — a region formed by merging two non-reference nodes joined by a floating voltage source, used to write one combined KCL equation when the source's own current isn't directly known.

## From [[6.002-circuits-and-electronics/notes/03-dependent-sources-superposition-thevenin-norton]]
- **Dependent source** — a source (voltage or current) whose value is not fixed but is instead proportional to some other voltage or current elsewhere in the circuit (its "controlling variable"); never zeroed during superposition.
- **VCVS (voltage-controlled voltage source)** — a dependent voltage source $v_o = \mu v_c$, output proportional to a controlling voltage $v_c$; $\mu$ dimensionless (V/V).
- **VCCS (voltage-controlled current source)** — a dependent current source $i_o = g_m v_c$, output proportional to a controlling voltage $v_c$; $g_m$ in siemens (A/V).
- **CCVS (current-controlled voltage source)** — a dependent voltage source $v_o = r_m i_c$, output proportional to a controlling current $i_c$; $r_m$ in ohms (V/A).
- **CCCS (current-controlled current source)** — a dependent current source $i_o = \beta i_c$, output proportional to a controlling current $i_c$; $\beta$ dimensionless (A/A).
- **$\mu$** — the dimensionless gain (V/V) of a VCVS.
- **$g_m$** — the transconductance gain (A/V, siemens) of a VCCS.
- **$r_m$** — the transresistance gain (V/A, ohms) of a CCVS.
- **$\beta$** — the dimensionless gain (A/A) of a CCCS.
- **Superposition** — for a linear circuit, the response to several independent sources acting together equals the sum of the responses to each independent source acting alone with all *other* independent sources zeroed (voltage sources shorted, current sources opened); dependent sources are never zeroed.
- **One-port** — a network exposing exactly two external terminals, characterized (if linear and resistive) entirely by its Thevenin or Norton equivalent as seen from those terminals.
- **Thevenin voltage $v_{th}$** — the open-circuit voltage $v_{oc}$ of a one-port: the terminal voltage with no load attached and all real (independent) sources active.
- **Thevenin resistance $R_{th}$** — the resistance seen at a one-port's terminals with all independent sources zeroed (dependent sources left active); equals $v_{oc}/i_{sc}$ when $v_{oc}\ne0$.
- **Norton current $i_{no}$** — the short-circuit current $i_{sc}$ of a one-port; $i_{no} = v_{th}/R_{th}$.
- **Norton resistance $R_{no}$** — equals $R_{th}$; the resistance in parallel with $i_{no}$ in the Norton equivalent.
- **$v_{oc}$ — open-circuit voltage** — the terminal voltage of a one-port measured with no load attached (terminals left open, current $=0$).
- **$i_{sc}$ — short-circuit current** — the current flowing out of a one-port's $+$ terminal, through a direct short, when the terminals are shorted together ($v=0$).

## From [[6.002-circuits-and-electronics/notes/04-nonlinear-elements-digital-abstraction-mosfet-model]]

- **Nonlinear element** — a lumped element whose $v$-$i$ relationship $i=f(v)$ is not a straight line through the origin, so superposition/Thevenin reduction cannot be applied to the element itself.
- **Load line** — the linear $i$-$v$ equation, $i = (v_{TH}-v)/R_{TH}$, obtained from KVL after reducing the rest of a circuit to its Thevenin equivalent around a nonlinear element's terminals.
- **$Q$-point / operating point** — the $(V_Q, I_Q)$ pair where the load line intersects the nonlinear device's $i=f(v)$ curve; the actual steady-state voltage/current at the device's terminals.
- **Digital abstraction** — the modeling convention of treating a continuous analog voltage as representing one of two discrete logic values (0 or 1), valid only within the static discipline.
- **Gain** — the ratio of a circuit's output signal change to its input signal change (output/input); gain greater than 1 in a gate's transition region lets it restore a degraded input signal rather than pass the degradation through.
- **$V_{OL}$ / $V_{OH}$** — the maximum voltage a driving gate is guaranteed to output for logic 0 ($V_{OL}$) / the minimum voltage it is guaranteed to output for logic 1 ($V_{OH}$).
- **$V_{IL}$ / $V_{IH}$** — the maximum input voltage a receiving gate is guaranteed to still interpret as logic 0 ($V_{IL}$) / the minimum input voltage it is guaranteed to still interpret as logic 1 ($V_{IH}$).
- **Static discipline** — the pair of design inequalities $V_{OL}\le V_{IL}$ and $V_{OH}\ge V_{IH}$ that guarantee a driving gate's output is never misread by a receiving gate, under worst-case DC conditions.
- **Noise margin ($NM_L$, $NM_H$)** — spare voltage tolerance built into the static discipline: $NM_L = V_{IL}-V_{OL} \ge 0$, $NM_H = V_{OH}-V_{IH} \ge 0$.
- **MOSFET** — a voltage-controlled transistor (here: NMOS) whose drain-source terminals behave as a nonlinear element controlled by the gate-source voltage $v_{GS}$; gate draws zero current in the models used here.
- **$V_T$ — threshold voltage** — the value of $v_{GS}$ below which an NMOS is in cutoff ($i_D=0$) and above which it can conduct.
- **Switch-level model** — the simplest MOSFET model: an open switch ($i_D=0$) for $v_{GS}<V_T$, a resistor of value $R_{ON}$ ($i_D=v_{DS}/R_{ON}$) for $v_{GS}\ge V_T$.
- **$R_{ON}$** — the effective on-state drain-source resistance in the switch-level MOSFET model (ohms).
- **Piecewise-linear (three-region) model** — the more detailed MOSFET model with cutoff, triode, and saturation regions, each with its own $i_D(v_{GS},v_{DS})$ formula, continuous across region boundaries.
- **Leakage current** — a small, unwanted current that flows through insulation or a reverse-biased junction that is ideally supposed to block current entirely; the real-device effect that breaks the "gate draws exactly zero current" idealization.
- **$K$** — the transconductance-like scale factor (units A/V$^2$) in the piecewise-linear MOSFET model's triode/saturation current formulas.

## From [[6.002-circuits-and-electronics/notes/05-amplifiers-large-small-signal-analysis]]

- **Operating point (bias point, quiescent point), $(V_{GS}, V_{DS}, I_D)$** — the DC (no-signal) voltages/current at which a transistor sits under constant sources only; capital letter/capital subscript denotes this throughout the note.
- **Large-signal analysis** — solving the full nonlinear circuit equation (e.g. $i_D=\frac{K}{2}(v_{GS}-V_T)^2$) directly, with no approximation; valid for signals of any size.
- **Small-signal (incremental) analysis** — linear approximation of circuit behavior for small deviations about the operating point; every quantity written as DC value + small deviation, with quadratic-and-higher terms in the deviation dropped. Lowercase letter/lowercase subscript (e.g. $v_{gs}$, $i_d$) denotes these deviations.
- **Total instantaneous quantity** — operating-point value + small-signal deviation, e.g. $v_{GS}^{total}(t)=V_{GS}+v_{gs}(t)$; mixed-case notation.
- **Transconductance $g_m$** — $g_m \equiv \partial i_D/\partial v_{GS}\big|_{\text{operating point}}$, units A/V (siemens); for this note's MOSFET saturation model, $g_m=K(V_{GS}-V_T)$.
- **Small-signal (linearized) MOSFET model** — gate–source open circuit ($i_g=0$) plus a drain–source dependent current source of value $g_m v_{gs}$; valid only for small deviations about a bias point in saturation.
- **Amplifier** — a circuit biasing a device (here, a MOSFET in saturation) so that a small-signal input voltage produces a scaled, larger-magnitude small-signal output voltage.
- **Voltage gain $A_v$** — $A_v \equiv v_{out}/v_{in}$ (dimensionless); for the common-source amplifier derived in this note, $A_v=-g_mR_D$ (inverting).

## From [[6.002-circuits-and-electronics/notes/06-capacitors-inductors-energy-storage]]

- **Passive sign convention** — current $i(t)$ defined as entering the terminal marked "+" for voltage $v(t)$; under this convention instantaneous power delivered to the element is $p(t) = v(t)\,i(t)$.
- **Capacitor** — two-terminal lumped element storing energy in an electric field; defining relation $i = C\,dv/dt$.
- **Capacitance $C$** — constant of proportionality between charge and voltage on a capacitor, $q = Cv$; units farads (F), $1\text{ F} = 1\text{ C/V}$.
- **Inductor** — two-terminal lumped element storing energy in a magnetic field; defining relation $v = L\,di/dt$.
- **Inductance $L$** — constant of proportionality between flux linkage and current on an inductor, $\lambda = Li$; units henries (H), $1\text{ H} = 1\text{ V·s/A}$.
- **Energy storage element** — a circuit element that can absorb, hold, and later return energy without (ideally) dissipating it as heat; contrasted with a resistor.
- **State variable** — quantity summarizing an energy-storage element's "memory": capacitor voltage $v_C$ (or charge $q$) and inductor current $i_L$ (or flux linkage $\lambda$); cannot change discontinuously, and together with topology determines all future behavior.

## From [[6.002-circuits-and-electronics/notes/09-sinusoidal-steady-state-impedance]]

- **Sinusoidal steady state (SSS)** — the condition of a linear circuit, driven by a sinusoidal source of a single fixed frequency $\omega$, after all transient ("natural response") terms have decayed to zero, leaving only a response that oscillates at the same frequency $\omega$ as the drive, forever.
- **Phasor** — a complex number representing a real sinusoid of known angular frequency $\omega$ by encoding only its amplitude and phase: for $x(t) = X\cos(\omega t + \phi)$, the phasor is $\hat{X} = Xe^{j\phi}$.
- **Complex exponential representation** — the identity $\cos(\omega t + \phi) = \mathrm{Re}\{e^{j(\omega t + \phi)}\}$, letting a real sinusoid $x(t)$ be replaced by the complex signal $\hat{X}e^{j\omega t}$, with $x(t)$ recovered at the end by taking the real part.
- **Impedance $Z$** — the complex-valued ratio of phasor voltage to phasor current across a two-terminal element in sinusoidal steady state, $Z = \hat{V}/\hat{I}$; units ohms ($\Omega$).
- **Admittance $Y$** — the reciprocal of impedance, $Y = 1/Z = \hat{I}/\hat{V}$; units siemens (S).
- **Reactance $X$** — the imaginary part of impedance, $Z = R_{\text{eff}} + jX$.
- **Angular frequency $\omega$** — rate of oscillation in radians per second, $\omega = 2\pi f = 2\pi/T$.

## From [[6.002-circuits-and-electronics/notes/10-op-amps-ideal-model-feedback-circuits]]

- **Op-amp (operational amplifier)** — a multi-terminal amplifier IC with two inputs (non-inverting $v_+$, inverting $v_-$) and one output $v_{out}$, modeled internally as a VCVS: $v_{out} = A(v_+-v_-)$.
- **Open-loop gain $A$** — the op-amp's intrinsic, very large ($10^4$–$10^6$ in real devices) voltage gain between its input difference and its output, absent any external feedback network.
- **Ideal op-amp model** — the idealization $A\to\infty$ and $i_+=i_-=0$ (zero input current), used together with an external resistor network to derive closed-loop behavior.
- **Feedback** — routing some fraction of an amplifier's output back to one of its inputs through an external network, so the output influences its own future value.
- **Negative feedback** — feedback routed to the inverting input $v_-$; self-correcting, yields a stable, finite output and (for an ideal op-amp) the virtual-short condition $v_+=v_-$. Contrasted with positive feedback (routed to $v_+$), which is unstable and drives $v_{out}$ to a supply rail.
- **Virtual short** — the condition $v_+ = v_-$ that holds under ideal-op-amp negative feedback with finite, stable $v_{out}$; an equality of voltages, not an actual wire (no current flows between the two terminals).
- **Closed-loop gain** — the overall $v_{out}/v_{in}$ ratio of an op-amp plus its feedback resistor network, determined by the resistors alone once $A\to\infty$ (independent of the op-amp's actual open-loop gain).
- **Inverting amplifier** — op-amp circuit with $v_+$ grounded and the input applied to $v_-$ through $R_1$, feedback resistor $R_2$ from $v_-$ to $v_{out}$; closed-loop gain $-R_2/R_1$.
- **Non-inverting amplifier** — op-amp circuit with the input applied directly to $v_+$ and $v_-$ set by a feedback divider ($R_1$ to ground, $R_2$ to $v_{out}$); closed-loop gain $1+R_2/R_1$.
- **Summing amplifier** — inverting-amplifier variant with multiple input resistors $R_1,\dots,R_n$ all feeding the same $v_-$ node and one feedback resistor $R_f$; output $v_{out} = -R_f\sum_k v_k/R_k$, a weighted, inverted sum.

## From [[6.002-circuits-and-electronics/notes/11-frequency-response-bode-plots]]

- **Transfer function $H(j\omega)$** — for a linear circuit driven by a sinusoidal input phasor $\hat V_{in}$ at angular frequency $\omega$, producing steady-state output phasor $\hat V_{out}$, the complex ratio $H(j\omega) = \hat V_{out}/\hat V_{in}$.
- **Magnitude response $|H(j\omega)|$** — the factor by which the input sinusoid's amplitude is scaled at frequency $\omega$.
- **Phase response $\angle H(j\omega)$** — the phase shift (radians or degrees) added to the input sinusoid at frequency $\omega$.
- **Decibel (dB)** — logarithmic unit for magnitude response, $|H|_{dB} = 20\log_{10}|H(j\omega)|$ (factor of 20, not 10, since power $\propto$ voltage$^2$).
- **Decade** — a frequency interval where the frequency multiplies by 10.
- **Octave** — a frequency interval where the frequency doubles.
- **Pole / corner frequency / break frequency / cutoff frequency $\omega_0$** — the angular frequency at which a first-order circuit's magnitude response transitions from roughly flat to roughly falling; $\omega_0 = 1/\tau$ for a single-time-constant circuit ($f_0 = \omega_0/2\pi$ in Hz).
- **Bode plot** — a pair of plots of a transfer function vs. frequency on a logarithmic frequency axis: magnitude in dB vs. $\log\omega$, and phase in degrees vs. $\log\omega$.
- **Bode magnitude asymptote** — the piecewise-linear straight-line approximation to a Bode magnitude plot, exact at $\omega\to0$ and $\omega\to\infty$, meeting at the corner frequency; deviates from the true curve by at most about 3 dB right at the corner.
- **Low-pass filter** — a circuit whose $|H(j\omega)|$ is largest at low $\omega$ and falls off at high $\omega$.
- **High-pass filter** — a circuit whose $|H(j\omega)|$ is largest at high $\omega$ and falls off at low $\omega$.
- **Roll-off rate** — the slope of the Bode magnitude asymptote past the corner frequency, in dB/decade (20 dB/decade for a single pole, equivalently 6 dB/octave).
