---
title: "Capacitors, Inductors, and Energy Storage"
course: "6.002"
topic_number: 06
prerequisites: ["01-02 lumped circuit abstraction, KVL/KCL", "8.02 electric and magnetic fields (recapped inline)"]
status: done
---

# Capacitors, Inductors, and Energy Storage

## Why this matters

Every circuit element analyzed so far in this course (resistors, sources) relates voltage and current *instantaneously and algebraically*: $v = iR$ holds at every instant with no memory of the past. Capacitors and inductors break that pattern — their voltage-current relationships involve *rates of change* (derivatives) and *history* (integrals). This is the single biggest conceptual jump in 6.002 so far: circuits containing these elements are described by differential equations, not algebraic ones. That is precisely why they show up here, right before the transient-response topics (RC and RL circuits, then RLC) — you cannot understand *why* an RC circuit charges up smoothly over time, or why a switching power supply needs an inductor, without first nailing down what a capacitor and inductor *are* and how they store energy. Skip this topic and every later topic in the course — transients, sinusoidal steady state, filters, op-amp compensation — becomes a set of memorized formulas with no physical grounding.

## Builds on

- [[6.002-circuits-and-electronics/notes/01-lumped-circuit-abstraction-kvl-kcl]] — we still treat capacitors and inductors as lumped two-terminal elements with a well-defined terminal voltage $v(t)$ and terminal current $i(t)$, and KVL/KCL still apply unchanged around loops and at nodes containing them.
- [[6.002-circuits-and-electronics/notes/02-resistive-networks-node-mesh-analysis]] — the passive sign convention and $v$-$i$ element-law bookkeeping used for resistors is extended here to capacitors and inductors.
- 8.02 energy-storage field concepts — recapped self-containedly below, since this note assumes only the passive-sign convention and Ohm's law from topics 01-02, not a completed 8.02 course.

## Core definitions

- **Passive sign convention** — current $i(t)$ is defined as entering the terminal marked "+" for voltage $v(t)$. Under this convention, instantaneous power *delivered to* the element is $p(t) = v(t)\,i(t)$. Positive $p$ means the element is absorbing energy; negative $p$ means it is delivering energy back to the circuit.
- **Capacitor** — a two-terminal lumped element that stores energy in an electric field. Its defining relation is $i = C\,\dfrac{dv}{dt}$, where $C$ (units: farads, F) is the capacitance, a fixed nonnegative constant for an ideal (linear, time-invariant) capacitor.
- **Capacitance $C$** — the constant of proportionality between charge stored on the capacitor's plates and the voltage across it: $q = Cv$. Measured in farads (F); $1\text{ F} = 1\text{ C/V}$.
- **Inductor** — a two-terminal lumped element that stores energy in a magnetic field. Its defining relation is $v = L\,\dfrac{di}{dt}$, where $L$ (units: henries, H) is the inductance, a fixed nonnegative constant for an ideal (linear, time-invariant) inductor.
- **Inductance $L$** — the constant of proportionality between magnetic flux linkage through the inductor's coil and the current through it: $\lambda = Li$. Measured in henries (H); $1\text{ H} = 1\text{ V·s/A}$.
- **Energy storage element** — a circuit element that can absorb energy from the circuit, hold it (without dissipating it as heat, in the ideal case), and later return it. Contrasted with a resistor, which only dissipates energy irreversibly as heat.
- **State variable** — a quantity that summarizes the "memory" of an energy-storage element: capacitor voltage $v_C$ (equivalently, charge $q$) and inductor current $i_L$ (equivalently, flux linkage $\lambda$) are the state variables of a circuit, because they cannot change discontinuously (see Common pitfalls) and they, together with the circuit topology, determine all future behavior.

## Intuition

