
# RTL Design and Logic Synthesis Using Yosys

## 1. Introduction

This repository documents the process of designing, simulating, verifying, and synthesizing digital circuits using Verilog HDL and open-source EDA tools.

The project focuses on understanding how a digital design written in RTL (Register Transfer Level) is converted into a gate-level netlist through logic synthesis.

The complete workflow includes RTL coding, testbench development, functional simulation, waveform analysis, synthesis using Yosys, and verification of the synthesized design.

The objective is to understand the relationship between RTL descriptions, digital logic, simulation results, and hardware implementation.

## 2. Objectives

- Understand the fundamentals of RTL design using Verilog HDL.
- Develop synthesizable Verilog modules.
- Create testbenches to verify the functionality of RTL designs.
- Perform functional simulation and analyze waveforms.
- Understand the working principles of the Yosys synthesis tool.
- Convert RTL descriptions into gate-level representations.
- Understand logic synthesis and technology mapping.
- Explore the purpose of Liberty (`.lib`) files in technology mapping.
- Verify the functionality of the synthesized netlist.

## 3. Tools and Technologies

| Tool / Technology | Purpose |
|---|---|
| Verilog HDL | Hardware description language for RTL design |
| Icarus Verilog | Verilog compilation and simulation |
| GTKWave | Waveform visualization and analysis |
| Yosys | RTL synthesis and logic optimization |
| SKY130 PDK | Open-source 130 nm semiconductor technology |
| Liberty (.lib) | Standard-cell timing and characterization information |
| Ubuntu Linux | Development environment |

## 4. Repository Setup

The RTL Design and Synthesis Workshop repository is obtained from GitHub and configured in the Ubuntu environment. This repository contains the Verilog source files and practical exercises required to understand RTL design, simulation, and synthesis.

### Step 1: Clone the GitHub Repository

The Git repository is downloaded using the `git clone` command.

```bash
git clone https://github.com/kunalg123/sky130RTLDesignAndSynthesisWorkshop.git
```
<img width="955" height="539" alt="screenshot1" src="https://github.com/user-attachments/assets/765d0950-8c2e-4f43-9be0-dfe47baf4700" />



### Step 2: Navigate to the Project Directory

Move into the cloned repository to access the workshop files.

```bash
cd sky130RTLDesignAndSynthesisWorkshop
```

### Step 3: Explore the Repository Contents

List the available files and directories to understand the project structure.

```bash
ls
```

The Verilog source files required for RTL design and simulation are available in the `verilog_files` directory.

```bash
cd verilog_files
ls
```
<img width="953" height="536" alt="scre2" src="https://github.com/user-attachments/assets/254b7be5-3213-477c-94ba-ad3414330847" />


### Repository Setup Screenshot

The following screenshot shows the successful cloning of the GitHub repository and access to the project directory in Ubuntu.

<img width="953" height="539" alt="scren6" src="https://github.com/user-attachments/assets/99f86174-5e93-4282-a5e1-68edb31688e7" />


## 4. Understanding RTL Design

RTL stands for Register Transfer Level. It describes the behavior and data flow of a digital circuit using a hardware description language such as Verilog.

RTL specifies how data moves between registers and how combinational and sequential logic processes that data.

A Verilog design generally contains:

- Module declaration
- Input and output ports
- Internal signals
- Combinational logic
- Sequential logic
- Parameters and generate blocks, when required


### RTL Design Example

A simple multiplexer can be described using Verilog.

The multiplexer selects one of two input signals depending on the select signal.

**Design file:** `good_mux.v`

```verilog
module good_mux (
    input i0,
    input i1,
    input sel,
    output y
);

assign y = sel ? i1 : i0;

endmodule
```

**Explanation:**

- `i0` and `i1` are the input signals.
- `sel` is the selection signal.
- `y` is the output.
- When `sel = 0`, the output follows `i0`.
- When `sel = 1`, the output follows `i1`.

**ScreenShot of RTL code**

<img width="956" height="537" alt="scr11" src="https://github.com/user-attachments/assets/487d279e-52c2-4ce7-9620-1ac1f5aa1f0e" />



## 5. Understanding the Testbench

A testbench is a separate Verilog module used to verify the functionality of the RTL design.

It applies different input combinations to the Design Under Test (DUT) and observes the corresponding outputs.

A testbench does not represent physical hardware. It is used only for simulation and verification.

### Main Components of a Testbench

1. **DUT Instantiation:** Connects the RTL design to the testbench.
2. **Input Stimulus:** Generates different input combinations.
3. **Output Monitoring:** Observes the output response.
4. **Verification:** Checks whether the output matches the expected result.
5. **Simulation Control:** Defines the simulation duration and completion.

### Testbench Example

**Testbench file:** `tb_good_mux.v`

