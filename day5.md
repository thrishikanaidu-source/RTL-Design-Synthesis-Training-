# DAY 5 — RTL SYNTHESIS

## 1. OPTIMISATION IN SYNTHESIS

### `if` and `else` statements

- `if-else` statements can create priority logic.
- Conditions are checked in order.
- The first condition that is true gets priority.
- The synthesized hardware can contain cascaded MUXes.

Example:

    if (cond1)
        y = c1;
    else if (cond2)
        y = c2;
    else if (cond3)
        y = c3;
    else
        y = default_value;

The priority is:

    cond1 > cond2 > cond3 > else

---

## 2. PRIORITY LOGIC

- `if-else` naturally represents priority logic.
- If more than one condition is true, the first matching condition is selected.
- The order of conditions therefore matters.

Conceptually:

    C1 ──→ MUX ──→
                  MUX ──→ Y
    C2 ──→ MUX ──→
                  ↑
    C3 ───────────┘

---

## 3. DANGER / CAUTION WITH `if`

- In combinational logic, an incomplete `if` statement can result in an inferred latch.
- If an output is not assigned for every possible condition, the hardware may need to retain its previous value.
- This causes storage behaviour even though the designer intended combinational logic.
- An inferred latch is generally undesirable when combinational logic is intended.

Example:

    if (cond1)
        y = a;
    else if (cond2)
        y = b;

If neither `cond1` nor `cond2` is true, `y` is not assigned.

Therefore:

    Incomplete if
          ↓
    Inferred Latch
          ↓
    Previous value retained

### Safer approach

Give the output a default value:

    always_comb begin
        y = default_value;

        if (cond1)
            y = a;
        else if (cond2)
            y = b;
    end

This ensures that `y` always has a value.

---

# CASE STATEMENT

- `case` statements are used for selection logic.
- A `case` statement is generally used inside an `always` block.
- The expression is compared with the case items.
- The matching case item determines the output.

Example:

    always_comb begin
        case (sel)
            2'b00: y = a;
            2'b01: y = b;
            2'b10: y = c;
            2'b11: y = d;
        endcase
    end

A `case` statement can be used to describe MUX/selection logic.

---

# CAVEAT WITH CASE

## 1. INCOMPLETE CASE

- An incomplete `case` does not cover all possible values.
- If no case item matches, the output may not get an assignment.
- In combinational logic, this can result in an inferred latch.

Example:

    always_comb begin
        case (sel)
            2'b00: y = a;
            2'b01: y = b;
            2'b10: y = c;
        endcase
    end

For:

    sel = 2'b11

there is no matching case.

Therefore:

    Incomplete case
          ↓
    Output not assigned
          ↓
    Possible latch

---

## 2. PARTIAL ASSIGNMENTS IN CASE

- Partial assignment in a `case` can also cause latch inference.
- Every output should be assigned for every possible condition when designing combinational logic.
- A default assignment can be used before the `case`.
- A `default` case can also be used to cover unmatched values.

Example:

    always_comb begin
        y = default_value;

        case (sel)
            2'b00: y = a;
            2'b01: y = b;
            2'b10: y = c;
            2'b11: y = d;
        endcase
    end

---

# NOTE

- Assign all the outputs in the combinational block.
- Make sure every possible condition has an assignment.
- This prevents unintended latch inference.

---

# COUNTER

A counter is sequential logic and requires storage.

Example:

    logic [3:0] count;

    always_ff @(posedge clk) begin
        if (reset)
            count <= 4'b0000;
        else
            count <= count + 1'b1;
    end

Important:

    always_comb
        → Combinational logic

    always_ff
        → Sequential logic

A counter uses flip-flops to store its value.

---

# FOR LOOP / GENERATE

- A `for` loop can be used to repeat operations.
- A generate `for` loop is used to create repeated hardware structures.
- `genvar` is commonly used with generate loops.
- Generate constructs are expanded during elaboration.

Example:

    genvar i;

    generate
        for (i = 0; i < N; i = i + 1) begin
            // repeated hardware
        end
    endgenerate

### Important Difference

    for loop
        ↓
    Repeat operations/statements

    generate for
        ↓
    Repeat hardware structures/instances

---

# EXAMPLE — RIPPLE CARRY ADDER (RCA)

