
# Day 3: Introduction to Logic Optimizations

## Objective

Understand combinational and sequential logic optimization techniques using Verilog RTL and Yosys. Explore constant propagation, Boolean simplification, multiple-module optimization, sequential constant optimization, and removal of unused logic during synthesis.

## Tools Used

- **Verilog HDL:** Hardware description language for digital design.
- **Icarus Verilog:** RTL compilation and functional simulation.
- **GTKWave:** Simulation waveform analysis.
- **Yosys:** RTL synthesis and logic optimization.
- **ABC:** Logic optimization and technology mapping.
- **SKY130 Library:** Standard-cell library for technology mapping.
- **Linux Terminal:** Environment for executing synthesis and simulation commands.
- **GVim:** Verilog code editing.

## 1. Combinational Logic Optimization

**Combinational Logic Optimization:** Simplifies digital circuits while preserving their original functionality.

**Constant Propagation:** Replaces signals with known constant values (0 or 1) to simplify logic expressions.

**Boolean Logic Optimization:** Simplifies Boolean expressions using algebraic transformations and logic minimization.

### Experiment 1: opt_check

**Verilog Code (`opt_check.v`)**

```verilog
module opt_check (input a, input b, output y);
    assign y = a ? b : 0;
endmodule
```

**Explanation:**

The design contains a 2:1 multiplexer with one input connected to logic 0.

Using constant propagation:

**y = a & b**

The multiplexer is simplified into an AND operation.

**Yosys Synthesis Result:**

The optimized logic is equivalent to an AND gate.
<img width="1917" height="492" alt="Screenshot 2026-11-20 162706" src="https://github.com/user-attachments/assets/f2ccfd70-2858-4797-818b-d4b304da8cfa" />

### Experiment 2: opt_check2

**Verilog Code (`opt_check2.v`)**

```verilog
module opt_check2 (input a, input b, output y);
    assign y = a ? 1 : b;
endmodule
```

**Explanation:**

One input of the multiplexer is connected to logic 1.

Using Boolean simplification:

**y = a | b**

The multiplexer is simplified into an OR operation.

**Yosys Synthesis Result:**

The optimized logic is equivalent to an OR gate.
<img width="1917" height="503" alt="Screenshot 2026-11-20 163107" src="https://github.com/user-attachments/assets/8807edbe-1daa-4253-8c6f-e9cc272056e7" />

### Experiment 3: opt_check3

**Verilog Code (`opt_check3.v`)**

```verilog
module opt_check3 (input a, input b, input c, output y);
    assign y = a ? (c ? b : 0) : 0;
endmodule
```

**Explanation:**

The design contains nested conditional operators.

Using constant propagation and Boolean simplification:

**y = a & b & c**

The resulting logic is equivalent to a three-input AND gate.

**Yosys Synthesis Result:**

The nested multiplexer structure is simplified into equivalent AND logic.
<img width="1917" height="495" alt="Screenshot 2026-11-20 163645" src="https://github.com/user-attachments/assets/d7e52dea-028a-4a73-b03e-96f0d82c92ab" />

### Experiment 4: opt_check4

**Verilog Code (`opt_check4.v`)**

```verilog
module opt_check4 (input a, input b, input c, output y);
    assign y = a ? (b ? (a & c) : c) : (!c);
endmodule
```

**Explanation:**

The design contains nested conditional operators and Boolean expressions.

Using Boolean algebra:

**y = (a & b & c) | (!a & !c)**

The expression simplifies to:

**y = !(a ^ c)**

The resulting logic is equivalent to an XNOR operation between inputs a and c.

**Yosys Synthesis Result:**

The original conditional logic is simplified into an XNOR-equivalent implementation.
<img width="1917" height="498" alt="Screenshot 2026-11-20 164046" src="https://github.com/user-attachments/assets/6edeef83-54e2-4d43-9f43-d800026f9e90" />

## 2. Multiple Module Optimization

