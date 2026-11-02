# Day 03 – Introduction to Optimizations

## Introduction

Logic optimization is an important part of the RTL synthesis process. It simplifies the digital circuit by removing unnecessary logic while maintaining the original functionality.

In this session, I explored constant propagation, Boolean logic optimization, combinational logic optimization, and synthesis of multiple Verilog modules using Yosys and the SKY130 standard-cell library.

## Learning Objectives

- Understand how constant propagation and Boolean optimization simplify digital circuits.
- Explore combinational logic and multiple-module synthesis using Yosys.

---

## 1. Introduction to Optimizations

### Constant Propagation

Constant propagation is an optimization technique in which fixed values such as logic `0` and logic `1` are used to simplify logic expressions.

For example, when a multiplexer has one input connected to a constant value, its logic can sometimes be simplified into a basic logic gate.

### Boolean Logic Optimization

Boolean logic optimization simplifies complex logic expressions using Boolean algebra and minimization techniques such as Karnaugh Maps and the Quine–McCluskey method.

These techniques help reduce unnecessary logic gates and simplify the hardware implementation.

---

## 2. Combinational Logic Optimizations

In this section, different Verilog designs are synthesized using Yosys to observe how combinational logic is optimized.

### 2.1 opt_check

This experiment demonstrates how a multiplexer with one constant input can be simplified into an equivalent logic gate.

**Observation:** The multiplexer logic is optimized into a simpler AND gate.

<img width="959" height="539" alt="day3-1" src="https://github.com/user-attachments/assets/7b564ada-fc1a-42e6-85b1-407225a18552" />


### 2.2 opt_check2

This experiment uses a multiplexer with one input connected to logic `1`.

**Observation:** The multiplexer is simplified into an equivalent OR gate.

<img width="959" height="538" alt="day3-2" src="https://github.com/user-attachments/assets/f5467530-36ee-4f2f-bf84-ad6f42e21f1f" />


### 2.3 opt_check3

This experiment contains two multiplexers connected through conditional logic.

**Observation:** The logic is simplified into a single three-input AND gate.

<img width="959" height="536" alt="day3-3" src="https://github.com/user-attachments/assets/49a9ff01-4b7b-4d7c-bdfa-8d0bc8263777" />


### 2.4 opt_check4

This experiment uses nested conditional expressions and Boolean operations.

**Observation:** The synthesized circuit can be examined to understand how Yosys simplifies the conditional logic and maps it into standard cells.

<img width="959" height="534" alt="day3-4" src="https://github.com/user-attachments/assets/e4a9bf03-a2f4-43fa-b851-fb0362e8a79b" />


---

## 3. Multiple Module Synthesis

This section focuses on understanding how Yosys handles designs containing multiple Verilog modules. The experiments help examine module hierarchy and the effect of synthesis optimization on the overall design.

### 3.1 multiple_modules_opt

This experiment explores synthesis optimization in a design containing multiple modules.

The design is analyzed to understand how the individual modules are represented and how the synthesis process handles the module hierarchy.

<img width="959" height="538" alt="day3-5" src="https://github.com/user-attachments/assets/795ee6ac-73ee-427b-b2c0-ad961c9d6a31" />


### 3.2 multiple_modules_opt2

This experiment continues the multiple-module synthesis analysis using a different optimization setup.

The synthesized output is examined to observe how Yosys processes the modules and represents the resulting logic.

<img width="958" height="539" alt="day3-6" src="https://github.com/user-attachments/assets/649df0eb-fb02-4324-b32b-43446eda1408" />


---
## 3. Sequential Logic Optimizations

Sequential logic optimization is used to simplify digital circuits containing flip-flops and other storage elements. Unlike combinational circuits, sequential circuits depend on the current inputs, clock transitions and previously stored values.

The main concept explored in this section is **Sequential Constant Propagation**. It identifies whether a flip-flop output always produces a constant value. If a register is unnecessary, the synthesis tool can remove it and replace it with a constant connection.

In this section, five different Verilog designs are simulated and synthesized using Icarus Verilog and Yosys to understand how sequential logic is optimized.

### 3.1 dff_const1 – Flip-Flop with Asynchronous Reset

The `dff_const1` design contains a flip-flop with an asynchronous active-high reset.

When reset is high, the output `q` is assigned logic `0`. When reset is low, the output is assigned logic `1` at the next positive edge of the clock.

