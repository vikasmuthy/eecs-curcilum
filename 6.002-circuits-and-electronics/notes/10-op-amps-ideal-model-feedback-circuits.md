---
title: "Op-amps: Ideal Model, Feedback Circuits"
course: "6.002"
topic_number: 10
prerequisites: ["05-amplifiers-large-small-signal-analysis", "02-resistive-networks-node-mesh-analysis"]
status: in progress
---

# Op-amps: Ideal Model, Feedback Circuits

## Why this matters

Every real amplifier built from a single transistor (see Topic 05) has gain that depends on the transistor's exact characteristics, the supply voltage, temperature, and the load it's driving — none of which the designer can pin down precisely or hold constant. An **operational amplifier** ("op-amp") is a pre-built, high-gain amplifier chip that, when wrapped in the right external resistor network (a **feedback** circuit), produces a gain that depends *only* on those external resistors — not on the messy internal details of the chip. This is the single idea that makes practical analog circuit design possible: you buy a "good enough" amplifier once, and then feedback lets you dial in whatever precise gain, filtering, or buffering behavior you want using cheap, stable resistors. Skip this topic and you cannot analyze or design any real-world amplifier stage, active filter, instrumentation front end, or analog-to-digital signal-conditioning chain — all of which dominate the rest of 6.002's applications and the analog portions of 6.302.

## Builds on

- [[6.002-circuits-and-electronics/notes/05-amplifiers-large-small-signal-analysis]] — the idea of an amplifier as a circuit element with an input port and output port related by a gain, and the notion of a dependent source used to model gain.
- [[6.002-circuits-and-electronics/notes/02-resistive-networks-node-mesh-analysis]] — KCL (Kirchhoff's Current Law: currents into a node sum to zero) and KVL (Kirchhoff's Voltage Law: voltage drops around any closed loop sum to zero), plus solving resistor networks by writing node equations.

Because this is the first op-amp note in the course, and because it leans on two results that are foundational but easy to forget, both are recapped self-containedly below before they're used.

**Recap — dependent source (from Topic 05):** A *dependent source* is an ideal source (voltage or current) whose value is not fixed, but is instead proportional to some other voltage or current elsewhere in the circuit. For example, a *voltage-controlled voltage source* (VCVS) with gain $A$ produces an output voltage $v_{out} = A\,v_{in}$, where $v_{in}$ is a voltage measured somewhere else in the same circuit, and this relationship holds no matter what is connected to the output terminals (an ideal VCVS has zero output resistance — it can supply any current at that fixed voltage). This is exactly the internal model we will use for the op-amp's gain stage.