- A Ripple Carry Adder is constructed using multiple Full Adders.
- Each Full Adder performs one-bit addition.
- The carry output of one Full Adder is connected to the carry input of the next Full Adder.
- The carry propagates from the LSB towards the MSB.
- This is called ripple carry.

For a 4-bit RCA:

    C0 → FA0 → C1 → FA1 → C2 → FA2 → C3 → FA3 → C4
           ↓          ↓          ↓          ↓
          SUM0       SUM1       SUM2       SUM3

---

# HARDWARE INSTANTIATION USING FOR-GENERATE

- Generate can be used to instantiate multiple Full Adders.
- The same hardware structure is repeated for every value of `i`.
- This is useful for creating parameterized and scalable hardware.

Example:

    genvar i;

    generate
        for (i = 0; i < N; i = i + 1) begin : FA_GEN

            full_adder FA (
                .a    (a[i]),
                .b    (b[i]),
                .cin  (carry[i]),
                .sum  (sum[i]),
                .cout (carry[i+1])
            );

        end
    endgenerate

For N = 4:

    FA0
    FA1
    FA2
    FA3

are generated.

---

# SYNTHESIS / ELABORATION FLOW

    RTL Code
        ↓
    Elaboration
        ↓
    Generate Hardware
        ↓
    Synthesis
        ↓
    Gate-Level Hardware

---
# LAB 1 — INCOMPLETE IF

<img width="937" height="615" alt="WhatsApp Image 2026-08-21 at 9 52 33 PM" src="https://github.com/user-attachments/assets/5fd4967e-f226-4b31-b96f-355dea1ced4a" />

## 1. CODE
```
module incomp_if2 (
    input i0,
    input i1,
    input i2,
    output reg y
);

always @(*) begin
    if (i0)
        y <= i1;
end

endmodule
```

## 2. CODE EXPLANATION

- `i0`, `i1`, and `i2` are input signals.
- `y` is the output signal.
- `always @(*)` is used to describe combinational logic.
- The `if (i0)` condition checks whether `i0` is HIGH.
- When `i0 = 1`, the output `y` gets the value of `i1`.
- When `i0 = 0`, there is no assignment to `y`.
- Therefore, `y` retains its previous value.
- Because the previous value needs to be retained, a latch is inferred.
- `i2` is not used anywhere in the logic.


## 3. WAVEFORM EXPLANATION

<img width="947" height="896" alt="image" src="https://github.com/user-attachments/assets/9a9c32f9-267d-46c3-b526-d1bb2397bef1" />



- When `i0 = 1`, `y` follows `i1`.
- When `i0 = 0`, `y` holds its previous value.
- The output remains unchanged while `i0` is LOW because there is no assignment to `y`.
- This holding behaviour indicates latch behaviour.

### WAVEFORM BEHAVIOUR

    i0 = 1
    i1 changes
       ↓
    y follows i1

    i0 = 0
       ↓
    y holds previous value


## 4. SYNTHESIS RESULT

<img width="952" height="903" alt="image" src="https://github.com/user-attachments/assets/78efbeb8-c2fd-4d80-8590-d882fa3df898" />




Yosys reports:

    $_DLATCH_P_ : 1

This means one latch is inferred.

The latch connections are:

    D = i1
    E = i0
    Q = y


    

## 5. CONCLUSION

The incomplete `if` statement causes latch inference because `y` is not
assigned when `i0 = 0`.

    Incomplete if
         ↓
    y not assigned
         ↓
    Previous value retained
         ↓
    Latch inferred


    # LAB 2 — INCOMPLETE IF / ELSE IF

## 1. CODE

<img width="1148" height="683" alt="image" src="https://github.com/user-attachments/assets/67c4ea0b-6cd5-48f7-a865-f44a68e5e5bd" />


module incomp_if2 (
    input i0,
    input i1,
    input i2,
    input i3,
    output reg y
);

always @(*) begin

    if (i0)
        y <= i1;

    else if (i2)
        y <= i3;

end

endmodule


## 2. CODE EXPLANATION

- `i0`, `i1`, `i2`, and `i3` are input signals.
- `y` is the output signal.
- `always @(*)` is used for combinational logic.

- First condition:

      if (i0)
          y <= i1;

  When `i0 = 1`, `y` gets the value of `i1`.

