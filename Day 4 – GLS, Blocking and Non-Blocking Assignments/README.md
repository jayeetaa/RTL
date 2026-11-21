
# Day 4: Gate-Level Simulation (GLS), Blocking vs Non-Blocking Assignments and Synthesis-Simulation Mismatch

## Objective

Understand Gate-Level Simulation (GLS), the differences between RTL and gate-level simulation, and the behavior of blocking and non-blocking assignments in Verilog HDL.

The objectives of this session are:

- Understand the concept and flow of Gate-Level Simulation.
- Perform functional GLS using a synthesized SKY130 gate-level netlist.
- Understand synthesis-simulation mismatches caused by incomplete sensitivity lists.
- Study the effect of blocking assignment order on combinational logic.
- Understand blocking and non-blocking assignments in sequential circuits.
- Learn the recommended RTL coding practices for combinational and sequential logic.

---

## 1. Introduction to Gate-Level Simulation (GLS)

### Theory

RTL simulation verifies the design functionality against the given specification using a testbench.

After synthesis, the RTL design is converted into a gate-level netlist consisting of standard cells from the target technology library.

Gate-Level Simulation (GLS) is performed by simulating this synthesized netlist using the same testbench used for RTL simulation.

The main objective is to verify that the functionality of the design is preserved after synthesis.

### 1.1 RTL Simulation vs Gate-Level Simulation

| RTL Simulation | Gate-Level Simulation |
|---|---|
| Uses RTL Verilog code | Uses synthesized gate-level netlist |
| Describes the design using behavioral constructs | Represents the design using standard-cell instances |
| Checks intended RTL functionality | Checks functionality after synthesis |
| Does not require standard-cell models | Requires standard-cell Verilog models |
| Generally faster | Generally slower |

### 1.2 GLS Flow

The general Gate-Level Simulation flow is:

1. Write the RTL design and testbench.
2. Simulate the RTL using Icarus Verilog.
3. Synthesize the RTL using Yosys.
4. Generate the gate-level netlist.
5. Simulate the netlist using standard-cell Verilog models and the same testbench.
6. Compare RTL and gate-level simulation waveforms.

**Note:** Timing-aware GLS uses Standard Delay Format (SDF) annotation to include cell and interconnect delays. This workshop covers functional GLS without SDF-based timing simulation.

---

## 2. Functional GLS – Ternary Operator MUX

The first experiment demonstrates the functional GLS flow using a 2:1 multiplexer implemented with the Verilog ternary operator.

### 2.1 RTL Code

The ternary operator selects one of two inputs depending on the select signal.

```verilog
assign y = sel ? i1 : i0;
```

- When `sel = 1`, output `y` follows input `i1`.
- When `sel = 0`, output `y` follows input `i0`.

### 2.2 RTL Simulation

Compile and simulate the RTL design using Icarus Verilog.

```bash
iverilog ternary_operator_mux.v tb_ternary_operator_mux.v
./a.out
gtkwave tb_ternary_operator_mux.vcd
```
<img width="1917" height="862" alt="Screenshot 2026-11-21 093211" src="https://github.com/user-attachments/assets/d9c0d0d5-dbea-4e61-96fd-a6fa966b555f" />
<img width="1917" height="850" alt="Screenshot 2026-11-21 094459" src="https://github.com/user-attachments/assets/8c0541fa-1c8d-4bd2-b26d-3497ddaad3be" />

The generated VCD file can be opened in GTKWave to observe the input and output waveforms.

### 2.3 Synthesis Using Yosys

The RTL design is synthesized using the SKY130 HD standard-cell library.

```tcl
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_verilog ternary_operator_mux.v
synth -top ternary_operator_mux
dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
write_verilog -noattr ternary_operator_mux_net.v
```
<img width="1911" height="507" alt="Screenshot 2026-11-21 093752" src="https://github.com/user-attachments/assets/c39f50c5-a990-4294-89c5-f25ae307b35d" />

The synthesized gate-level netlist is generated as `ternary_operator_mux_net.v`.