**Recap — KCL/KVL and node analysis (from Topic 02):** KCL says the sum of currents leaving any node is zero (charge cannot accumulate at a node in a lumped circuit). KVL says the sum of voltage drops around any closed loop is zero (equivalent to saying voltage is a well-defined function of position — path-independent — in the lumped abstraction). "Node analysis" means picking every distinct node in a circuit, writing one KCL equation per node in terms of unknown node voltages (using Ohm's law $i = v/R$ to convert resistor currents into voltage differences divided by resistance), and solving the resulting system of linear equations.

## Core definitions

- **Operational amplifier (op-amp)** — a packaged, high-gain differential amplifier with two input terminals (non-inverting input $v_+$ and inverting input $v_-$), one output terminal $v_{out}$, and (usually two, sometimes omitted from diagrams) power-supply terminals. Internally it behaves approximately like a VCVS with gain $A$ applied to the *difference* of its two inputs: $v_{out} = A(v_+ - v_-)$.
- **Open-loop gain $A$** — the op-amp's own internal voltage gain, with no external feedback path connected. For real op-amps $A$ is large (typically $10^4$ to $10^6$) but not infinite, not perfectly constant, and not perfectly linear over the full output range.
- **Ideal op-amp model** — the idealization used for hand analysis, defined by three assumptions:
  1. Infinite open-loop gain: $A \to \infty$.
  2. Infinite input resistance: no current flows into either input terminal ($i_+ = i_- = 0$).
  3. Zero output resistance: the output can supply/sink any current needed while its voltage stays exactly $v_{out} = A(v_+ - v_-)$ (before taking the limit).
- **Feedback** — connecting some of the output signal back to one of the inputs through an external network (here, resistors), so that the circuit's behavior is governed by that external network rather than by $A$ alone.
- **Negative feedback** — feedback routed back to the *inverting* input ($v_-$), which (as the derivation below shows) makes the closed-loop circuit self-correcting and stable. All circuits in this note use negative feedback.
- **Virtual short (or "virtual ground" when one input is grounded)** — the emergent fact that, under negative feedback with an ideal (infinite-gain) op-amp, $v_+ = v_-$ even though no wire directly connects those two nodes. Justified rigorously in the derivation, not assumed.
- **Closed-loop gain** — the overall gain of the op-amp *plus* its feedback network, from the circuit's external input to $v_{out}$, once feedback is in place. This is what a designer actually wants to control.
- **Inverting amplifier** — a feedback configuration where the input signal is applied (through a resistor) to $v_-$, while $v_+$ is grounded; output is an inverted (negated), scaled copy of the input.
- **Non-inverting amplifier** — a feedback configuration where the input signal is applied directly to $v_+$, and feedback returns part of $v_{out}$ to $v_-$; output is a non-inverted, scaled copy of the input.
- **Summing amplifier** — an inverting-amplifier variant with multiple input resistors, producing a weighted sum of several input voltages at the output.

## Intuition

Think of the bare op-amp as an almost pathologically twitchy comparator: it looks at $v_+ - v_-$ and, because $A$ is enormous, blows that difference up into a huge output swing. If $v_+$ is even a microvolt above $v_-$, an ideal op-amp with $A \to \infty$ wants to slam $v_{out}$ to $+\infty$; if $v_+$ is a microvolt below $v_-$, it wants $v_{out} \to -\infty$. Left alone (open-loop), this is nearly useless as a linear amplifier — it's a very sensitive threshold detector.

Feedback tames this. If you take the output and route part of it back to $v_-$ through a resistor network, you create a self-correcting loop: any tiny difference $v_+ - v_-$ gets amplified hugely at the output, but that same output change (fed back to $v_-$) acts to *reduce* the difference $v_+ - v_-$ that caused it. The loop only settles down (reaches equilibrium) at the one operating point where $v_+ - v_-$ has shrunk to essentially zero — because that's the only point where the enormous gain $A$ isn't blowing the output off to a rail. This is why, in *every* correctly-built negative-feedback op-amp circuit analyzed with the ideal model, you can just write $v_+ = v_-$ and go from there: it's not that the two nodes are wired together, it's that the feedback loop forces them to the same voltage as a consequence of the amplifier trying (and being allowed, by a finite linear output) to equalize its inputs.

The other ideal assumption — no current into the inputs — says the op-amp's input terminals behave like true voltmeter probes: they sense voltage without disturbing the circuit by drawing current. Combine "$v_+ = v_-$" with "no input current" and "apply KCL/KVL to the external resistors," and every closed-loop gain formula in this note falls out of ordinary node analysis (Topic 02) with no new physics.

## Derivation / formalism

### Setting up the ideal op-amp equations

By definition, the ideal op-amp obeys these constraints at its two input terminals, for *any* external circuit connected to it, as long as the overall circuit reaches a stable linear operating point — i.e. $v_{out}$ settles to a finite value strictly between the amplifier's positive and negative supply voltages ("rails"), rather than **saturating**: pinned at (near) one of those rails, a nonlinear regime discussed further in Common Pitfalls below:

$$i_+ = 0, \qquad i_- = 0 \qquad \text{(no current into either input)}$$

and internally,

$$v_{out} = A(v_+ - v_-), \qquad A \to \infty.$$

We cannot use $v_{out} = A(v_+-v_-)$ directly with $A=\infty$ (that would force $v_+-v_-=0$ trivially and give no information about $v_{out}$ itself — we need to combine it with the external feedback network to pin down $v_{out}$). The standard method: keep $A$ finite and large, write the full circuit equations, then take the limit $A \to \infty$ at the end. We do this once below for the inverting amplifier so the "virtual short" claim is *proven*, not asserted; afterward we take the shortcut of assuming $v_+ = v_-$ directly, since the limit has already been justified.

### Inverting amplifier — full derivation (justifying the virtual-short shortcut)

Circuit: input voltage source $v_{in}$ connects through resistor $R_1$ to node $v_-$. A feedback resistor $R_2$ connects node $v_-$ to the output node $v_{out}$. The non-inverting input $v_+$ is tied directly to ground, so $v_+ = 0$. We want $v_{out}$ in terms of $v_{in}$, $R_1$, $R_2$.

**Step 1 — KCL at the $v_-$ node.** Current flowing into node $v_-$ from the source through $R_1$ must equal the current flowing out of node $v_-$ through $R_2$ toward the output, *plus* the current flowing into the op-amp's inverting terminal itself, $i_-$:

$$\frac{v_{in} - v_-}{R_1} = \frac{v_- - v_{out}}{R_2} + i_-$$

(Ohm's law: current from node $a$ to node $b$ through resistor $R$ is $(v_a - v_b)/R$; here current flows from the source node $v_{in}$ into $v_-$ across $R_1$, and from $v_-$ into the output node across $R_2$.)

**Step 2 — apply the ideal no-input-current condition.** $i_- = 0$, so:

$$\frac{v_{in} - v_-}{R_1} = \frac{v_- - v_{out}}{R_2} \tag{1}$$

**Step 3 — write the op-amp's internal gain relation**, keeping $A$ finite for now, with $v_+ = 0$:

$$v_{out} = A(0 - v_-) = -A\,v_- \quad \Longrightarrow \quad v_- = -\frac{v_{out}}{A} \tag{2}$$

**Step 4 — substitute (2) into (1)** to eliminate $v_-$:

$$\frac{v_{in} - \left(-\dfrac{v_{out}}{A}\right)}{R_1} = \frac{-\dfrac{v_{out}}{A} - v_{out}}{R_2}$$

$$\frac{v_{in} + \dfrac{v_{out}}{A}}{R_1} = \frac{-v_{out}\left(\dfrac{1}{A} + 1\right)}{R_2}$$

**Step 5 — multiply both sides by $R_1 R_2$** to clear denominators:

$$R_2\left(v_{in} + \frac{v_{out}}{A}\right) = -R_1\, v_{out}\left(\frac{1}{A} + 1\right)$$

$$R_2 v_{in} + \frac{R_2 v_{out}}{A} = -\frac{R_1 v_{out}}{A} - R_1 v_{out}$$

**Step 6 — collect all $v_{out}$ terms on one side:**

$$R_2 v_{in} = -\frac{R_1 v_{out}}{A} - R_1 v_{out} - \frac{R_2 v_{out}}{A}$$

$$R_2 v_{in} = -v_{out}\left(R_1 + \frac{R_1 + R_2}{A}\right)$$

**Step 7 — solve for $v_{out}$:**

$$v_{out} = \frac{-R_2\, v_{in}}{R_1 + \dfrac{R_1+R_2}{A}}$$

**Step 8 — take the limit $A \to \infty$.** The term $\dfrac{R_1+R_2}{A} \to 0$, leaving:

$$\boxed{v_{out} = -\frac{R_2}{R_1}\, v_{in}} \qquad \text{(inverting amplifier, closed-loop gain } -R_2/R_1\text{)}$$

**Step 9 — check the virtual-short claim.** From (2), $v_- = -v_{out}/A$. As $A \to \infty$ with $v_{out}$ finite (which Step 8 shows it is), $v_- \to 0$. Since $v_+ = 0$ by construction, this confirms $v_+ = v_- = 0$ in the limit — the "virtual ground" is a *derived consequence* of infinite gain plus a finite, stable output, not an extra assumption.

This same argument generalizes: whenever an ideal op-amp is wired with negative feedback (output fed back to $v_-$) and settles to a finite $v_{out}$, the internal relation $v_{out} = A(v_+-v_-)$ forces $v_+ - v_- = v_{out}/A \to 0$ as $A\to\infty$. So from here on we use the shortcut directly:

$$v_+ = v_- \qquad \text{(valid for any ideal op-amp in negative feedback, finite-output operation)}$$

### Non-inverting amplifier — derivation using the shortcut

Circuit: input voltage source $v_{in}$ connects directly to $v_+$. The output $v_{out}$ connects to $v_-$ through a feedback resistor $R_2$, and $v_-$ also connects to ground through resistor $R_1$ (so $R_1, R_2$ form a voltage divider between $v_{out}$ and ground, tapped at $v_-$).

**Step 1 — apply the virtual-short result:** $v_+ = v_-$, and $v_+ = v_{in}$ directly, so:

$$v_- = v_{in} \tag{3}$$

**Step 2 — KCL at node $v_-$.** No current flows into the op-amp's $v_-$ terminal ($i_- = 0$), so all the current flowing from $v_{out}$ through $R_2$ into node $v_-$ must continue on through $R_1$ to ground:

$$\frac{v_{out} - v_-}{R_2} = \frac{v_- - 0}{R_1}$$

**Step 3 — substitute (3), $v_- = v_{in}$:**

$$\frac{v_{out} - v_{in}}{R_2} = \frac{v_{in}}{R_1}$$

**Step 4 — solve for $v_{out}$.** Multiply both sides by $R_2$:

$$v_{out} - v_{in} = \frac{R_2}{R_1}\, v_{in}$$

$$v_{out} = v_{in} + \frac{R_2}{R_1} v_{in} = v_{in}\left(1 + \frac{R_2}{R_1}\right)$$

$$\boxed{v_{out} = \left(1 + \frac{R_2}{R_1}\right) v_{in}} \qquad \text{(non-inverting amplifier, closed-loop gain } 1+R_2/R_1\text{)}$$

Note the gain is always $\geq 1$ and positive (no inversion), unlike the inverting configuration.

### Summing amplifier — quick extension

Take the inverting amplifier and connect $n$ input resistors $R_1, R_2, \dots, R_n$ (each from its own input source $v_1, \dots, v_n$) all meeting at the same $v_-$ node, with a single feedback resistor $R_f$ from $v_-$ to $v_{out}$. Since $v_+ = 0$ (grounded) forces $v_- = 0$ (virtual ground), KCL at $v_-$ (currents in from each input resistor, current out through $R_f$, zero current into the op-amp) gives:

$$\frac{v_1 - 0}{R_1} + \frac{v_2-0}{R_2} + \cdots + \frac{v_n - 0}{R_n} = \frac{0 - v_{out}}{R_f}$$

Solving for $v_{out}$:

$$\boxed{v_{out} = -R_f\left(\frac{v_1}{R_1} + \frac{v_2}{R_2} + \cdots + \frac{v_n}{R_n}\right)}$$

a weighted, inverted sum — the algebra is identical in structure to the single-input inverting case, just with more KCL terms at the same node.

## Worked examples

### Example 1 — Inverting amplifier, numeric gain and output swing check

An ideal op-amp is wired as an inverting amplifier with $R_1 = 2\ \text{k}\Omega$ and $R_2 = 20\ \text{k}\Omega$. The input is $v_{in} = 0.3\ \text{V}$. The op-amp is powered from $\pm 15\ \text{V}$ supplies (so $v_{out}$ must physically stay between $-15\ \text{V}$ and $+15\ \text{V}$).

**Step 1 — closed-loop gain:**

$$\text{Gain} = -\frac{R_2}{R_1} = -\frac{20\,000\ \Omega}{2\,000\ \Omega} = -10$$

**Step 2 — output voltage:**

$$v_{out} = -10 \times 0.3\ \text{V} = -3\ \text{V}$$

**Step 3 — sanity check against supply rails:** $-3\ \text{V}$ is well within $[-15\ \text{V}, +15\ \text{V}]$, so the ideal linear model is self-consistent here (the op-amp is not being asked for an output it cannot physically produce).

**Step 4 — check input node current, confirming virtual ground:** $v_- = 0\ \text{V}$ (virtual ground, since $v_+ = 0$). Current through $R_1$: $i_1 = (v_{in}-v_-)/R_1 = (0.3\ \text{V} - 0\ \text{V})/2000\ \Omega = 1.5\times10^{-4}\ \text{A} = 0.15\ \text{mA}$. This same current must flow through $R_2$ (since $i_-=0$): $i_2 = (v_- - v_{out})/R_2 = (0\ \text{V} - (-3\ \text{V}))/20\,000\ \Omega = 3\ \text{V}/20\,000\ \Omega = 1.5\times10^{-4}\ \text{A} = 0.15\ \text{mA}$. Matches $i_1$, confirming KCL is satisfied.

**Result:** $v_{out} = -3\ \text{V}$.

### Example 2 — Non-inverting amplifier feeding a summing amplifier

Two stages in series: Stage A is a non-inverting amplifier with $R_1 = 1\ \text{k}\Omega$, $R_2 = 9\ \text{k}\Omega$, driven by $v_{in} = 0.5\ \text{V}$. Stage A's output feeds into one input of a summing amplifier (Stage B), which also takes a second, independent input $v_{2} = -1\ \text{V}$. Stage B has input resistors $R_a = 10\ \text{k}\Omega$ (from Stage A's output) and $R_b = 5\ \text{k}\Omega$ (from $v_2$), and feedback resistor $R_f = 20\ \text{k}\Omega$.

**Step 1 — Stage A output:**

$$v_{A} = \left(1 + \frac{R_2}{R_1}\right)v_{in} = \left(1 + \frac{9000}{1000}\right)(0.5\ \text{V}) = (1+9)(0.5\ \text{V}) = 10 \times 0.5\ \text{V} = 5\ \text{V}$$

**Step 2 — check Stage A is a valid input to Stage B.** $v_A = 5\ \text{V}$, a normal signal level; fine as an input to Stage B.

**Step 3 — Stage B output (summing amplifier formula):**

$$v_{out} = -R_f\left(\frac{v_A}{R_a} + \frac{v_2}{R_b}\right) = -20\,000\left(\frac{5}{10\,000} + \frac{-1}{5\,000}\right)$$

**Step 4 — compute each term inside the parentheses:**

$$\frac{5\ \text{V}}{10\,000\ \Omega} = 5\times10^{-4}\ \text{A} = 0.5\ \text{mA}$$

$$\frac{-1\ \text{V}}{5\,000\ \Omega} = -2\times10^{-4}\ \text{A} = -0.2\ \text{mA}$$

Sum: $0.5\ \text{mA} + (-0.2\ \text{mA}) = 0.3\ \text{mA} = 3\times10^{-4}\ \text{A}$.

**Step 5 — multiply by $-R_f$:**

$$v_{out} = -20\,000\ \Omega \times 3\times10^{-4}\ \text{A} = -6\ \text{V}$$

**Result:** $v_{out} = -6\ \text{V}$. This demonstrates that op-amp stages **cascade** cleanly: because each stage's output is an ideal (zero output resistance) voltage source and each stage's input draws zero current, connecting stages in series never changes any individual stage's gain formula — a key practical payoff of the ideal model.

## Common pitfalls

- **Treating $v_+ = v_-$ as a real short circuit.** It is *not* a wire — no current flows directly between $v_+$ and $v_-$. It's an equality of voltages that emerges only under negative feedback with finite, stable output. Confusing this with an actual connection leads to wrongly computing currents (e.g., assuming current can flow "through" the virtual short).
- **Applying the virtual-short shortcut without negative feedback.** If feedback is routed to $v_+$ instead of $v_-$ ("positive feedback"), the loop is unstable and drives $v_{out}$ to a supply rail rather than settling — $v_+ = v_-$ does *not* hold, and the ideal linear formulas above do not apply.
- **Forgetting the input current is exactly zero, not "small."** The ideal model sets $i_+ = i_- = 0$ exactly. This is what allows "whatever current flows into a summing junction through the input resistors must flow entirely out through the feedback resistor" — dropping this assumption (as with a real, non-ideal op-amp with finite input resistance) changes the gain formulas.
- **Ignoring supply-rail (saturation) limits.** The formulas $v_{out} = -\frac{R_2}{R_1}v_{in}$, etc., are only valid when the *computed* $v_{out}$ is physically achievable given the supply voltages. If the formula predicts $v_{out}$ outside $[-V_{supply}, +V_{supply}]$ (roughly), the real op-amp saturates (clips) at (near) the rail instead, and the linear ideal-model formula no longer describes the actual output.
- **Sign errors in the inverting-amplifier gain.** The gain is $-R_2/R_1$, with $R_2$ the *feedback* resistor (from $v_-$ to $v_{out}$) and $R_1$ the *input* resistor (from $v_{in}$ to $v_-$) — swapping which resistor is which, or dropping the minus sign, is the single most common algebra mistake with this circuit.
- **Mixing up which input is grounded in inverting vs. non-inverting configurations.** Inverting: $v_+$ grounded, signal into $v_-$ via $R_1$. Non-inverting: signal into $v_+$ directly, $v_-$ set by the feedback divider. Swapping these swaps the entire circuit's behavior.
- **Assuming the "no current into inputs" rule means no current flows anywhere near the op-amp.** Current absolutely flows through the external feedback and input resistors (see Example 1, Step 4) — it's only the op-amp's own input *terminals* that draw zero current.

## Self-check

### Questions

1. In an ideal op-amp under negative feedback, why does $v_+ = v_-$ even though there is no wire directly connecting those two terminals?
2. An inverting amplifier has $R_1 = 4\ \text{k}\Omega$ and $R_2 = 12\ \text{k}\Omega$. What is its closed-loop gain?
3. Using the amplifier from Question 2, what is $v_{out}$ if $v_{in} = 0.5\ \text{V}$?
4. A non-inverting amplifier has $R_1 = 5\ \text{k}\Omega$ and $R_2 = 15\ \text{k}\Omega$. Find $v_{out}$ for $v_{in} = 1\ \text{V}$.
5. Explain, in terms of the ideal-model assumptions, why cascading two op-amp stages (feeding one stage's output directly into the next stage's input resistor) does not change either stage's individual gain formula.
6. A summing amplifier has inputs $v_1 = 2\ \text{V}$ through $R_1 = 10\ \text{k}\Omega$ and $v_2 = 4\ \text{V}$ through $R_2 = 20\ \text{k}\Omega$, with feedback resistor $R_f = 10\ \text{k}\Omega$. Find $v_{out}$.
7. Suppose an inverting amplifier with $R_1=1\ \text{k}\Omega$, $R_2 = 100\ \text{k}\Omega$ is driven by $v_{in} = 0.2\ \text{V}$, and the op-amp is powered from $\pm 10\ \text{V}$ rails. What does the *ideal-model formula* predict for $v_{out}$, and why can the real circuit not actually produce that output? What would you expect to observe instead?
8. Re-derive the non-inverting amplifier's closed-loop gain formula keeping $A$ finite (do not use the $v_+=v_-$ shortcut), and show it reduces to $1+R_2/R_1$ as $A\to\infty$. (This mirrors the inverting-amplifier derivation in this note but for the non-inverting topology.)

### Answers

1. Because the op-amp's internal relation is $v_{out} = A(v_+-v_-)$ with $A\to\infty$. If the circuit settles to any finite output voltage $v_{out}$, then $v_+-v_- = v_{out}/A \to 0$ as $A\to\infty$. It's a consequence of infinite gain plus a finite, stable (non-saturated) output — not a physical wire. (Full justification: see the Step-by-step derivation, Steps 8–9.)
2. Gain $= -R_2/R_1 = -12\,000/4\,000 = -3$.
3. $v_{out} = (-3)(0.5\ \text{V}) = -1.5\ \text{V}$.
4. Gain $= 1 + R_2/R_1 = 1 + 15\,000/5\,000 = 1+3=4$. $v_{out} = 4 \times 1\ \text{V} = 4\ \text{V}$.
5. Because each op-amp stage's output behaves as an ideal voltage source (zero output resistance — it holds its computed $v_{out}$ regardless of the current drawn from it), and each downstream stage's input resistor sees only that fixed voltage; meanwhile the ideal input draws exactly zero current ($i_+=i_-=0$), so connecting a next stage never "loads down" or diverts current away from the previous stage's feedback network. Both gain formulas were derived assuming exactly this — an unloaded, ideal-source output and a non-current-drawing input — so cascading changes nothing in either formula.
6. $v_{out} = -R_f\left(\dfrac{v_1}{R_1}+\dfrac{v_2}{R_2}\right) = -10\,000\left(\dfrac{2}{10\,000}+\dfrac{4}{20\,000}\right) = -10\,000\left(2\times10^{-4}+2\times10^{-4}\right) = -10\,000 \times 4\times10^{-4} = -4\ \text{V}$.
7. Ideal formula: gain $=-R_2/R_1 = -100\,000/1\,000=-100$; predicted $v_{out} = -100 \times 0.2\ \text{V} = -20\ \text{V}$. This exceeds the $\pm10\ \text{V}$ supply rails, which is physically impossible — an op-amp's output can never exceed (in magnitude) roughly its supply voltage, since it is itself powered from those supplies. In the real circuit, the op-amp saturates: $v_{out}$ clips at approximately $-10\ \text{V}$ (or slightly less in magnitude, accounting for the op-amp's own internal voltage drop) instead of reaching $-20\ \text{V}$, and the output-vs-input relationship is no longer linear once saturated.
8. Circuit: $v_+ = v_{in}$ directly; $v_-$ set by divider between $v_{out}$ and ground through $R_2, R_1$. KCL at $v_-$ (with $i_-=0$): $\dfrac{v_{out}-v_-}{R_2} = \dfrac{v_-}{R_1}$. Internal relation with finite $A$: $v_{out} = A(v_{in}-v_-) \Rightarrow v_- = v_{in} - v_{out}/A$. Substitute into KCL equation: $\dfrac{v_{out} - v_{in}+v_{out}/A}{R_2} = \dfrac{v_{in}-v_{out}/A}{R_1}$. Multiply both sides by $R_1R_2$: $R_1\left(v_{out}-v_{in}+\dfrac{v_{out}}{A}\right) = R_2\left(v_{in}-\dfrac{v_{out}}{A}\right)$. Expand: $R_1 v_{out} - R_1 v_{in} + \dfrac{R_1 v_{out}}{A} = R_2 v_{in} - \dfrac{R_2 v_{out}}{A}$. Collect $v_{out}$ terms: $v_{out}\left(R_1 + \dfrac{R_1+R_2}{A}\right) = v_{in}(R_1+R_2)$. So $v_{out} = \dfrac{(R_1+R_2)v_{in}}{R_1 + (R_1+R_2)/A}$. As $A\to\infty$, the denominator's second term vanishes, giving $v_{out} = \dfrac{(R_1+R_2)}{R_1}v_{in} = \left(1+\dfrac{R_2}{R_1}\right)v_{in}$, matching the shortcut-derived formula.

