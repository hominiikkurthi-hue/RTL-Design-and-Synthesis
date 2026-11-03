# Day 04 – Gate-Level Simulation, Synthesis-Simulation Mismatch and Blocking vs Non-Blocking Assignments

## Overview

Day 04 focuses on understanding how Verilog RTL code behaves during simulation and how the synthesized hardware behaves after converting the RTL design into a gate-level netlist.

The main objective of this session is to understand Gate-Level Simulation (GLS), identify the reasons behind synthesis-simulation mismatches, and learn the correct usage of blocking and non-blocking assignments in Verilog.

## Topics Covered

- Gate-Level Simulation (GLS)
- RTL Simulation vs Gate-Level Simulation
- Functional GLS Flow using Icarus Verilog
- Synthesis-Simulation Mismatch
- Incomplete Sensitivity List
- Blocking Assignments (`=`)
- Non-Blocking Assignments (`<=`)
- Blocking Assignment Issues in Combinational and Sequential Logic

---

## 1. Gate-Level Simulation (GLS)

Gate-Level Simulation is the process of verifying a synthesized gate-level netlist using a Verilog simulator.

During RTL simulation, we verify the functionality of the original Verilog code. After synthesis, the RTL code is converted into a network of standard cells.

GLS helps verify whether the synthesized circuit maintains the expected functionality of the original RTL design.

### RTL Simulation vs Gate-Level Simulation

| RTL Simulation | Gate-Level Simulation |
|---|---|
| Uses RTL Verilog code | Uses synthesized netlist |
| Verifies design functionality | Verifies synthesized logic functionality |
| Does not require standard-cell models | Requires standard-cell simulation models |
| Runs before synthesis | Runs after synthesis |

### Functional GLS Procedure

1. Write the RTL design and testbench.
2. Perform RTL simulation using Icarus Verilog.
3. Synthesize the design using Yosys.
4. Generate the gate-level netlist.
5. Simulate the netlist using SKY130 standard-cell models.
6. Compare both simulation waveforms using GTKWave.

### Tools Used

- Icarus Verilog – Simulation
- Yosys – Synthesis
- GTKWave – Waveform analysis
- SKY130 HD – Standard-cell library

**Observation:** The RTL simulation and gate-level simulation should produce equivalent functional results when the design is correctly described and the appropriate simulation models are used.

---

## 2. Functional GLS – 2:1 Multiplexer

A 2:1 multiplexer is used to understand the complete RTL-to-gate-level simulation process.

The design selects one of two input signals depending on the select line.

### Verilog Logic

```verilog
assign y = sel ? i1 : i0;
```

### Working Principle

- When `sel = 0`, output `y` follows input `i0`.
- When `sel = 1`, output `y` follows input `i1`.

### Experiment Flow

- RTL simulation using Icarus Verilog.
- Logic synthesis using Yosys.
- Gate-level netlist generation.
- GLS using SKY130 standard-cell models.
- Waveform comparison using GTKWave.

### Result

The synthesized multiplexer is checked against the original RTL design to verify that the expected selection functionality is maintained.

**Screenshot:** Add your RTL waveform, synthesized netlist and GLS waveform here.
**RTL Waveform**

<img width="959" height="528" alt="day4-1" src="https://github.com/user-attachments/assets/38a415c7-0f94-4831-82a4-7018fe349ac8" />

**Synthesized netlist**

<img width="959" height="538" alt="day4-2" src="https://github.com/user-attachments/assets/08467edf-e9e7-4e30-bec5-48bb7b745327" />

**GLS Waveform**
<img width="959" height="532" alt="day4-3gls" src="https://github.com/user-attachments/assets/82c916d9-c4d2-4743-bd6d-ded34594035d" />



---

## 3. Understanding Synthesis-Simulation Mismatch

A synthesis-simulation mismatch occurs when the behavior observed during RTL simulation differs from the behavior of the synthesized hardware.

One important cause is writing an incomplete sensitivity list in a combinational `always` block.

### Incomplete Sensitivity List

Consider the following example:

```verilog
always @(sel)
begin
    if (sel)
        d = b;
    else
        d = a;
end
```

Here, only `sel` is included in the sensitivity list.

If input `a` or `b` changes while `sel` remains unchanged, the simulator does not execute this block again.

As a result, the output may retain its previous value.

### Correct Coding Method

For combinational logic, use `always @(*)`.