- Second condition:

      else if (i2)
          y <= i3;

  When `i0 = 0` and `i2 = 1`, `y` gets the value of `i3`.

- There is no final `else` condition.
- When both `i0 = 0` and `i2 = 0`, `y` is not assigned.
- Therefore, `y` retains its previous value.
- This results in latch inference during synthesis.


## 3. WAVEFORM EXPLANATION

<img width="1136" height="710" alt="image" src="https://github.com/user-attachments/assets/def2fb1e-53c9-4107-9d4a-a81291fdcff4" />


- When `i0 = 1`, `y` follows `i1`.
- When `i0 = 0` and `i2 = 1`, `y` follows `i3`.
- When `i0 = 0` and `i2 = 0`, `y` holds its previous value.
- The initial red/unknown value of `y` occurs because the output has not
  received a valid assignment at the beginning of simulation.

### WAVEFORM BEHAVIOUR

    i0 = 1
        ↓
    y = i1


    i0 = 0
    i2 = 1
        ↓
    y = i3


    i0 = 0
    i2 = 0
        ↓
    y holds previous value
        ↓
    LATCH BEHAVIOUR


## 4. SYNTHESIS / NETLIST EXPLANATION

<img width="1151" height="851" alt="image" src="https://github.com/user-attachments/assets/9a51dc2f-21f7-4da6-bcff-09e5edd917d7" />


The synthesized circuit contains:

    sky130_fd_sc_hd__mux2_1
    sky130_fd_sc_hd__nor2_1
    $_DLATCH_N_

The MUX selects between `i1` and `i3`.

    i1 ──┐
         ├── MUX ──→ LATCH ──→ y
    i3 ──┘

The selection depends on `i0`.

The latch enable is controlled by the condition involving `i0` and `i2`.

The synthesized circuit therefore represents the behaviour of the
incomplete `if / else if` statement.


## 5. CONCLUSION

The incomplete `if / else if` statement causes latch inference because
there is no assignment to `y` when both conditions are FALSE.

    if (i0)
        y = i1;
    else if (i2)
        y = i3;

    i0 = 0
    i2 = 0
        ↓
    y is not assigned
        ↓
    Previous value of y is retained
        ↓
    LATCH INFERRED

    # LAB 4 — PARTIAL CASE ASSIGNMENT

## 1. CODE
<img width="1158" height="761" alt="image" src="https://github.com/user-attachments/assets/f502b5e8-75d2-45bf-a1df-1fd0ebf36aaa" />


module partial_case_assign (
    input i0,
    input i1,
    input i2,
    input [1:0] sel,
    output reg y,
    output reg x
);

always @(*) begin

    case(sel)

        2'b00 : begin
            y = i0;
            x = i2;
        end

        2'b01 : y = i1;

        default : begin
            x = i1;
            y = i2;
        end

    endcase

end

endmodule


## 2. CODE EXPLANATION

- `i0`, `i1`, and `i2` are input signals.
- `sel` is a 2-bit select signal.
- `y` and `x` are output signals.
- `always @(*)` is used for combinational logic.
- A `case(sel)` statement is used to select the output values based on `sel`.

### CASE 2'b00

When:

    sel = 2'b00

The code executes:

    y = i0;
    x = i2;

Therefore:

    y → i0
    x → i2

### CASE 2'b01

When:

    sel = 2'b01

The code executes:

    y = i1;

Here, `y` is assigned but `x` is NOT assigned.

Therefore, `x` has to retain its previous value.

This causes latch inference for `x`.

### DEFAULT CASE

For all other values of `sel`:

    x = i1;
    y = i2;

Therefore:

    y → i2
    x → i1

### IMPORTANT POINT

The problem is in:

    2'b01 : y = i1;

Since `x` is not assigned in this case, `x` retains its previous value.

Therefore, a latch is inferred for `x`.


## 3. WAVEFORM / FUNCTIONAL BEHAVIOUR
<img width="332" height="328" alt="image" src="https://github.com/user-attachments/assets/ca8f7028-4d39-40ad-9190-e785d56102cf" />


The output behaviour depends on the value of `sel`.

    sel = 2'b00
        ↓
    y = i0
    x = i2


    sel = 2'b01
        ↓
    y = i1
    x holds previous value


    sel = 2'b10 or 2'b11
        ↓
    y = i2
    x = i1