### 2.4 Gate-Level Simulation

To simulate the synthesized netlist, compile it along with the SKY130 standard-cell Verilog models and the same testbench.

```bash
iverilog ../my_lib/verilog_model/primitives.v ../my_lib/verilog_model/sky130_fd_sc_hd.v ternary_operator_mux_net.v tb_ternary_operator_mux.v
./a.out
gtkwave tb_ternary_operator_mux.vcd
```

### 2.5 Simulation Results

**RTL Simulation, Gate-Level Simulation and Synthesis**

**Observation:**

- The RTL waveform shows the expected multiplexer functionality.
- The gate-level waveform shows the behavior of the synthesized netlist.
- The Yosys schematic represents the synthesized MUX logic.
- Both RTL and gate-level simulations preserve the expected functionality.

**Conclusion:** The synthesized netlist preserves the intended functionality of the original RTL design.

---

## 3. Synthesis-Simulation Mismatch Due to Missing Sensitivity List

### Theory

A synthesis-simulation mismatch occurs when the behavior observed during RTL simulation differs from the behavior of the synthesized hardware.

One common cause is an incomplete sensitivity list in a combinational `always` block.

### 3.1 Understanding the Sensitivity List

Consider the following Verilog code:

```verilog
always @(sel)
begin
    if (sel)
        d = b;
    else
        d = a;
end
```

Here, only `sel` is included in the sensitivity list.

**What happens?**

- When `sel` changes, the always block executes.
- When `a` or `b` changes without a change in `sel`, the block does not execute.
- The RTL simulation may therefore retain an old output value.
- However, the synthesized hardware represents a multiplexer whose output responds to changes in all its inputs.

### 3.2 Correct Coding Style

For combinational logic, use `always @(*)` so that all signals read inside the block are included in the inferred sensitivity list.

```verilog
always @(*)
begin
    if (sel)
        d = b;
    else
        d = a;
end
```

This ensures that changes in any input used by the block trigger its execution during simulation.

### 3.3 Simulation Results

**Missing Sensitivity List – RTL vs Gate-Level Simulation**

<img width="1912" height="881" alt="Screenshot 2026-11-21 095622" src="https://github.com/user-attachments/assets/b68e162d-ab3a-46b0-a356-8c25ed060067" />
<img width="1917" height="705" alt="Screenshot 2026-11-21 100702" src="https://github.com/user-attachments/assets/57c0d5f1-5fad-4222-8e15-417a41765b0b" />


**Observation:**

- The upper waveform shows the RTL simulation.
- The lower waveform shows the gate-level simulation.
- In RTL simulation, changes in `a` and `b` alone do not trigger the `always @(sel)` block.
- The synthesized MUX responds to changes in its input signals as expected.

**Conclusion:** An incomplete sensitivity list can cause RTL simulation to behave differently from synthesized hardware. Using `always @(*)` is the recommended coding style for combinational logic.

---

## 4. Blocking and Non-Blocking Assignments

Verilog provides two types of procedural assignments.

| Blocking Assignment (`=`) | Non-Blocking Assignment (`<=`) |
|---|---|
| Executes immediately | Schedules updates for the end of the current simulation time step |
| Statements execute in procedural order | RHS expressions are evaluated before scheduled updates |
| Commonly used for combinational logic | Recommended for sequential logic |
| Can introduce race conditions when misused | Helps avoid common sequential assignment race conditions |

### 4.1 Blocking Assignment – Combinational Logic

Consider the following code:

```verilog
always @(*)
begin
    d = x & c;
    x = a | b;
end
```

Here:

- `a`, `b`, and `c` are inputs.
- `x` is an internal signal.
- `d` is the output.

Because blocking assignments execute immediately in procedural order, `d` is evaluated using the value of `x` available before the assignment `x = a | b` is executed.

Now consider the statements in reverse order:

```verilog
always @(*)
begin
    x = a | b;
    d = x & c;
end
```

In this case:

- `x` is updated first.
- `d` uses the newly assigned value of `x`.