**Simulation Observation:**

When reset becomes active, the output changes to `0` immediately. When reset is deactivated, the output changes to `1` only at the next positive clock edge.
**Simulation of dff_const1 – RTL Code, Testbench Execution and Waveform**
<img width="959" height="538" alt="day3-7seq" src="https://github.com/user-attachments/assets/e0e279cf-2e5b-43eb-994d-ef79cb5fc556" />

**Synthesis Observation:**

The flip-flop is retained because the output can have both logic `0` and logic `1` values depending on the reset and clock conditions. Therefore, it cannot be replaced with a constant connection.

The SKY130 library implementation uses an active-low reset flip-flop, so an inverter is used to obtain the required reset polarity.

**Screenshots**



### 3.2 dff_const2 – Sequential Constant Propagation

The `dff_const2` design assigns logic `1` to the output `q` during both reset and normal operating conditions.

**Simulation Observation:**

The output remains at logic `1` regardless of the clock and reset conditions.

**Synthesis Observation:**

Since the output is always constant, the flip-flop is unnecessary. Yosys identifies this condition and optimizes the sequential element away, leaving a constant logic connection.

**Screenshots**

**Simulation of dff_const2 – RTL Code, Testbench Execution and Waveform**

<img width="959" height="539" alt="day3-8seq2" src="https://github.com/user-attachments/assets/ba12295f-8ea5-4b3f-be48-c840def2d7b0" />

<img width="959" height="539" alt="day3-9seq" src="https://github.com/user-attachments/assets/ee7dad0e-feb5-44e4-8f13-709c073b5c51" />


### 3.3 dff_const3 – Sequential Dependency Between Flip-Flops

The `dff_const3` design contains two flip-flops. The output of one flip-flop is connected to the input of another flip-flop.

During reset, `q` is assigned `1` and `q1` is assigned `0`. During normal operation, `q1` is assigned `1`, while `q` receives the previous stored value of `q1`.

**Simulation Observation:**

When reset is deactivated, `q1` becomes `1` at the next positive clock edge. The output `q` receives the previous value of `q1` and changes to `1` at a subsequent clock edge.

This demonstrates the dependency between the two sequential elements.

**Synthesis Observation:**

Both flip-flops are retained because the outputs depend on their stored values and clock transitions. The sequential behavior cannot be replaced with a constant connection.

**Screenshots**
**GTKWave observation**

<img width="959" height="538" alt="day3-10" src="https://github.com/user-attachments/assets/8400fd32-4271-47dc-9ad3-ad20cc19084d" />


### 3.4 dff_const4 – Constant Sequential Optimization

The `dff_const4` design contains two flip-flops with reset and normal operating conditions assigning logic `1` to their respective outputs.

During reset, both `q` and `q1` are assigned `1`. During normal operation, `q1` is assigned `1` and `q` receives the value of `q1`.

**Simulation Observation:**

Both outputs remain at logic `1` regardless of the clock and reset conditions.

**Synthesis Observation:**

Since both sequential outputs are constant, the flip-flops are unnecessary. Yosys optimizes the sequential elements away and replaces them with constant connections.

**Screenshots**

<img width="959" height="538" alt="day3-12" src="https://github.com/user-attachments/assets/d087f7bd-b29a-44d7-87ff-0eaa782802df" />


### 3.5 dff_const5 – Registers Preserved During Synthesis

The `dff_const5` design contains two flip-flops with both outputs assigned logic `0` during reset. During normal operation, `q1` is assigned logic `1`, and `q` receives the stored value of `q1`.

**Simulation Observation:**

During reset, both outputs become `0`. After reset is deactivated, `q1` becomes `1` at the next positive clock edge. The output `q` follows the stored value of `q1` at a subsequent clock edge.

**Synthesis Observation:**

Unlike the previous constant-output experiment, the outputs are not always constant. The sequential behavior must be preserved, so the registers remain in the synthesized design.

**Screenshots**

<img width="959" height="536" alt="day3-12 1" src="https://github.com/user-attachments/assets/2d36e14d-edc8-46e2-b8e1-d40d459e06e0" />


---

## 4. Sequential Optimizations for Unused Outputs

Unused-output optimization is another important technique in sequential synthesis.

A digital design may contain several registers, but not every register necessarily contributes to the required output. If some register bits do not affect the primary output, the synthesis tool can remove the unnecessary logic.