**Capacitor.** Picture two parallel metal plates separated by a thin insulating gap (this is literally the physical structure of the simplest capacitor). If you push positive charge onto one plate and pull it off the other, the plates build up equal and opposite charge $+q$ and $-q$. This separated charge creates an electric field in the gap, and that field represents stored energy — exactly like stretching a spring stores mechanical energy. The more charge you push, the higher the voltage between the plates; for a linear capacitor this relationship is exactly proportional, $q = Cv$. Current is charge in motion, so the *rate* at which charge accumulates, $dq/dt$, is the current flowing into the capacitor: $i = dq/dt = C\,dv/dt$ (since $C$ is constant). This is the key qualitative fact to internalize: **a capacitor's voltage cannot jump instantaneously**, because that would require moving a finite amount of charge in zero time, i.e. infinite current. A capacitor "wants" to keep its voltage the same from one instant to the next; it fights sudden voltage changes. At DC (constant voltage, $dv/dt = 0$) a capacitor carries zero current — it looks like an open circuit once everything has settled.

**Inductor.** Picture a coil of wire. When current flows through the coil, it creates a magnetic field threading through the loops (this is the 8.02 fact: a current loop produces a magnetic field, and the total flux through the loops for a fixed geometry is proportional to the current, $\lambda = Li$). If the current changes, the magnetic flux through the coil changes, and by Faraday's law (also from 8.02: a changing magnetic flux through a loop induces an EMF around that loop) a voltage is induced across the coil that opposes the change — $v = d\lambda/dt = L\,di/dt$. The inductor "wants" to keep its current the same; it fights sudden current changes. This is the dual intuition to the capacitor: **an inductor's current cannot jump instantaneously**, because that would require an infinite induced voltage. At DC (constant current, $di/dt = 0$) an inductor has zero voltage across it — it looks like a short circuit (plain wire) once everything has settled.

**Duality.** Capacitor and inductor are mirror images of each other with voltage and current swapped: $i = C\,dv/dt$ versus $v = L\,di/dt$; capacitor voltage is the "sluggish" state variable versus inductor current is the "sluggish" state variable; capacitor is open at DC versus inductor is a short at DC. This duality is not a coincidence — it will resurface constantly in the transient-response and impedance topics later in the course, so it is worth memorizing now.

## Derivation / formalism

### 1. The capacitor $i$-$v$ relationship, from charge

Start from the physical definition of capacitance (a fact carried over from 8.02, recapped here): for two conductors carrying charge $+q$ and $-q$, the voltage between them is proportional to $q$ for a fixed geometry,
$$q(t) = C\,v(t), \qquad C = \text{const} \ge 0.$$

Current into the "+" terminal, under the passive sign convention, is by definition the rate at which charge flows onto that plate:
$$i(t) = \frac{dq(t)}{dt}.$$

Substitute $q(t) = Cv(t)$, and since $C$ is a constant (does not depend on $t$), the derivative passes through it by the constant-multiple rule of differentiation:
$$i(t) = \frac{d}{dt}\big[C\,v(t)\big] = C\,\frac{dv(t)}{dt}.$$

This is the capacitor's defining equation. Integrating both sides from some reference time $t_0$ to $t$ gives the equivalent integral form (useful for finding $v(t)$ given $i(t)$):
$$\int_{t_0}^{t} i(\tau)\,d\tau = \int_{t_0}^{t} C\,\frac{dv}{d\tau}\,d\tau = C\big[v(t) - v(t_0)\big]$$
$$\Rightarrow\quad v(t) = v(t_0) + \frac{1}{C}\int_{t_0}^{t} i(\tau)\,d\tau.$$

### 2. The inductor $v$-$i$ relationship, from flux linkage

By the dual physical fact from 8.02 (recapped here): for a coil of fixed geometry carrying current $i$, the total magnetic flux linkage through the coil is proportional to $i$,
$$\lambda(t) = L\,i(t), \qquad L = \text{const} \ge 0.$$

Faraday's law of induction (also an 8.02 fact) states that the voltage induced across the coil equals the rate of change of flux linkage:
$$v(t) = \frac{d\lambda(t)}{dt}.$$