### LATCH BEHAVIOUR

When:

    sel = 2'b01

Only `y` receives a new value.

    y = i1
    x = previous x

Therefore, `x` needs storage to remember its previous value.


## 4. SYNTHESIS RESULT

Yosys reports:

    Number of wires:        13
    Number of wire bits:    14
    Number of public wires: 6
    Number of public wire bits: 7
    Number of ports:        6
    Number of port bits:    7
    Number of memories:     0
    Number of processes:    0
    Number of cells:        9

The synthesized cells include:

    $_ANDNOT_ : 1
    $_AND_    : 1
    $_DLATCH_P_ : 1
    $_MUX_    : 2
    $_NOR_    : 1
    $_ORNOT_  : 1
    $_OR_     : 1


## 5. SYNTHESIZED NETLIST EXPLANATION

<img width="1155" height="952" alt="image" src="https://github.com/user-attachments/assets/b326ecf4-fd2b-4a7f-b3fc-326802ada88c" />


The synthesized circuit contains:

    MUX
    AND / OR / NOR logic
    LATCH

The important part is:

    x → $_DLATCH_P_ → x output

The latch is inferred because `x` is not assigned for every possible
value of `sel`.

The MUX logic is used to select the required input values based on `sel`.

The synthesized hardware therefore represents the partial assignment
present in the original `case` statement.


## 6. CONCLUSION

This experiment demonstrates partial assignment inside a `case` statement.

    sel = 2'b00
        ↓
    y = i0
    x = i2

    sel = 2'b01
        ↓
    y = i1
    x not assigned
        ↓
    x retains previous value
        ↓
    LATCH INFERRED

    sel = 2'b10 / 2'b11
        ↓
    y = i2
    x = i1

The synthesis result confirms the presence of:

    $_DLATCH_P_ : 1

Therefore, a partial assignment in a combinational `case` statement can
cause unintended latch inference.
# LAB — MUX USING FOR LOOP

## Verilog Code

<img width="947" height="416" alt="image" src="https://github.com/user-attachments/assets/cdd50fa5-9e09-409e-be02-8776d6cb96cc" />


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
    for (k = 0; k < 4; k = k + 1) begin
        if (k == sel)
            y = i_int[k];
    end
end

endmodule


## Understanding

A 4:1 MUX is implemented using a `for` loop.

The inputs are combined into `i_int`:

i_int[0] = i0
i_int[1] = i1
i_int[2] = i2
i_int[3] = i3

The `for` loop evaluates each value of `k` from 0 to 3 and checks whether it matches the select signal.

The important point is that the `for` loop is used for **evaluation during RTL elaboration/synthesis**, not for creating sequential hardware that repeatedly executes.

The synthesis tool effectively unrolls the loop and generates the required MUX hardware.


## Waveform Output

<img width="952" height="696" alt="image" src="https://github.com/user-attachments/assets/50230e2d-c852-49bd-82ce-e4657d052881" />


The GTKWave waveform verifies the functionality of the MUX.

The `sel` signal changes through:

00 → 01 → 10 → 11 → 00 → 01

Accordingly, the output `y` selects the corresponding input:

sel = 00 → y = i0
sel = 01 → y = i1
sel = 10 → y = i2
sel = 11 → y = i3

The waveform confirms that the output follows the selected input correctly.


## Synthesis Output

<img width="941" height="466" alt="image" src="https://github.com/user-attachments/assets/8540ac97-cc7a-42ab-aad2-b5e86530813c" />


The synthesized schematic shows that the `for` loop has been converted into MUX/selection logic.

This demonstrates that a Verilog `for` loop is a convenient way to describe repetitive hardware structures.

## Key Learning

- `for` loops are useful for evaluating repetitive RTL logic.
- The loop does not represent repeated hardware execution.
- Synthesis unrolls the loop into hardware.
- `for` statements are especially handy when describing **wide MUXes, DEMUXes, and other repetitive hardware structures**.
- The waveform confirms the functional behavior and the synthesized schematic confirms the generated hardware.

-# LAB — DEMUX USING CASE vs FOR LOOP
### LAB TO COMPARE CASE STATEMENT & FOR LOOP
## Objective

