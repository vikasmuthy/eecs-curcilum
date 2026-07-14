---
title: "Clock Skew and the Static Timing Discipline"
course: "6.004"
topic_number: 15
prerequisites: ["6.004 Topic 14 — Propagation Delay, Setup/Hold Time", "6.004 Topic 13 — Implementing FSMs with Flip-Flops + Combinational Logic", "6.002 Topic 04 — Nonlinear Elements, Digital Abstraction, MOSFET Model"]
status: done
---

# Clock Skew and the Static Timing Discipline

## Why this matters

Topic 14's setup and hold inequalities silently assumed something Topic 13's Common Pitfalls already flagged as fragile: that every flip-flop in a "single clock domain" sees the *exact same* clock edge at the *exact same instant*. In any real circuit, the clock signal has to physically travel — through wires, through buffers used to drive many flip-flops — from its source to each flip-flop, and those paths are essentially never exactly equal in delay. The resulting per-flip-flop difference in when "the same" edge actually arrives is **clock skew**, and this note re-derives Topic 14's setup and hold constraints to account for it, arriving at the complete, general set of inequalities — the **static timing discipline** — that a real synchronous circuit must satisfy everywhere to be guaranteed correct. This is a direct structural echo of 6.002 Topic 04's static discipline for voltage levels: a fixed package of worst-case inequalities that, if satisfied throughout a design, guarantee correct behavior regardless of the exact (bounded) values things actually take — there for volts, here for nanoseconds.

## Builds on

- [[6.004-computation-structures/notes/14-propagation-delay-setup-hold-time]] — the setup constraint ($T\ge t_{pd,FF}+t_{pd,\text{logic}}+t_{setup}$) and hold constraint ($t_{cd,FF}+t_{cd,\text{logic}}\ge t_{hold}$), both derived assuming zero clock skew; this note re-derives both to include a skew term, and shows the zero-skew case is exactly Topic 14's original result.
- [[6.004-computation-structures/notes/13-implementing-fsms-flip-flops]] — the state register's single shared "clock domain" (Common Pitfalls: "giving different flip-flops in the state register different clock signals... risks breaking the architecture's correctness guarantee") and its Self-check Question 5, which already flagged — without deriving — that a buffered, slightly-delayed clock copy to one flip-flop is a real hazard; this note supplies the missing derivation.
- [[6.002-circuits-and-electronics/notes/04-nonlinear-elements-digital-abstraction-mosfet-model]] — the **static discipline** for voltage levels ($V_{OL}\le V_{IL}$ and $V_{OH}\ge V_{IH}$), recapped in full below since that note is still `in progress`; this note's static *timing* discipline is a direct structural analogue, substituting worst-case delay/skew bounds for worst-case voltage bounds.

## Core definitions

- **Clock skew** — for a specific pair of flip-flops (a **launching flip-flop**, whose output feeds a combinational path, and a **capturing flip-flop**, whose $D$ input is fed by that path), the difference in arrival time of the clock edge at the two flip-flops, caused by unequal clock-wire/buffer delay from a common clock source to each. Written $t_{skew}$, treated in this note as a **bounded but unknown** quantity: the design specifies a maximum magnitude $t_{skew,\max}$, but the actual sign and exact value at any given flip-flop pair is not assumed known or controlled.
- **Launching flip-flop / capturing flip-flop** — the source and destination flip-flops of one specific timing path through combinational logic (Topic 14, Section 1); the same flip-flop can be a launching flip-flop for one path and a capturing flip-flop for another, depending on which signal is being traced.
- **Skew-adjusted setup constraint** — the setup inequality of Topic 14, Section 3, re-derived to account for the *worst-case* direction of clock skew that could shrink the effective time available before the capturing edge (derived in Section 2 below).
- **Skew-adjusted hold constraint** — the hold inequality of Topic 14, Section 4, re-derived to account for the *worst-case* direction of clock skew that could shrink the effective time available after the (shared) launching/capturing edge (derived in Section 3 below).
- **Static timing discipline** — the complete, named package consisting of both the skew-adjusted setup constraint and the skew-adjusted hold constraint, required to hold for *every* timing path in a synchronous circuit; a circuit satisfying this discipline throughout is guaranteed to operate correctly regardless of the specific (within-bound) delay and skew values any individual path happens to have — the direct timing-domain analogue of 6.002 Topic 04's voltage-domain static discipline.
- **Skew budget** — for a given path, the maximum clock skew magnitude that path's hold margin (or setup margin) can absorb before the corresponding constraint is violated; derived and computed concretely in Worked Example 2.

