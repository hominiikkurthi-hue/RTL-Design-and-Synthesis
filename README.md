
# RTL Design and Logic Synthesis Using Yosys

## 1. Introduction

This repository documents the process of designing, simulating, verifying, and synthesizing digital circuits using Verilog HDL and open-source EDA tools.

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
| Verilog HDL | RTL Design |
| Icarus Verilog | Functional Simulation |
| GTKWave | Waveform Analysis |
| Yosys | RTL Synthesis |
| SKY130 PDK | Standard Cell Technology |
| Liberty (.lib) | Standard Cell Characterization |
| Oracle VirtualBox | Virtualization Platform |
| Linux (VSD Squadron) | Development Environment |

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

### RTL Design Example

A simple multiplexer can be described using Verilog.

The multiplexer selects one of two input signals depending on the select signal.

**Design file:** `good_mux.v`

module good_mux(input i0, input i1, input sel, output reg y);

always @(*)
begin
    if(sel)
        y <= i1;
    else
        y <= i0;
end

endmodule

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

Yosys is an open-source RTL synthesis framework used to convert Verilog hardware descriptions into optimized logic representations and gate-level netlists.

### Why is Yosys Used?

- Reads and analyzes Verilog RTL.
- Converts behavioral descriptions into logic.
- Performs logic optimization.
- Supports technology mapping.
- Generates gate-level Verilog netlists.
- Provides synthesis statistics.

### Basic Yosys Synthesis Flow

1. Read Verilog RTL files.
2. Check design hierarchy.
3. Convert processes into logic.
4. Perform logic optimization.
5. Execute synthesis.
6. Perform technology mapping.
7. Generate the gate-level netlist.
8. Analyze synthesis statistics.


## 8. Understanding Logic Synthesis

Logic synthesis is the process of converting an RTL description into an equivalent network of logic gates and sequential elements.

### Stages of Logic Synthesis

**1. RTL Reading:** The synthesis tool reads the Verilog source code and identifies modules, ports, and signals.

**2. Process Conversion:** Behavioral statements are converted into internal logic representations.

**3. Logic Optimization:** Redundant logic is removed and Boolean expressions are simplified.

**4. Technology Mapping:** Optimized logic is mapped to standard cells available in the target technology library.

**5. Netlist Generation:** The final synthesized design is written as a gate-level Verilog netlist.

## 9. Introduction to Liberty (.lib) Files

A Liberty file (`.lib`) is a standard library description file containing the characteristics of standard cells used in digital IC design.

### Information Available in a .lib File

- Standard-cell names and logic functions.
- Input and output pin information.
- Timing arcs and propagation delays.
- Setup and hold timing constraints.

### Importance of Liberty Files

The `.lib` file provides the synthesis tool with information about the available standard cells in a technology.

For SKY130, it contains descriptions of cells such as NAND, NOR, AND, OR, inverters, flip-flops, buffers, and multiplexers.

Yosys uses this information during technology mapping to select suitable standard cells.

**Note:** A Liberty file describes logical, timing, and electrical characteristics. It does not contain the physical layout geometry of the cells.

## 10. RTL Synthesis and Technology Mapping Using Yosys

Yosys is used to synthesize the Verilog RTL design and map the optimized logic into standard cells from the SKY130 technology library.

Run the following commands at the Yosys prompt.

### Step 1: Read the SKY130 Standard-Cell Library

```tcl
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

This loads the standard-cell definitions for synthesis and technology mapping.

### Step 2: Read the RTL Design

```tcl
read_verilog good_mux.v
```

This command reads the Verilog source code into Yosys.

### Step 3: Synthesize the Design

```tcl
synth -top good_mux -noabc
```

This performs RTL synthesis and logic optimization while leaving technology mapping to the ABC step.

### Step 4: Map Logic to Standard Cells

```tcl
abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

ABC performs logic optimization and maps the design into standard cells using the specified Liberty library.

### Step 5: View the Synthesized Logic

```tcl
show
```

This command generates a graphical representation of the synthesized design, showing the logic cells and their connections.

**Synthesized Logic Diagram:**

<img width="959" height="535" alt="synthesis" src="https://github.com/user-attachments/assets/f611dd0c-96c8-4c2c-973f-fe01a2cd409a" />


### Step 6: Display Synthesis Statistics

```tcl
stat -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib
```

This displays the standard-cell usage and synthesis statistics.

### Step 7: Generate the Gate-Level Netlist

```tcl
write_verilog -noattr mapped_good_mux.v
```

This generates the technology-mapped Verilog netlist.

<img width="959" height="539" alt="syn1" src="https://github.com/user-attachments/assets/62c1ddaf-ca7a-4c05-9bf6-d9c4ca20e638" />


## 13. Results and Observations

The practical implementation demonstrates the following results:

| Stage | Observation |
|---|---|
| RTL Design | Verilog module created |
| Testbench | Input stimulus applied |
| Simulation | Functional behavior verified |
| Waveform Analysis | Signal transitions observed |
| Yosys Synthesis | RTL converted into logic |
| Optimization | Logic optimized |
| Technology Mapping | Standard cells selected |
| Netlist Generation | Gate-level Verilog generated |
| Verification | Synthesized design checked |

## 16. Conclusion

This project provides practical understanding of RTL design, functional simulation, and logic synthesis using open-source VLSI tools.

It demonstrates how Verilog RTL is processed, optimized, and converted into a gate-level netlist using Yosys and the SKY130 standard-cell library.

The project also introduces Liberty files, technology mapping, and post-synthesis verification, providing a foundation for further learning in ASIC design and physical design.

## 17. References

- [Yosys Documentation](https://yosyshq.readthedocs.io/)
- [Icarus Verilog Documentation](https://steveicarus.github.io/iverilog/)
- [GTKWave](https://gtkwave.sourceforge.net/)
- [SkyWater SKY130 PDK](https://github.com/google/skywater-pdk)
- [Reference Repository](https://github.com/Ragavarshini24/SKY130A-CMOS-Inverter.git)