```verilog
module tb_good_mux;

reg i0, i1, sel;
wire y;

good_mux dut (
    .i0(i0),
    .i1(i1),
    .sel(sel),
    .y(y)
);

initial begin

    $dumpfile("good_mux.vcd");
    $dumpvars(0, tb_good_mux);

    i0 = 0; i1 = 0; sel = 0;
    #10;

    i0 = 0; i1 = 1; sel = 0;
    #10;

    i0 = 1; i1 = 0; sel = 1;
    #10;

    i0 = 1; i1 = 1; sel = 1;
    #10;

    $finish;

end

endmodule
```
**ScreenShot of Testbenchfile**
<img width="959" height="532" alt="scre10" src="https://github.com/user-attachments/assets/9d176657-6a2a-469d-aa65-b62d041c9993" />


## 6. Functional Simulation

Functional simulation checks whether the RTL design behaves according to the intended logic before synthesis.

The simulator executes the Verilog design along with its testbench and produces the output responses for the applied input combinations.

### What is a Simulator?

A simulator is a software tool that models the behavior of digital hardware over time.

It helps identify:

- Incorrect logical conditions.
- Unexpected output behavior.
- Missing input combinations.
- Functional errors in the RTL code.

### Simulation Using Icarus Verilog

Icarus Verilog is an open-source Verilog compiler and simulator.

**Step 1: Compile the design and testbench**

```bash
iverilog -o mux_sim good_mux.v tb_good_mux.v
```


**Step 3: Generate and view the waveform**

```bash
gtkwave good_mux.vcd
```

### Waveform Analysis

GTKWave is used to visualize the simulation results.

It displays the input signals, select signal, and output signal over simulation time.

The waveform helps confirm whether the output follows the expected multiplexer behavior.

**Simulation Waveform:**

<img width="955" height="537" alt="scren8" src="https://github.com/user-attachments/assets/11c63ea0-98c8-4ba6-8167-a9f8fc8a7cd2" />




## 7. Introduction to Yosys

Yosys is an open-source RTL synthesis framework used to convert Verilog hardware descriptions into optimized gate-level logic representations.

It reads the RTL source code, processes the design, performs logic optimization, and generates a synthesized netlist.

### Why is Yosys Used?

Yosys helps to:

- Read and analyze Verilog RTL.
- Convert procedural descriptions into internal logic.
- Perform logic optimization.
- Identify combinational and sequential elements.
- Map logic into standard-cell libraries.
- Generate gate-level Verilog netlists.
- Analyze cell usage and synthesis statistics.

### Basic Yosys Synthesis Flow

1. Read RTL source files.
2. Check design hierarchy.
3. Convert processes into logic.
4. Perform optimization.
5. Execute synthesis.
6. Perform technology mapping.
7. Generate the gate-level netlist.
8. Analyze synthesis statistics.

**Yosys Flow Diagram:**

```text
       Verilog RTL
            |
            v
       Read Verilog
            |
            v
      Design Hierarchy
            |
            v
       Process Logic
            |
            v
       Optimization
            |
            v
      Logic Synthesis
            |
            v
    Technology Mapping
            |
            v
     Gate-Level Netlist
            |
            v
     Synthesis Statistics
```

## 8. Understanding Logic Synthesis

Logic synthesis is the process of converting an RTL description into an equivalent network of logic gates and sequential elements.

During synthesis, the tool interprets the behavioral and structural descriptions and transforms them into hardware logic.

### Stages of Logic Synthesis

**1. RTL Reading**

The synthesis tool reads the Verilog source code and identifies modules, ports, signals, and design structures.

**2. Process Conversion**

Behavioral Verilog statements are converted into internal logic representations.

**3. Logic Optimization**

Unnecessary logic is removed, redundant expressions are simplified, and the circuit is optimized.

**4. Technology Mapping**

The optimized logic is mapped to available standard cells from a target technology library.

**5. Netlist Generation**

The final mapped design is written as a gate-level Verilog netlist.

## 9. Introduction to Liberty (.lib) Files

A Liberty file (`.lib`) is a standard library description file that contains information about standard cells used in digital integrated circuit design.

It provides the synthesis and timing-analysis tools with the electrical and timing characteristics of individual cells.

### What Information Does a .lib File Contain?

A Liberty file generally includes:

- Standard-cell names.
- Logic functions.
- Input and output pin information.
- Timing arcs and propagation delays.
- Setup and hold timing information.
- Clock-to-output timing.
- Input capacitance.
- Output drive characteristics.
- Power-related information.
- Operating conditions.

### Why is a .lib File Important?

The `.lib` file helps the synthesis tool understand which standard cells are available in a particular technology.

For example, when synthesizing a design using the SKY130 technology, the library provides information about cells such as:

- AND gates
- OR gates
- NAND gates
- NOR gates
- Inverters
- Flip-flops
- Buffers
- Multiplexers

Yosys uses the library information during technology mapping to select appropriate standard cells for the optimized logic.

**Important:** A Liberty file is not a layout file. It describes the logical, timing, and electrical characteristics of standard cells, not their physical geometries.

## 10. Performing RTL Synthesis Using Yosys

Yosys can be used to synthesize a Verilog design and generate a gate-level representation.

### Step 1: Launch Yosys

```bash
yosys
```

### Step 2: Read the RTL Design

```tcl
read_verilog good_mux.v
```

This command reads the Verilog source code into Yosys.


### Step 3: Perform Synthesis

```tcl
synth -top good_mux
```

This performs the synthesis process, including logic transformation and optimization.

### Step 6: Generate the Synthesized Netlist

```tcl
write_verilog -noattr synth_good_mux.v
```

This generates a Verilog file containing the synthesized logic representation.

## 11. Technology Mapping Using a .lib File

Technology mapping converts the optimized generic logic into actual standard cells available in the selected technology library.

For SKY130, the corresponding standard-cell Liberty file is used.


## 12. Verification of the Synthesized Design

After synthesis, it is important to verify that the generated gate-level netlist preserves the intended functionality of the original RTL design.

This step checks whether the synthesized hardware produces the expected output for the applied testbench inputs.

### Verification Procedure

1. Generate the synthesized gate-level Verilog netlist.
2. Use the same testbench developed for the original RTL.
3. Compile the synthesized netlist with the testbench.
4. Execute the simulation.
5. Compare the output responses with the original RTL simulation.

### Commands

```bash
iverilog -o synth_sim mapped_good_mux.v tb_good_mux.v

vvp synth_sim
```

For waveform analysis:

```bash
gtkwave good_mux.vcd
```

**Verification Objective:**

The output of the synthesized design should match the expected functional behavior of the original RTL design.

For stronger verification, the RTL and synthesized netlist can also be compared using formal equivalence checking.

## 13. Understanding the Complete RTL-to-Synthesis Flow

The complete workflow connects the different stages of digital design, beginning with Verilog coding and ending with a synthesized standard-cell netlist.

```text
       RTL Design
            |
            v
       Testbench
            |
            v
    Functional Simulation
            |
            v
     Waveform Analysis
            |
            v
       RTL Validation
            |
            v
        Yosys Tool
            |
            v
      Logic Synthesis
            |
            v
      Optimization
            |
            v
     Technology Mapping
            |
            v
       SKY130 .lib
            |
            v
   Gate-Level Netlist
            |
            v
  Post-Synthesis Verification
```

## 14. Results and Observations

The following observations can be documented after completing the simulation and synthesis stages.

| Stage | Observation |
|---|---|
| RTL Design | Verilog design successfully created |
| Testbench | Input stimulus applied |
| Functional Simulation | RTL behavior examined |
| Waveform Analysis | Signal transitions observed |
| Yosys Synthesis | RTL converted into logic representation |
| Logic Optimization | Unnecessary logic optimized |
| Technology Mapping | Logic mapped to standard cells |
| Netlist Generation | Gate-level Verilog generated |
| Post-Synthesis Verification | Functional behavior checked |

*Update the observations according to your actual execution results.*

## 15. Screenshots and Project Documentation

The following screenshots can be included to document the practical implementation.

1. RTL Verilog source code.
2. Testbench source code.
3. Functional simulation terminal output.
4. GTKWave simulation waveform.
5. Yosys synthesis execution.
6. Yosys synthesis statistics.
7. SKY130 Liberty file.
8. Technology mapping results.
9. Synthesized gate-level netlist.
10. Post-synthesis verification waveform.

## 16. Conclusion

This project provides practical exposure to the fundamental stages of RTL design and logic synthesis using open-source VLSI tools.

Through Verilog coding, testbench development, functional simulation, and waveform analysis, the functionality of digital circuits can be understood and verified.

The use of Yosys demonstrates how RTL descriptions are transformed into optimized logic and mapped to standard cells using technology library information.

Understanding the role of Liberty files and synthesized netlists provides a foundation for further learning in ASIC design, synthesis, static timing analysis, and physical design.

## 17. Reference

- [SKY130A CMOS Inverter – Reference Repository](https://github.com/Ragavarshini24/SKY130A-CMOS-Inverter.git)
- [Yosys Official Documentation](https://yosyshq.readthedocs.io/)
- [Icarus Verilog Documentation](https://steveicarus.github.io/iverilog/)
- [GTKWave Documentation](https://gtkwave.sourceforge.net/)
- [SkyWater SKY130 PDK](https://github.com/google/skywater-pdk)

This documentation is independently written for the RTL design and synthesis workflow.