Multiple-module optimization examines how synthesis tools simplify hierarchical Verilog designs by optimizing logic across module boundaries.

### Experiment 5: multiple_module_opt

**Verilog Code (`multiple_module_opt.v`)**

```verilog
module sub_module1(input a, input b, output y);
    assign y = a & b;
endmodule

module sub_module2(input a, input b, output y);
    assign y = a ^ b;
endmodule

module multiple_module_opt(
    input a,
    input b,
    input c,
    input d,
    output y
);

    wire n1, n2, n3;

    sub_module1 U1 (.a(a), .b(1'b1), .y(n1));
    sub_module2 U2 (.a(n1), .b(1'b0), .y(n2));
    sub_module2 U3 (.a(b), .b(d), .y(n3));

    assign y = c | (b & n1);

endmodule
```

**Explanation:**

The design consists of two submodules implementing AND and XOR operations.

Constant inputs are applied to the submodules.

Since:

**n1 = a & 1 = a**

The output becomes:

**y = c | (a & b)**

The unused submodule outputs and unnecessary logic can be removed during optimization.
<img width="1917" height="1135" alt="Screenshot 2026-11-20 165616" src="https://github.com/user-attachments/assets/4106812f-57a4-469e-beba-ecb1c6947794" />

**Yosys Synthesis Result:**

The design is flattened and optimized, removing unnecessary logic while preserving the output functionality.

### Experiment 6: multiple_module_opt2

**Explanation:**

This experiment explores another hierarchical Verilog design and studies how Yosys performs module flattening and logic optimization.

**Optimization Techniques:**

- Module flattening.
- Constant propagation.
- Boolean simplification.
- Removal of unused logic.
- Optimization across module boundaries.

The final synthesized implementation depends on the RTL design and the selected top module.
<img width="1917" height="1127" alt="Screenshot 2026-11-20 165850" src="https://github.com/user-attachments/assets/42ac7365-43ad-4594-b12d-d8bec6a4773d" />

## 3. Sequential Logic Optimization

**Sequential Logic Optimization:** Simplifies circuits containing registers and flip-flops while preserving their required sequential behavior.

The techniques studied include:

- **Sequential Constant Propagation:** Identifies registers whose values can be determined under specified conditions.
- **State Optimization:** Removes redundant states or simplifies state representations.
- **Retiming:** Moves registers across combinational logic while preserving required sequential behavior.
- **Sequential Logic Cloning:** Duplicates sequential elements when beneficial for timing or fanout.

### 3.1 D Flip-Flop Constant Optimization

#### Experiment 7: dff_const1

**Verilog Code (`dff_const1.v`)**

```verilog
module dff_const1(input clk, input reset, output reg q);

always @(posedge clk, posedge reset)
begin
    if(reset)
        q <= 1'b0;
    else
        q <= 1'b1;
end

endmodule
```
<img width="1917" height="613" alt="Screenshot 2026-11-20 183057" src="https://github.com/user-attachments/assets/68c364fe-1fe2-4867-96d9-0800dc6788c9" />
<img width="1917" height="720" alt="Screenshot 2026-11-20 201909" src="https://github.com/user-attachments/assets/c33ed520-c8f2-4b98-81da-03b143e1fc92" />

**Explanation:**

The asynchronous reset sets the output to 0. During normal operation, the D input is constantly 1.

The flip-flop is retained because its output depends on the asynchronous reset condition.

#### Experiment 8: dff_const2

**Verilog Code (`dff_const2.v`)**

```verilog
module dff_const2(input clk, input reset, output reg q);

always @(posedge clk, posedge reset)
begin
    if(reset)
        q <= 1'b1;
    else
        q <= 1'b1;
end

endmodule
```
<img width="1587" height="697" alt="Screenshot 2026-11-20 200131" src="https://github.com/user-attachments/assets/3d8bc55c-b90f-4646-9dc5-169023db69fb" />

**Explanation:**

Both asynchronous reset and normal operating conditions assign logic 1 to the output.

