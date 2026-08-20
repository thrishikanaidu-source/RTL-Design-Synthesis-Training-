# Day 4: Gate Level Simulation (GLS) and Synthesis-Simulation Mismatch

Today, I explored **Gate Level Simulation (GLS), synthesis-simulation mismatch, sensitivity lists, and blocking assignments**, along with practical labs using **Icarus Verilog, GTKWave, and Yosys**.

## 1. Introduction to Gate Level Simulation (GLS)

Gate Level Simulation (GLS) is the process of simulating the synthesized gate-level netlist using a testbench.

The gate-level netlist is generated after synthesis and contains actual logic gates and flip-flops instead of the original RTL description.

### Why GLS?

- To verify the logical correctness of the design after synthesis.
- To check whether the synthesized design behaves the same as the RTL design.
- To identify synthesis-related issues that may not be visible during RTL simulation.
- To check timing behavior when timing information is available.

- <img width="1216" height="632" alt="WhatsApp Image 2026-08-20 at 9 27 09 PM" src="https://github.com/user-attachments/assets/2a907c0c-db3b-494d-9707-5b0fd5fdd747" />


### Important Point

The testbench used for GLS should be logically the same as the RTL simulation testbench.
```text
RTL Design + Testbench
        |
        v
   RTL Simulation
        |
        v
   Functional Output

RTL Design
        |
        v
    Synthesis
        |
        v
Gate-Level Netlist
        |
        +------> Same Testbench
        |
        v
   Gate-Level Simulation
        |
        v
Functional / Timing Output
```
---

# 2. Synthesis and Simulation Mismatch

A synthesis-simulation mismatch occurs when the behavior observed during simulation does not match the behavior of the synthesized hardware.

### Common Reasons

1. Missing signals in the sensitivity list.
2. Incorrect use of blocking and non-blocking assignments.
3. Non-standard or incorrect Verilog coding styles.

---

# 3. Missing Sensitivity List

A sensitivity list tells an `always` block when it should execute.

For example:

```verilog
always @(a)
begin
    y = a & b;
end
```

Here, the block is sensitive only to changes in `a`.

If `b` changes while `a` remains unchanged, the block will not execute again. This can cause simulation behavior to differ from the intended combinational hardware.

### Correct Approach for Combinational Logic

All signals that affect the output should be included in the sensitivity list.

```verilog
always @(a or b)
begin
    y = a & b;
end
```

A better and safer approach is:

```verilog
always @(*)
begin
    y = a & b;
end
```

`always @(*)` automatically includes the signals read inside the block in the sensitivity list.

---

# 4. Blocking and Non-Blocking Statements in Verilog

Verilog mainly uses two types of procedural assignments:

```verilog
=
```

and

```verilog
<=
```

## 4.1 Blocking Assignment

Blocking assignment uses:

```verilog
=
```

The statement is executed immediately, and the next statement waits for it to complete.

Example:

```verilog
always @(*)
begin
    q0 = d;
    q1 = q0;
end
```

Here:

```verilog
q0 = d;
```

is executed first.

Then:

```verilog
q1 = q0;
```

uses the updated value of `q0`.

Therefore, the statements execute in the order in which they are written.

### Blocking Assignment

```text
Statement 1
    |
    v
Statement 2
    |
    v
Statement 3
```

The statements execute sequentially within the procedural block.

---

# 5. Non-Blocking Assignment

Non-blocking assignment uses:

```verilog
<=
```

The right-hand side values are evaluated first, and the assignments are updated without immediately affecting the following statements.

Example:

```verilog
always @(posedge clk)
begin
    q0 <= d;
    q1 <= q0;
end
```

At the clock edge:

```text
q0 <= d
q1 <= q0
```

Both right-hand side values are evaluated using the old values.

Therefore, non-blocking assignments are commonly used for sequential logic such as flip-flops.

### Non-Blocking Assignment

```text
At the same clock edge:

d  --------> q0
              |
              v
           old q0
              |
              v
             q1
```

The assignments appear together to occur at the same clock event.

---

# 6. Blocking vs Non-Blocking

| Blocking Assignment | Non-Blocking Assignment |
|---|---|
| Uses `=` | Uses `<=` |
| Executes immediately | Update is scheduled |
| Statements execute in order | Multiple assignments can update together |
| Commonly used for combinational logic | Commonly used for sequential logic |
| Later statements can see updated values | Later statements generally see old values until update |

### Example

Using blocking assignment:

```verilog
always @(*)
begin
    q0 = d;
    q1 = q0;
end
```

Using non-blocking assignment:

```verilog
always @(posedge clk)
begin
    q0 <= d;
    q1 <= q0;
end
```

---

# 7. Caveat with Blocking Statements

Consider:

```verilog
always @(*)
begin
    q0 = d;
    q1 = q0;
end
```