Substitute $\lambda(t) = Li(t)$, and since $L$ is constant:
$$v(t) = \frac{d}{dt}\big[L\,i(t)\big] = L\,\frac{di(t)}{dt}.$$

Integrating both sides from $t_0$ to $t$ gives the dual integral form:
$$i(t) = i(t_0) + \frac{1}{L}\int_{t_0}^{t} v(\tau)\,d\tau.$$

### 3. Energy stored in a capacitor

Instantaneous power delivered *to* the capacitor (passive sign convention) is
$$p(t) = v(t)\,i(t) = v(t)\cdot C\,\frac{dv(t)}{dt}.$$

Total energy delivered to the capacitor from time $t_0$ (assume $v(t_0) = 0$, i.e. starting uncharged) up to time $t$ is the integral of power:
$$w(t) = \int_{t_0}^{t} p(\tau)\,d\tau = \int_{t_0}^{t} C\,v(\tau)\,\frac{dv(\tau)}{d\tau}\,d\tau.$$

Change variables: let $u = v(\tau)$, so $du = \dfrac{dv(\tau)}{d\tau}\,d\tau$. When $\tau = t_0$, $u = v(t_0) = 0$; when $\tau = t$, $u = v(t)$. The integral becomes
$$w(t) = \int_{0}^{v(t)} C\,u\,du = C\left[\frac{u^2}{2}\right]_0^{v(t)} = \frac{1}{2}C\,v(t)^2.$$

So the energy stored in a capacitor charged to voltage $v$ is
$$\boxed{w_C = \tfrac{1}{2}C v^2}.$$

Because this expression depends only on the present value of $v$ (not on the path taken to get there), the energy is fully recoverable: charge the capacitor up, then discharge it, and you get the same energy back out (ideal case, no resistive losses in the charging path).

### 4. Energy stored in an inductor

By the exactly dual argument: power delivered to the inductor is
$$p(t) = v(t)\,i(t) = L\,\frac{di(t)}{dt}\cdot i(t).$$

Assume $i(t_0) = 0$ (starting with no current). Total energy delivered up to time $t$:
$$w(t) = \int_{t_0}^{t} L\,i(\tau)\,\frac{di(\tau)}{d\tau}\,d\tau.$$

Let $u = i(\tau)$, $du = \dfrac{di(\tau)}{d\tau}\,d\tau$; limits go from $u=0$ to $u = i(t)$:
$$w(t) = \int_0^{i(t)} L\,u\,du = \frac{1}{2}L\,i(t)^2.$$

So the energy stored in an inductor carrying current $i$ is
$$\boxed{w_L = \tfrac{1}{2}L i^2}.$$

### 5. Series and parallel combinations (stated with derivation sketch, needed for later topics)

**Capacitors in parallel** (same voltage $v$ across each, by KVL applied to the loop formed by the two branches): total current into the parallel combination is the sum of branch currents (KCL),
$$i = i_1 + i_2 = C_1\frac{dv}{dt} + C_2\frac{dv}{dt} = (C_1+C_2)\frac{dv}{dt} \;\Rightarrow\; C_{\text{parallel}} = C_1 + C_2.$$
Parallel capacitors add — intuitively, you're just increasing the plate area available to store charge at a given voltage.

**Capacitors in series** (same current $i$ through each, by KCL at the node joining them, since no charge can accumulate at that internal node): each capacitor's voltage obeys $v_k(t) = v_k(t_0) + \frac{1}{C_k}\int i\,d\tau$. Assuming both start at $0$, total voltage across the series pair is
$$v = v_1 + v_2 = \frac{1}{C_1}\int_{t_0}^t i\,d\tau + \frac{1}{C_2}\int_{t_0}^t i\,d\tau = \left(\frac{1}{C_1}+\frac{1}{C_2}\right)\int_{t_0}^t i\,d\tau \;\Rightarrow\; \frac{1}{C_{\text{series}}} = \frac{1}{C_1}+\frac{1}{C_2}.$$
Series capacitors combine like the *reciprocal* rule (same form as resistors in parallel) — this is the flip side of the duality noted above.