## Summary / cheat sheet

**Ideal op-amp assumptions:** $A\to\infty$ (infinite open-loop gain); $i_+=i_-=0$ (no input current); zero output resistance ($v_{out}$ holds regardless of load current) — valid only while $v_{out}$ stays within the supply rails (not saturated).

**Virtual short (negative feedback only):** $v_+ = v_-$ — derived from $v_{out}=A(v_+-v_-)$ as $A\to\infty$ with finite $v_{out}$; not a real wire; does *not* hold for positive feedback.

**Inverting amplifier** (input via $R_1$ into $v_-$, feedback $R_2$ from $v_-$ to $v_{out}$, $v_+$ grounded):
$$v_{out} = -\frac{R_2}{R_1}\,v_{in}$$

**Non-inverting amplifier** (input directly into $v_+$; $v_-$ set by divider $R_1$ (to ground), $R_2$ (to $v_{out}$)):
$$v_{out} = \left(1+\frac{R_2}{R_1}\right)v_{in}$$

**Summing (inverting) amplifier** (inputs $v_1,\dots,v_n$ each via its own $R_k$ into $v_-$; feedback $R_f$; $v_+$ grounded):
$$v_{out} = -R_f\sum_k \frac{v_k}{R_k}$$

**Method for any ideal-op-amp negative-feedback circuit:** (1) set $v_+=v_-$; (2) set $i_+=i_-=0$; (3) write KCL at the $v_-$ (and/or $v_+$) node(s) using Ohm's law on the external resistors; (4) solve the resulting linear equation(s) for $v_{out}$ — pure node analysis, no op-amp-specific tricks beyond steps (1)-(2).

**Always sanity-check:** does the computed $v_{out}$ fit within the supply rails? If not, the real circuit saturates and the linear formula is invalid there.

## Used later in
(none yet)
