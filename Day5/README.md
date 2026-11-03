# Day 05 – Optimization in Synthesis

## Overview

Day 05 focuses on understanding how different Verilog coding styles influence hardware synthesis and the resulting circuit implementation.

This session covers conditional constructs, incomplete assignments, overlapping case conditions, procedural loops, and generate constructs. The main objective is to understand how RTL descriptions are interpreted during synthesis and how coding styles affect hardware inference.

## Topics Covered

- If and Case Constructs
- Incomplete If and Case Statements
- Incomplete and Overlapping Case Statements
- For Loop and For Generate
- MUX and DEMUX Implementations
- Ripple Carry Adder Using Generate
- Synthesis Optimization Techniques

---

# 1. If and Case Constructs

Conditional statements are used in Verilog to describe decision-making operations in digital circuits. The two commonly used constructs are `if-else` and `case`.

## 1.1 If-Else Construct

The `if-else` statement evaluates a condition and executes the corresponding block based on whether the condition is true or false.

**Key characteristics:**
- Supports Boolean and relational expressions.
- Conditions are evaluated sequentially.
- Multiple conditions can be handled using `else if`.
- Commonly used to describe priority-based logic.
- Can infer multiplexers during synthesis.

### Verilog Code – If-Else

```verilog
module if_else_mux (
    input cond1,
    input cond2,
    input a,
    input b,
    input c,
    output reg y
);

always @(*) begin
    if (cond1)
        y = a;
    else if (cond2)
        y = b;
    else
        y = c;
end

endmodule
```

## 1.2 Case Construct

The `case` statement compares an expression with multiple possible values and executes the corresponding matching branch.

**Key characteristics:**
- Supports multiple selection conditions.
- Provides a structured way to describe selection logic.
- Commonly used in multiplexers, decoders, and control logic.
- A `default` branch handles unmatched values.
- Complete assignments help avoid unintended latch inference.

### Verilog Code – Case

```verilog
module case_mux (
    input i0,
    input i1,
    input i2,
    input i3,
    input [1:0] sel,
    output reg y
);

always @(*) begin
    case (sel)
        2'b00: y = i0;
        2'b01: y = i1;
        2'b10: y = i2;
        2'b11: y = i3;
        default: y = 1'b0;
    endcase
end

endmodule
```

**Explanation:**

The select signal determines which input is connected to the output. Since every possible 2-bit selection is covered, the output is assigned for all conditions.

## 1.3 Difference Between If-Else and Case

<img width="428" height="182" alt="ifandelse" src="https://github.com/user-attachments/assets/3e0a68ae-07ba-4048-a9dd-ccec2ae860f1" />


| If-Else | Case |
|---|---|
| Evaluates conditions sequentially. | Compares an expression against multiple values. |
| Supports complex Boolean expressions. | Suitable for matching specific values or patterns. |
| Naturally represents priority-based logic. | Commonly used for multi-way selection. |
| Useful for range-based conditions. | Useful for decoders and multiplexers. |
| Conditions are checked in order. | Case items represent alternative matching conditions. |

**Observation:** Both constructs can describe combinational logic. The resulting hardware depends on the RTL structure and the conditions used.

---

# 2. Incomplete If and Case Statements

Incomplete assignments occur when an output is not assigned for every possible execution path. In combinational logic, this can result in unintended latch inference.

## 2.1 Incomplete If Statement

An incomplete `if` statement occurs when an output is assigned only under a particular condition, without specifying its behavior for the remaining conditions.

### Verilog Code – Incomplete If

```verilog
module incomp_if (
    input i0,
    input i1,
    output reg y
);

always @(*) begin
    if (i0)
        y = i1;
end

endmodule
```

**Important points:**
- Missing assignments can introduce latches.
- The previous output value is retained when no assignment occurs.
- All possible conditions should be covered in combinational logic.
- Complete assignments help produce predictable hardware.

### Simulation Results

<img width="959" height="539" alt="day5-2" src="https://github.com/user-attachments/assets/960ec27b-dcc8-4b33-bfa4-b4fd8099786e" />


**Explanation:** The waveform illustrates how the output retains its previous value when the specified assignment condition is not satisfied.

### Synthesis Results