**Inductors** combine by exactly the opposite rule, because $v = L\,di/dt$ is structurally the dual of $i = C\,dv/dt$:
$$L_{\text{series}} = L_1 + L_2 \qquad(\text{same current through each, KVL adds voltages}),$$
$$\frac{1}{L_{\text{parallel}}} = \frac{1}{L_1}+\frac{1}{L_2} \qquad(\text{same voltage across each, KCL adds currents}).$$
Inductors in series add like resistors in series; inductors in parallel combine like resistors in parallel. (Capacitors are the mirror image: series capacitors combine like parallel resistors, and vice versa.)

## Worked examples

### Example 1 — Charging a capacitor at constant current; energy stored

A $C = 10\ \mu\text{F} = 10\times10^{-6}\text{ F}$ capacitor is initially uncharged, $v(0) = 0\text{ V}$. Starting at $t=0$, a constant current source forces $i(t) = 2\text{ mA} = 2\times10^{-3}\text{ A}$ into it for $t \ge 0$.

**(a) Find $v(t)$ for $t \ge 0$.**

Use the integral form derived above:
$$v(t) = v(0) + \frac{1}{C}\int_0^t i(\tau)\,d\tau = 0 + \frac{1}{10\times10^{-6}}\int_0^t (2\times10^{-3})\,d\tau.$$

The integrand is constant, so $\int_0^t (2\times10^{-3})\,d\tau = 2\times10^{-3}\,t$. Thus
$$v(t) = \frac{2\times10^{-3}}{10\times10^{-6}}\,t = 200\,t \quad\text{[volts, with } t \text{ in seconds]}.$$

Check dimensions: $\dfrac{\text{A}}{\text{F}} = \dfrac{\text{A}}{\text{C/V}} = \dfrac{\text{A}\cdot\text{V}}{\text{C}} = \dfrac{\text{A}\cdot\text{V}}{\text{A}\cdot\text{s}} = \text{V/s}$ — correct, $v(t)$ grows linearly in volts.

**(b) At $t = 5\text{ ms} = 5\times10^{-3}\text{ s}$, find $v$, the charge $q$, and the stored energy $w_C$.**

$$v(5\text{ ms}) = 200 \times (5\times10^{-3}) = 1.0\text{ V}.$$

Charge: $q = Cv = (10\times10^{-6}\text{ F})(1.0\text{ V}) = 1.0\times10^{-5}\text{ C} = 10\ \mu\text{C}$.

(Sanity check via a second route: $q = \int_0^t i\,d\tau = (2\times10^{-3}\text{ A})(5\times10^{-3}\text{ s}) = 1.0\times10^{-5}\text{ C}$ — matches.)

Stored energy: $w_C = \tfrac{1}{2}Cv^2 = \tfrac12 (10\times10^{-6}\text{ F})(1.0\text{ V})^2 = 5\times10^{-6}\text{ J} = 5\ \mu\text{J}$.

### Example 2 — Inductor current ramp, energy, and a check via power integration

An $L = 4\text{ mH} = 4\times10^{-3}\text{ H}$ inductor carries $i(0) = 0\text{ A}$. A voltage source applies a constant $v(t) = 20\text{ mV} = 0.02\text{ V}$ across it for $0 \le t \le 3\text{ s}$.

**(a) Find $i(t)$.**

$$i(t) = i(0) + \frac{1}{L}\int_0^t v(\tau)\,d\tau = 0 + \frac{1}{4\times10^{-3}}\int_0^t 0.02\,d\tau = \frac{0.02}{4\times10^{-3}}\,t = 5\,t \quad\text{[amps, } t \text{ in seconds]}.$$

