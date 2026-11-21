
# Day 5: Optimization in Synthesis

## Objective

Understand how different Verilog RTL coding styles affect hardware synthesis and optimization. Explore conditional statements, latch inference, CASE constructs, procedural FOR loops, Generate FOR loops, multiplexers, demultiplexers, and Ripple Carry Adders using RTL simulation and Yosys synthesis with the SKY130 standard-cell library.

## Tools Used

- **Verilog HDL:** Hardware description language for describing digital circuits.
- **Icarus Verilog:** RTL compilation and functional simulation.
- **GTKWave:** Simulation waveform analysis.
- **Yosys:** RTL synthesis, optimization, and hardware visualization.
- **GVim:** Verilog RTL code editing.
- **Graphviz / xdot:** Visualization of synthesized logic schematics.
- **SKY130:** Open-source standard-cell technology library.

## 1. IF-ELSE Constructs and Priority Logic

The IF-ELSE construct is used to describe conditional behavior in Verilog.

When multiple conditions are written using `if` and `else if`, they are evaluated sequentially. The first condition that evaluates to true determines which assignment is executed.

### Syntax

```verilog
always @(*) begin
    if (cond1)
        y = a;
    else if (cond2)
        y = b;
    else
        y = c;
end
```

### Observation

- IF-ELSE statements can infer priority logic.
- The first condition has the highest priority.
- Multiple conditions can result in cascaded multiplexers.
- The final ELSE branch handles all remaining conditions.

## 2. Latch Inference – Incomplete IF Statement

A latch is inferred when an output is not assigned a value in every possible execution path of a combinational block.

### RTL Code

```verilog
module incomp_if (
    input i0,
    input i1,
    output reg y
);

always @(*) begin
    if (i0)
        y = i1;
end

endmodule
```
<img width="1917" height="702" alt="Screenshot 2026-11-21 111703" src="https://github.com/user-attachments/assets/1a9398cc-2564-4e07-b397-75d01168311e" />
<img width="1917" height="412" alt="Screenshot 2026-11-21 112239" src="https://github.com/user-attachments/assets/d1afaba7-8c8f-402c-998d-b83da8a4b53e" />

### Explanation

- When `i0 = 1`, output `y` follows `i1`.
- When `i0 = 0`, no assignment is made to `y`.
- Therefore, the output retains its previous value.
- Synthesis infers a D latch to preserve the previous value.

### Synthesis Observation

Yosys identifies the inferred hardware as:

```text
$_DLATCH_P_
```

This represents a positive-level-sensitive D latch.

### Key Learning

Incomplete assignments in combinational logic can cause unintended latch inference.

## 3. Multiple Incomplete IF Conditions

### RTL Code

```verilog
module incomp_if2 (
    input i0,
    input i1,
    input i2,
    input i3,
    output reg y
);

always @(*) begin
    if (i0)
        y = i1;
    else if (i2)
        y = i3;
end

endmodule
```
<img width="1915" height="832" alt="Screenshot 2026-11-21 113952" src="https://github.com/user-attachments/assets/3899982a-381d-49d6-8479-209d441e116f" />
<img width="1917" height="725" alt="Screenshot 2026-11-21 113610" src="https://github.com/user-attachments/assets/dc40a441-ef2a-4223-ae33-a604ee2e4221" />

### Explanation

The first condition selects `i1`, while the second condition selects `i3` only when `i0` is false.

When both `i0` and `i2` are low, no assignment is made to `y`. Therefore, the output retains its previous value, resulting in latch inference.

### Observation

- The first condition has higher priority.
- The second condition is evaluated only when the first condition is false.
- When both conditions are false, the latch holds its previous value.

## 4. CASE Constructs

A CASE statement is used to select one among multiple alternatives based on a selector signal.

### Syntax

```verilog
always @(*) begin
    case (sel)
        2'b00: y = i0;
        2'b01: y = i1;
        2'b10: y = i2;
        2'b11: y = i3;
        default: y = 1'b0;
    endcase
end
```

### Complete CASE Statement

A complete CASE statement assigns an output for every possible selector condition.

```verilog
module comp_case (
    input i0,
    input i1,
    input i2,
    input [1:0] sel,
    output reg y
);

always @(*) begin
    case (sel)
        2'b00: y = i0;
        2'b01: y = i1;
        default: y = i2;
    endcase
end

endmodule
```

### Observation