<img width="959" height="538" alt="day5-3" src="https://github.com/user-attachments/assets/5a7f3f46-56a6-42e0-9344-1acd56bce83a" />


**Explanation:** The synthesis result helps identify the latch inferred due to the incomplete assignment.

## 2.2 Multiple Incomplete If Conditions

### Verilog Code – Multiple Incomplete If

```verilog
module incomp_if2 (
    input i0,
    input i1,
    input i2,
    input i3,
    output reg y
);

always @(*) begin
    if (i0)
        y = i1;
    else if (i2)
        y = i3;
end

endmodule
```

**Explanation:**

- When `i0` is high, output `y` follows `i1`.
- When `i0` is low and `i2` is high, output `y` follows `i3`.
- When both conditions are low, `y` retains its previous value.

Since there is no final `else` assignment, a latch is inferred.

### Simulation Results

<img width="959" height="539" alt="day5-4" src="https://github.com/user-attachments/assets/a88de0b7-525f-4737-bfab-292d1deade3f" />


### Synthesis Results

<img width="959" height="534" alt="day5-5" src="https://github.com/user-attachments/assets/a4ee8b68-7420-4c75-9d09-5ee6796ca5a1" />


## 2.3 Incomplete Case Statement

An incomplete `case` statement does not specify an output assignment for every possible input combination.

### Verilog Code – Incomplete Case

```verilog
module incomp_case (
    input i0,
    input i1,
    input [1:0] sel,
    output reg y
);

always @(*) begin
    case (sel)
        2'b00: y = i0;
        2'b01: y = i1;
    endcase
end

endmodule
```

**Explanation:**

For `sel = 2'b10` or `sel = 2'b11`, there is no assignment to `y`. Therefore, the output retains its previous value and synthesis infers a latch.

### Simulation Results

<img width="959" height="538" alt="day5-6case" src="https://github.com/user-attachments/assets/c55bb772-b30b-477d-be28-cf56ba321ea5" />


### Synthesis Results

<img width="959" height="539" alt="day5-6 1case" src="https://github.com/user-attachments/assets/cef84e81-10c4-4a0f-86b1-cce0f7053cfd" />


## 2.4 Complete vs Incomplete Case

### Complete Case Code

```verilog
always @(*) begin
    case (sel)
        2'b00: y = i0;
        2'b01: y = i1;
        default: y = i2;
    endcase
end
```

The `default` branch handles the remaining input combinations.

### Incomplete Case Code

```verilog
always @(*) begin
    case (sel)
        2'b00: y = i0;
        2'b01: y = i1;
    endcase
end
```

For the remaining input combinations, the output is not assigned.

**Observation:** The complete case can be implemented as combinational logic, whereas the incomplete case can infer a latch.

---

# 3. Incomplete and Overlapping Case Statements

This section focuses on understanding the effects of incomplete conditions and overlapping selection conditions in Verilog.

## 3.1 Partial Assignments in Case Statements

Even when a `default` branch is present, a latch can still be inferred if every output is not assigned in every branch.

### Verilog Code – Partial Case Assignment

```verilog
module partial_case_assign (
    input i0,
    input i1,
    input i2,
    input [1:0] sel,
    output reg y,
    output reg x
);

always @(*) begin
    case (sel)
        2'b00: begin
            y = i0;
            x = i2;
        end

        2'b01: begin
            y = i1;
        end

        default: begin
            y = i2;
            x = i1;
        end
    endcase
end

endmodule
```

**Explanation:**

- `y` is assigned in every case branch.
- `x` is assigned in the `00` and default branches.
- When `sel = 01`, `x` has no assignment.
- Therefore, synthesis infers a latch for `x`, but not for `y`.

**Key observation:** Latch inference is determined separately for each output.

### Simulation Results

<img width="959" height="531" alt="day5partialwaveform" src="https://github.com/user-attachments/assets/2cb5d22f-017f-44fa-a301-30d77a3b6d1a" />



### Synthesis Results

<img width="959" height="537" alt="partialsymetatic" src="https://github.com/user-attachments/assets/126e97d3-8635-41f9-9cdf-e812c99f0052" />


## 3.2 Overlapping Case Conditions

Overlapping conditions occur when multiple selection patterns can match the same input combination.