Therefore, the output remains constant at 1, allowing the sequential element to be optimized away.

#### Experiment 9: dff_const3

**Verilog Code (`dff_const3.v`)**

```verilog
module dff_const3(input clk, input reset, output reg q);

reg q1;

always @(posedge clk, posedge reset)
begin
    if(reset)
    begin
        q  <= 1'b1;
        q1 <= 1'b0;
    end
    else
    begin
        q1 <= 1'b1;
        q  <= q1;
    end
end

endmodule
```
<img width="1496" height="912" alt="Screenshot 2026-11-20 212453" src="https://github.com/user-attachments/assets/a119e9e0-776e-49a0-a907-1299ab7121fe" />
<img width="1917" height="618" alt="Screenshot 2026-11-20 213805" src="https://github.com/user-attachments/assets/79e62d1a-9770-458e-9d53-61941ee36cfa" />

**Explanation:**

The design contains two sequential elements, with the output of one flip-flop connected to the data input of another.

Because of the sequential dependency, the output depends on the previous clock cycle.

Both registers are preserved in the observed synthesis result.

## 4. Counter Optimization

Counter optimization demonstrates how synthesis tools remove unnecessary registers and logic based on observable outputs.

### Experiment 12: counter_opt

**Verilog Code (`counter_opt.v`)**

```verilog
module counter_opt (
    input clk,
    input reset,
    output q
);

reg [2:0] count;

assign q = count[0];

always @(posedge clk, posedge reset)
begin
    if(reset)
        count <= 3'b000;
    else
        count <= count + 1;
end

endmodule
```
<img width="1917" height="645" alt="Screenshot 2026-11-20 215707" src="https://github.com/user-attachments/assets/a19f8115-f4d9-4ab0-8529-eb32234423a3" />

**Explanation:**

The design implements a 3-bit counter. However, only the least significant bit, count[0], is connected to the output.

The upper two bits do not influence the observable output and can be removed during synthesis.

The least significant bit toggles on every clock cycle.

**Yosys Synthesis Result:**

The optimized implementation requires only the sequential logic needed to generate the least significant bit.

## 5. Yosys Optimization Commands

The following commands are used to read the Verilog design, perform synthesis, optimize the logic, and display the resulting circuit.

**Yosys Commands:**

```tcl
read_liberty -lib ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

read_verilog opt_check.v

synth -top opt_check

opt_clean -purge

abc -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

show
```

**Description of Commands:**

| Command | Purpose |
|---|---|
| `read_liberty` | Reads the SKY130 standard-cell library |
| `read_verilog` | Reads the Verilog RTL design |
| `synth` | Performs synthesis for the specified top module |
| `opt_clean -purge` | Removes unused wires, cells, and unnecessary objects |
| `abc` | Performs technology mapping and logic optimization |
| `show` | Displays the synthesized logic diagram |

**Note:** The library path should match the actual location of the SKY130 Liberty file in the project.

## 6. Key Learnings

- Understood combinational logic optimization using constant propagation and Boolean simplification.
- Observed how nested multiplexers can be reduced to simpler logic gates.
- Learned how module flattening enables optimization across hierarchical designs.
- Studied sequential constant propagation in D flip-flop circuits.
- Understood how asynchronous reset and sequential dependencies affect register optimization.
- Observed how synthesis removes unused counter bits.
- Compared counter implementations based on observable outputs.
- Learned how Yosys and ABC optimize RTL designs for standard-cell implementation.

## 7. Conclusion

Day 3 focused on understanding combinational and sequential logic optimization using Verilog HDL and Yosys.

Different experiments were performed to study constant propagation, Boolean simplification, hierarchical module optimization, sequential constant optimization, and unused logic removal.

The counter and D flip-flop experiments demonstrated how synthesis tools analyze the functionality of sequential circuits before deciding which registers and logic elements can be removed.

These experiments provided practical understanding of how RTL designs are optimized into efficient hardware implementations while preserving their required functionality.