There are two statements:

```verilog
q0 = d;
q1 = q0;
```

Since blocking assignment is used, the first statement completes before the second statement is executed.

Therefore:

```text
d --> q0 --> q1
```

The value propagates from `d` to `q0` and then from `q0` to `q1` during the procedural execution.

---

# 8. Caveat with Non-Blocking Statements

Consider:

```verilog
always @(*)
begin
    q0 <= d;
    q1 <= q0;
end
```

Here, non-blocking assignments are used.

Both right-hand side expressions are evaluated before the updates occur.

Therefore, `q1` receives the previous value of `q0`.

```text
Current:
d  ------> q0

Previous q0 ------> q1
```

This behavior is important when describing sequential circuits.

---

# 9. Example of Combinational Logic

Consider the following logic:

```verilog
module mux(
    input  a,
    input  b,
    input  c,
    output y
);

reg q0;

always @(*)
begin
    q0 = a | b;
    y  = q0 & c;
end

endmodule
```

The corresponding logic can be represented as:

```text
a ----\
       OR ---- q0 ----\
b ----/                AND ---- y
                       /
c --------------------/
```

For combinational logic, blocking assignments are generally preferred because the statements describe the flow of combinational calculations.

---

# 10. Why Non-Blocking Statements Are Used for Sequential Logic

Sequential logic depends on clock events.

Example:

```verilog
always @(posedge clk)
begin
    q0 <= d;
    q1 <= q0;
end
```

At every positive edge of the clock:

```text
d  ---> q0
       |
       v
      q1
```

`q1` receives the previous value of `q0`, which represents the behavior of cascaded flip-flops.

---

# 11. Key Learning from Day 4

```text
RTL Simulation
      |
      v
Synthesis
      |
      v
Gate-Level Netlist
      |
      v
Gate-Level Simulation
```

Gate Level Simulation helps verify that the synthesized netlist still represents the intended RTL functionality.

The main causes of synthesis-simulation mismatch studied today were:

```text
1. Missing Sensitivity List
2. Incorrect Blocking / Non-Blocking Assignment
3. Non-Standard Verilog Coding Style
```

Important coding rules:

```text
Combinational Logic
        |
        +--> always @(*)
        |
        +--> Blocking assignment (=)

Sequential Logic
        |
        +--> always @(posedge clk)
        |
        +--> Non-blocking assignment (<=)
```
## Lab 1: Ternary Operator MUX

### Verilog Design
<img width="946" height="577" alt="image" src="https://github.com/user-attachments/assets/f17933b8-70e4-4fc9-9960-95d8f6e2eedd" />


```verilog
module ternary_operator_mux (
    input i0,
    input i1,
    input sel,
    output y
);

assign y = sel ? i1 : i0;

endmodule
```

### Explanation

The ternary operator is used to implement a 2:1 multiplexer.

```verilog
assign y = sel ? i1 : i0;
```

The operation is:

```text
sel = 0 → y = i0
sel = 1 → y = i1
```

### RTL Simulation
<img width="936" height="657" alt="image" src="https://github.com/user-attachments/assets/e2876718-0aa8-4e45-a1d5-6fd9cb3048ca" />


The design was simulated using Icarus Verilog and the output waveform was viewed using GTKWave.

The waveform contains the following signals:

```text
i0
i1
sel
y
```

The waveform verifies that the output `y` follows the selected input according to the value of `sel`.

### Synthesis using Yosys

<img width="1600" height="630" alt="image" src="https://github.com/user-attachments/assets/dde6b9fd-1baa-4385-9743-e3ffc7e6eaf2" />


The design was synthesized using Yosys.

The synthesis statistics obtained were:

```text
=== ternary_operator_mux ===

Number of wires:
    4

Number of wire bits:
    4

Number of public wires:
    4

Number of public wire bits:
    4

Number of ports:
    4

Number of port bits:
    4

Number of memories:
    0

Number of memory bits:
    0

Number of processes:
    0

Number of cells:
    1

$_MUX:
    1
```

### Gate-Level Netlist

After synthesis, the ternary operator MUX was converted into a gate-level representation using SKY130 standard cells.

The synthesized netlist contains cells such as:

```text
sky130_fd_sc_hd__clkinv_1
sky130_fd_sc_hd__nand2_1
sky130_fd_sc_hd__o21ai_0
```

The synthesized circuit represents the same MUX functionality as the original RTL design.

### Lab Flow

```text
Verilog RTL
    ↓
Icarus Verilog Simulation
    ↓
GTKWave
    ↓
Yosys Synthesis
    ↓
Synthesis Statistics
    ↓
Gate-Level Netlist
```

### Result

The ternary operator MUX was successfully simulated and synthesized. The GTKWave waveform verified the functional behavior, and Yosys generated the corresponding gate-level netlist using standard cells.