### Verilog Code – Bad Case

```verilog
module bad_case (
    input i0,
    input i1,
    input i2,
    input i3,
    input [1:0] sel,
    output reg y
);

always @(*) begin
    case (sel)
        2'b00: y = i0;
        2'b01: y = i1;
        2'b10: y = i2;
        2'b1?: y = i3;
    endcase
end

endmodule
```

**Explanation:**

In a regular `case` statement, matching is exact. The `?` is not a general wildcard.

For wildcard matching, `casez` can be used.

### Verilog Code – Casez

```verilog
always @(*) begin
    casez (sel)
        2'b00: y = i0;
        2'b01: y = i1;
        2'b10: y = i2;
        2'b1?: y = i3;
        default: y = 1'b0;
    endcase
end
```

Here, `2'b1?` matches both `2'b10` and `2'b11`. Therefore, it overlaps with the earlier `2'b10` condition.

**Important observations:**
- Conditions must be examined carefully.
- Priority-based conditional statements follow their evaluation order.
- Wildcard case constructs require additional attention.
- Overlapping patterns can cause differences between RTL and synthesized behavior.
- `casex` should generally be avoided in synthesizable RTL because it can hide unknown values.

### Simulation Results

<img width="959" height="539" alt="day5badcasewv" src="https://github.com/user-attachments/assets/a1860ac3-3ab3-41a2-b3b5-437fdad70bbe" />


### Synthesis Results



---

# 4. For Loop and For Generate

Verilog provides loop constructs to simplify repetitive coding and describe repeated hardware structures. The procedural `for` loop and `generate-for` construct serve different purposes in RTL design.

## 4.1 Procedural For Loop

A `for` loop is used to repeat a set of procedural statements based on an initialization, condition, and update expression.

**Key characteristics:**
- Reduces repetitive RTL coding.
- Commonly used inside procedural blocks.
- Useful for repeated assignments and bit-level operations.
- Statically bounded synthesizable loops are generally unrolled during synthesis.
- Does not automatically create a sequential hardware loop.

### Syntax

```verilog
integer i;

always @(*) begin
    for (i = 0; i < N; i = i + 1) begin
        // Repeated operation
    end
end
```

### Verilog Code – Bitwise Operation Using For Loop

```verilog
module bitwise_loop (
    input [3:0] a,
    input [3:0] b,
    output reg [3:0] y
);

integer i;

always @(*) begin
    for (i = 0; i < 4; i = i + 1) begin
        y[i] = a[i] & b[i];
    end
end

endmodule
```

**Explanation:** The loop describes four bitwise AND operations. During synthesis, the loop is expanded into corresponding combinational hardware.

### Simulation Results

*Insert your GTKWave screenshot here.*

### Synthesis Results

*Insert your Yosys schematic here.*

## 4.2 MUX Using a For Loop

A multiplexer selects one of several input signals and forwards the selected value to a single output.

### Verilog Code – MUX Using For Loop

```verilog
module mux_generate (
    input i0,
    input i1,
    input i2,
    input i3,
    input [1:0] sel,
    output reg y
);

wire [3:0] i_int;
assign i_int = {i3, i2, i1, i0};

integer k;

always @(*) begin
    y = 1'b0;

    for (k = 0; k < 4; k = k + 1) begin
        if (k == sel)
            y = i_int[k];
    end
end

endmodule
```

**Explanation:**

1. The four input signals are combined into a 4-bit vector.
2. The loop checks each index against the select signal.
3. When the index matches, the corresponding input bit is assigned to `y`.
4. The initial assignment gives `y` a default value.

This describes MUX selection logic using a loop rather than explicitly writing every selection condition.

**Observation:** Providing a default assignment prevents unintended latch inference when no loop iteration assigns the output.

### Simulation Results

*Insert your RTL and gate-level waveform screenshots here.*

### Synthesis Results

*Insert your Yosys schematic here.*

## 4.3 DEMUX Using a Loop

A DEMUX directs a single input to one of several outputs based on the select signal.

### Verilog Code – DEMUX Using For Loop

