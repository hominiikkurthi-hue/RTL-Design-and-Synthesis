# Day 2 – Timing Libraries, Synthesis Hierarchy and Flip-Flop Optimization

## Objective

To explore standard-cell timing information, understand different synthesis architectures, examine sequential logic implementation, and study RTL optimization techniques using Yosys and the SKY130 standard-cell library.

---

## 1. Introduction to Timing Libraries

A Liberty (`.lib`) file describes the electrical and timing characteristics of standard cells available in a technology library.

It provides the information required by synthesis and timing analysis tools to understand the behavior of different standard cells.

**Important parameters:**
- Cell area
- Timing arcs and propagation delay
- Setup and hold time
- Input capacitance
- Output transition
- Power characteristics
- Process, voltage and temperature (PVT) conditions

**Process Corners:**
- TT – Typical Process
- FF – Fast Process
- SS – Slow Process

---

## 2. Hierarchical vs. Flat Synthesis

Synthesis converts RTL descriptions into a gate-level representation. The way modules are handled during synthesis affects the resulting design structure and optimization opportunities.

### A. Hierarchical Synthesis

Hierarchical synthesis maintains the original module boundaries of the Verilog design.

In this experiment, a design containing multiple modules, including OR and AND logic blocks, was analyzed using Yosys.

**What happens during synthesis?**
- The top-level module retains its submodule instances.
- Each module maintains its own logical structure.
- The connections between modules remain visible.
- Individual modules can be examined separately.

**Observation:** The synthesized design preserves the module hierarchy, making it easier to understand the relationship between different logic blocks.

**Screenshot:** 
<img width="959" height="539" alt="day2-2" src="https://github.com/user-attachments/assets/21b6ef28-0596-40d0-b087-6adfa8ee5059" />


### B. Flat Synthesis

Flat synthesis converts a hierarchical design into a single-level representation by removing module boundaries.

The Yosys `flatten` command was used to combine the submodules into the top-level design.

**What happens during synthesis?**
- Submodule instances are expanded into the parent module.
- Module boundaries are removed.
- Logic from different modules becomes available for optimization together.
- Redundant logic can be simplified across the expanded design.

**Observation:** The resulting design represents the combined logic without the original submodule hierarchy.

**Screenshot:** `2_multiple_flat.png`
<img width="959" height="539" alt="day2-3" src="https://github.com/user-attachments/assets/736400a1-3bed-4fe1-8879-57ad0dde19c3" />


### C. Submodule Synthesis

Submodule synthesis focuses on synthesizing an individual module independently instead of processing the complete top-level design.

In this experiment, an individual logic block was selected and analyzed using Yosys.

**What happens during synthesis?**
- A specific module is selected as the synthesis target.
- Only the logic belonging to that module is considered.
- The module's inputs and outputs are analyzed independently.
- Its logic implementation can be examined without the complete top-level design.

**Observation:** Individual module synthesis helps understand the internal logic of reusable design blocks and supports modular analysis.

**Screenshot:** `3_sub_module.png`
<img width="959" height="539" alt="day2-5submod" src="https://github.com/user-attachments/assets/c29702ec-c910-4752-bbad-eb3123a2349b" />


### Comparison of Synthesis Approaches

| Feature | Hierarchical | Flat | Submodule |
|---|---|---|---|
| Module boundaries | Preserved | Removed | Selected module analyzed independently |
| Design structure | Multiple levels | Single level | Individual block |
| Optimization scope | Within module boundaries | Across flattened logic | Selected module |
| Main purpose | Maintain design hierarchy | Global logic optimization | Independent block analysis |

---

## 3. Various Flip-Flop Coding Styles

Flip-flops are sequential elements used to store one bit of information. Their output changes according to clock events and control signals such as reset and set.

Different RTL coding styles describe how a flip-flop should respond to clock, reset, and set signals. During synthesis, Yosys identifies these behaviors and maps them into suitable standard-cell implementations.

### A. Asynchronous Reset

An asynchronous reset flip-flop resets its output immediately when the reset signal is activated, without waiting for a clock edge.

**Working principle:**
- When reset is asserted, the output becomes logic 0.
- Reset has priority over the clock.
- When reset is inactive, the flip-flop captures input data at the active clock edge.

