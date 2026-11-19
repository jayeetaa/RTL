# Lab 1 — Introduction to RTL Design, Simulation and Synthesis Using SKY130

## Objective

To understand the fundamentals of Verilog RTL design, testbench development, functional simulation, waveform analysis, and logic synthesis using the SKY130 standard-cell library.

This lab focuses on implementing and verifying a 2:1 Multiplexer (MUX) using Verilog HDL, Icarus Verilog, GTKWave, and Yosys.

## Tools and Technologies Used

- **Verilog HDL** – Hardware Description Language used to describe digital circuits.
- **Icarus Verilog (iverilog)** – Used to compile and simulate Verilog designs.
- **GTKWave** – Used to visualize and analyze simulation waveforms.
- **Yosys** – Open-source RTL synthesis tool.
- **SKY130 Standard-Cell Library** – Provides standard-cell definitions and characteristics for technology mapping.
- **Ubuntu Linux** – Development environment running inside Oracle VirtualBox.

## Repository and Design Information

| Parameter | Description |
|---|---|
| Workshop Repository | sky130RTLDesignAndSynthesisWorkshop |
| Design Name | good_mux |
| Design Type | 2:1 Multiplexer |
| RTL Language | Verilog HDL |
| Technology Library | SKY130 |
| Synthesis Tool | Yosys |

---

# 1. Introduction to RTL Design

## What is RTL Design?

Register Transfer Level (RTL) design describes the behavior and data flow of a digital circuit using a Hardware Description Language (HDL), such as Verilog.

RTL design specifies how data moves between registers and how combinational logic processes input signals to generate outputs.

## What is a Testbench?

A testbench is a verification environment used to test a design under different input conditions.

It applies input stimuli to the Design Under Test (DUT) and observes the outputs to verify whether the circuit behaves as expected.

## What is a Simulator?

A simulator evaluates the behavior of a Verilog design over simulation time.

In this lab, Icarus Verilog is used to compile and simulate the RTL design along with its testbench.

## RTL Simulation Flow

1. Write the RTL design in Verilog.
2. Develop a testbench to generate input stimuli.
3. Compile the design and testbench using Icarus Verilog.
4. Execute the simulation.
5. Generate a Value Change Dump (VCD) file.
6. Open the VCD file in GTKWave.
7. Analyze the waveforms and verify the output behavior.

**Simulation Flow:**

RTL Design + Testbench → Icarus Verilog → Simulation → VCD File → GTKWave

---

# 2. Lab 1 — SKY130 Library Setup and Repository Exploration

## 2.1 Clone the Workshop Repository

Open the Ubuntu terminal and execute:

```bash
cd ~/Desktop
git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git
```

If the repository is already cloned, there is no need to clone it again.

Navigate to the repository:

```bash
cd ~/Desktop/sky130RTLDesignAndSynthesisWorkshop
```

Inspect the repository contents:

```bash
ls -la
```

## 2.2 Locate the SKY130 Liberty Library

Navigate to the library directory:

```bash
ls -l lib/
```

The standard-cell Liberty file used in this lab is:

`lib/sky130_fd_sc_hd__tt_025C_1v80.lib`

### What is a Liberty File?

A Liberty (.lib) file contains information about standard cells, including:

- Cell names and descriptions.
- Input and output pin information.
- Timing characteristics.
- Cell area.
- Power characteristics.
- Timing arcs and operating conditions.

The SKY130 library provides standard-cell information required for technology mapping during synthesis.

## 2.3 Process Corners

The SKY130 standard-cell library supports different process conditions.

- **TT (Typical-Typical):** Typical NMOS and PMOS characteristics.
- **FF (Fast-Fast):** Both NMOS and PMOS operate at faster process conditions.
- **SS (Slow-Slow):** Both NMOS and PMOS operate at slower process conditions.

Process corners are considered during timing analysis to evaluate circuit behavior under different manufacturing and operating conditions.

## 2.4 Navigate to the Verilog Directory

```bash
cd ~/Desktop/sky130RTLDesignAndSynthesisWorkshop/verilog_files
ls
```

The important design files are:

- `good_mux.v`
- `tb_good_mux.v`

## 2.5 Read the SKY130 Library in Yosys

Start Yosys:

```bash
yosys
```

At the Yosys prompt, execute:

```tcl
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

This loads the standard-cell library definitions into Yosys.

Exit Yosys:

```tcl
exit
```

**Screenshot 1 — SKY130 Library Inspection**

![SKY130 Library](library.png)

---

# 3. Lab 2 — RTL Design and Functional Simulation of a 2:1 MUX

## 3.1 Understanding a 2:1 Multiplexer

A 2:1 Multiplexer is a combinational logic circuit that selects one of two input signals based on a select signal.

It consists of:

- Two data inputs: `i0` and `i1`
- One select input: `sel`
- One output: `y`

### Truth Table

| i0 | i1 | sel | y |
|---|---|---|---|
| 0 | 0 | 0 | 0 |
| 0 | 1 | 0 | 0 |
| 1 | 0 | 0 | 1 |
| 1 | 1 | 0 | 1 |
| 0 | 0 | 1 | 0 |
| 0 | 1 | 1 | 1 |
| 1 | 0 | 1 | 0 |
| 1 | 1 | 1 | 1 |

### Boolean Expression

\[
y = (\overline{sel} \land i0) \lor (sel \land i1)
\]

When `sel = 0`, the output follows `i0`.

When `sel = 1`, the output follows `i1`.

## 3.2 RTL Design — good_mux.v

```verilog
module good_mux (
    input  i0,
    input  i1,
    input  sel,
    output reg y
);

always @(*) begin
    if (sel)
        y <= i1;
    else
        y <= i0;
end

endmodule
```

The RTL code describes the combinational behavior of the 2:1 multiplexer.

## 3.3 Testbench — tb_good_mux.v

The testbench applies different input combinations to verify the functionality of the multiplexer.

```verilog
`timescale 1ns/1ps

module tb_good_mux;

reg i0, i1, sel;
wire y;

good_mux uut (
    .i0(i0),
    .i1(i1),
    .sel(sel),
    .y(y)
);

initial begin
    $dumpfile("tb_good_mux.vcd");
    $dumpvars(0, tb_good_mux);

    i0 = 0; i1 = 0; sel = 0;
    #10 i0 = 1; i1 = 0; sel = 0;
    #10 i0 = 0; i1 = 1; sel = 0;
    #10 i0 = 0; i1 = 1; sel = 1;
    #10 i0 = 1; i1 = 0; sel = 1;
    #10 i0 = 1; i1 = 1; sel = 0;
    #10 i0 = 0; i1 = 0; sel = 1;
    #10 $finish;
end

endmodule
```

The `$dumpfile` and `$dumpvars` commands generate the VCD waveform file.

## 3.4 Install Icarus Verilog

If Icarus Verilog is not installed:

```bash
sudo apt update
sudo apt install iverilog -y
```

## 3.5 Compile the Verilog Design

Navigate to the Verilog workspace:

```bash
cd ~/Desktop/sky130RTLDesignAndSynthesisWorkshop/verilog_files
```

Compile the design and testbench:

```bash
iverilog -o good_mux_sim good_mux.v tb_good_mux.v
```

## 3.6 Run the Simulation

```bash
vvp good_mux_sim
```

The simulation generates:

`tb_good_mux.vcd`

Verify the generated file:

```bash
ls -l tb_good_mux.vcd
```

## 3.7 Open the Waveform in GTKWave

Install GTKWave if required:

```bash
sudo apt install gtkwave -y
```

Open the waveform:

```bash
gtkwave tb_good_mux.vcd
```

Add the following signals to the waveform viewer:

- i0
- i1
- sel
- y

## 3.8 Waveform Analysis

The GTKWave waveform is used to observe the input and output transitions of the multiplexer.

Expected behavior:

- When `sel = 0`, output `y` follows `i0`.
- When `sel = 1`, output `y` follows `i1`.
- The output changes according to the selected input.

**Screenshot 2 — MUX Simulation Waveform**

![MUX Waveform](mux_waveform.png)

The waveform confirms the functional behavior of the 2:1 multiplexer for the applied testbench inputs.

---

# 4. Lab 3 — Logic Synthesis Using Yosys

## 4.1 What is Logic Synthesis?

Logic synthesis is the process of converting an RTL description into a logic-level implementation.

It involves:

1. Reading and elaborating the RTL code.
2. Performing logic optimization.
3. Mapping the logic into standard cells.
4. Generating a gate-level netlist.

Yosys is used in this lab to synthesize the 2:1 multiplexer.