```verilog
module demux_loop (
    input din,
    input [1:0] sel,
    output reg [3:0] y
);

integer k;

always @(*) begin
    y = 4'b0000;

    for (k = 0; k < 4; k = k + 1) begin
        if (k == sel)
            y[k] = din;
    end
end

endmodule
```

**Explanation:**

- The input `din` is routed to one selected output.
- The loop compares each output index with the select signal.
- The default assignment clears all outputs.
- Only the selected output receives the input value.

### Simulation Results

*Insert your GTKWave screenshot here.*

### Synthesis Results

*Insert your Yosys schematic here.*

## 4.4 DEMUX Using Case

The same DEMUX functionality can also be described using a `case` statement.

### Verilog Code – DEMUX Using Case

```verilog
module demux_case (
    input din,
    input [1:0] sel,
    output reg [3:0] y
);

always @(*) begin
    y = 4'b0000;

    case (sel)
        2'b00: y[0] = din;
        2'b01: y[1] = din;
        2'b10: y[2] = din;
        2'b11: y[3] = din;
        default: y = 4'b0000;
    endcase
end

endmodule
```

**Explanation:** The select signal determines which output receives the input. The default assignment ensures that the remaining outputs stay at zero.

**Observation:** Comparing the loop-based and case-based implementations helps understand how different RTL coding styles can describe the same hardware functionality.

### Simulation Results

<img width="959" height="539" alt="day5demuxwf" src="https://github.com/user-attachments/assets/4ae21aab-00a4-44ec-a995-a5479e3e7feb" />


### Synthesis Results

<img width="959" height="539" alt="day5demuxsyn" src="https://github.com/user-attachments/assets/a093174c-8da1-40f4-b349-ff282ba3d046" />


---

# 5. Generate For Loop

A `generate for` loop is used to replicate hardware structures during elaboration.

Unlike a procedural `for` loop, a generate loop is written outside procedural `always` blocks.

**Key characteristics:**
- Used at module level.
- Creates multiple hardware instances.
- Uses a `genvar` variable.
- Useful for repeated module instantiations.
- Supports scalable and parameterized hardware designs.

### Syntax

```verilog
genvar i;

generate
    for (i = 0; i < N; i = i + 1) begin : gen_block
        // Hardware or module instantiation
    end
endgenerate
```

### Verilog Code – Generate For Loop

```verilog
module and_gate (
    input a,
    input b,
    output y
);

assign y = a & b;

endmodule
```

```verilog
module and_array (
    input [3:0] a,
    input [3:0] b,
    output [3:0] y
);

genvar i;

generate
    for (i = 0; i < 4; i = i + 1) begin : gen_block
        and_gate u1 (
            .a(a[i]),
            .b(b[i]),
            .y(y[i])
        );
    end
endgenerate

endmodule
```

**Explanation:** The generate loop creates four instances of the `and_gate` module, one for each bit.

### Generate If

A conditional generate construct can instantiate hardware only when a condition is satisfied.

```verilog
generate
    if (CONDITION) begin
        // Hardware instantiated when condition is true
    end
endgenerate
```

Generate conditions are evaluated during elaboration, not during normal runtime operation.

### Elaboration Results
<img width="959" height="538" alt="day5casegenerate" src="https://github.com/user-attachments/assets/40a20f6f-f902-455a-9613-77ca02ae6d49" />


### Synthesis Results

*Insert your Yosys schematic here.*

## 5.1 Difference Between For Loop and For Generate

| For Loop | For Generate |
|---|---|
| Repeats procedural statements. | Creates repeated hardware structures. |
| Used inside procedural blocks. | Used at module level. |
| Uses a procedural loop variable. | Uses a `genvar` variable. |
| Useful for repeated operations and assignments. | Useful for repeated module instantiations. |
| Static loops are generally unrolled during synthesis. | Instances are created during elaboration. |
| Describes repeated operations. | Describes repeated hardware architecture. |

---

# 6. Ripple Carry Adder Using Generate

A Ripple Carry Adder (RCA) is constructed by connecting multiple Full Adders in a chain.

- Each Full Adder adds two input bits and a carry-in.
- The carry-out of one stage becomes the carry-in of the next stage.
- A generate loop can replicate the Full Adder hardware for each bit.

## 6.1 Full Adder Module

The Full Adder is the basic building block used to construct the Ripple Carry Adder.

