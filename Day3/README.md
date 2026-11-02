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

## 4. Yosys Commands

The following commands are used for reading the SKY130 library, loading Verilog files, performing synthesis, and viewing the optimized circuits.

### Open Yosys

```bash
cd ~/sky130RTLDesignAndSynthesisWorkshop/verilog_files
yosys
```

### Synthesis and Technology Mapping

Run these commands inside the Yosys prompt:

```tcl
read_liberty -lib ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

read_verilog opt_check.v

synth -top opt_check -noabc

opt_clean -purge

abc -liberty ../lib/sky130_fd_sc_hd__tt_025C_1v80.lib

show
```

For each experiment, change the Verilog filename and top-module name to match the design being synthesized.

| Experiment | Verilog file | Top module |
|---|---|---|
| opt_check | `opt_check.v` | `opt_check` |
| opt_check2 | `opt_check2.v` | `opt_check2` |
| opt_check3 | `opt_check3.v` | `opt_check3` |
| opt_check4 | `opt_check4.v` | `opt_check4` |

For the multiple-module experiments, use the corresponding Verilog source files and the appropriate top module from your design.

### Command Description

| Command | Description |
|---|---|
| `read_liberty -lib` | Loads the SKY130 standard-cell library. |
| `read_verilog` | Reads the Verilog source file. |
| `synth -top -noabc` | Performs synthesis without ABC technology mapping. |
| `opt_clean -purge` | Removes unused logic and design elements. |
| `abc -liberty` | Performs technology mapping using the SKY130 library. |
| `show` | Displays the synthesized circuit. |

---