To implement and compare an 1:8 DEMUX using:

1. `case` statement
2. `for` loop

The purpose is to understand how both RTL coding styles represent the same hardware and why a `for` loop becomes more convenient for larger MUX/DEMUX designs.

---

## 1. DEMUX USING CASE

<img width="942" height="933" alt="image" src="https://github.com/user-attachments/assets/886936a7-b7a6-4958-bcd7-302110c8b7a9" />


### Verilog Code

```verilog
module demux_case (
    output o0,
    output o1,
    output o2,
    output o3,
    output o4,
    output o5,
    output o6,
    output o7,
    input [2:0] sel,
    input i
);

reg [7:0] y_int;

assign {o7,o6,o5,o4,o3,o2,o1,o0} = y_int;

always @(*) begin
    y_int = 8'b0;

    case(sel)
        3'b000 : y_int[0] = i;
        3'b001 : y_int[1] = i;
        3'b010 : y_int[2] = i;
        3'b011 : y_int[3] = i;
        3'b100 : y_int[4] = i;
        3'b101 : y_int[5] = i;
        3'b110 : y_int[6] = i;
        3'b111 : y_int[7] = i;
    endcase
end

endmodule
```

The `case` statement explicitly defines every select condition and the corresponding DEMUX output.

---

## 2. DEMUX USING FOR LOOP

### Verilog Code

```verilog
module demux_generate (
    output o0,
    output o1,
    output o2,
    output o3,
    output o4,
    output o5,
    output o6,
    output o7,
    input [2:0] sel,
    input i
);

reg [7:0] y_int;

assign {o7,o6,o5,o4,o3,o2,o1,o0} = y_int;

integer k;

always @(*) begin
    y_int = 8'b0;

    for(k = 0; k < 8; k = k + 1) begin
        if(k == sel)
            y_int[k] = i;
    end
end

endmodule
```

Here, instead of writing eight individual conditions, the `for` loop evaluates `k` from `0` to `7` and checks:

```text
k == sel
```

When the condition matches, the input is routed to the corresponding output.

---

## 3. WAVEFORM OBSERVATION

<img width="955" height="872" alt="image" src="https://github.com/user-attachments/assets/aeec2c04-47a2-4bc8-a4c8-ed5aa7278fb8" />
waveform for case statement


<img width="957" height="797" alt="image" src="https://github.com/user-attachments/assets/c6477c11-83e7-4c2f-96c8-2d30e12b29c2" />
waveform for forloop

The GTKWave results for both implementations show the same functional behavior.

The select signal changes through:

```text
010 → 011 → 100 → 101
```

Accordingly, the input `i` is routed to:

```text
sel = 010 → o2
sel = 011 → o3
sel = 100 → o4
sel = 101 → o5
```

Only the selected DEMUX output follows the input signal while the other outputs remain inactive.

The waveforms of the `case` and `for` implementations are functionally equivalent.

---

## 4. SYNTHESIS COMPARISON

The synthesized schematics show that both RTL descriptions are converted into DEMUX hardware.

### CASE

<img width="935" height="433" alt="image" src="https://github.com/user-attachments/assets/55789c71-5b2c-4782-b4f9-f42c4315d923" />


The `case` implementation explicitly describes each selection:

```text
sel → individual conditions → output
```

As the number of outputs increases, the RTL becomes longer because every case item has to be written explicitly.

### FOR LOOP

<img width="946" height="683" alt="image" src="https://github.com/user-attachments/assets/41fb2622-eb02-45fe-8835-9b48b39ef01d" />

The `for` loop describes the repetitive selection in a compact way:

```text
for k = 0 to 7
        |
        └── if k == sel
                |
                └── y_int[k] = i
```

The synthesis tool unrolls the loop and generates the required hardware.

---

## 5. KEY LEARNING

The important point is that `for` loops are **not hardware that repeatedly executes**.

The `for` loop is used to describe/evaluate repetitive RTL structure, and during synthesis it is **unrolled into hardware**.

For a small MUX/DEMUX, a `case` statement is easy to understand and write.

But as the design becomes wider:

```text
4:1 MUX
8:1 MUX
16:1 MUX
32:1 MUX
64:1 MUX
...
```

or:

```text
1:4 DEMUX
1:8 DEMUX
1:16 DEMUX
1:32 DEMUX
...
```

writing every selection explicitly using `case` becomes lengthy and repetitive.

Therefore:

**For larger/wider MUX and DEMUX structures, `for` statements are very handy because they allow the repetitive logic to be described compactly while synthesis generates the corresponding hardware.**

---

## FINAL CONCLUSION

`case` and `for` can describe the same DEMUX functionality.

```text
CASE
→ Explicit
→ Easy for small selection logic
→ Becomes lengthy for large designs

FOR LOOP
→ Compact
→ Handles repetitive structures efficiently
→ Very useful for wide MUX/DEMUX designs
→ Synthesizer unrolls it into actual hardware
```

This lab helped me understand that RTL coding style can make a major difference in **code scalability and readability**, even though both descriptions can synthesize to functionally equivalent hardware.

# LAB — GENERATE FOR LOOP — RIPPLE CARRY ADDER

## Objective

To understand the use of a `generate for` loop in Verilog for creating multiple instances of the same module.

In this lab, an 8-bit Ripple Carry Adder (RCA) is designed by instantiating multiple 1-bit Full Adder modules using a `generate for` loop.

---

### Lab on FOR generate loop

## 1. FULL ADDER MODULE

<img width="946" height="640" alt="image" src="https://github.com/user-attachments/assets/6c8dc72d-5f2a-4d58-995a-801f4c05198b" />


### Verilog Code

```verilog
module fa (
    input a,
    input b,
    input c,
    output co,
    output sum
);

assign {co,sum} = a + b + c;

endmodule
```

The Full Adder takes three inputs:

- `a`
- `b`
- `c` (carry-in)

and produces:

- `sum`
- `co` (carry-out)

---

## 2. RCA USING GENERATE FOR LOOP

### Verilog Code

```verilog
module rca (
    input [7:0] num1,
    input [7:0] num2,
    output [8:0] sum
);

wire [7:0] int_sum;
wire [7:0] int_co;

genvar i;

generate
    for (i = 1; i < 8; i = i + 1) begin
        fa u_fa_1 (
            .a(num1[i]),
            .b(num2[i]),
            .c(int_co[i-1]),
            .co(int_co[i]),
            .sum(int_sum[i])
        );
    end
endgenerate

fa u_fa_0 (
    .a(num1[0]),
    .b(num2[0]),
    .c(1'b0),
    .co(int_co[0]),
    .sum(int_sum[0])
);

assign sum[7:0] = int_sum;
assign sum[8] = int_co[7];

endmodule
```

---

## 3. UNDERSTANDING GENERATE FOR

The important point in this lab is that this is a **generate `for` loop**.

```verilog
genvar i;

generate
    for (i = 1; i < 8; i = i + 1) begin
        fa u_fa_1 (...);
    end
endgenerate
```

Unlike a procedural `for` loop used inside an `always` block, this `generate for` loop is used to **instantiate multiple hardware modules**.

It is written outside the `always` block.

---

## 4. WHY GENVAR IS USED

`genvar` is used as the loop variable for a generate block.

```verilog
genvar i;
```

The elaboration tool uses `i` to create multiple instances of the Full Adder.

Conceptually, the loop creates:

```text
i = 1 → Full Adder instance
i = 2 → Full Adder instance
i = 3 → Full Adder instance
i = 4 → Full Adder instance
i = 5 → Full Adder instance
i = 6 → Full Adder instance
i = 7 → Full Adder instance
```

So instead of manually writing seven Full Adder instantiations, the generate loop creates them automatically.

---

## 5. RIPPLE CARRY ADDER STRUCTURE

The RCA consists of multiple 1-bit Full Adders connected in series.

```text
num1[0] ─┐
num2[0] ─┤
          FA0 ── carry ──> FA1 ──> FA2 ──> ... ──> FA7
          │
        sum[0]
```

The carry output of one Full Adder becomes the carry input of the next Full Adder.

```text
FA0 → FA1 → FA2 → FA3 → FA4 → FA5 → FA6 → FA7
```

This is why it is called a **Ripple Carry Adder**.

---

## 6. WHY THE FIRST FULL ADDER IS OUTSIDE THE LOOP

The generate loop starts from:

```verilog
for (i = 1; i < 8; i = i + 1)
```