### Verilog Code – `fa.v`

```verilog
module fa (
    input a,
    input b,
    input cin,
    output sum,
    output cout
);

assign sum = a ^ b ^ cin;

assign cout = (a & b) | (b & cin) | (a & cin);

endmodule
```

## 6.2 Ripple Carry Adder Module

The RCA design uses multiple Full Adder instances connected through carry signals.

### Verilog Code – `rca.v`

```verilog
module rca (
    input [7:0] a,
    input [7:0] b,
    input cin,
    output [7:0] sum,
    output cout
);

wire [8:0] carry;

assign carry[0] = cin;
assign cout = carry[8];

genvar i;

generate
    for (i = 0; i < 8; i = i + 1) begin : genblk1
        fa u_fa_1 (
            .a(a[i]),
            .b(b[i]),
            .cin(carry[i]),
            .sum(sum[i]),
            .cout(carry[i+1])
        );
    end
endgenerate

endmodule
```

**Explanation:**

1. Two 8-bit input operands are provided.
2. The initial carry input is connected to `carry[0]`.
3. The generate loop creates eight Full Adder instances.
4. Each Full Adder calculates one sum bit and one carry bit.
5. The carry output from one stage is connected to the next stage.
6. The final carry is available at `cout`.

## 6.3 Ripple Carry Adder Block Diagram

```text
       a[0], b[0]       a[1], b[1]                  a[7], b[7]
            |                |                           |
cin ------> FA0 --carry----> FA1 --carry--> ... -------> FA7
            |                |                           |
          sum[0]           sum[1]                      sum[7]
                                                          |
                                                        cout
```

## 6.4 Testbench – `tb_rca.v`

```verilog
module tb_rca;

reg [7:0] a, b;
reg cin;

wire [7:0] sum;
wire cout;

rca uut (
    .a(a),
    .b(b),
    .cin(cin),
    .sum(sum),
    .cout(cout)
);

initial begin
    $dumpfile("rca.vcd");
    $dumpvars(0, tb_rca);

    a = 8'd10;
    b = 8'd20;
    cin = 0;
    #10;

    a = 8'd100;
    b = 8'd50;
    cin = 0;
    #10;

    a = 8'd255;
    b = 8'd1;
    cin = 0;
    #10;

    a = 8'd25;
    b = 8'd30;
    cin = 1;
    #10;

    $finish;
end

endmodule
```

### Simulations Results

*Insert your RCA Yosys schematic here.*

<img width="959" height="538" alt="day5rcawf" src="https://github.com/user-attachments/assets/00d92f38-4578-40ce-9126-bf8f31b6246f" />


### Synthesis Results

<img width="958" height="539" alt="day5rcasyn" src="https://github.com/user-attachments/assets/9429ab75-59d5-4076-aff4-c0a933713492" />


---

# 7. Key Learnings

Through these experiments, I developed an understanding of:

- Differences between `if-else` and `case` constructs.
- Conditional logic representation in RTL.
- Latch inference caused by incomplete assignments.
- Behavior of incomplete and overlapping case conditions.
- Differences between procedural `for` loops and `generate-for`.
- Hardware expansion during synthesis and elaboration.
- MUX and DEMUX implementation using different RTL coding styles.
- Hierarchical design using Full Adder modules.
- Ripple Carry Adder implementation using generate loops.
- The importance of writing complete and synthesizable Verilog code.

# 8. Tools Used

| Tool | Purpose |
|---|---|
| Icarus Verilog | RTL simulation |
| Yosys | Synthesis and hardware visualization |
| GTKWave | Waveform analysis |
| Xdot | Viewing synthesized circuit diagrams |
| SKY130 Standard Cell Library | Gate-level simulation and synthesis |

# 9. Conclusion

Day 05 provided a deeper understanding of how RTL coding styles influence hardware synthesis. By exploring conditional constructs, incomplete assignments, overlapping case conditions, and loop-based descriptions, I gained insight into hardware inference and the conversion of Verilog RTL into digital circuits.

The Ripple Carry Adder experiment helped me understand hierarchical design and repeated module instantiation using generate loops.

These concepts are important for developing clean, efficient, and synthesizable RTL designs.