## Lab 2: Synthesis and Simulation Mismatch

### Objective

To understand synthesis and simulation mismatch using a MUX example with an incomplete sensitivity list.

### Verilog Design

<img width="897" height="602" alt="image" src="https://github.com/user-attachments/assets/595d5433-1df7-4395-a3d3-8f0a93096814" />


```verilog
module bad_mux (input i0, input i1, input sel, output reg y);

always @(sel)
begin
    if(sel)
        y <= i1;
    else
        y <= i0;
end

endmodule
```

### Explanation

The above design describes a 2:1 MUX.

```verilog
if(sel)
    y <= i1;
else
    y <= i0;
```

The intended operation is:

```text
sel = 0 → y = i0
sel = 1 → y = i1
```

However, the sensitivity list is:

```verilog
always @(sel)
```

Only `sel` is included in the sensitivity list.

The inputs `i0` and `i1` are missing from the sensitivity list. Therefore, the `always` block is triggered only when `sel` changes.

This can cause a mismatch between simulation behavior and the actual synthesized hardware.

### Simulation

<img width="945" height="580" alt="image" src="https://github.com/user-attachments/assets/222d5d3a-fc91-4a1b-b585-c21b88535ce5" />


The design was simulated and the waveform was observed using GTKWave.

The following signals were observed:

```text
i0
i1
sel
y
```

The RTL simulation waveform shows the behavior of the `bad_mux` design.

Because the sensitivity list contains only `sel`, changes in `i0` or `i1` alone do not trigger the `always` block.

### Synthesis

<img width="937" height="615" alt="image" src="https://github.com/user-attachments/assets/31ab6c27-6f48-4cdd-9404-8d98f6dee7cc" />


The design was synthesized using Yosys.

The synthesis statistics obtained were:

```text
=== bad_mux ===

Number of wires:
    4

Number of wire bits:
    4

Number of public wires:
    4

Number of public wire bits:
    4

Number of ports:
    4

Number of port bits:
    4

Number of memories:
    0

Number of memory bits:
    0

Number of processes:
    0

Number of cells:
    1

$_MUX:
    1
```

### Synthesis Result

Yosys recognizes the logic as a MUX and synthesizes it into:

```text
$_MUX
```

The synthesis result contains:

```text
Number of cells: 1
$_MUX: 1
```

### Gate-Level Simulation

After synthesis, the synthesized design was simulated at gate level using the testbench.

The gate-level waveform was viewed using GTKWave.

The waveform contains:

```text
i0
i1
sel
y
```

The synthesized design shows the MUX behavior at gate level.

### RTL Simulation vs Gate-Level Simulation

```text
RTL Design
    ↓
RTL Simulation
    ↓
Waveform
    ↓
Synthesis
    ↓
Gate-Level Netlist
    ↓
Gate-Level Simulation
    ↓
Waveform Comparison
```

The purpose of comparing the two simulations is to identify differences between the RTL simulation behavior and the synthesized gate-level behavior.

### Cause of Synthesis-Simulation Mismatch

The main issue in this example is the incomplete sensitivity list:

```verilog
always @(sel)
```

The signals `i0` and `i1` are used inside the `always` block but are not included in the sensitivity list.

For combinational logic, the preferred coding style is:

```verilog
always @(*)
```

For example:

```verilog
always @(*)
begin
    if(sel)
        y <= i1;
    else
        y <= i0;
end
```

This ensures that changes in the signals used by the combinational logic trigger the `always` block.

### Result

The synthesis and simulation mismatch example was successfully studied using the `bad_mux` design. The design was simulated using GTKWave and synthesized using Yosys. The synthesis statistics showed one MUX cell. The experiment demonstrated how an incomplete sensitivity list can cause simulation behavior to differ from the intended combinational hardware behavior.
This lab demonstrated the importance of using a complete sensitivity list when describing combinational logic. The `bad_mux` example used `always @(sel)`, while `i0` and `i1` were also required for the logic. This can lead to synthesis and simulation mismatch. Using `always @(*)` is a safer coding style for combinational logic.

# Lab 3: Blocking Assignment Caveat

## Objective

To understand the behavior of blocking assignments in Verilog and observe the difference between RTL simulation and the synthesized gate-level circuit.

---

## Verilog Design

<img width="897" height="602" alt="image" src="https://github.com/user-attachments/assets/7dd990d1-b23e-41c4-9736-c67abb92d2a0" />


```verilog
module blocking_caveat(input a, input b, input c, output reg d);

reg x;

always @(*)
begin
    d = x & c;
    x = a | b;
end

endmodule
```

---

## Explanation

In this design, blocking assignments are used inside the `always @(*)` block.

The two statements are:

```verilog
d = x & c;
x = a | b;
```

Since blocking assignment `=` is used, statements are executed sequentially from top to bottom.

First:

```verilog
d = x & c;
```

The output `d` is calculated using the current value of `x`.

Then:

```verilog
x = a | b;
```

The value of `x` is updated using `a` and `b`.

Therefore, `d` uses the **previous value of `x`**, while `x` is updated afterward.

### Logic Flow

```text
a ───┐
     ↓
     OR ───→ x
     ↑       │
b ───┘       │
             ↓
            AND ───→ d
             ↑
             c
```

Because of the order of the blocking assignments, the simulation behavior can show a caveat compared with the combinational hardware inferred during synthesis.

---

## RTL Simulation

<img width="997" height="647" alt="image" src="https://github.com/user-attachments/assets/ecdfa876-f2e7-45f9-8a9a-d8e13fe9ddb8" />


The design was simulated and the waveform was viewed using GTKWave.

The following signals were observed:

```text
a
b
c
d
```

The waveform shows the changes in the input signals and the corresponding behavior of the output `d`.

The RTL simulation demonstrates the effect of the blocking assignment order.

---

## Synthesis using Yosys


<img width="1206" height="641" alt="image" src="https://github.com/user-attachments/assets/a11776e1-d2ff-4151-9e02-9cf8dc02b662" />



The design was synthesized using Yosys.

The synthesis statistics obtained were:

```text
=== blocking_caveat ===

Number of wires:
    5

Number of wire bits:
    5

Number of public wires:
    4

Number of public wire bits:
    4

Number of ports:
    4

Number of port bits:
    4

Number of memories:
    0

Number of memory bits:
    0

Number of processes:
    0

Number of cells:
    2

$_ANDNOT_:
    1

$_NOR_:
    1
```

### Synthesis Result

Yosys synthesized the design into two logic cells:

```text
$_ANDNOT_ → 1
$_NOR_    → 1
```

Total number of cells:

```text
2
```

---

## Gate-Level Netlist

<img width="692" height="633" alt="image" src="https://github.com/user-attachments/assets/7fe77977-72f0-431a-bc9d-f1eae4aac0d1" />

The synthesized design was converted into a gate-level representation.

The synthesized circuit uses the SKY130 standard cell:

```text
sky130_fd_sc_hd__o21ai_1
```

The schematic contains:

```text
Inputs:
a
b
c

Output:
d
```

The synthesized circuit represents the hardware generated from the RTL description.

---

## Gate-Level Simulation

<img width="1203" height="640" alt="image" src="https://github.com/user-attachments/assets/07f1656d-531d-4de5-bdd2-06904618df03" />


The synthesized gate-level design was simulated and the waveform was viewed using GTKWave.

The observed signals were:

```text
a
b
c
d
```

The gate-level waveform shows the behavior of the synthesized circuit.

---

## RTL Simulation and Gate-Level Simulation

The overall flow is:

```text
RTL Design
    ↓
RTL Simulation
    ↓
GTKWave
    ↓
Yosys Synthesis
    ↓
Gate-Level Netlist
    ↓
Gate-Level Simulation
    ↓
GTKWave
```

The purpose of comparing RTL and gate-level simulation is to understand how the RTL coding style affects simulation and synthesized hardware.

---

## Blocking Assignment Caveat

The important point in this experiment is the order of the blocking assignments:

```verilog
d = x & c;
x = a | b;
```

The first statement uses the old value of `x`.

The second statement updates `x`.

Therefore, the procedural execution is:

```text
First:
d = x & c
       ↓
Uses previous x

Then:
x = a | b
       ↓
Updates x
```

This demonstrates why the order of statements matters when blocking assignments are used.

---

## Result

The blocking assignment caveat was successfully studied using the `blocking_caveat` module.

The RTL design was simulated using GTKWave and synthesized using Yosys.

The synthesis statistics showed:

```text
Number of wires: 5
Number of wire bits: 5
Number of public wires: 4
Number of public wire bits: 4
Number of ports: 4
Number of port bits: 4
Number of memories: 0
Number of memory bits: 0
Number of processes: 0
Number of cells: 2
```

The two synthesized cells were:

```text
$_ANDNOT_: 1
$_NOR_:    1
```

The synthesized gate-level representation used the SKY130 standard cell:

```text
sky130_fd_sc_hd__o21ai_1
```

---

## Conclusion

This lab demonstrated the caveat associated with blocking assignments in combinational logic. Since blocking assignments execute sequentially, the order of statements affects the simulation behavior. In this example, `d` is calculated using the previous value of `x` before `x` is updated. The synthesis result shows how the RTL description is converted into actual hardware logic.

### Overall Conclusion:
The labs helped me understand Gate Level Simulation, synthesis-simulation mismatch, sensitivity lists, and blocking assignments. I also learned how RTL code is simulated using Icarus Verilog and GTKWave and synthesized using Yosys to obtain and verify the gate-level netlist.


