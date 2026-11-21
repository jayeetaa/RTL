
# RTL Design and Synthesis Using SKY130

## About This Repository

This repository documents my hands-on learning and practical implementation of **RTL Design and Synthesis** using Verilog HDL and open-source VLSI tools.

The workshop focuses on understanding digital design concepts, writing and simulating Verilog RTL, analyzing synthesis results, exploring timing libraries, and observing how RTL descriptions are converted into gate-level implementations using the SKY130 standard-cell library.

Each day contains the concepts explored, practical experiments, commands used, simulation results, synthesis observations, and supporting screenshots.

## Objectives

- Understand the fundamentals of RTL design using Verilog HDL.
- Develop and simulate Verilog modules and testbenches.
- Understand timing libraries and standard-cell concepts.
- Explore hierarchical and flattened synthesis.
- Analyze RTL optimization techniques.
- Understand blocking and non-blocking assignments.
- Perform Gate-Level Simulation (GLS).
- Study synthesis behavior for different Verilog coding styles.
- Gain practical experience with the RTL-to-Gate-Level flow.

## Tools and Technologies

- **Verilog HDL:** Hardware description language used to describe digital circuits.
- **Icarus Verilog:** Used for Verilog compilation and functional simulation.
- **GTKWave:** Used to view and analyze simulation waveforms.
- **Yosys:** Open-source RTL synthesis tool.
- **SKY130 PDK:** Open-source process design kit used for digital circuit implementation.
- **Liberty Files (.lib):** Provide standard-cell functionality and timing information.
- **Graphviz:** Used to generate graphical representations of synthesized circuits.
- **Ubuntu Linux (WSL):** Development and execution environment.
- **Git and GitHub:** Used for version control and documentation.

## Workshop Progress

### Day 00 – Tool Setup

- Set up the Linux development environment.
- Prepared the required open-source tools.
- Explored the workshop directory structure.
- Organized the environment for RTL design and synthesis experiments.

[View Day 00 – Tool Setup](./Day-00-Tool-Setup/)

### Day 01 – Introduction to Verilog

- Explored the basics of Verilog RTL design.
- Developed Verilog modules and testbenches.
- Performed RTL simulation using Icarus Verilog.
- Analyzed simulation waveforms using GTKWave.
- Introduced RTL synthesis using Yosys.
- Explored the SKY130 standard-cell library.

[View Day 01 – Introduction to Verilog](./Day-01-Introduction-to-Verilog/)

### Day 02 – Timing Libraries, Hierarchical Synthesis and RTL Optimization

- Explored timing libraries and Liberty files.
- Studied SKY130 standard-cell characteristics.
- Analyzed hierarchical Verilog designs.
- Compared hierarchical and flattened synthesis.
- Examined submodule functionality and design hierarchy.
- Explored reset configurations and arithmetic optimization.

[View Day 02 – Timing Libraries and Hierarchical Synthesis](./Day-02-Timing-Libraries-Hierarchical-Synthesis/)

### Day 03 – Introduction to Optimizations

- Explored RTL optimization techniques.
- Analyzed how Yosys optimizes different Verilog coding styles.
- Studied the effect of RTL descriptions on synthesized logic.
- Examined optimized circuit representations.

[View Day 03 – Introduction to Optimizations](./Day-03-Introduction-to-Optimizations/)

### Day 04 – Gate-Level Simulation, Blocking and Non-Blocking Assignments

- Explored Gate-Level Simulation (GLS).
- Studied blocking and non-blocking assignments.
- Compared RTL simulation and gate-level simulation.
- Examined the role of simulation models and standard-cell libraries.
- Analyzed simulation behavior using waveform visualization.

[View Day 04 – GLS, Blocking and Non-Blocking Assignments](./Day%204%20%E2%80%93%20GLS,%20Blocking%20and%20Non-Blocking%20Assignments/)

### Day 05 – Optimization in Synthesis

- Studied the effect of different RTL coding styles on synthesis.
- Explored IF-ELSE and CASE statements.
- Analyzed incomplete conditional assignments and latch inference.
- Examined priority logic and multiplexer implementation.
- Explored CASE, CASEZ and CASEX behavior.
- Studied procedural and generate FOR loops.
- Analyzed MUX and DEMUX implementations.
- Explored Ripple Carry Adder (RCA) design and synthesis.

[View Day 05 – Optimization in Synthesis](./Day-05-Optimization-in-Synthesis/)

## Repository Structure

```text
RTL/
│
├── Day-00-Tool-Setup/
│   └── README.md
│
├── Day-01-Introduction-to-Verilog/
│   └── README.md
│
├── Day-02-Timing-Libraries-Hierarchical-Synthesis/
│   └── README.md
│
├── Day-03-Introduction-to-Optimizations/
│   └── README.md
│
├── Day 4 – GLS, Blocking and Non-Blocking Assignments/
│   └── README.md
│
├── Day-05-Optimization-in-Synthesis/
│   └── README.md
│
└── README.md
```

## Key Learnings

- Developed a practical understanding of Verilog RTL design.
- Gained experience in writing, compiling, and simulating Verilog modules.
- Learned to analyze digital circuit behavior through waveform visualization.
- Understood the role of timing libraries and standard cells in synthesis.
- Explored hierarchical synthesis and design flattening.
- Studied the impact of RTL coding styles on synthesized hardware.
- Understood latch inference, multiplexers, and combinational logic.
- Gained hands-on experience with Yosys and the SKY130 standard-cell library.

## Conclusion

This repository represents my hands-on learning journey in RTL Design and Synthesis using Verilog HDL and open-source VLSI tools.

Through the workshop experiments, I explored the relationship between RTL coding styles, simulation behavior, synthesis optimization, and gate-level implementation.

The practical exercises helped me build a foundation in digital design, RTL synthesis, and the tools used in the VLSI design flow.

---

**Author:** Jayeetaa Yeddanapalli

**GitHub:** [jayeetaa](https://github.com/jayeetaa)