## Intuition

**Clock skew makes the "shared clock edge" a convenient fiction that this note stops relying on.** Topic 13's proof and Topic 14's inequalities both implicitly treated "the rising edge" as a single, shared instant every flip-flop in a clock domain experiences simultaneously. Physically, a clock signal is generated at one point and has to propagate — through wires with real resistance and capacitance, through buffers needed to drive the fan-out of many flip-flops — to reach each flip-flop, and those propagation paths are essentially never perfectly matched. The "same" logical edge therefore arrives at slightly different real times at different flip-flops; this note names that difference (clock skew) and asks precisely how large a difference the design can tolerate before Topic 14's guarantees break.

**Skew hurts setup and hold in opposite ways, because the two checks are anchored to different pairs of edges.** The setup check (Topic 14, Section 3) relates the *launching* flip-flop's edge $N$ to the *capturing* flip-flop's *next* edge, $N{+}1$ — the worst case for setup is when the capturing flip-flop's clock arrives *earlier* than nominal, shrinking the time window before edge $N{+}1$ even further. The hold check (Topic 14, Section 4) relates the *same, shared* edge $N$ at both flip-flops — the worst case for hold is when the capturing flip-flop's clock arrives *later* than nominal (relative to the launching flip-flop's edge $N$), giving the launching flip-flop's just-updated output *more* time to sneak through the combinational logic and corrupt $D$ before the capturing flip-flop's hold window (measured from *its own*, now-later edge) has even started. Because these two checks use opposite-direction worst cases, a single skew magnitude bound $t_{skew,\max}$ ends up *subtracting* margin from *both* checks when each is analyzed independently for its own worst case — this is not a contradiction, it is simply that "worst case for setup" and "worst case for hold" are different physical scenarios, and a real, unknown-sign skew could be either one on any given pair of wires.

**"Static timing discipline" is 6.002 Topic 04's static discipline, translated from volts to nanoseconds.** That note's static discipline was a pair of worst-case inequalities ($V_{OL}\le V_{IL}$, $V_{OH}\ge V_{IH}$) guaranteeing that as long as every gate's guaranteed output band and required input band satisfied those bounds, a whole chain of gates would correctly interpret bits regardless of the exact analog voltage any individual signal happened to sit at. This note's static timing discipline plays the identical structural role one level up: as long as every timing path in a circuit satisfies the (skew-adjusted) setup and hold bounds, the whole synchronous system is guaranteed to correctly capture and propagate bits from cycle to cycle, regardless of the exact delay and skew values any individual path happens to have — "static" in both cases refers to using fixed, worst/best-case *bounds* rather than tracking exact, time-varying analog or timing detail.

## Derivation / formalism

### 1. Formalizing clock skew as a bounded, sign-unknown quantity

Let $t_{launch}$ be the arrival time of a given clock edge (call it edge $N$) at the launching flip-flop, and let $t_{skew}$ be defined as the (signed) difference between the capturing flip-flop's clock arrival time and the launching flip-flop's, for the *same* physical clock signal reaching both: $t_{skew} \triangleq t_{\text{clock arrival at capturing FF}} - t_{\text{clock arrival at launching FF}}$, evaluated at corresponding edges. A design specifies only a bound $|t_{skew}| \le t_{skew,\max}$ (from the physical layout's known worst-case wire/buffer delay mismatch), not the exact value or sign for every flip-flop pair — different pairs of flip-flops in the same circuit can have different actual skew, and even the sign is generally not controlled by the designer without deliberate effort. Consequently, any constraint that must hold *for every pair of flip-flops in the circuit* must hold for the *worst-case* value of $t_{skew}$ within $[-t_{skew,\max}, +t_{skew,\max}]$ — and, as Section 2 and Section 3 show, the worst-case *direction* differs between the setup and hold checks.

### 2. Re-deriving the setup constraint with skew

Setup (Topic 14, Section 3) relates the launching flip-flop's edge $N$ (arriving at $t_{launch}$) to the capturing flip-flop's *next* edge, nominally at $t_{launch}+T$. With skew, the capturing flip-flop's actual edge-$(N{+}1)$ arrival time is $t_{capture} = t_{launch} + T + t_{skew}$ (the skew term shifts the capturing edge earlier or later relative to the nominal, exactly as defined in Section 1). Data at $D$ is guaranteed stable (worst case) by $t_{launch}+t_{pd,FF}+t_{pd,\text{logic}}$ (Topic 14, Section 3's derivation, unchanged). Requiring this to occur at least $t_{setup}$ before the *actual* capturing edge:
$$t_{launch}+t_{pd,FF}+t_{pd,\text{logic}} \ \le\ t_{capture} - t_{setup} = t_{launch}+T+t_{skew}-t_{setup}$$
Canceling $t_{launch}$ and solving for $T$:
$$T \ \ge\ t_{pd,FF}+t_{pd,\text{logic}}+t_{setup} - t_{skew}$$
The *worst case* (the value of $t_{skew}$ that makes the right-hand side largest, i.e. hardest to satisfy) is the most negative allowed skew, $t_{skew}=-t_{skew,\max}$ (the capturing flip-flop's clock arriving *earlier* than nominal, per the Intuition section's reasoning):
$$\boxed{T \ \ge\ t_{pd,FF}+t_{pd,\text{logic}}+t_{setup} + t_{skew,\max}}$$
This is the **skew-adjusted setup constraint**. Setting $t_{skew,\max}=0$ recovers Topic 14, Section 3's original inequality exactly, confirming this is a strict generalization, not a replacement.

### 3. Re-deriving the hold constraint with skew

Hold (Topic 14, Section 4) relates the *same* physical edge $N$ at both flip-flops — the launching flip-flop's $Q$ starts possibly changing (best case) at $t_{launch}+t_{cd,FF}$, propagating through the combinational logic (best case) to reach $D$ no earlier than $t_{launch}+t_{cd,FF}+t_{cd,\text{logic}}$. The capturing flip-flop's *own* copy of this same edge $N$ arrives at $t_{launch}+t_{skew}$ (by Section 1's definition, applied to edge $N$ itself rather than edge $N{+}1$), and its hold requirement is that $D$ must remain stable until $t_{skew}$-shifted-edge-time-plus-$t_{hold}$, i.e. until $t_{launch}+t_{skew}+t_{hold}$. Requiring $D$'s earliest possible change to occur no sooner than this:
$$t_{launch}+t_{cd,FF}+t_{cd,\text{logic}} \ \ge\ t_{launch}+t_{skew}+t_{hold}$$
Canceling $t_{launch}$:
$$t_{cd,FF}+t_{cd,\text{logic}} \ \ge\ t_{hold}+t_{skew}$$
The *worst case* here (the value of $t_{skew}$ making the right-hand side largest, hardest to satisfy) is the most *positive* allowed skew, $t_{skew}=+t_{skew,\max}$ (the capturing flip-flop's clock arriving *later* than the launching flip-flop's, per the Intuition section) — the opposite-signed extreme from Section 2's setup case:
$$\boxed{t_{cd,FF}+t_{cd,\text{logic}} \ \ge\ t_{hold}+t_{skew,\max}}$$
This is the **skew-adjusted hold constraint**. As in Section 2, setting $t_{skew,\max}=0$ recovers Topic 14, Section 4's original inequality exactly. Notice this constraint still contains **no** $T$ term — Topic 14's observation that hold violations cannot be fixed by a slower clock remains true even with skew, since $T$ never entered either derivation above.

### 4. The static timing discipline, and its parallel to 6.002's static discipline

**Recap of 6.002 Topic 04's static discipline (since that note is not yet `done`, per this vault's zero-assumed-knowledge rule):** that note defined four voltage thresholds — $V_{OL}$ (max guaranteed output for a driven logic 0), $V_{OH}$ (min guaranteed output for a driven logic 1), $V_{IL}$ (max input a gate is guaranteed to still read as 0), $V_{IH}$ (min input a gate is guaranteed to still read as 1) — and required $V_{OL}\le V_{IL}$ and $V_{OH}\ge V_{IH}$ for correct, cascadable digital operation. The key structural feature: these are *worst-case* bounds (guaranteed output ranges, required input ranges), and satisfying the two inequalities guarantees correct bit interpretation *regardless* of the exact analog voltage any real signal happens to take, as long as it stays within its guaranteed band.

**This note's static timing discipline** is the pair of boxed inequalities from Section 2 and Section 3, required to hold simultaneously for *every* launching/capturing flip-flop pair (i.e., every timing path) in a synchronous circuit:
$$T \ge t_{pd,FF}+t_{pd,\text{logic}}+t_{setup}+t_{skew,\max} \qquad\text{and}\qquad t_{cd,FF}+t_{cd,\text{logic}} \ge t_{hold}+t_{skew,\max}$$
Exactly like 6.002's static discipline, these are worst-case bounds (using worst-case $t_{pd}$ for setup, best-case $t_{cd}$ for hold, and the worst-case *direction* of skew for each), and satisfying both, for every path, guarantees correct cycle-by-cycle operation of the whole synchronous circuit *regardless* of the exact delay and skew values any individual gate, flip-flop, or clock wire happens to have — the designer never needs to know the precise values, only that they stay within the specified worst-case bounds used in the inequalities. This is the payoff of building a *discipline* (a fixed, checkable set of rules) rather than requiring an exact, instant-by-instant simulation of every possible circuit configuration.

## Worked examples

### Example 1 — Skew exposes a hidden hold violation in Topic 14 Example 2's circuit

Recall Topic 14 Example 2's circuit and numbers: $t_{pd,FF}=0.3\ \text{ns}$, $t_{cd,FF}=0.15\ \text{ns}$, $t_{setup}=0.15\ \text{ns}$, $t_{hold}=0.1\ \text{ns}$; two next-state paths, $D_1$ (one AND gate: $t_{pd}=0.2\ \text{ns}, t_{cd}=0.1\ \text{ns}$) and $D_0$ (direct wire: $t_{pd}=0, t_{cd}=0$). Without skew, Topic 14 found both setup (at $T=1\ \text{ns}$) and hold satisfied for both paths, with hold on the $D_0$ path being the tightest case ($0.15\ \text{ns} \ge 0.10\ \text{ns}$, only $0.05\ \text{ns}$ margin).

**Introduce $t_{skew,\max}=0.1\ \text{ns}$ and re-check using Section 2/3's boxed inequalities.**

**Setup, $D_1$ path (the binding path for setup, per Topic 14):** $T \ge 0.3+0.2+0.15+0.1 = 0.75\ \text{ns}$. At $T=1\ \text{ns}$: $1 \ge 0.75$ — still satisfied, though the slack has shrunk from $0.35\ \text{ns}$ (no skew) to $0.25\ \text{ns}$.

**Hold, $D_0$ path (the binding path for hold, per Topic 14):** required $t_{cd,FF}+t_{cd,\text{logic}} \ge t_{hold}+t_{skew,\max}$, i.e. $0.15+0 \ge 0.1+0.1=0.2$. This gives $0.15 \ge 0.2$ — **false**. **The hold constraint is now violated**, purely because of clock skew, even though the exact same circuit and clock period passed every check under Topic 14's zero-skew analysis. This is a direct, numeric demonstration of why Topic 13's Common Pitfalls flagged clock-buffering mismatches as a real hazard: a design that looks completely correct under an idealized shared-clock assumption can fail once real, unavoidable clock skew is accounted for — and, per Topic 14 Section 4's own observation (still true here, since Section 3's hold inequality has no $T$ term), this specific violation **cannot be fixed by slowing the clock down**; only reducing the skew itself, or adding deliberate extra delay to the $D_0$ path, could resolve it.