Dimension check: $\dfrac{\text{V}}{\text{H}} = \dfrac{\text{V}}{\text{V·s/A}} = \dfrac{\text{A}}{\text{s}}$ — correct, current ramps linearly in amps.

**(b) Find $i(3\text{ s})$ and the energy stored at that instant.**

$$i(3\text{ s}) = 5\times 3 = 15\text{ A}.$$
$$w_L = \tfrac12 L i^2 = \tfrac12 (4\times10^{-3}\text{ H})(15\text{ A})^2 = \tfrac12 (4\times10^{-3})(225) = 0.45\text{ J}.$$

**(c) Verify by direct power integration** (this checks the energy formula against the raw definition, not just the shortcut):
$$p(t) = v(t)\,i(t) = (0.02)(5t) = 0.1\,t \quad\text{[watts]}.$$
$$w(3\text{ s}) = \int_0^3 p(t)\,dt = \int_0^3 0.1\,t\,dt = 0.1\left[\frac{t^2}{2}\right]_0^3 = 0.1 \times \frac{9}{2} = 0.1 \times 4.5 = 0.45\text{ J}.$$
Matches part (b). Good — the shortcut formula $w_L = \tfrac12 Li^2$ is confirmed consistent with integrating $p=vi$ directly for this case.

## Common pitfalls

- **Treating $C$ or $L$ as if they relate $v$ and $i$ algebraically like Ohm's law.** $i=C\,dv/dt$ is a *differential* relation: you must know $dv/dt$, not just $v$, to get $i$. A constant nonzero voltage across a capacitor gives *zero* current (not some fixed current computed from $v$ alone), which surprises students used to resistor thinking.
- **Assuming capacitor voltage or inductor current can jump discontinuously.** They cannot, in any physically realizable circuit (jumping would need infinite current/voltage, hence infinite instantaneous power). This is precisely why $v_C$ and $i_L$ are called *state variables* — they are continuous functions of time even when other quantities in the circuit (like resistor voltages, or switch positions) change abruptly. A very common error in later transient problems is writing $v_C(0^+) \ne v_C(0^-)$ across a switching instant; in fact $v_C(0^+) = v_C(0^-)$ always (same for $i_L$), and *that* continuity is what you use to solve for initial conditions.
- **Confusing $q=Cv$ (algebraic, always true instantaneously) with the derived $i=C\,dv/dt$ (also always true).** Both are correct; $q=Cv$ describes the *state*, $i=C\,dv/dt$ describes how fast the state is changing. Don't try to use $q=Cv$ to compute current directly — you have to differentiate first.
- **Sign convention slips.** If $i$ is defined as leaving the "+" terminal instead of entering it, every formula above picks up a minus sign ($i = -C\,dv/dt$, and power delivered *to* the element becomes $p=-vi$). Always redraw the reference arrows and confirm passive sign convention before plugging into these formulas.
- **Forgetting the series/parallel combination rules flip between C and L.** Capacitors in series behave like resistors in parallel (reciprocal sum); inductors in series behave like resistors in series (direct sum) — mixing these up is a very common exam mistake.
- **Using energy formulas $w=\tfrac12Cv^2$ or $w=\tfrac12Li^2$ with the wrong reference/zero point.** These formulas assume the element started with zero stored energy (uncharged capacitor, zero-current inductor) at the reference time. If there's a nonzero initial condition and you want *energy delivered during an interval*, you must integrate power over that interval and account for the initial stored energy separately, or use $\Delta w = \tfrac12 C(v_2^2 - v_1^2)$ style differences.
- **Unit mix-ups with prefixes.** $\mu\text{F}$ ($10^{-6}$), $\text{nF}$ ($10^{-9}$), $\text{pF}$ ($10^{-12}$) for capacitance; $\text{mH}$ ($10^{-3}$), $\mu\text{H}$ ($10^{-6}$) for inductance are all common in real components — convert to base SI units (F, H) before computing, as done in the worked examples, to avoid factor-of-1000 errors.

