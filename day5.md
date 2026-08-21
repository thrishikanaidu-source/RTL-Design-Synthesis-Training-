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
