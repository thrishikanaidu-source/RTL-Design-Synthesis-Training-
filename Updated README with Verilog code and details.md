Detailed explanation, simulation steps, screenshots, and results are available here

# 🎯 Objective:

To understand the fundamentals of Verilog HDL by designing, simulating, and synthesizing a 2:1 Multiplexer using iverilog and analysis of waveform using GTKwave
________
Contents:

-Digital Design Verification

-Simulation Workflow with Icarus Verilog

-Practical Exercise – Simulating a 2:1 Multiplexer

-Multiplexer Design Explanation

-Conclusion

_______
## Theory:

# Simulator

A simulator is a software tool that executes your Verilog design and shows how it behaves before it is implemented in hardware.

# Design

A design is the Verilog code that describes the functionality and behavior of a digital hardware circuit.

# Testbench

A testbench is a Verilog module used to apply test inputs to a design and verify that it produces the expected outputs.
<img width="1121" height="557" alt="image" src="https://github.com/user-attachments/assets/d920664a-cb75-4b57-b5ac-f594613a47f8" />


--------
# Simulation Flow:

Verilog Design + Testbench

          │
          ▼
          
 Icarus Verilog (Simulator)
 
          │
          ▼
          
      VCD Waveform
      
          │
          ▼
          
 GTKWave (Waveform Viewer)
 
 

 <img width="1227" height="592" alt="image" src="https://github.com/user-attachments/assets/96913e1c-4997-4e9f-9de5-96c38dfe1eb4" />

 -----
#  Step-by-Step Commands Used
 1. Installation of Icarus Verilog and GTKWave

```bash
sudo apt install iverilog 
sudo apt install gtkwave 
```

 2. Compilation 

```bash
iverilog good_mux.v tb_good_mux.v
```
3. Run the Simulation

```bash
./a.out
```
4. View the Waveform

```bash
gtkwave tb_good_mux.vcd
```    
 GTKwave form output
 <img width="1257" height="662" alt="image" src="https://github.com/user-attachments/assets/14b2a4c2-362e-4f4f-a2de-911279f6d6bd" />
----
## Verilog Code

```verilog
module good_mux (
    input i0,
    input i1,
    input sel,
    output reg y
);

always @(*)
begin
    if (sel)
        y <= i1;
    else
        y <= i0;
end

endmodule
```
-----
# verilog screen shot

<img width="1265" height="655" alt="image" src="https://github.com/user-attachments/assets/fe1a7e42-72d2-4351-83fe-36a618e921aa" />

---
## Conclusion
Through this experiment, I learned the basic RTL design flow using Verilog. I understood the purpose of a simulator, design, and testbench, successfully compiled and simulated a 2:1 Multiplexer using Icarus Verilog, and verified the circuit's functionality using GTKWave. This experiment provided a strong foundation for further digital design experiments.