This demonstrates how the order of blocking assignments can affect RTL simulation behavior.

The intended combinational relationship is:

```verilog
d = (a | b) & c;
```

The synthesized combinational circuit is determined by the logic relationships, rather than simply by the procedural execution order used by the simulator.

**Simulation Results – Blocking Assignment Caveat**

<img width="1917" height="1032" alt="Screenshot 2026-11-21 102547" src="https://github.com/user-attachments/assets/7e8205b3-702a-4015-bab0-fc27b825ef6d" />


**Observation:**

- The upper waveform shows the RTL simulation.
- The lower waveform shows the gate-level simulation.
- The RTL waveform demonstrates the effect of blocking assignment order.
- The synthesized logic represents the combinational relationship inferred by synthesis.

**Conclusion:** Blocking assignments in combinational logic must be written carefully. The order of statements can affect simulation behavior and may lead to an RTL-synthesis mismatch.

### 4.2 Blocking Assignment – Sequential Logic

Consider a two-flip-flop shift-register example:

```verilog
always @(posedge clk)
begin
    q = q0;
    q0 = d;
end
```

With blocking assignments:

- `q` receives the current value of `q0`.
- `q0` is then updated with `d`.

If the order is reversed:

```verilog
always @(posedge clk)
begin
    q0 = d;
    q = q0;
end
```

Now:

- `q0` is updated first.
- `q` receives the newly assigned value of `q0`.

This changes the behavior because blocking assignments execute immediately.
<img width="1917" height="778" alt="Screenshot 2026-11-21 102901" src="https://github.com/user-attachments/assets/eee90d76-470c-4ae1-ae02-dce25cec1b20" />
<img width="1461" height="740" alt="Screenshot 2026-11-21 103633" src="https://github.com/user-attachments/assets/399c6db2-178b-40d4-9183-6ecdf3e9ddbe" />

### 4.3 Non-Blocking Assignment – Recommended Sequential Coding Style

The recommended way to describe the two-flip-flop shift register is:

```verilog
always @(posedge clk)
begin
    q  <= q0;
    q0 <= d;
end
```

With non-blocking assignments:

- `q` receives the previous value of `q0`.
- `q0` receives the value of `d`.
- Both updates occur at the end of the current simulation time step.
- The result is independent of the order of these two assignments.

**Key Takeaway:** Use blocking assignments (`=`) for combinational procedural logic and non-blocking assignments (`<=`) for sequential logic to avoid simulation mismatches and describe the intended hardware clearly.
---

## 5. Key Learnings

- Understood the purpose and flow of Gate-Level Simulation.
- Compared RTL simulation with functional gate-level simulation.
- Learned how to simulate a synthesized SKY130 gate-level netlist using standard-cell Verilog models.
- Understood how incomplete sensitivity lists can cause synthesis-simulation mismatches.
- Studied the effect of blocking assignment order on simulation behavior.
- Understood why non-blocking assignments are preferred for sequential logic.
- Learned that timing-aware GLS requires SDF annotation and was not covered in this workshop.
- Understood the importance of writing synthesizable and simulation-consistent RTL code.

---

## 6. Tools Used

- **Verilog HDL:** RTL design and testbench development.
- **Icarus Verilog:** RTL and gate-level simulation.
- **Yosys:** RTL synthesis and gate-level netlist generation.
- **GTKWave:** Waveform visualization and analysis.
- **SKY130 HD Standard Cell Library:** Standard-cell library and Verilog simulation models.
- **Linux Terminal:** Compilation and execution of simulation and synthesis commands.

---

## Conclusion

Day 4 focused on Gate-Level Simulation, blocking and non-blocking assignments, and synthesis-simulation mismatches.

The experiments demonstrated how a synthesized gate-level netlist can be verified against the original RTL design. They also highlighted how incomplete sensitivity lists and incorrect use of blocking assignments can cause differences between RTL simulation and synthesized hardware behavior.

The session emphasized the importance of using correct Verilog coding practices, particularly `always @(*)` for combinational logic and non-blocking assignments for sequential logic.