**Verilog Code:**

```verilog
module dff_async_reset (
    input clk,
    input reset,
    input d,
    output reg q
);

always @(posedge clk or posedge reset)
begin
    if (reset)
        q <= 1'b0;
    else
        q <= d;
end

endmodule
```

**Synthesis observation:**

Yosys identifies the asynchronous reset behavior and maps the design to a suitable resettable flip-flop. If the available library cell has an active-low reset pin, additional inversion logic may be required to match the RTL reset polarity.

**Screenshot:** `4_async_res.png`

### B. Asynchronous Set

An asynchronous set flip-flop forces its output to logic 1 immediately when the set signal is activated, independently of the clock.

**Working principle:**
- When set is asserted, the output becomes logic 1.
- The operation does not depend on a clock transition.
- When set is inactive, the flip-flop captures input data at the active clock edge.

**Verilog Code:**

```verilog
module dff_async_set (
    input clk,
    input set,
    input d,
    output reg q
);

always @(posedge clk or posedge set)
begin
    if (set)
        q <= 1'b1;
    else
        q <= d;
end

endmodule
```

**Synthesis observation:**

Yosys recognizes the asynchronous set condition and maps the RTL to a suitable settable flip-flop implementation from the target standard-cell library.

**Screenshot:** `5_async_set.png`

### C. Synchronous Reset

A synchronous reset flip-flop resets its output only at the active clock edge. Unlike asynchronous reset, activating the reset signal alone does not immediately change the output.

**Working principle:**
- When reset is active at the clock edge, the output becomes logic 0.
- When reset is inactive, the input data is captured.
- Reset operation is synchronized with the clock.

**Verilog Code:**

```verilog
module dff_sync_reset (
    input clk,
    input reset,
    input d,
    output reg q
);

always @(posedge clk)
begin
    if (reset)
        q <= 1'b0;
    else
        q <= d;
end

endmodule
```

**Synthesis observation:**

Yosys implements synchronous reset behavior using a suitable flip-flop and associated logic. Depending on the target library, the reset functionality may be implemented using a dedicated synchronous-reset cell or additional combinational logic at the flip-flop input.

**Screenshot:** `6_sync_reset.png`

### Comparison of Flip-Flop Types

| Parameter | Async Reset | Async Set | Sync Reset |
|---|---|---|---|
| Control signal | Reset | Set | Reset |
| Output during activation | 0 | 1 | Changes at clock edge |
| Clock required for control action | No | No | Yes |
| Main purpose | Initialize to 0 | Initialize to 1 | Synchronous initialization |

---

## 4. RTL Logic Optimization

RTL optimization reduces unnecessary hardware complexity by expressing arithmetic operations in simpler forms.

### A. Multiplication by 2

For a 3-bit unsigned input `a`, multiplying by 2 is equivalent to shifting the bits one position to the left.

**Original expression:**

```verilog
assign y = a * 2;
```

**Optimized expression:**

```verilog
assign y = {a, 1'b0};
```

The least significant bit is filled with zero. This operation can be implemented using simple wiring instead of a general-purpose multiplier.

**Screenshot:** `7_mult_2.png`

### B. Multiplication by 9

Multiplication by 9 can be expressed as the sum of the input and its value shifted left by three positions.

Since:

`9 × a = (8 × a) + a`

**Original expression:**

```verilog
assign y = a * 9;
```

**Equivalent expression:**

```verilog
assign y = (a << 3) + a;
```

For a 3-bit input, this can also be represented as:

```verilog
assign y = {a, a};
```

This representation demonstrates how constant multiplication can be simplified using bit concatenation and arithmetic properties.

**Screenshot:** `8_mult_8.png`

---

## 5. Key Learnings

- Explored the structure and importance of Liberty timing libraries.
- Understood how hierarchical and flat synthesis represent digital designs.
- Analyzed individual submodules using Yosys.
- Studied asynchronous reset, asynchronous set, and synchronous reset flip-flops.
- Observed how RTL sequential behavior is translated into standard-cell implementations.
- Learned to simplify constant multiplication operations through RTL optimization.

## Tools and Technologies

**Verilog HDL | Yosys | SKY130 PDK | Liberty Timing Library**