In this section, two counter designs are used to understand how Yosys identifies unused counter bits and optimizes the circuit.

### 4.1 counter_opt – Optimization of Unused Counter Bits

The `counter_opt` design describes a 3-bit up counter with an asynchronous reset.

The counter increments by one at every positive edge of the clock. However, the output is connected only to the least significant bit of the counter.

```verilog
assign q = count[0];
```

The other two counter bits, `count[1]` and `count[2]`, are not used to generate the output.

**Synthesis Observation:**

Although the original design contains a 3-bit counter, Yosys identifies that only the least significant bit is required for the output.

The unused counter bits and their associated logic are removed. The optimized circuit requires only one flip-flop instead of three.

**Screenshots**

<img width="959" height="539" alt="day3-13" src="https://github.com/user-attachments/assets/e6fbfe7d-2339-47d5-8746-c75cff8ea617" />


### 4.2 counter_opt2 – Counter with All Bits Used in Output Logic

The `counter_opt2` experiment modifies the output condition of the previous counter design.

Instead of using only the least significant bit, the output is generated by comparing the complete 3-bit counter value with a constant value.

```verilog
assign q = (count[2:0] == 3'b100);
```

Here, all three counter bits are required to determine whether the counter has reached the value `100`.

**Synthesis Observation:**

Since all three counter bits contribute to the output comparison, Yosys must retain the required counter registers and associated logic.

Unlike `counter_opt`, the unused-bit optimization cannot remove the upper counter bits because they affect the output.

**Screenshots**

<img width="959" height="538" alt="day3-14" src="https://github.com/user-attachments/assets/3a6dc505-ce79-4377-b3b0-377c9b38d012" />


## 5. Simulation and Synthesis

The following tools and commands are used to simulate the sequential designs and examine their synthesized implementations.

### 5.1 Verilog Simulation Using Icarus Verilog

Move to the Verilog files directory:

```bash
cd ~/sky130RTLDesignAndSynthesisWorkshop/verilog_files
```

Use the following command sequence for each experiment. Change the design and testbench filenames accordingly.

```bash
iverilog dff_const1.v tb_dff_const1.v
./a.out
gtkwave tb_dff_const1.vcd
```

Repeat the same procedure for `dff_const2`, `dff_const3`, `dff_const4` and `dff_const5` using their corresponding testbench files.

For the counter designs, use their corresponding testbenches and VCD files if available.

### 5.2 Synthesis Using Yosys

Start Yosys from the Verilog files directory:

```bash
yosys
```

Run the following commands inside the Yosys prompt:

```tcl
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

read_verilog dff_const1.v

synth -top dff_const1 -noabc

dfflibmap -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

clean

show
```

### 5.3 Understanding the Yosys Commands

| Command | Description |
|---|---|
| `read_liberty -lib` | Loads the SKY130 standard-cell library. |
| `read_verilog` | Reads the Verilog source file. |
| `synth -top -noabc` | Performs synthesis without ABC technology mapping. |
| `dfflibmap -liberty` | Maps internal flip-flop cells to available library cells. |
| `abc -liberty` | Maps combinational logic using the SKY130 library. |
| `clean` | Removes unused design elements. |
| `show` | Displays the synthesized circuit. |

---

## 6. Key Observations

- Sequential constant propagation identifies flip-flops whose outputs always remain constant.
- Flip-flops are retained when their outputs depend on clock transitions and stored values.
- Sequential dependencies between registers can prevent unnecessary removal of flip-flops.
- Unused counter bits can be removed when they do not contribute to the primary output.
- When all counter bits are required for output logic, the necessary registers are retained.
- Yosys optimizes the circuit based on the functionality required at the outputs.

## Conclusion

Through these experiments, I gained practical understanding of sequential logic optimization and unused-output optimization using Yosys. By simulating and synthesizing different flip-flop and counter designs, I observed how constant sequential elements are removed, required registers are preserved, and unused counter bits are eliminated.

The synthesized schematics helped me understand how RTL designs are optimized and mapped into the SKY130 standard-cell library, reducing unnecessary hardware while maintaining the required functionality.

## Tools Used

- Verilog HDL
- Icarus Verilog
- GTKWave
- Yosys
- SKY130 HD Standard-Cell Library
- Graphviz