### Example 2 — Computing a path's skew budget

For a given path, the **skew budget** is the largest $t_{skew,\max}$ that path's hold margin can absorb before Section 3's inequality breaks — found by solving the boxed hold inequality for $t_{skew,\max}$ with equality (the break-even point):
$$t_{skew,\max}^{\text{(budget)}} = t_{cd,FF}+t_{cd,\text{logic}} - t_{hold}$$

**For the $D_0$ path** (Example 1's numbers, $t_{cd,\text{logic}}=0$): $t_{skew,\max}^{\text{(budget)}} = 0.15+0-0.1 = 0.05\ \text{ns}$. This path can tolerate at most $0.05\ \text{ns}$ of clock skew before its hold constraint breaks — confirming Example 1's finding: the applied skew there, $0.1\ \text{ns}$, exceeds this path's $0.05\ \text{ns}$ budget (by exactly $0.05\ \text{ns}$), which is precisely why Example 1 found a violation. **For the $D_1$ path** ($t_{cd,\text{logic}}=0.1\ \text{ns}$): $t_{skew,\max}^{\text{(budget)}} = 0.15+0.1-0.1=0.15\ \text{ns}$ — a much larger budget, three times the $D_0$ path's, consistent with $D_1$'s path having *more* combinational delay (a slower, not a faster, path) providing *more* of a buffer against a same-edge race — directly illustrating why hold-time problems in real designs are characteristically associated with the *fastest* (often close to zero-delay) paths in a circuit, exactly the ones with the smallest skew budget.

## Common pitfalls

- **Applying the same worst-case skew direction to both setup and hold.** Section 2 and Section 3 use *opposite*-signed worst cases ($-t_{skew,\max}$ for setup, $+t_{skew,\max}$ for hold) because the two checks are anchored to different pairs of edges (launch-edge-$N$-to-capture-edge-$(N{+}1)$ for setup; shared edge $N$ for hold) — applying the same sign to both either overstates one check's difficulty or understates the other's.
- **Assuming a slower clock fixes a skew-induced hold violation.** Exactly as in Topic 14, Section 3's boxed inequality (setup) is the only one with a $T$ term; Section 3's boxed inequality (hold) has none, skew or no skew — Example 1's violation is present at *every* clock period, not just $T=1\ \text{ns}$.
- **Treating clock skew as always harmful.** This note conservatively bounds skew's magnitude and takes the worst case for each check independently, which is the safe, standard design approach — but real designs sometimes deliberately introduce **useful skew** (intentionally delaying one flip-flop's clock relative to another) to *relax* a tight setup constraint at the cost of tightening hold, or vice versa; this note only develops the conservative worst-case-both-ways analysis, not the deliberate-skew-insertion technique, which is beyond this note's scope.
- **Forgetting the skew budget (Example 2) is specific to one path, not a single circuit-wide number.** Different paths have different $t_{cd,\text{logic}}$ values and therefore different skew budgets (Example 2 found $0.05\ \text{ns}$ for one path and $0.15\ \text{ns}$ for another in the *same* circuit) — a circuit's true tolerable skew is the *minimum* skew budget over all its paths, exactly mirroring Topic 14 Section 1's "circuit delay is the max/min over all paths" framing.
- **Conflating the static timing discipline with the static (voltage) discipline from 6.002.** They are structurally analogous (both are worst-case-bound inequality packages guaranteeing correctness regardless of exact within-bound values) but govern entirely different physical quantities (nanoseconds vs. volts) and are checked independently — satisfying one gives no information about the other; a circuit needs both disciplines satisfied to work correctly in practice, since Topics 03–13's whole framework already assumed the voltage-domain digital abstraction underneath everything this note's timing analysis is built on.
- **Believing zero clock skew is achievable in a real circuit and this note's machinery is therefore optional.** Physical clock distribution to many flip-flops (via necessarily-imperfectly-matched wires and buffers) makes some nonzero $t_{skew,\max}$ essentially unavoidable in any real chip; treating skew as negligible without actually verifying $t_{skew,\max}$ against a specific design's skew budgets (Example 2) is exactly the mistake Example 1 was constructed to warn against.

