
# Day 1: Introduction to Verilog RTL Design and Synthesis

## Objective

Understand Verilog RTL design, testbench development, functional simulation, waveform analysis, and logic synthesis using the SKY130 standard-cell library.

## Tools Used

- **Verilog HDL:** Hardware description language for digital design.
- **Icarus Verilog:** RTL compilation and simulation.
- **GTKWave:** Simulation waveform analysis.
- **Yosys:** RTL synthesis and logic optimization.
- **SKY130 Library:** Standard-cell library for technology mapping.

## 1. Introduction to Verilog RTL Design

**RTL Design:** Describes the behavior and data flow of a digital circuit using Verilog.

**Testbench:** Applies input stimuli to the Design Under Test (DUT) and checks its output behavior.

**Simulation Flow:**

RTL Design + Testbench → Icarus Verilog → Simulation → VCD Waveform → GTKWave

## 2. RTL Simulation of 2:1 Multiplexer

**Design File:** `good_mux.v`

**Testbench File:** `tb_good_mux.v`

Compile the design:

```bash
iverilog good_mux.v tb_good_mux.v
```

Run the simulation:

```bash
./a.out
```

Open the waveform:

```bash
gtkwave tb_good_mux.vcd
```

### Waveform Observation

- When `sel = 0`, output `y` follows `i0`.
- When `sel = 1`, output `y` follows `i1`.

**Screenshot 1: MUX Simulation Waveform**

<img width="1917" height="1198" alt="Screenshot 2026-11-19 151745" src="https://github.com/user-attachments/assets/f3f8e1df-8e99-42fe-b0f7-b2bef6348cd6" />


## 3. SKY130 Library Exploration

The SKY130 Liberty (`.lib`) file contains standard-cell information such as cell names, pins, timing, and area.

Library file:

`sky130_fd_sc_hd__tt_025C_1v80.lib`

Process Corners:

- **TT:** Typical-Typical
- **FF:** Fast-Fast
- **SS:** Slow-Slow

<img width="1917" height="1198" alt="Screenshot 2026-11-19 153011" src="https://github.com/user-attachments/assets/ad8eca7e-45c8-48ee-aba8-e4c2fcc9d4af" />


## 4. MUX Synthesis Using Yosys

Start Yosys:

```bash
yosys
```

Read the SKY130 library:

```tcl
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

Read the RTL design:

```tcl
read_verilog good_mux.v
```

Synthesize the design:

```tcl
synth -top good_mux
```

Technology mapping:

```tcl
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

View synthesis statistics:

```tcl
stat
```

Display the synthesized schematic:

```tcl
show
```

Generate the synthesized netlist:

```tcl
write_verilog -noattr good_mux_netlist.v
```

**Screenshot 2: Synthesized MUX Schematic**

<img width="1033" height="757" alt="Screenshot 2026-11-19 185449" src="https://github.com/user-attachments/assets/e4a28a72-2c18-4842-814b-594f2f2f6ec1" />


## 5. Key Learnings

- Understood Verilog RTL design and testbench development.
- Simulated a 2:1 MUX using Icarus Verilog.
- Analyzed waveforms using GTKWave.
- Explored SKY130 standard-cell libraries.
- Performed RTL synthesis and technology mapping using Yosys.
- Generated a synthesized Verilog netlist.

## References

- [VSD RTL Design and Synthesis Workshop](https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop)
- [Icarus Verilog](https://steveicarus.github.io/iverilog/)
- [Yosys Documentation](https://yosyshq.readthedocs.io/)
- [GTKWave](https://gtkwave.sourceforge.net/)