## Self-check

### Questions

1. A capacitor has a constant voltage of 5 V across it for all time. What is the current through it? Why?
2. Write, from memory, the defining differential equation for a capacitor and for an inductor, and state which physical quantity ($v$ or $i$) is the "sluggish" state variable for each.
3. A $2\ \mu\text{F}$ capacitor and a $3\ \mu\text{F}$ capacitor are connected in series, both initially uncharged. What is the equivalent capacitance of the combination?
4. An inductor has $i(t) = 0$ for $t<0$ and $i(t) = 3t$ A for $t\ge 0$ (a current ramp starting at $t=0$), with $L = 0.5$ H. Find $v(t)$ for $t>0$, and find the energy stored at $t=2\text{ s}$.
5. A $100\ \mu\text{F}$ capacitor is charged from $0$ to $12$ V by a battery. How much energy is stored in the capacitor at the end? (Note: this is *not* the same as the total energy the battery supplies — the battery also loses some energy to resistance in the charging path — but for this question, just find the capacitor's stored energy.)
6. True or false, with justification: "Since an inductor looks like a short circuit at DC, you can always replace an inductor with a plain wire in any circuit analysis." What is the precise condition under which this replacement is valid?
7. A capacitor $C=1\text{ F}$ starts at $v(0) = 2\text{ V}$. A current $i(t) = 0.5\text{ A}$ (constant) flows into it starting at $t=0$. Find $v(t)$ and the energy stored at $t=0$ and at $t=4\text{ s}$. How much energy was *delivered to* the capacitor during those 4 seconds, and does it match $\int_0^4 p\,dt$?
8. Two inductors, $L_1 = 6$ mH and $L_2 = 3$ mH, are connected in parallel. A third inductor $L_3 = 4$ mH is then placed in series with that parallel combination. Find the total equivalent inductance.

### Answers

1. Current is $i = C\,dv/dt$. Since $v$ is constant, $dv/dt = 0$, so $i = 0$. A capacitor with a constant (DC) voltage across it draws no current — it behaves like an open circuit in steady state. (There is no contradiction with $q=Cv\ne 0$: the capacitor holds a fixed nonzero charge, but a *fixed* charge means no *flow* of charge, hence no current.)
2. Capacitor: $i = C\dfrac{dv}{dt}$, with $v$ (voltage) the state variable that cannot jump. Inductor: $v = L\dfrac{di}{dt}$, with $i$ (current) the state variable that cannot jump.
3. Series capacitors combine reciprocally: $\dfrac{1}{C_{\text{eq}}} = \dfrac{1}{2\ \mu\text{F}} + \dfrac{1}{3\ \mu\text{F}} = \dfrac{3+2}{6\ \mu\text{F}} = \dfrac{5}{6\ \mu\text{F}^{-1}}$, so $C_{\text{eq}} = \dfrac{6}{5}\ \mu\text{F} = 1.2\ \mu\text{F}$.
4. $v(t) = L\dfrac{di}{dt} = 0.5 \times \dfrac{d}{dt}(3t) = 0.5 \times 3 = 1.5\text{ V}$ for $t>0$ (constant, since the current ramp has constant slope). At $t=2\text{ s}$: $i(2) = 3\times2 = 6\text{ A}$, so $w_L = \tfrac12 L i^2 = \tfrac12(0.5)(6)^2 = \tfrac12(0.5)(36) = 9\text{ J}$.
5. $w_C = \tfrac12 C v^2 = \tfrac12 (100\times10^{-6}\text{ F})(12\text{ V})^2 = \tfrac12(100\times10^{-6})(144) = 7.2\times10^{-3}\text{ J} = 7.2\text{ mJ}$.
6. False as an unconditional statement. It is only valid for the DC steady-state (constant-current, all transients settled) portion of the analysis, and only for finding voltages/currents *in that steady state* — replacing the inductor with a wire discards its ability to store energy or oppose changing current, which matters the instant anything switches or varies with time. The precise condition: replace an inductor with a short only when solving for the DC (constant, $di/dt=0$) steady-state operating point of a circuit that has already settled; never during a transient analysis, and never in a circuit driven by a time-varying (e.g. sinusoidal or switching) source.
7. $v(t) = v(0) + \frac{1}{C}\int_0^t i\,d\tau = 2 + \frac{1}{1}(0.5)t = 2 + 0.5t$ volts. At $t=0$: $v=2$ V, $w_C(0) = \tfrac12(1)(2)^2 = 2\text{ J}$. At $t=4$: $v(4) = 2+0.5(4) = 4$ V, $w_C(4) = \tfrac12(1)(4)^2 = 8\text{ J}$. Energy delivered $=w_C(4)-w_C(0) = 8-2 = 6\text{ J}$. Check via power integral: $p(t) = v(t)i(t) = (2+0.5t)(0.5) = 1 + 0.25t$; $\int_0^4 (1+0.25t)\,dt = [t + 0.125t^2]_0^4 = 4 + 0.125(16) = 4+2 = 6\text{ J}$. Matches.
8. Parallel combination: $\dfrac{1}{L_{12}} = \dfrac{1}{6}+\dfrac{1}{3} = \dfrac{1}{6}+\dfrac{2}{6} = \dfrac{3}{6} = \dfrac{1}{2}$, so $L_{12} = 2$ mH. Series with $L_3$: $L_{\text{total}} = L_{12}+L_3 = 2+4 = 6$ mH.

## Summary / cheat sheet

| Quantity | Capacitor | Inductor |
|---|---|---|
| Defining law | $i = C\dfrac{dv}{dt}$ | $v = L\dfrac{di}{dt}$ |
| State variable | $q = Cv$; $v$ can't jump | $\lambda = Li$; $i$ can't jump |
| Integral form | $v(t) = v(t_0)+\frac1C\int_{t_0}^t i\,d\tau$ | $i(t) = i(t_0)+\frac1L\int_{t_0}^t v\,d\tau$ |
| At DC (steady state) | open circuit ($i=0$) | short circuit ($v=0$) |
| Stored energy | $w_C = \tfrac12 Cv^2$ | $w_L = \tfrac12 Li^2$ |
| Series combine | $\dfrac{1}{C_{\text{eq}}}=\sum \dfrac{1}{C_k}$ (like R parallel) | $L_{\text{eq}}=\sum L_k$ (like R series) |
| Parallel combine | $C_{\text{eq}}=\sum C_k$ (like R series is additive, but this is C so it's direct sum) | $\dfrac{1}{L_{\text{eq}}}=\sum \dfrac{1}{L_k}$ (like R parallel) |
| Units | farad, F ($=$ C/V) | henry, H ($=$ V·s/A) |

Key facts to internalize: capacitor voltage and inductor current are continuous in time (no jumps, ever, in a physical circuit); power $p=vi$ can be positive (absorbing) or negative (releasing stored energy) for these elements, unlike a resistor which only ever absorbs; energy stored depends only on the present state ($v$ or $i$), not on history.

## Used later in
- [[6.002-circuits-and-electronics/notes/07-first-order-transients-rc-rl]] — reuses the capacitor/inductor element laws and the no-instantaneous-jump property of $v_C$/$i_L$ as the starting point for RC/RL transient analysis.
- [[6.002-circuits-and-electronics/notes/08-second-order-transients-rlc-damping]] — reuses the same element laws for the RLC second-order case.
- [[6.002-circuits-and-electronics/notes/09-sinusoidal-steady-state-impedance]] — reuses $i=C\,dv/dt$ and $v=L\,di/dt$ as the basis for deriving capacitor/inductor impedance under sinusoidal excitation.