```verilog
always @(*)
begin
    if (sel)
        d = b;
    else
        d = a;
end
```

The `@(*)` automatically includes the signals read inside the block in the sensitivity list.

### Important Observation

- An incomplete sensitivity list can cause incorrect RTL simulation behavior.
- Synthesized combinational hardware responds to changes in its input signals.
- Using `always @(*)` helps avoid this type of mismatch.

**Screenshot:** Add the RTL and GLS waveforms showing the mismatch and corrected behavior.

<img width="959" height="539" alt="day4-4" src="https://github.com/user-attachments/assets/79d0c5dc-c062-46ee-a362-3e2f9257635c" />

---

## 4. Blocking and Non-Blocking Assignments

Verilog provides two types of procedural assignments.

### Blocking Assignment (`=`)

A blocking assignment executes immediately. The next statement uses the updated value.

Example:

```verilog
always @(*)
begin
    x = a | b;
    d = x & c;
end
```

Here:
- First, `x` gets the result of `a | b`.
- Next, `d` uses the updated value of `x`.
- The statements execute in sequential order.

Blocking assignments are generally preferred for combinational logic.

### Non-Blocking Assignment (`<=`)

A non-blocking assignment schedules the update for the end of the current simulation time step.

Example:

```verilog
always @(posedge clk)
begin
    q  <= q0;
    q0 <= d;
end
```

Here:
- `q` receives the previous value of `q0`.
- `q0` receives the value of `d`.
- Both updates occur together after the current procedural evaluation.

Non-blocking assignments are generally preferred for sequential logic.


**Screenshot:** Add your RTL waveform, synthesized netlist and GLS waveform here

**RTL Waveform**

<img width="959" height="539" alt="day4-6" src="https://github.com/user-attachments/assets/2655a622-1c1e-4c66-be7d-06c0a9d8e913" />

**Synthesized netlist**

<img width="959" height="539" alt="day4-5" src="https://github.com/user-attachments/assets/bc110078-5823-4369-a45f-caf45560fa3d" />

**GLS Waveform**

![Uploading day4-7gls.png…]()


### Comparison

| Blocking (`=`) | Non-Blocking (`<=`) |
|---|---|
| Updates immediately | Schedules the update |
| Executes in statement order | Updates occur later in the time step |
| Preferred for combinational logic | Preferred for sequential logic |
| Can cause order-dependent behavior | Helps model simultaneous register updates |

---

## 5. Blocking Assignment in Sequential Logic

Blocking assignments inside clock-triggered blocks can create unexpected simulation behavior.

Consider a two-stage shift-register example:

```verilog
always @(posedge clk)
begin
    q = q0;
    q0 = d;
end
```

Since blocking assignments execute immediately, the value assigned to `q` depends on the current value of `q0` at that point in the execution.

Changing the statement order can change the simulation result.

This is why blocking assignments should generally be avoided for sequential register updates.

### Recommended Sequential Coding Style

```verilog
always @(posedge clk)
begin
    q  <= q0;
    q0 <= d;
end
```

With non-blocking assignments, both right-hand-side values are evaluated before the scheduled updates take place.

This correctly represents the behavior of two sequential registers.

---

## 6. Key Observations

- GLS checks the functional behavior of synthesized hardware.
- RTL and gate-level simulations can be compared using GTKWave.
- Incomplete sensitivity lists can create simulation mismatches.
- `always @(*)` is recommended for combinational logic.
- Blocking assignments execute immediately and depend on statement order.
- Non-blocking assignments schedule updates together.
- Non-blocking assignments are recommended for clock-driven sequential logic.

## 7. Tools and Environment

| Tool | Purpose |
|---|---|
| Oracle VirtualBox | Ubuntu development environment |
| Ubuntu | Linux operating environment |
| Icarus Verilog | RTL and gate-level simulation |
| Yosys | RTL synthesis |
| GTKWave | Waveform visualization |
| SKY130 HD Library | Standard-cell models and technology mapping |

## Conclusion

Day 04 helped me understand the relationship between RTL simulation and synthesized hardware. I explored the functional GLS flow, studied how incomplete sensitivity lists cause synthesis-simulation mismatches, and understood the differences between blocking and non-blocking assignments.

These concepts are important for writing reliable Verilog RTL, avoiding simulation mismatches, and developing correct digital designs for ASIC implementation.