- A complete CASE statement handles all selector combinations.
- The default branch handles the remaining conditions.
- Complete assignments allow combinational logic to be synthesized without an unintended latch.
<img width="1917" height="790" alt="Screenshot 2026-11-21 115302" src="https://github.com/user-attachments/assets/d9340748-f2db-430d-b4fe-06e8bd0ae128" />
<img width="1917" height="890" alt="Screenshot 2026-11-21 115525" src="https://github.com/user-attachments/assets/f49cd8e1-5b76-4209-8d5d-a9facb9edbb3" />

## 5. Incomplete CASE Statement

An incomplete CASE statement does not assign an output for every possible selector value.

### RTL Code

```verilog
module incomp_case (
    input i0,
    input i1,
    input i2,
    input [1:0] sel,
    output reg y
);

always @(*) begin
    case (sel)
        2'b00: y = i0;
        2'b01: y = i1;
    endcase
end

endmodule
```
<img width="1917" height="780" alt="Screenshot 2026-11-21 114425" src="https://github.com/user-attachments/assets/fa56ea68-46cd-40a3-87f9-0a0f5c089ae7" />
<img width="1173" height="835" alt="Screenshot 2026-11-21 114714" src="https://github.com/user-attachments/assets/07c9dbc7-e8d7-4eb8-bdb1-7748c4a781dc" />

### Explanation

The selector is two bits wide and can represent four combinations:

- `00`
- `01`
- `10`
- `11`

Only `00` and `01` are handled in the CASE statement.

For selector values `10` and `11`, no assignment is made to `y`. Therefore, synthesis infers a latch.

### Observation

Incomplete CASE assignments can cause latch inference because the output must retain its previous value for unspecified conditions.

## 6. Partial Assignments in CASE Statements

A latch can be inferred even when a default branch is present if every output is not assigned in every branch.

### RTL Code

```verilog
module partial_case_assign (
    input i0,
    input i1,
    input i2,
    input [1:0] sel,
    output reg y,
    output reg x
);

always @(*) begin
    case (sel)

        2'b00: begin
            y = i0;
            x = i2;
        end

        2'b01: begin
            y = i1;
        end

        default: begin
            y = i2;
            x = i1;
        end

    endcase
end

endmodule
```
<img width="1917" height="1198" alt="Screenshot 2026-11-21 120650" src="https://github.com/user-attachments/assets/127e03d8-97e4-4903-9eae-47317e349eeb" />

### Explanation

- Output `y` is assigned in every branch.
- Output `x` is not assigned when `sel = 2'b01`.
- Therefore, `x` must retain its previous value during that condition.
- Synthesis infers a latch for `x`.

### Key Learning

A default branch alone does not prevent latch inference. Every output must be assigned in every branch of a combinational block.

## 7. CASE, CASEZ and CASEX

Verilog provides different CASE constructs with different matching behaviors.

| Construct | Matching Behavior |
|---|---|
| `case` | Exact four-state matching |
| `casez` | Treats Z and ? as wildcards |
| `casex` | Treats X, Z and ? as wildcards |

### Example Using CASEZ

```verilog
always @(*) begin
    casez (sel)
        2'b00: y = i0;
        2'b01: y = i1;
        2'b10: y = i2;
        2'b1?: y = i3;
        default: y = 1'b0;
    endcase
end
```

### Important Points

- Regular CASE performs exact matching.
- CASEZ treats Z and ? as wildcard bits.
- CASEX treats X, Z and ? as wildcard bits.
- Overlapping wildcard patterns should be avoided.
- CASEX is generally discouraged in synthesizable RTL because it can hide unknown values and cause simulation mismatches.

## 8. Bad CASE – Overlapping Conditions

This experiment demonstrates the importance of avoiding overlapping CASE conditions.

### RTL Code

```verilog
module bad_case (
    input i0,
    input i1,
    input i2,
    input i3,
    input [1:0] sel,
    output reg y
);

always @(*) begin
    case (sel)
        2'b00: y = i0;
        2'b01: y = i1;
        2'b10: y = i2;
        2'b1?: y = i3;
    endcase
end

endmodule
```
<img width="1917" height="985" alt="Screenshot 2026-11-21 121651" src="https://github.com/user-attachments/assets/e334cf2e-21f5-47ee-b502-7e6bd06f8a7b" />

### Understanding the CASE Pattern

In a regular CASE statement, `?` is not a general wildcard.

For wildcard matching, CASEZ can be used:

```verilog
casez (sel)
    2'b00: y = i0;
    2'b01: y = i1;
    2'b10: y = i2;
    2'b1?: y = i3;
endcase
```