because the first Full Adder, `FA0`, needs a constant carry-in:

```verilog
.c(1'b0)
```

Therefore, it is instantiated separately:

```verilog
fa u_fa_0 (
    .a(num1[0]),
    .b(num2[0]),
    .c(1'b0),
    .co(int_co[0]),
    .sum(int_sum[0])
);
```

The remaining Full Adders use the carry from the previous stage.

---

## 7. HARDWARE GENERATED BY THE FOR LOOP

The generate loop:

```verilog
for (i = 1; i < 8; i = i + 1)
```

effectively creates multiple Full Adder instances.

Conceptually:

```text
i = 1 → FA1
i = 2 → FA2
i = 3 → FA3
i = 4 → FA4
i = 5 → FA5
i = 6 → FA6
i = 7 → FA7
```

Together with the separately instantiated `FA0`, the final hardware contains:

```text
FA0 + FA1 + FA2 + FA3 + FA4 + FA5 + FA6 + FA7
```

Therefore, the RCA contains **8 Full Adder instances**.

---

## 8. WAVEFORM OBSERVATION

<img width="948" height="753" alt="image" src="https://github.com/user-attachments/assets/9c83ecec-68d0-4e01-b32d-75eedd02cb55" />


The GTKWave waveform shows:

```text
num1[7:0]
num2[7:0]
sum[8:0]
```

The output `sum` changes according to the addition of `num1` and `num2`.

For example, in the waveform, `num1` changes continuously while `num2` remains at a particular value for a period of time. The `sum` output correspondingly changes with the addition result.

The waveform confirms that the generated Full Adder instances are correctly connected and that the 8-bit RCA performs the expected addition.

---

## 9. SYNTHESIS OBSERVATION
<img width="976" height="825" alt="image" src="https://github.com/user-attachments/assets/740d6f5f-aa5f-49b6-b18c-159cb688d64d" />


The synthesized schematic shows multiple Full Adder blocks connected together:

```text
FA0 → FA1 → FA2 → FA3 → FA4 → FA5 → FA6 → FA7
```

The carry signals connect the Full Adders in sequence.

The schematic demonstrates that the `generate for` loop has resulted in multiple instances of the `fa` module.

---

## 10. IMPORTANT DIFFERENCE — PROCEDURAL FOR vs GENERATE FOR

There are two important uses of `for` loops in Verilog.

### Procedural FOR Loop

A procedural `for` loop is generally written inside an `always` block.

It is used to describe or evaluate repetitive RTL logic.

```verilog
always @(*) begin
    for (k = 0; k < 8; k = k + 1) begin
        ...
    end
end
```

### Generate FOR Loop

A generate `for` loop is written outside procedural blocks and is used for **repeated module or hardware instantiation**.

```verilog
genvar i;

generate
    for (i = 1; i < 8; i = i + 1) begin
        fa u_fa_1 (...);
    end
endgenerate
```

So the key difference is:

```text
Procedural FOR
→ Used for repetitive RTL logic/evaluation
→ Generally used inside always blocks

Generate FOR
→ Used for repeated hardware/module instantiation
→ Used outside always/procedural blocks
→ Uses genvar
```

---

## 11. KEY LEARNING

This lab helped me understand that a **generate `for` loop is mainly used to replicate hardware structures**.

It is especially useful when the same module needs to be instantiated many times.

Examples include:

```text
Ripple Carry Adder
Register arrays
Parallel processing elements
Repeated combinational blocks
Wide datapaths
Bit-sliced architectures
```

Instead of manually writing:

```text
FA0
FA1
FA2
FA3
FA4
FA5
FA6
FA7
```

a generate `for` loop can create the repeated instances compactly.

---

## FINAL CONCLUSION

The `generate for` loop is different from a procedural `for` loop.

```text
GENERATE FOR
        ↓
Used outside always/procedural blocks
        ↓
Uses genvar
        ↓
Used for repeated module/hardware instantiation
        ↓
Creates multiple hardware instances
```

In this lab, the generate `for` loop is used to instantiate multiple Full Adders and construct an 8-bit Ripple Carry Adder.

The main learning is:

**Generate `for` is used for hardware/module replication and is very useful when the same hardware block needs to be instantiated multiple times.**
