# RVMyth — Simulation, Synthesis and GLS

## Objective

To perform functional simulation of the RVMyth core, synthesize the RTL design, generate the synthesized netlist, perform Gate-Level Simulation (GLS), and compare the pre-synthesis and post-synthesis results.

---



## 1. Functional Simulation

The BabySoC RTL design was first simulated using the testbench.

The functional simulation was performed to verify the behavior of the RTL design before synthesis.

### Simulation Flow

```text
RTL Design
    ↓
Testbench
    ↓
Functional Simulation
    ↓
GTKWave
```

### Functional Simulation Waveform

<img width="1600" height="598" alt="image" src="https://github.com/user-attachments/assets/34f89ae8-f024-49c1-9b48-ab000ac77363" />

The waveform was observed using GTKWave to verify the functional behavior of the design.

---

## 2. Synthesis

<img width="1600" height="402" alt="image" src="https://github.com/user-attachments/assets/a266718d-4e91-4299-9aae-00564b782ebd" />


<img width="1567" height="817" alt="image" src="https://github.com/user-attachments/assets/be5a2ffd-45ea-4187-ab98-d1e2c86d2258" />


After functional simulation, the BabySoC RTL design was synthesized using Yosys.

The required RTL files and library files were read first, followed by synthesis of the top module.

### Yosys Commands

```yosys
yosys

read_verilog src/module/vsdbabysoc.v

read_verilog -I src/include src/module/rvmyth.v

read_verilog -I src/include src/module/clk_gate.v

read_liberty -lib src/lib/avsdpll.lib

read_liberty -lib src/lib/avsddac.lib

read_liberty -lib src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

synth -top vsdbabysocabc -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

flatten

setundef -zero

clean -purge

rename -enumerate
```

### Synthesis Flow

```text
vsdbabysoc.v
     ↓
rvmyth.v
     ↓
clk_gate.v
     ↓
Library Files
     ↓
Yosys Synthesis
     ↓
Synthesized Netlist
```

---


##  Gate-Level Simulation (GLS)

The synthesized netlist generated during synthesis was used for Gate-Level Simulation.

The same testbench was used to simulate the synthesized design.

### GLS Flow

```text
Synthesized Netlist
        ↓
    Testbench
        ↓
       GLS
        ↓
    GTKWave
```

### GLS Waveform

<img width="1307" height="640" alt="image" src="https://github.com/user-attachments/assets/a4467dcb-0f36-425d-95b7-c356ad372b3a" />

The GLS waveform was observed using GTKWave.

---

##  Pre-Synthesis vs Post-Synthesis Comparison

<img width="1600" height="819" alt="image" src="https://github.com/user-attachments/assets/42484470-046d-4994-a682-75b3d84bcb93" />


The pre-synthesis functional simulation and post-synthesis GLS waveforms were compared.

| Parameter | Pre-Synthesis | Post-Synthesis / GLS |
|-----------|---------------|----------------------|
| Design | RTL | Synthesized Netlist |
| Simulation | Functional Simulation | Gate-Level Simulation |
| Testbench | Same Testbench | Same Testbench |
| Functionality | Expected behavior | Same functional behavior |

### Observation

The post-synthesis GLS waveform follows the expected functional behavior of the pre-synthesis simulation.

Small timing differences may occur in GLS because the synthesized design contains gate and cell propagation delays.

---

## Results

- Functional simulation was completed successfully.
- The RTL design was synthesized using Yosys.
- A synthesized gate-level netlist was generated.
- The synthesized netlist was used for Gate-Level Simulation.
- The GLS waveform was observed using GTKWave.
- Pre-synthesis and post-synthesis waveforms were compared.
- The synthesized design maintains the expected functionality of the RTL design.

---

##  Complete RTL-to-GLS Flow

```text
                  RTL Design
                      ↓
             Functional Simulation
                      ↓
                   GTKWave
                      ↓
                Yosys Synthesis
                      ↓
             Synthesized Netlist
                      ↓
             Gate-Level Simulation
                      ↓
                   GTKWave
                      ↓
       Pre-Synthesis vs Post-Synthesis
                  Comparison
```

---

## Conclusion

The BabySoC design was successfully taken through the complete RTL-to-GLS flow.

The design was first functionally simulated and verified. It was then synthesized using Yosys with the Sky130 standard-cell library. The synthesized netlist was used for Gate-Level Simulation, and the GLS waveform was compared with the pre-synthesis functional simulation.

The comparison confirms that the synthesized design maintains the intended functionality of the RTL design.