Here, `2'b1?` matches both `2'b10` and `2'b11`.

The pattern overlaps with the explicitly defined `2'b10` condition.

### Observation

- Overlapping wildcard patterns can create ambiguous selection behavior.
- The simulator selects the first matching case item.
- Differences between RTL simulation and synthesized implementation should be investigated.
- Mutually exclusive conditions are preferred for reliable RTL design.

## 9. Procedural FOR Loops

A procedural FOR loop is written inside an always block and is used to describe repetitive operations.

During synthesis, a statically bounded FOR loop is generally unrolled into hardware. It does not necessarily create a physical hardware counter.

### Syntax

```verilog
integer k;

always @(*) begin
    for (k = 0; k < 4; k = k + 1) begin
        // Repeated operation
    end
end
```

### Applications

- Multiplexer implementation.
- Demultiplexer implementation.
- Repetitive bit-level operations.
- Wide bus processing.
- Combinational logic implementation.

## 10. Multiplexer Using a FOR Loop

A multiplexer selects one input from multiple input lines based on the select signal.

### RTL Code

```verilog
module mux_generate (
    input i0,
    input i1,
    input i2,
    input i3,
    input [1:0] sel,
    output reg y
);

wire [3:0] i_int;

assign i_int = {i3, i2, i1, i0};

integer k;

always @(*) begin
    for (k = 0; k < 4; k = k + 1) begin
        if (k == sel)
            y = i_int[k];
    end
end

endmodule
```
<img width="1917" height="905" alt="Screenshot 2026-11-21 123113" src="https://github.com/user-attachments/assets/48659b73-7906-4d3c-af6e-42bf463d7d74" />

### Explanation

- The four input signals are combined into a four-bit vector.
- The FOR loop iterates through all four input positions.
- The loop compares the index with the selector.
- The matching input is assigned to the output.

### Synthesis Observation

A default assignment should be provided before the loop to ensure that the output receives a value for every possible execution path.

```verilog
always @(*) begin
    y = 1'b0;

    for (k = 0; k < 4; k = k + 1) begin
        if (k == sel)
            y = i_int[k];
    end
end
```

This avoids unintended latch inference.

## 11. DEMUX Using a CASE Statement

A demultiplexer directs a single input to one of several output lines based on the select signal.

The following example implements a 1:8 DEMUX.

### RTL Code

```verilog
module demux_case (
    output o0,
    output o1,
    output o2,
    output o3,
    output o4,
    output o5,
    output o6,
    output o7,
    input [2:0] sel,
    input i
);

reg [7:0] y_int;

assign {o7,o6,o5,o4,o3,o2,o1,o0} = y_int;

always @(*) begin
    y_int = 8'b0;

    case (sel)
        3'b000: y_int[0] = i;
        3'b001: y_int[1] = i;
        3'b010: y_int[2] = i;
        3'b011: y_int[3] = i;
        3'b100: y_int[4] = i;
        3'b101: y_int[5] = i;
        3'b110: y_int[6] = i;
        3'b111: y_int[7] = i;
    endcase
end

endmodule
```
<img width="1917" height="950" alt="Screenshot 2026-11-21 123658" src="https://github.com/user-attachments/assets/9c3fc0e1-08aa-4947-adfb-c3e0221ba75e" />

### Explanation

- The input signal is routed to one of eight outputs.
- The three-bit selector determines the active output.
- The internal vector is initialized to zero.
- All non-selected outputs remain zero.

### Observation

The default assignment to `y_int` ensures complete combinational assignments.

## 12. DEMUX Using a FOR Loop

The same 1:8 DEMUX functionality can be implemented using a procedural FOR loop.

### RTL Code

```verilog
module demux_generate (
    output o0,
    output o1,
    output o2,
    output o3,
    output o4,
    output o5,
    output o6,
    output o7,
    input [2:0] sel,
    input i
);

reg [7:0] y_int;

assign {o7,o6,o5,o4,o3,o2,o1,o0} = y_int;

integer k;

always @(*) begin
    y_int = 8'b0;

    for (k = 0; k < 8; k = k + 1) begin
        if (k == sel)
            y_int[k] = i;
    end
end

endmodule
```
<img width="1917" height="906" alt="Screenshot 2026-11-21 123448" src="https://github.com/user-attachments/assets/981c63f5-3a7d-4799-9916-b0f6076dfbf5" />

### Explanation

