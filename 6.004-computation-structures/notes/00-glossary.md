# 6.004 Glossary

Running list of terms/symbols introduced in 6.004 notes, in order of first appearance.

## From [[6.004-computation-structures/notes/01-boolean-algebra-logic-functions]]

- **Bit (Boolean value)** — an element of $\{0,1\}$; the abstract, voltage-free version of a digital-abstraction logic level.
- **Boolean variable** — a symbol standing for an unspecified bit.
- **Logic function (Boolean function)** — a function $f:\{0,1\}^n\to\{0,1\}$.
- **Truth table** — explicit tabulation of a logic function's output over all $2^n$ input combinations.
- **AND ($X\cdot Y$)** — outputs 1 iff both inputs are 1.
- **OR ($X+Y$)** — outputs 1 iff at least one input is 1.
- **NOT ($\bar X$ / $X'$)** — flips the bit.
- **Literal** — a variable or its complement.
- **Boolean expression** — built from variables, $0$/$1$, AND/OR/NOT; precedence NOT $>$ AND $>$ OR.
- **Logically equivalent expressions** — identical truth tables (same function), written with $=$.
- **Perfect induction** — proof by checking all $2^n$ truth-table rows; valid because the domain is finite.
- **Boolean algebra (Huntington's postulates)** — $\{0,1\}$ with AND/OR/NOT satisfying closure, identity, commutativity, distributivity (both directions), and complement.
- **Duality principle** — any valid law stays valid under simultaneous $+\leftrightarrow\cdot$, $0\leftrightarrow1$, because the postulates come in mirror pairs.
- **Dominance/null law** — $x+1=1$, $x\cdot0=0$.
- **Idempotent law** — $x+x=x$, $x\cdot x=x$.
- **Absorption law** — $x+xy=x$, $x(x+y)=x$.
- **De Morgan's laws** — $\overline{X+Y}=\bar X\bar Y$; $\overline{XY}=\bar X+\bar Y$.

## From [[6.004-computation-structures/notes/02-canonical-forms-simplification]]

- **Minterm ($m_i$)** — a product term with every variable exactly once, uncomplemented iff its bit in index $i$ is 1; $1$ at exactly one row.
- **Maxterm ($M_i$)** — a sum term with every variable exactly once, uncomplemented iff its bit in index $i$ is 0 (opposite of minterm convention); $0$ at exactly one row; $M_i=\overline{m_i}$.
- **Canonical SOP (minterm expansion)** — $F=\Sigma m(\text{indices where }F{=}1)$; direct translation of a truth table into an OR of minterms.
- **Canonical POS (maxterm expansion)** — $F=\Pi M(\text{indices where }F{=}0)$; direct translation into an AND of maxterms.
- **(General) sum-of-products / product-of-sums** — OR of AND terms / AND of OR terms, not necessarily full minterms/maxterms.
- **Karnaugh map (K-map)** — grid of truth-table rows in Gray-code order so grid-adjacent cells (with wraparound) differ in exactly one variable.
- **Gray code** — ordering of binary numbers where consecutive entries differ in exactly one bit.
- **Implicant** — a product term that is 1 only where $F$ is 1 (never covers a 0-row).
- **Prime implicant** — an implicant that can't be enlarged (no literal droppable) while remaining valid; largest valid K-map group containing a given cell.
- **Essential prime implicant** — the only prime implicant covering some particular minterm; must appear in every minimal SOP.
- **Don't-care ($d$/$\times$)** — an input row with unspecified required output; assignable to 0 or 1 freely during minimization.

## From [[6.004-computation-structures/notes/03-cmos-switches-mosfet-abstraction]]

- **NMOS (n-channel enhancement-mode MOSFET)** — three-terminal device (gate $G$, drain $D$, source $S$); switch-level model: closed iff $v_{GS}=v_G-v_S \ge V_{Tn}$ ($V_{Tn}>0$); $i_G=0$.
- **$V_{Tn}$, $V_{Tp}$** — NMOS/PMOS threshold voltages; refines the single $V_T$ used in 6.002 Topic 04 (single-device-type note) now that two device types are in play. $V_{Tn}>0$, $V_{Tp}<0$.
- **PMOS (p-channel enhancement-mode MOSFET)** — three-terminal device, same $v_{GS}$ definition as NMOS; switch-level model: closed iff $v_{GS} \le V_{Tp}$ ($V_{Tp}<0$); $i_G=0$.
- **CMOS (complementary MOS)** — circuits using NMOS and PMOS together, exploiting their opposite on-conditions for the same gate voltage.
- **Ideal switch abstraction** — further idealization of the switch-level model: closed = zero resistance, open = infinite resistance, state set purely by a Boolean control bit (no $R_{ON}$); used for connectivity ("is there a path") reasoning, not analog/delay analysis.
- **Control literal** — the condition (variable or its complement) under which a switch is closed; NMOS driven by bit $X$ with grounded source contributes literal $X$, PMOS driven by $X$ with source at $V_{DD}$ contributes literal $\bar X$.
- **Connectivity function $C(x_1,\dots,x_n)$** — the Boolean function equal to 1 exactly when a switch network provides a conducting path between its two terminals.
- **Series composition** — switches/subnetworks chained through a shared intermediate node; connectivity function is the AND of the individual conditions ($C_1\cdot C_2$).
- **Parallel composition** — switches/subnetworks both connecting the same two terminals; connectivity function is the OR of the individual conditions ($C_1+C_2$).

## From [[6.004-computation-structures/notes/04-static-cmos-gates]]

- **Output node** — the shared node whose voltage is a static CMOS gate's output signal, distinct from the fixed rails $V_{DD}$/$GND$.
- **Pull-down network (PDN)** — an NMOS-only switch network connecting the output node to ground; its connectivity function $F_{PD}$ is the condition under which the output is pulled to logic 0.
- **Pull-up network (PUN)** — a PMOS-only switch network connecting the output node to $V_{DD}$; its connectivity function $F_{PU}$ is the condition under which the output is pulled to logic 1.
- **Static CMOS gate** — one PDN + one PUN sharing an output node and input set, satisfying $F_{PU}=\overline{F_{PD}}$ (the complementary-network condition); output $=\overline{F_{PD}}$.
- **Complementary-network condition** — exactly one of \{PUN, PDN\} conducts for every input; rules out floating and contention.
- **Floating node** — a node connected to neither rail (both networks open); undefined voltage.
- **Contention / crowbar condition** — a node connected to both rails at once (both networks conducting); forbidden in a valid static gate.
- **CMOS inverter** — PDN $=$ one NMOS($A$), PUN $=$ one PMOS($A$); output $=\bar A$.
- **NAND gate** — PDN $=$ series NMOS (AND), PUN $=$ parallel PMOS (dual); output $=\overline{AB\cdots}$.
- **NOR gate** — PDN $=$ parallel NMOS (OR), PUN $=$ series PMOS (dual); output $=\overline{A+B+\cdots}$.
- **Complex gate** — a static CMOS gate whose PDN is an arbitrary series-parallel AND/OR tree, PUN built as its exact dual.
- **Dual network** — given a series-parallel network, the network with every series$\leftrightarrow$parallel swapped and every literal complemented (device type swapped NMOS$\leftrightarrow$PMOS); proved (by structural induction + De Morgan's) to compute the complement of the original network's connectivity function.

## From [[6.004-computation-structures/notes/05-multiplexers-decoders-encoders]]

- **Multiplexer (MUX) / $2^n$-to-1 MUX** — routes one of $2^n$ data inputs $D_0,\dots,D_{2^n-1}$ to a single output, chosen by $n$ select lines; $F=\sum_i m_i(S)\cdot D_i$.
- **Select lines** — the $n$ control inputs $S_{n-1},\dots,S_0$ of a MUX encoding which data input to route.
- **Transmission gate (pass gate)** — NMOS gated by $C$ + PMOS gated by $\bar C$, both between the same two terminals; closes for $C=1$ regardless of which terminal is driving/what data value is present, unlike a lone NMOS/PMOS (whose simple rule assumed a fixed-rail source, Topic 03).
- **Binary decoder / $n$-to-$2^n$ decoder** — $2^n$ outputs, exactly one ($Y_i=m_i$) is 1 for any input, matching Topic 02's minterm.
- **One-hot encoding** — a vector of $2^n$ bits with exactly one bit ($=$ the represented index) set to 1.
- **Binary encoder / $2^n$-to-$n$ encoder** — inverse of a decoder: given a one-hot input (precondition), outputs the binary index of the active bit; undefined if the precondition fails.

## From [[6.004-computation-structures/notes/06-adders-basic-arithmetic]]

- **Binary place-value number** — $n$-bit unsigned integer $A_{n-1}\cdots A_0$ representing $\sum_i A_i2^i$; $A_0$ is the LSB, $A_{n-1}$ the MSB.
- **Half adder** — 2-input ($A,B$), 2-output ($S,C_{out}$) circuit computing $A+B$; $S=A\oplus B$, $C_{out}=AB$. No carry-in.
- **XOR ($X\oplus Y$)** — $X\bar Y+\bar XY$; outputs 1 iff inputs differ. $X\oplus 0 = X$.
- **Full adder** — 3-input ($A,B,C_{in}$), 2-output ($S,C_{out}$) circuit computing $A+B+C_{in}$; $S=A\oplus B\oplus C_{in}$, $C_{out}=AB+BC_{in}+AC_{in}$ (majority function). Built from 2 half adders + 1 OR gate.
- **Ripple-carry adder** — $n$ full adders chained $C_{out}^{(i)}\to C_{in}^{(i+1)}$, $C_{in}^{(0)}=0$; correct by induction on bit position.
- **Overflow (unsigned)** — the final full adder's $C_{out}$ ($=C_n$) signals the true sum exceeded $2^n-1$.
- **Propagation delay of a ripple-carry chain** — informally introduced (full treatment deferred to Topic 22): worst-case time for a ripple-carry adder's output to become valid, dominated by the carry rippling sequentially through all $n$ full adders in the worst case.

## From [[6.004-computation-structures/notes/07-comparators-intro-alus]]

- **Two's complement representation** — $n$-bit signed encoding where $A_{n-1}\cdots A_0$ represents $-A_{n-1}2^{n-1}+\sum_{i=0}^{n-2}A_i2^i$; MSB doubles as sign bit.
- **Two's complement negation** — $-A = \overline A + 1 \pmod{2^n}$ ("invert and add one").
- **Subtractor (via adder reuse)** — computes $A-B$ as $A+\overline B+1$: adder fed $\overline B$ with $C_{in}^{(0)}=1$; same hardware as the Topic 06 adder.
- **Comparator** — circuit producing $A{=}B$/$A{<}B$/$A{>}B$ as single-bit outputs for $n$-bit $A,B$.
- **Zero-detect** — $n$-input NOR signaling "every bit is 0"; used for equality via $\overline{\bigvee_i(A_i\oplus B_i)}$.
- **Arithmetic logic unit (ALU)** — combinational circuit computing one of several functions of $A,B$, selected by a $k$-bit opcode via a MUX; all candidate results computed in parallel, MUX selects the exposed one.
- **Bit-slice** — one 1-bit-wide vertical slice of an $n$-bit-wide ALU/adder, replicated $n$ times with carries rippling between slices.

## From [[6.004-computation-structures/notes/08-bistability-sr-latch]]

- **Feedback (digital)** — wiring a gate's output back to (an input feeding) its own input, so the circuit's output depends on its own history.
- **State** — the current value(s) on a feedback circuit's internal node(s); not determined by external inputs alone.
- **Stable state (equilibrium)** — a self-consistent assignment: evaluating every gate with these values as inputs reproduces the same values.
- **Bistable circuit** — a feedback circuit with 2 distinct stable states, settable via external inputs and held after those inputs are released.
- **Metastable state** — a non-ideal equilibrium unstable to infinitesimal perturbation; real (not ideal-gate-model) phenomenon, unresolved until Topic 23.
- **Cross-coupled gates** — two 2-input gates wired output-to-input in a loop (NOR-NOR or NAND-NAND), forming a 2-gate feedback loop.
- **SR latch (set-reset latch)** — bistable circuit, inputs $S,R$, outputs $Q,\overline Q$; NOR version active-high ($S{=}R{=}0$ hold, $01$ reset, $10$ set, $11$ forbidden); NAND version active-low (polarities flipped).
- **Forbidden / restricted input combination** — the input combination breaking $Q=\overline{\overline Q}$ and/or causing unpredictable release behavior ($S{=}R{=}1$ for NOR, $S{=}R{=}0$ for NAND).

## From [[6.004-computation-structures/notes/09-d-latch-transparency-problem]]

- **D latch (data / transparent latch)** — one data input $D$, one control $\text{CLK}$; while $\text{CLK}=1$, $Q$ tracks $D$ continuously; while $\text{CLK}=0$, $Q$ holds. Built via $S=D\cdot\text{CLK}$, $R=\overline D\cdot\text{CLK}$ into an SR latch (forbidden state structurally unreachable), or via transmission gates + cross-coupled inverters.
- **Level-sensitive** — behavior governed by the sustained level of the control signal, not a single instant.
- **Transparency** — while enabled, output continuously follows every change of the input in real time, not just eventually.
- **Transparency problem (race-through)** — chaining two same-clock (or same-enable-condition) latches data-out-to-data-in lets a change propagate through both within one enabled interval, collapsing the intended one-stage delay ($Q_2=Q_1=D_1$ throughout the shared interval).
- **Setup time / hold time** — informally flagged interval around a capture instant during which $D$ must stay stable; formalized in Topic 22.

## From [[6.004-computation-structures/notes/10-edge-triggered-d-flip-flop-master-slave]]

- **Edge-triggered device** — output changes only at a clock transition (an edge), not throughout a sustained level; contrast level-sensitive (Topic 09).
- **Rising edge / positive edge** — the instant $\text{CLK}$ transitions $0\to1$.
- **Falling edge / negative edge** — the instant $\text{CLK}$ transitions $1\to0$.
- **Positive-edge-triggered D flip-flop** — $Q$ takes $D$'s value at the most recent rising edge, holds otherwise (even between edges if $D$ changes).
- **Master-slave construction** — two D latches in series (master, slave) driven by complementary clock phases ($\overline{\text{CLK}}$, $\text{CLK}$ for positive-edge triggering; swapped for negative-edge) so they're never simultaneously transparent; the standard implementation of an edge-triggered flip-flop.
- **Characteristic equation** — $Q^+=D$: the flip-flop's next state equals $D$ at the triggering edge.

## From [[6.004-computation-structures/notes/11-moore-mealy-machines]]

- **Finite state machine (FSM)** — 5-tuple $(S,I,O,\delta,\lambda)$ with initial state $s_0$: finite states $S$, input alphabet $I$, output alphabet $O$, next-state function $\delta:S\times I\to S$, output function $\lambda$ (signature differs by flavor).
- **Present state / next state** — $s$ (current) vs. $s^+=\delta(s,x)$ (next), generalizing $Q^+=D$ to a function of state and input.
- **State diagram** — directed graph: nodes = states, edges = transitions labeled by input (and output, for Mealy).
- **State transition table** — tabular form of $\delta$ (and $\lambda$); losslessly equivalent to the state diagram.
- **Moore machine** — $\lambda:S\to O$; output depends on state alone; states labeled with output.
- **Mealy machine** — $\lambda:S\times I\to O$; output depends on state and current input; transitions labeled with output.
- **Synchronous FSM** — all state updates happen simultaneously, once per clock edge; requires edge-triggered storage (Topic 10) to be well-defined.

## From [[6.004-computation-structures/notes/12-fsm-design-procedure]]

- **State variable** — one of the $k$ bits $Q_{k-1},\dots,Q_0$ encoding which abstract state an FSM occupies.
- **State encoding (state assignment)** — injective function $\text{code}:S\to\{0,1\}^k$, $k=\lceil\log_2|S|\rceil$, assigning each state a distinct code word.
- **Unused code word** — a $k$-bit pattern with $2^k>|S|$ left unassigned; treated as a don't-care row in every next-state/output truth table.
- **Encoded transition table** — Topic 11's state table with symbolic states replaced by code words, split into bit columns; an ordinary multi-output truth table.
- **Next-state equations** — one minimized Boolean expression per state variable, $Q_i^+=f_i(Q_{k-1},\dots,Q_0,x_1,\dots,x_m)$, derived via Topic 02 K-map minimization.
- **Output equations** — analogous minimized expressions for each output bit; Moore equations depend on state bits only, Mealy equations may depend on state and input bits.
- **$Z$** — this note's symbol for a single output bit, used throughout its worked examples' encoded transition tables.

## From [[6.004-computation-structures/notes/13-implementing-fsms-flip-flops]]

- **State register** — a bank of $k$ D flip-flops (one per state variable) sharing one clock, holding the FSM's current encoded state.
- **Next-state logic** — combinational circuit computing each $D_i=Q_i^+=f_i(Q,x)$ from state register outputs (fed back) and external input.
- **Output logic** — combinational circuit computing output bits; Moore version taps state register outputs only (no input wire); Mealy version taps state and input.
- **General synchronous FSM architecture** — state register + next-state logic + output logic, one shared clock; the standard implementation of any FSM from Topic 12's equations.
- **Clock domain** — the set of flip-flops sharing one clock signal, updating at exactly the same edges; this architecture uses a single clock domain for the whole state register.

## From [[6.004-computation-structures/notes/14-propagation-delay-setup-hold-time]]

- **Propagation delay ($t_{pd}$)** — worst-case time from a gate/circuit input change to its output being guaranteed to reach its final value.
- **Contamination delay ($t_{cd}$)** — best-case (earliest possible) time from an input change to the output possibly starting to change; $t_{cd}\le t_{pd}$.
- **Uncertain window** — the interval $(t_{cd},t_{pd})$; a statement about timing-model guarantee limits, not a third signal value.
- **Path delay / circuit delay** — path delay $=$ sum of gate delays along a path; circuit $t_{pd}=\max$ over paths (critical path), circuit $t_{cd}=\min$ over paths (short path).
- **Clock-to-$Q$ delay ($t_{pd,FF}$/$t_{cd,FF}$)** — flip-flop analogue of gate delay, from the triggering edge to $Q$ becoming valid (worst case) or possibly starting to change (best case).
- **Setup time ($t_{setup}$)** — minimum time before a triggering edge that $D$ must already be stable.
- **Hold time ($t_{hold}$)** — minimum time after a triggering edge that $D$ must remain stable.

## From [[6.004-computation-structures/notes/15-clock-skew-static-timing-discipline]]

- **Clock skew ($t_{skew}$)** — difference in clock arrival time between a launching and capturing flip-flop, for the same physical clock signal; bounded in magnitude by $t_{skew,\max}$, sign generally unknown/uncontrolled.
- **Launching flip-flop / capturing flip-flop** — source/destination flip-flops of one timing path.
- **Skew-adjusted setup constraint** — $T\ge t_{pd,FF}+t_{pd,\text{logic}}+t_{setup}+t_{skew,\max}$; worst case is capturing clock early.
- **Skew-adjusted hold constraint** — $t_{cd,FF}+t_{cd,\text{logic}}\ge t_{hold}+t_{skew,\max}$; worst case is capturing clock late; no $T$ term.
- **Static timing discipline** — both skew-adjusted inequalities, required for every path in a synchronous circuit; timing-domain analogue of 6.002's voltage-domain static discipline.
- **Skew budget** — $t_{skew,\max}^{\text{budget}}=t_{cd,FF}+t_{cd,\text{logic}}-t_{hold}$; the largest tolerable skew for one path before hold breaks.