## 4.2 Start Yosys

Navigate to the Verilog directory:

```bash
cd ~/Desktop/sky130RTLDesignAndSynthesisWorkshop/verilog_files
```

Start Yosys:

```bash
yosys
```

## 4.3 Read the SKY130 Library

At the Yosys prompt:

```tcl
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

This loads the standard-cell library information required for technology mapping.

## 4.4 Read the RTL Design

```tcl
read_verilog good_mux.v
```

## 4.5 Set the Top Module

```tcl
hierarchy -top good_mux
```

This identifies `good_mux` as the top-level design module.

## 4.6 Perform Logic Synthesis

```tcl
synth -top good_mux
```

Yosys performs synthesis and optimizes the RTL representation into a logic-level implementation.

## 4.7 Technology Mapping Using ABC

```tcl
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

ABC performs logic optimization and maps the logic into available standard cells from the SKY130 library.

## 4.8 Display Cell Statistics

```tcl
clean
stat
```

The `stat` command displays synthesis statistics, including the number and types of cells used in the design.

## 4.9 View the Synthesized Schematic

```tcl
show
```

This displays the synthesized logic representation of the MUX.

**Screenshot 3 — Synthesized MUX Schematic**

![MUX Schematic](mux.png)

The synthesized schematic shows the logic implementation generated by Yosys.

## 4.10 Generate the Gate-Level Netlist

```tcl
write_verilog -noattr good_mux_netlist.v
```

This generates the synthesized Verilog netlist.

To inspect the generated file:

```tcl
!cat good_mux_netlist.v
```

Alternatively, exit Yosys and execute:

```bash
cat good_mux_netlist.v
```

## 4.11 Exit Yosys

```tcl
exit
```

---

# 5. Verification of the Synthesized Netlist

The synthesized netlist should be checked against the original RTL design to ensure that the intended functionality is preserved.

### Verification Procedure

1. Simulate the original RTL design using `tb_good_mux.v`.
2. Simulate the synthesized netlist using the same testbench.
3. Apply identical input stimuli to both implementations.
4. Observe and compare the output behavior.
5. Verify that both implementations produce the expected results.

For a technology-mapped SKY130 netlist, the corresponding SKY130 Verilog standard-cell models may also be required during simulation.

Matching output behavior under the tested input conditions provides evidence that the synthesized implementation preserves the intended functionality.

---

# 6. Complete Command Reference

## Ubuntu Terminal Commands

```bash
cd ~/Desktop/sky130RTLDesignAndSynthesisWorkshop/verilog_files

iverilog -o good_mux_sim good_mux.v tb_good_mux.v

vvp good_mux_sim

gtkwave tb_good_mux.vcd
```

## Yosys Commands

```tcl
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

read_verilog good_mux.v

hierarchy -top good_mux

synth -top good_mux

abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

clean

stat

show

write_verilog -noattr good_mux_netlist.v

exit
```

---

# 7. Key Learnings

Through this lab, the following concepts were explored:

- Understood the fundamentals of RTL design using Verilog HDL.
- Learned the purpose of testbenches in functional verification.
- Implemented a 2:1 multiplexer using behavioral Verilog.
- Compiled and simulated Verilog designs using Icarus Verilog.
- Generated VCD files and analyzed signal transitions using GTKWave.
- Understood the structure and importance of SKY130 Liberty libraries.
- Learned the fundamentals of logic synthesis using Yosys.
- Performed technology mapping using the ABC synthesis engine.
- Generated and inspected a synthesized gate-level Verilog netlist.
- Understood the importance of verifying synthesized logic against the original RTL.

---

# 8. Repository Structure

```text
lab_1/
│
├── README.md
├── library.png
├── mux.png
└── mux_waveform.png
```

The screenshots are included to document the SKY130 library inspection, MUX simulation waveform, and synthesized logic schematic.

---

# 9. References

- [SKY130 RTL Design and Synthesis Workshop](https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop)
- [Icarus Verilog Documentation](https://steveicarus.github.io/iverilog/)
- [GTKWave](https://gtkwave.sourceforge.net/)
- [Yosys Documentation](https://yosyshq.readthedocs.io/)
- [SkyWater SKY130 PDK](https://github.com/google/skywater-pdk)

---

**End of Lab 1 — RTL Design, Simulation and Synthesis Using SKY130**