- The loop iterates through all eight output positions.
- Each iteration compares the loop index with the selector.
- When a match occurs, the corresponding output receives the input signal.
- All other outputs remain zero.

### Comparison

| DEMUX Using CASE | DEMUX Using FOR |
|---|---|
| Explicit selector conditions | Iterative selection logic |
| Individual output assignments | Repeated assignments through a loop |
| Easy to understand for small designs | Useful for repetitive and scalable designs |

Both implementations describe the same basic DEMUX functionality.


## 13. Ripple Carry Adder Using Generate

A Ripple Carry Adder (RCA) is constructed by connecting multiple full adders in a chain.

Each full adder adds two input bits along with a carry-in. The carry-out from one stage becomes the carry-in of the next stage.

### Full Adder Module

```verilog
module fa (
    input a,
    input b,
    input c,
    output co,
    output sum
);

assign {co, sum} = a + b + c;

endmodule
```

### Ripple Carry Adder Module

The following design implements an 8-bit Ripple Carry Adder using generated full-adder stages.

```verilog
module rca (
    input [7:0] num1,
    input [7:0] num2,
    output [8:0] sum
);

wire [7:0] int_sum;
wire [7:0] int_co;

genvar i;

generate
    for (i = 1; i < 8; i = i + 1) begin

        fa u_fa (
            .a(num1[i]),
            .b(num2[i]),
            .c(int_co[i-1]),
            .co(int_co[i]),
            .sum(int_sum[i])
        );

    end
endgenerate

fa u_fa_0 (
    .a(num1[0]),
    .b(num2[0]),
    .c(1'b0),
    .co(int_co[0]),
    .sum(int_sum[0])
);

assign sum[7:0] = int_sum;
assign sum[8] = int_co[7];

endmodule
```
<img width="1917" height="947" alt="Screenshot 2026-11-21 124121" src="https://github.com/user-attachments/assets/ad2c1244-fb63-49c4-8767-26ffe5d8beb2" />

### Working Principle

1. The first full adder processes the least significant bits.
2. Its carry-out is connected to the next full adder.
3. Each subsequent stage adds the corresponding operand bits and incoming carry.
4. The final carry becomes the most significant bit of the result.

### Output Width Calculation

For two unsigned n-bit operands, the maximum result width is:

\[
n+1
\]

For two 8-bit operands:

\[
8+1=9\text{ bits}
\]

Therefore, the output is declared as `output [8:0] sum`.

### Observation

The Generate FOR loop simplifies the construction of repetitive full-adder hardware and makes the design easier to scale.

## 14. Simulation and Synthesis Commands

### Compile Verilog Design

```bash
iverilog design.v testbench.v
```

### Execute Simulation

```bash
./a.out
```

### View Simulation Waveform

```bash
gtkwave testbench.vcd
```

### Start Yosys

```bash
yosys
```

### Read Verilog File

```tcl
read_verilog design.v
```

### Perform Synthesis

```tcl
synth -top module_name
```

### View Synthesis Statistics

```tcl
stat
```

### Generate Schematic

```tcl
show
```

### Exit Yosys

```tcl
exit
```

## 15. Key Learnings

- IF-ELSE statements can infer priority MUX logic.
- Incomplete IF statements can infer latches.
- Incomplete CASE statements can also infer latches.
- Complete assignments are essential for combinational logic.
- Partial assignments can cause latch inference for individual outputs.
- CASE, CASEZ and CASEX have different matching behaviors.
- Overlapping wildcard conditions should be avoided.
- Procedural FOR loops describe repetitive operations and are unrolled during synthesis.
- Generate FOR loops replicate hardware structures during elaboration.
- Multiplexers and demultiplexers can be implemented using different RTL coding styles.
- Full adders can be connected to construct Ripple Carry Adders.
- RTL coding style directly affects the hardware inferred during synthesis.
- RTL simulation and gate-level simulation help verify hardware behavior.

## 16. Conclusion

Day 5 provided practical exposure to optimization in synthesis through different Verilog RTL coding styles.

The experiments demonstrated how IF-ELSE statements, CASE constructs, incomplete assignments and overlapping conditions influence hardware inference. The session also covered procedural FOR loops, Generate FOR loops, multiplexers, demultiplexers and Ripple Carry Adders.

By analyzing simulation waveforms and Yosys-generated schematics, I gained a better understanding of how Verilog RTL descriptions are translated into hardware.

Writing correct and synthesis-friendly RTL is essential for reliable digital hardware design and effective synthesis optimization.
