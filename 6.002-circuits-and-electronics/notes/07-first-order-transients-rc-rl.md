---
title: "First-Order Transients (RC, RL)"
course: "6.002"
topic_number: 07
prerequisites: ["06-capacitors-inductors-energy-storage", "18.03 first-order linear ODEs (recapped inline below)"]
status: in progress
---

# First-Order Transients (RC, RL)

## Why this matters

Every real circuit takes *time* to react. Flip a switch feeding a capacitor or an inductor, and the voltage/current doesn't jump instantly to its final value — it glides there along an exponential curve. This matters practically (how fast can a logic gate charge the wire capacitance it's driving? how long until a relay coil's current stabilizes?) and it matters as the foundation for everything that follows in this course: second-order (RLC) transients, sinusoidal steady state, and frequency response are all built on the same exponential machinery introduced here, just with richer circuits. If you skip this topic, "time constant," "settling time," and "step response" — vocabulary used constantly for the rest of 6.002 — won't mean anything.

The key new fact, not available before this note: capacitors and inductors have *memory* (their voltage/current depends on the circuit's past, via $i = C\,dv/dt$ and $v = L\,di/dt$), so circuits containing them are described by *differential* equations, not the algebraic equations of resistive circuits. This note shows how to solve those differential equations for the simplest case: exactly one capacitor or inductor per circuit.

## Builds on

- [[6.002-circuits-and-electronics/notes/06-capacitors-inductors-energy-storage]] — the defining relations $i = C\,\dfrac{dv}{dt}$ for a capacitor and $v = L\,\dfrac{di}{dt}$ for an inductor, and the fact that capacitor voltage and inductor current cannot change instantaneously (doing so would require infinite current or infinite voltage, i.e. infinite power in zero time).
- Ordinary differential equations (normally 18.03, not yet written in this vault) — the solution method for a first-order linear ODE with constant coefficients is recapped from scratch in the Derivation section below, so no outside course is needed to follow this note.
- Basic resistive circuit analysis (KVL, KCL, Ohm's law) — assumed from earlier 6.002 topics (Topics 01–02): series/parallel resistor combination, and writing loop equations.

## Core definitions

- **Transient** — the time-varying part of a circuit's response that decays away, as opposed to the steady value it settles toward. "First-order transient" means the response is governed by a first-order (one derivative) differential equation.
- **Natural response** — the response of a circuit with no independent sources active (all independent voltage sources replaced by $0\,\text V$ short circuits, all independent current sources replaced by $0\,\text A$ open circuits), driven only by energy already stored in the capacitor/inductor.
- **Forced (particular) response** — the response the circuit would settle into if you waited forever with the sources active; also called the **steady-state** or **final value**, written $x(\infty)$.
- **Complete response** — natural response + forced response; the actual voltage or current as a function of time, $x(t)$.
- **Time constant, $\tau$** — the time scale of the exponential decay/growth. For an RC circuit, $\tau = RC$ (units: ohms $\times$ farads = seconds). For an RL circuit, $\tau = L/R$ (units: henries / ohms = seconds). After one time constant, a decaying exponential has fallen to $1/e \approx 36.8\%$ of its initial value above its final value; after $5\tau$ it is within $1\%$ of final value (used as the practical "settled" criterion).
- **Step input** — a source that switches abruptly from one constant value to another at some time $t_0$ (e.g., a switch closing at $t=0$, connecting a battery to a circuit that was previously at rest). This note only treats step inputs.
- **Initial condition, $x(0)$** — the value of the capacitor voltage or inductor current immediately after the switching event, at $t = 0^+$ (an infinitesimal time after $t=0$). Because capacitor voltage and inductor current cannot jump, $x(0^+) = x(0^-)$ (the value just before switching) for those two specific variables — but *not* necessarily for other variables in the circuit, like resistor voltages or currents, which can jump.
- **Thévenin-equivalent resistance seen by the capacitor/inductor, $R_{th}$** — the single resistance obtained by (a) turning off all independent sources (short voltage sources, open current sources) and (b) computing the resistance looking into the two terminals where the capacitor or inductor connects. This is the $R$ that appears in $\tau = RC$ or $\tau = L/R$, even in circuits with more than one resistor.

## Intuition

Picture a capacitor as a small tank that stores charge, and think of current as water flow. If you suddenly connect a water pump (voltage source) to an empty tank through a narrow pipe (resistor), the tank doesn't fill instantly — the fuller it gets, the more back-pressure it exerts, so the flow rate (current) tapers off exponentially and the tank's water level (voltage) rises toward the pump's pressure, also exponentially, slowing down as it approaches the final level. That "slowing down in proportion to remaining distance from the target" is exactly what defines exponential behavior: the rate of change of a quantity is proportional to how far the quantity still is from its final value.

An inductor is the dual: think of it as a heavy flywheel coupled to the current. Current, like the flywheel's spin, resists sudden changes because changing it means changing the energy stored in the associated magnetic field, and that requires a finite voltage $v = L\,di/dt$ — instantaneous current change would need infinite voltage. So when you suddenly apply a voltage across an inductor (in series with a resistor), the current ramps up gradually, again along an exponential curve, approaching whatever final current the resistor network and sources dictate.

In both cases the shape is the same exponential curve; only which variable can't jump (voltage for C, current for L) and which combination of circuit elements sets the time scale ($RC$ vs $L/R$) differ. Once you have the three numbers — initial value, final value, and time constant — you have the entire time-domain answer, because a first-order linear circuit with constant sources always relaxes as a single decaying (or growing-then-saturating) exponential toward its final value. No other shape is possible for this class of circuit.

## Derivation / formalism

### Setting up the differential equation: RC circuit

Consider a circuit with a voltage source $V_s$, a switch that closes at $t = 0$, a resistor $R$, and a capacitor $C$, all in one loop (a series RC circuit driven by a step). Let $v(t)$ be the capacitor voltage, with the sign convention that $v$ is the voltage across $C$ using the standard passive convention (current $i$ flows into the $+$ terminal of $C$).

For $t > 0$, KVL around the loop gives:
$$V_s = i R + v$$

The capacitor's defining relation (from Topic 06) is:
$$i = C\,\frac{dv}{dt}$$

Substitute:
$$V_s = RC\,\frac{dv}{dt} + v$$

Rearrange into standard form (all $v$-terms on the left):
$$RC\,\frac{dv}{dt} + v = V_s \qquad (t>0)$$

This is a first-order linear ordinary differential equation (ODE) with constant coefficients. It has exactly one derivative of the unknown ($dv/dt$), and the coefficients ($RC$ and $1$) don't depend on $t$ or $v$ — that's what "linear, constant-coefficient, first-order" means.

### Recap: solving a first-order linear constant-coefficient ODE from scratch

Because 18.03 (differential equations) may not yet be available in this vault, here is the complete method, self-contained.

We want to solve an equation of the form
$$\tau\,\frac{dx}{dt} + x = X_f \qquad (1)$$
where $\tau$ and $X_f$ are constants (in our RC case, $\tau = RC$ and $X_f = V_s$; $x$ stands for whatever unknown we're solving for, here $v$).

**Step 1 — guess the form of the total solution.** Because the equation is linear, its general solution is the sum of (a) any one particular solution $x_p(t)$ that satisfies the full equation, and (b) the general solution $x_h(t)$ of the associated *homogeneous* equation (the same equation with the right-hand side set to zero). This "particular + homogeneous" decomposition is a general property of linear ODEs: if $x_p$ solves $\tau \dot x_p + x_p = X_f$ and $x_h$ solves $\tau \dot x_h + x_h = 0$, then $x = x_p + x_h$ solves $\tau(\dot x_p + \dot x_h) + (x_p + x_h) = X_f + 0 = X_f$, by linearity of differentiation and addition. So we solve the two pieces separately.

**Step 2 — particular solution.** Since $X_f$ is a constant, try a constant $x_p(t) = A$. Then $\dot x_p = 0$, and the ODE becomes $\tau \cdot 0 + A = X_f$, so $A = X_f$. Thus $x_p(t) = X_f$ for all $t$ — this is exactly the steady-state / final value, which makes physical sense: once everything has settled, nothing is changing, so $dx/dt = 0$, and the ODE reduces to the algebraic statement $x_p = X_f$.

**Step 3 — homogeneous solution.** Solve $\tau\,\dfrac{dx_h}{dt} + x_h = 0$, i.e. $\dfrac{dx_h}{dt} = -\dfrac{x_h}{\tau}$. Separate variables (put all $x_h$ terms with $dx_h$, all $t$ terms with $dt$):
$$\frac{dx_h}{x_h} = -\frac{dt}{\tau}$$
Integrate both sides:
$$\int \frac{dx_h}{x_h} = -\int \frac{dt}{\tau} \implies \ln|x_h| = -\frac{t}{\tau} + K_1$$
where $K_1$ is a constant of integration. Exponentiate both sides:
$$|x_h| = e^{-t/\tau + K_1} = e^{K_1} e^{-t/\tau}$$
Absorb the constant $e^{K_1}$ (and the sign from the absolute value) into a single constant $K$:
$$x_h(t) = K e^{-t/\tau}$$
You can check this directly satisfies the homogeneous equation: $\dot x_h = -\frac{K}{\tau} e^{-t/\tau} = -\frac{1}{\tau} x_h$, so $\tau \dot x_h + x_h = \tau\left(-\frac{1}{\tau}x_h\right) + x_h = -x_h + x_h = 0$. ✓.

**Step 4 — total solution and applying the initial condition.**
$$x(t) = x_p(t) + x_h(t) = X_f + K e^{-t/\tau}$$
$K$ is fixed by the initial condition $x(0)$, known from the physical circuit (before switching). Setting $t=0$: $x(0) = X_f + K e^0 = X_f + K$, so $K = x(0) - X_f$. Substituting back:
$$\boxed{x(t) = X_f + \big(x(0) - X_f\big) e^{-t/\tau}}\qquad (2)$$

This is *the* master formula for every first-order step response in this course: **final value, plus (initial value minus final value) decaying exponentially with time constant $\tau$.** Note it automatically reduces to $x(0)$ at $t=0$ and to $X_f$ as $t \to \infty$ (since $e^{-t/\tau}\to 0$).

### Applying it to the RC circuit

Matching to Equation (1): $\tau = RC$, $X_f = V_s$ (the forced/final capacitor voltage — makes sense: at steady state, capacitor current $i = C\,dv/dt \to 0$, so no current flows, so no drop across $R$, so $v \to V_s$). The initial voltage $v(0)$ is whatever the capacitor was charged to before $t=0$ (often $0\,\text V$ if it started uncharged, but not always). So:
$$v(t) = V_s + \big(v(0) - V_s\big)e^{-t/RC}$$

The current follows from $i = C\,dv/dt$; differentiating,
$$\frac{dv}{dt} = \big(v(0)-V_s\big)\left(-\frac{1}{RC}\right)e^{-t/RC}$$
so
$$i(t) = C \cdot \big(v(0)-V_s\big)\left(-\frac{1}{RC}\right)e^{-t/RC} = \frac{V_s - v(0)}{R}e^{-t/RC}$$
This also matches physical intuition: at $t=0^+$, the current is $\big(V_s - v(0)\big)/R$ — exactly what Ohm's law gives if you treat the capacitor, at that instant, as a fixed voltage source of value $v(0)$ in series with $R$ (valid because $v$ can't jump).

### RL circuit, by the same method

Now a series loop with voltage source $V_s$, resistor $R$, inductor $L$, switch closing at $t=0$. Let $i(t)$ be the inductor current. KVL:
$$V_s = iR + v_L, \qquad v_L = L\,\frac{di}{dt}$$
Substituting:
$$V_s = iR + L\frac{di}{dt} \implies \frac{L}{R}\frac{di}{dt} + i = \frac{V_s}{R}$$
Matching to Equation (1) with $x = i$: $\tau = L/R$, $X_f = V_s/R$ (the final current — at steady state $di/dt \to 0$ so $v_L \to 0$, and all of $V_s$ appears across $R$, giving $i \to V_s/R$ by Ohm's law). Using the master formula (2):
$$i(t) = \frac{V_s}{R} + \left(i(0) - \frac{V_s}{R}\right)e^{-tR/L}$$
and the inductor voltage is $v_L(t) = L\,di/dt$, worked out the same way as for the RC case:
$$v_L(t) = \big(V_s - i(0)R\big)e^{-tR/L}$$

### Generalizing to circuits with many resistors and sources

Real circuits usually have more than one resistor and possibly multiple sources, but as long as there is **exactly one** capacitor or inductor, the same exponential shape still applies — you just need three numbers, obtained as follows:

1. **Initial value $x(0)$**: the capacitor voltage or inductor current immediately before switching (from the "before" circuit, assumed at DC steady state unless stated otherwise — meaning the capacitor acts as an open circuit and the inductor as a short circuit, because at true DC steady state nothing changes so $dv/dt = 0$ and $di/dt = 0$).
2. **Final value $x(\infty)$**: solve the "after" circuit (with the switch in its new state) at DC steady state — again capacitor→open, inductor→short — using ordinary resistive-circuit analysis (KVL/KCL, series/parallel reduction, or superposition) to find the capacitor voltage or inductor current there.
3. **Time constant $\tau$**: turn off all independent sources in the "after" circuit (voltage sources → short, current sources → open), find the equivalent resistance $R_{th}$ seen by the capacitor or inductor's two terminals (combine all other resistors as seen from those terminals), and set $\tau = R_{th}C$ (RC) or $\tau = L/R_{th}$ (RL).

Then the complete response for *any* variable $x$ in the circuit (not just the capacitor voltage/inductor current — every voltage and current in a first-order circuit shares the same exponential time-dependence, since they're all linear combinations of the state variable and the sources) is:
$$x(t) = x(\infty) + \big(x(0) - x(\infty)\big)e^{-t/\tau}, \qquad t \ge 0$$

This is why Step 1–3 above is the standard 6.002 recipe: you never need to re-derive the ODE for every new circuit topology — you only need three numbers, each obtainable from plain resistive-circuit analysis (which you already know) plus this one formula.

## Worked examples

### Example 1 — RC circuit, capacitor charging from rest

**Setup.** A $10\,\text{V}$ battery, a $2\,\text{k}\Omega$ resistor, and a $5\,\mu\text{F}$ capacitor are in series with a switch. The capacitor starts fully discharged ($v(0) = 0\,\text V$). At $t=0$ the switch closes, connecting the battery. Find $v(t)$ and $i(t)$, and evaluate them at $t = 10\,\text{ms}$.

**Step 1: time constant.**
$$\tau = RC = (2\times10^3\,\Omega)(5\times10^{-6}\,\text F) = 10\times10^{-3}\,\text s = 10\,\text{ms}$$

**Step 2: initial and final values.** Given $v(0) = 0\,\text V$. Final value: at steady state, capacitor current $\to 0$, so no drop across $R$, so $v(\infty) = V_s = 10\,\text V$.

**Step 3: assemble.**
$$v(t) = 10 + (0 - 10)e^{-t/0.01} = 10\big(1 - e^{-t/0.01}\big)\ \text{V}, \quad t \text{ in seconds}$$
$$i(t) = \frac{V_s - v(0)}{R}e^{-t/RC} = \frac{10 - 0}{2000}e^{-t/0.01} = 5\times10^{-3}\,e^{-t/0.01}\ \text A = 5\,e^{-t/0.01}\ \text{mA}$$

**Step 4: evaluate at $t = 10\,\text{ms} = \tau$.**
$$v(0.01) = 10\big(1 - e^{-1}\big) = 10(1 - 0.3679) = 10(0.6321) = 6.321\,\text V$$
$$i(0.01) = 5\,e^{-1} = 5(0.3679) = 1.839\,\text{mA}$$

**Sanity check:** at exactly one time constant, any charging exponential should be at $63.2\%$ of the way from initial to final value: $0 + 0.632\times(10-0) = 6.32\,\text V$. ✓ Matches. Also check power/units: $i(0)R = 5\,\text{mA} \times 2\,\text{k}\Omega = 10\,\text V = V_s$, consistent with $v(0)=0$ meaning the full battery voltage initially appears across $R$. ✓

### Example 2 — RL circuit with a Thévenin-resistance step, and a non-series resistor

**Setup.** An inductor $L = 4\,\text{mH}$ has, in parallel with it, a resistor $R_2 = 30\,\Omega$. This parallel combination is fed through a series resistor $R_1 = 20\,\Omega$ from a $12\,\text V$ source, with a switch that has been closed for a long time (so the circuit is at DC steady state) and then opens at $t=0$, disconnecting the source but leaving $R_1$'s far end floating (so after the switch opens, current can only circulate in the loop formed by $L$ and $R_2$; $R_1$ carries no current and drops out of the "after" circuit). Find $i_L(t)$ for $t>0$ and evaluate at $t = 0.2\,\text{ms}$.

**Step 1: initial value $i_L(0)$ — from the "before" circuit (switch closed, steady state).** At DC steady state the inductor acts as a short (zero voltage drop, since $v_L = L\,di/dt = 0$ when $i$ is constant). With the inductor shorting out $R_2$ (same two nodes), all current from the source flows through $R_1$ then through the inductor's short, bypassing $R_2$ entirely (a short circuit in parallel with $R_2$ carries all the current, since a short has zero resistance and current divides inversely with resistance — the $R_2$ branch would need zero resistance too to share current, which it doesn't have). So the "before" circuit is just $V_s$ in series with $R_1$:
$$i_L(0) = \frac{V_s}{R_1} = \frac{12\,\text V}{20\,\Omega} = 0.6\,\text A$$

**Step 2: final value $i_L(\infty)$ — from the "after" circuit.** After the switch opens, the source and $R_1$ are disconnected (no closed path through them), leaving only the loop of $L$ and $R_2$. With no source active in that loop, the final steady-state current is:
$$i_L(\infty) = 0\,\text A$$

**Step 3: time constant.** In the "after" circuit, the only resistance seen by the inductor's terminals is $R_2$ (the $R_1$ branch is open, contributing nothing):
$$\tau = \frac{L}{R_{th}} = \frac{4\times10^{-3}\,\text H}{30\,\Omega} = 1.333\times10^{-4}\,\text s = 0.1333\,\text{ms}$$

**Step 4: assemble.**
$$i_L(t) = 0 + (0.6 - 0)e^{-t/\tau} = 0.6\,e^{-t/1.333\times10^{-4}}\ \text A$$

**Step 5: evaluate at $t = 0.2\,\text{ms} = 2\times10^{-4}\,\text s$.**
$$\frac{t}{\tau} = \frac{2\times10^{-4}}{1.333\times10^{-4}} = 1.5$$
$$i_L(2\times10^{-4}) = 0.6\,e^{-1.5} = 0.6(0.2231) = 0.1339\,\text A \approx 133.9\,\text{mA}$$

**Sanity check on units:** $L/R$ has units $\text H/\Omega = (\text V\cdot\text s/\text A)/(\text V/\text A) = \text s$. ✓ And the inductor current decays monotonically from $0.6\,\text A$ toward $0\,\text A$, consistent with a passive $R_2$ dissipating the energy that was stored in $L$ (energy stored was $\frac12 L i_L(0)^2 = \frac12(4\times10^{-3})(0.6)^2 = 7.2\times10^{-4}\,\text J$, all of which is eventually dissipated in $R_2$ as the current decays to zero) — no source remains to sustain a nonzero current, so final value $0$ is the only physically sensible answer. ✓

## Common pitfalls

- **Forgetting that only $v_C$ and $i_L$ are continuous.** Every *other* voltage or current in the circuit (resistor voltages, resistor currents, the capacitor's *current*, the inductor's *voltage*) can and generally does jump discontinuously at $t=0$. Don't assume $i(0^-) = i(0^+)$ for a capacitor's current, or $v(0^-)=v(0^+)$ for an inductor's voltage.
- **Using the wrong resistance in $\tau = RC$ or $\tau = L/R$.** It's the *Thévenin-equivalent* resistance seen by the capacitor/inductor terminals in the circuit that's active *after* switching, with all independent sources zeroed (shorted/opened) — not just "the resistor in the circuit," especially when there are multiple resistors or the circuit reconfigures at $t=0$.
- **Mixing up the "before" and "after" circuits.** The initial condition comes from the steady state of the circuit *before* switching; the final value and time constant come from the circuit *after* switching. Using the after-circuit's source values to compute the initial condition (or vice versa) gives a wrong answer even if every individual formula is applied correctly.
- **Forgetting the capacitor is an open circuit and the inductor is a short circuit only at DC steady state**, not at every instant. This shortcut is valid exactly when $dv/dt = 0$ (capacitor) or $di/dt = 0$ (inductor), i.e. long after any switching transient has settled (rule of thumb: $t \gg 5\tau$) — it is not valid at $t=0^+$ or during the transient.
- **Sign errors in the master formula.** $x(t) = x(\infty) + (x(0)-x(\infty))e^{-t/\tau}$ — double check whether the exponential term should be growing toward or decaying away from the final value by plugging in $t=0$ and $t\to\infty$ mentally and confirming they match the known initial/final values.
- **Time constant units.** $RC$ must come out in seconds — if $C$ is given in $\mu\text F$ or $\text{pF}$, convert to farads (multiply by $10^{-6}$ or $10^{-12}$) before multiplying by $R$ in ohms, or the exponent $t/\tau$ will be dimensionally wrong by orders of magnitude. Same caution for $L$ in $\text{mH}$/$\mu\text H$ in $L/R$.
- **Confusing "5 time constants to settle" with "the response is linear."** The exponential never *exactly* reaches its final value in finite time; "$5\tau$" is an engineering convention (within $\approx 0.67\%$ of final value: $e^{-5}\approx 0.0067$) for "close enough," not a mathematical endpoint.

## Self-check

### Questions

1. A capacitor has $v(0) = 5\,\text V$ and is connected through a resistor to a $0\,\text V$ (grounded) node. Is this charging or discharging? What is $v(\infty)$?
2. Write the master formula $x(t) = x(\infty) + (x(0)-x(\infty))e^{-t/\tau}$ from memory and state, in one sentence each, what each of the three symbols $x(0)$, $x(\infty)$, $\tau$ physically represents for an RC circuit.
3. A series RC circuit has $R = 1\,\text{k}\Omega$, $C = 1\,\mu\text F$. What is $\tau$? After how much time (in the same units) is the transient within $1\%$ of its final value?
4. True or false, with justification: "At the instant a switch closes in an RL circuit, the inductor current can change abruptly if the source voltage is large enough." 
5. A capacitor charged to $v(0) = 20\,\text V$ discharges through a $5\,\text{k}\Omega$ resistor and reaches $v(\infty) = 0\,\text V$ with $\tau = 2\,\text{ms}$. Find $v(t)$ and the current $i(t)$ (through the resistor) as functions of time, and evaluate both at $t=4\,\text{ms}$.
6. An RL circuit has $L = 100\,\text{mH}$, and the Thévenin resistance seen by the inductor after switching is $R_{th} = 25\,\Omega$. If $i(0) = 0\,\text A$ and $i(\infty) = 2\,\text A$, at what time does the current first reach $1.8\,\text A$? (Hint: solve the exponential equation for $t$ using natural logs.)
7. A circuit has two resistors $R_1 = 6\,\text k\Omega$ and $R_2 = 3\,\text k\Omega$ in parallel, and this parallel combination is the only resistance seen by a $2\,\mu\text F$ capacitor after switching. What time constant should you use — $R_1 C$, $R_2 C$, or something else? Compute it.
8. In Example 2 above, suppose instead the switch, when it opens, leaves $R_1$ still connected in a loop with $R_2$ and $L$ (i.e., $R_1$ and $R_2$ end up in parallel with each other, and that combination in the loop with $L$, with no source). Find the new $R_{th}$ seen by $L$ and the new $\tau$, given $R_1=20\,\Omega$, $R_2=30\,\Omega$, $L=4\,\text{mH}$.

### Answers

1. Discharging (capacitor voltage starts at $5\,\text V$, above its final value, and decays down toward it). $v(\infty) = 0\,\text V$, matching the grounded node it settles to (no current flows at steady state, so no drop is needed to explain — the capacitor voltage equals the node it's tied toward through the resistor since eventually no current flows and both ends of R are at the same potential... more precisely, at $t\to\infty$, $i\to 0$, so the capacitor's node floats at the same potential as the grounded node, i.e. $0\,\text V$).
2. $x(t) = x(\infty) + \big(x(0)-x(\infty)\big)e^{-t/\tau}$. $x(0)$: the capacitor's voltage right after switching (equal to its value right before switching, since capacitor voltage can't jump). $x(\infty)$: the voltage the capacitor would settle to if you waited forever after switching (found by treating the capacitor as an open circuit in the post-switch circuit). $\tau = R_{th}C$: the time scale of the exponential approach, where $R_{th}$ is the resistance seen by the capacitor's terminals with independent sources zeroed.
3. $\tau = RC = (1000\,\Omega)(1\times10^{-6}\,\text F) = 1\times10^{-3}\,\text s = 1\,\text{ms}$. Within $1\%$ means $e^{-t/\tau} \le 0.01$, i.e. $t/\tau \ge \ln(100) \approx 4.605$, so $t \ge 4.605\,\text{ms} \approx 4.6\,\text{ms}$ (consistent with the "$5\tau$" rule of thumb, which is slightly more conservative than needed for exactly $1\%$).
4. **False.** Inductor current $i_L$ cannot change abruptly regardless of source voltage magnitude, because an instantaneous jump in $i_L$ would require $di_L/dt \to \infty$, and since $v_L = L\,di_L/dt$, that would require infinite voltage across the inductor — which no finite source can supply. What *can* jump abruptly when the switch closes is the inductor's *voltage* $v_L$, not its current.
5. $v(t) = 0 + (20-0)e^{-t/0.002} = 20\,e^{-t/0.002}\ \text V$ (t in seconds). Current through the resistor (discharging, so current flows from the capacitor's $+$ terminal through R): $i(t) = v(t)/R = \dfrac{20\,e^{-t/0.002}}{5000} = 4\times10^{-3}e^{-t/0.002}\ \text A = 4\,e^{-t/0.002}\ \text{mA}$. At $t=4\,\text{ms} = 2\tau$: $v = 20\,e^{-2} = 20(0.1353) = 2.707\,\text V$; $i = 4\,e^{-2} = 4(0.1353) = 0.5413\,\text{mA}$.
6. Master formula: $i(t) = 2 + (0-2)e^{-t/\tau} = 2\big(1-e^{-t/\tau}\big)$, with $\tau = L/R_{th} = 0.1/25 = 4\times10^{-3}\,\text s = 4\,\text{ms}$. Set $i(t) = 1.8$: $1.8 = 2(1-e^{-t/\tau}) \Rightarrow 0.9 = 1-e^{-t/\tau} \Rightarrow e^{-t/\tau} = 0.1 \Rightarrow -t/\tau = \ln(0.1) = -2.3026 \Rightarrow t = 2.3026\,\tau = 2.3026(4\,\text{ms}) = 9.21\,\text{ms}$.
7. The equivalent resistance $R_{th}$ seen by the capacitor is $R_1 \parallel R_2$ (both resistors are in the path from the capacitor's terminals to the rest of the zeroed-source circuit, in parallel), not either one alone: $R_{th} = \dfrac{R_1 R_2}{R_1+R_2} = \dfrac{(6000)(3000)}{6000+3000} = \dfrac{18{,}000{,}000}{9000} = 2000\,\Omega = 2\,\text k\Omega$. Then $\tau = R_{th}C = (2000)(2\times10^{-6}) = 4\times10^{-3}\,\text s = 4\,\text{ms}$.
8. With $R_1$ and $R_2$ now both in the loop with $L$ and no source, and $R_1 \parallel R_2$ as stated: $R_{th} = \dfrac{R_1R_2}{R_1+R_2} = \dfrac{(20)(30)}{50} = \dfrac{600}{50} = 12\,\Omega$. Then $\tau = L/R_{th} = (4\times10^{-3})/12 = 3.333\times10^{-4}\,\text s = 0.3333\,\text{ms}$. This is larger than Example 2's $\tau$ of $0.1333\,\text{ms}$: there, $R_{th}=R_2=30\,\Omega$ alone (since $R_1$'s branch was open); here $R_1\parallel R_2 = 12\,\Omega$ is smaller than $30\,\Omega$ alone, and since $\tau = L/R_{th}$ is inversely proportional to $R_{th}$, a smaller $R_{th}$ gives a larger $\tau$ — consistent with the numbers.

## Summary / cheat sheet

**Defining relations (from Topic 06):** capacitor $i = C\,\dfrac{dv}{dt}$; inductor $v = L\,\dfrac{di}{dt}$. Neither $v_C$ nor $i_L$ can jump instantaneously (would require infinite power).

**Universal first-order step-response formula:**
$$x(t) = x(\infty) + \big(x(0)-x(\infty)\big)e^{-t/\tau}, \quad t\ge 0$$
applies to *every* voltage and current in a circuit containing exactly one capacitor or inductor and step sources.

**Recipe (3 numbers):**
1. $x(0)$ — from the pre-switch circuit at DC steady state (C → open, L → short).
2. $x(\infty)$ — from the post-switch circuit at DC steady state (C → open, L → short).
3. $\tau$ — from the post-switch circuit with sources zeroed: $\tau = R_{th}C$ (RC) or $\tau = L/R_{th}$ (RL), where $R_{th}$ is the resistance seen by the capacitor/inductor terminals.

**Time-constant facts:** at $t=\tau$, response is $63.2\%$ of the way from initial to final value; at $t=5\tau$, within $\approx 0.67\%$ (treated as "settled"). $\tau_{RC}=RC$ [s = $\Omega\cdot$F]; $\tau_{RL}=L/R$ [s = H/$\Omega$].

**Sign/shape check:** plug $t=0$ into the formula (must recover $x(0)$) and let $t\to\infty$ (must recover $x(\infty)$) as a fast sanity check on any derived expression.

## Used later in
(none yet — will be filled in once later notes, e.g. second-order RLC transients, cite this note)