## Self-check

### Questions

1. (Easy) Define clock skew, in one sentence, for a launching/capturing flip-flop pair.
2. (Easy) True or false: the skew-adjusted hold constraint contains a $T$ (clock period) term. Justify in one sentence, citing Section 3.
3. (Medium) Using Section 2's boxed setup inequality, compute the required minimum clock period for $t_{pd,FF}=0.25\ \text{ns}$, $t_{pd,\text{logic}}=0.3\ \text{ns}$, $t_{setup}=0.1\ \text{ns}$, $t_{skew,\max}=0.05\ \text{ns}$.
4. (Medium) Using Section 3's boxed hold inequality, determine whether $t_{cd,FF}=0.2\ \text{ns}$, $t_{cd,\text{logic}}=0.05\ \text{ns}$, $t_{hold}=0.15\ \text{ns}$, $t_{skew,\max}=0.05\ \text{ns}$ satisfies hold.
5. (Medium) Explain, in your own words, why the worst-case skew direction for setup (capturing clock early) is the *opposite* of the worst-case skew direction for hold (capturing clock late).
6. (Hard) A path has $t_{cd,FF}=0.1\ \text{ns}$, $t_{cd,\text{logic}}=0.05\ \text{ns}$, $t_{hold}=0.1\ \text{ns}$. Compute its skew budget (Example 2's formula), and state whether a measured $t_{skew,\max}=0.08\ \text{ns}$ on this specific path satisfies hold.
7. (Hard) Explain why a path with *more* combinational logic delay in its short-path ($t_{cd,\text{logic}}$) tends to have a *larger* skew budget, using Section 3's formula — and explain why this means the fastest paths in a circuit (e.g. a bare wire) are generally the ones most vulnerable to skew-induced hold violations, connecting to Example 1 and Example 2's specific numeric results.
8. (Hard) Prove that setting $t_{skew,\max}=0$ in both of Section 2 and Section 3's boxed inequalities exactly reproduces Topic 14, Section 3 and Section 4's original (unskewed) inequalities — i.e., confirm this note's results are a strict generalization, not a different claim.

### Answers

1. Clock skew is the difference in arrival time of the (nominally same) clock edge at two different flip-flops (a launching flip-flop and a capturing flip-flop), caused by unequal delay along the clock distribution paths reaching each one.
2. False. Per Section 3, the skew-adjusted hold constraint is $t_{cd,FF}+t_{cd,\text{logic}}\ge t_{hold}+t_{skew,\max}$ — no $T$ appears anywhere in this inequality, exactly as in Topic 14's original (unskewed) hold constraint.
3. $T \ge 0.25+0.3+0.1+0.05 = 0.70\ \text{ns}$.
4. Required: $t_{cd,FF}+t_{cd,\text{logic}} \ge t_{hold}+t_{skew,\max}$, i.e. $0.2+0.05=0.25 \ge 0.15+0.05=0.20$. Since $0.25\ge0.20$, hold is satisfied, with $0.05\ \text{ns}$ of margin.
5. Setup (Section 2) measures whether data has enough time to arrive *before* the capturing flip-flop's *next* edge — if that edge arrives *earlier* than nominal (negative skew, by this note's sign convention), there is *less* time for the data to arrive in time, which is the worse case. Hold (Section 3) measures whether data stays stable long enough *after* the *same, shared* edge at both flip-flops — if the capturing flip-flop's copy of that shared edge arrives *later* than the launching flip-flop's (positive skew), the launching flip-flop's output has *more* extra time to start changing and corrupt the data before the capturing flip-flop's (now-later) hold window even begins, which is the worse case there. The two checks reference different pairs of edges (launch-$N$-to-capture-$(N{+}1)$ vs. shared edge $N$), which is exactly why the "bad" skew direction differs between them.
6. Skew budget $= t_{cd,FF}+t_{cd,\text{logic}}-t_{hold} = 0.1+0.05-0.1=0.05\ \text{ns}$. A measured $t_{skew,\max}=0.08\ \text{ns}$ exceeds this $0.05\ \text{ns}$ budget, so hold is **violated** on this path (equivalently, checking Section 3's inequality directly: $0.1+0.05=0.15 \ge 0.1+0.08=0.18$? No — $0.15<0.18$, confirming the violation).
7. Section 3's formula, $t_{skew,\max}^{\text{(budget)}}=t_{cd,FF}+t_{cd,\text{logic}}-t_{hold}$, shows the budget grows linearly with $t_{cd,\text{logic}}$ — a path with more combinational delay in its fastest (contamination-delay) case has a proportionally larger budget, since there's simply more "buffer time" between when the launching flip-flop's output could earliest start changing and when it could earliest reach $D$. A bare wire (Example 1/2's $D_0$ path) has $t_{cd,\text{logic}}=0$, the smallest possible value, giving the smallest possible budget ($0.05\ \text{ns}$ in that concrete case, versus $0.15\ \text{ns}$ for the AND-gate path) — meaning the fastest paths in a circuit, precisely because they offer the *least* delay to absorb any skew, are the ones a real design needs to scrutinize most carefully for hold violations, exactly what Example 1's violation (on the zero-delay $D_0$ path specifically, not the slower $D_1$ path) demonstrated concretely.
8. Section 2's boxed inequality: $T\ge t_{pd,FF}+t_{pd,\text{logic}}+t_{setup}+t_{skew,\max}$. Setting $t_{skew,\max}=0$: $T\ge t_{pd,FF}+t_{pd,\text{logic}}+t_{setup}+0 = t_{pd,FF}+t_{pd,\text{logic}}+t_{setup}$ — identical to Topic 14, Section 3's original inequality. Section 3's boxed inequality: $t_{cd,FF}+t_{cd,\text{logic}}\ge t_{hold}+t_{skew,\max}$. Setting $t_{skew,\max}=0$: $t_{cd,FF}+t_{cd,\text{logic}}\ge t_{hold}+0=t_{hold}$ — identical to Topic 14, Section 4's original inequality. Both reduce exactly, confirming this note's skew-adjusted constraints are strict generalizations of Topic 14's (recovering them exactly as the special case of zero skew), not separate or conflicting claims. $\blacksquare$

## Summary / cheat sheet

**Clock skew:** $t_{skew}$, the (sign-unknown, magnitude-bounded by $t_{skew,\max}$) difference in clock arrival time between a launching and capturing flip-flop.

**Skew-adjusted setup (Section 2):** $T\ge t_{pd,FF}+t_{pd,\text{logic}}+t_{setup}+t_{skew,\max}$ — worst case is capturing clock early.

**Skew-adjusted hold (Section 3):** $t_{cd,FF}+t_{cd,\text{logic}}\ge t_{hold}+t_{skew,\max}$ — worst case is capturing clock late; still no $T$ term, so unfixable by a slower clock.

**Skew budget (per path):** $t_{skew,\max}^{\text{budget}}=t_{cd,FF}+t_{cd,\text{logic}}-t_{hold}$ — fast (low-$t_{cd,\text{logic}}$) paths have the smallest budget and are most hold-vulnerable.

**Static timing discipline:** both boxed inequalities, satisfied for every path in the circuit — the timing-domain structural analogue of 6.002's static (voltage) discipline: fixed worst-case bounds guaranteeing correctness regardless of exact within-bound values.

## Used later in
(none yet)
