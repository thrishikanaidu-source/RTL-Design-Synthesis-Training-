# Day 3: Combinational and sequential optmizations

## 1. Combinational Logic Optimization

Combinational logic optimization means simplifying the logic circuit to obtain an efficient implementation without changing its functionality.

The main objectives are:

- Reduce circuit area
- Reduce power consumption
- Reduce propagation delay
- Improve overall design performance

### Important Concepts

- **Boolean Logic Optimization** – Simplifying Boolean expressions to reduce the required logic gates.
- **Constant Propagation** – Identifying signals that have constant values such as `0` or `1` and simplifying the logic accordingly.
- **Critical Path Optimization** – Reducing the delay of the longest timing path in the circuit.

---

## 2. Sequential Logic Optimization

Sequential logic optimization deals with optimizing circuits containing storage elements such as flip-flops.

Important techniques include:

- **State Optimization** – Reducing unnecessary states in a sequential circuit.
- **Cloning** – Replicating logic or registers to improve timing or reduce fanout-related problems.
- **Retiming** – Moving flip-flops across combinational logic while preserving functionality to improve timing.

---

# 11. Lab: Constant Propagation / Logic Optimization

In the lab, a simple Verilog design was created to observe logic optimization.

## Verilog Code

```verilog
module opt_check (input a, input b, output y);
    assign y = a ? b : 0;
endmodule
```

## Explanation

The conditional expression:

```text
y = a ? b : 0
```

means:

- When `a = 1`, `y = b`
- When `a = 0`, `y = 0`

Therefore, the logic is equivalent to:

```text
y = a AND b
```

This demonstrates how synthesis tools can identify and simplify logic while maintaining the same functionality.
<img width="1106" height="250" alt="WhatsApp Image 2026-08-19 at 9 57 11 PM" src="https://github.com/user-attachments/assets/cf7c31b8-6e4c-4480-b31c-671130af5d04" />

<img width="342" height="330" alt="image" src="https://github.com/user-attachments/assets/9b70c190-cad2-4135-9c9e-06d97344eab2" />

<img width="697" height="270" alt="image" src="https://github.com/user-attachments/assets/33c6f5e5-b7da-4826-b3b5-f26e7f5d0ac3" />

----
# Lab 2: Sequential Logic Optimization

## Verilog Code

~~~verilog
module dff_const1(input clk, input reset, output reg q);

always @(posedge clk, posedge reset)
begin
    if(reset)
        q <= 1'b0;
    else
        q <= 1'b1;
end

endmodule
~~~

## Simulation

The simulation verifies the behavior of the D flip-flop with reset.

- When `reset` is active, `q` becomes `0`.
- When reset is released, `q` becomes `1` at the next positive clock edge.

## Synthesis

During synthesis, the tool identifies that the flip-flop receives a constant value of `1` when reset is not active. Therefore, the logic can be optimized.

## Yosys Output
<img width="898" height="471" alt="image" src="https://github.com/user-attachments/assets/dd4543c6-e51b-4541-8bf3-3edd309e4164" />

<img width="1262" height="417" alt="image" src="https://github.com/user-attachments/assets/f3b80e31-543f-4d51-a8be-3d07d27048e9" />

<img width="1315" height="602" alt="image" src="https://github.com/user-attachments/assets/fe1b295e-9c6a-4ce3-8c85-35eb2e364c32" />




The Yosys output shows the optimized design and the cell statistics generated during synthesis.

<img width="1038" height="435" alt="image" src="https://github.com/user-attachments/assets/1a30f13b-9335-4174-9945-50ecdab2cd39" />



---

# Lab 3: Unused Logic Optimization

## Verilog Code

~~~verilog
module counter_opt(input clk, input reset, output q);

reg [2:0] count;

assign q = count[0];

always @(posedge clk, posedge reset)
begin
    if(reset)
        count <= 3'b000;
    else
        count <= count + 1;
end

endmodule
~~~

## Simulation

The simulation verifies the operation of the 3-bit counter.

- When `reset` is active, the counter is cleared to `3'b000`.
- On every positive clock edge, the counter increments by `1`.
- The output `q` is connected only to `count[0]`.

## Synthesis

The synthesis tool identifies logic that does not affect the required output.

The output is connected only to:

~~~verilog
assign q = count[0];
~~~

Therefore, the higher-order bits `count[2:1]` are not directly connected to the output and unnecessary logic can be optimized.


## Lab Screenshots
<img width="442" height="243" alt="image" src="https://github.com/user-attachments/assets/a67a7b6d-9fa1-4aec-a1b9-730dc3b29b39" />





<img width="442" height="243" alt="image" src="https://github.com/user-attachments/assets/a67a7b6d-9fa1-4aec-a1b9-730dc3b29b39" />



<img width="1220" height="462" alt="image" src="https://github.com/user-attachments/assets/d4b95e92-9de0-449b-b89c-0aa1725147bb" />




# Yosys Commands

## Start Yosys

~~~bash
yosys
~~~

## Read Verilog File

~~~bash
read_verilog dff_const1.v
~~~

## Process Conversion

~~~bash
proc
~~~

## Optimization

~~~bash
opt
~~~

## Check Design

~~~bash
check
~~~

## Display Statistics

~~~bash
stat
~~~

## Map Flip-Flops Using Liberty File

~~~bash
dfflibmap -liberty ../my_lib/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
~~~

## View Synthesized Circuit

~~~bash
show
~~~

## Write Optimized Verilog

~~~bash
write_verilog optimized.v
~~~

## Conclusion

The experiments on Sequential Logic Optimization and Combinational Logic Optimization demonstrated how synthesis tools such as Yosys optimize RTL designs. Sequential logic optimization simplifies constant or unnecessary flip-flop logic, while combinational logic optimization removes redundant logic and simplifies Boolean expressions. These optimizations reduce hardware area and complexity while maintaining the required functionality of the design.
