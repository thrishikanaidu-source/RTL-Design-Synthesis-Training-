# RVMyth — Functional Simulation, Synthesis and Gate-Level Simulation

## Objective

To verify the functionality of the RVMyth RISC-V core by performing functional simulation, synthesis, Gate-Level Simulation (GLS), and comparison of pre-synthesis and post-synthesis results.

### Flow

```text
RVMyth RTL
    ↓
Functional Simulation
    ↓
Synthesis
    ↓
Synthesized Netlist
    ↓
Gate-Level Simulation (GLS)
    ↓
Pre-Synthesis vs Post-Synthesis Comparison
```

---

## 1. Functional Simulation

The RVMyth RTL design was first simulated using the testbench.

The functional simulation was performed to verify the behavior of the RTL design before synthesis.

### Simulation

```text
RVMyth RTL
    ↓
Testbench
    ↓
Functional Simulation
    ↓
GTKWave
```

### Functional Simulation Waveform

<img width="1599" height="899" alt="image" src="https://github.com/user-attachments/assets/40a39e86-319e-4a88-af9b-0cb8b18cf975" />

The waveform was observed in GTKWave and the expected signals were verified.

---

## 2. Synthesis

After successful functional simulation, the RVMyth RTL design was synthesized using Yosys.

Synthesis converts the RTL design into a gate-level netlist.

### Synthesis Flow

```text
RVMyth RTL
    ↓
Yosys
    ↓
Synthesized Netlist
```

The synthesized netlist generated from this step was used for Gate-Level Simulation.

### Synthesized Schematic

<img width="1600" height="402" alt="image" src="https://github.com/user-attachments/assets/f7887ac8-216c-4e3c-a9df-34bbdee79a08" />


----


<img width="1567" height="817" alt="image" src="https://github.com/user-attachments/assets/bc2ecedc-f7e1-4c29-875e-8c89d0663926" />

---

## 3. Synthesized Netlist

The output of the synthesis process is a gate-level netlist of the RVMyth design.

```text
RTL Design
    ↓
Synthesis
    ↓
Gate-Level Netlist
```

This synthesized netlist was used as the design under test for the GLS.

---

## 4. Gate-Level Simulation (GLS)

The synthesized netlist was simulated using the same testbench.

Gate-Level Simulation was performed to verify the behavior of the synthesized design.

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

<img width="1185" height="570" alt="image" src="https://github.com/user-attachments/assets/58e334d2-2327-4bf4-b990-bdfe12ca0bcd" />

The GLS waveform was observed using GTKWave.

---

## 5. Pre-Synthesis vs Post-Synthesis Comparison

<img width="1311" height="650" alt="image" src="https://github.com/user-attachments/assets/8a819883-589a-45ca-a468-d74c020e526a" />



The functional simulation waveform was compared with the post-synthesis GLS waveform.

| Parameter | Pre-Synthesis | Post-Synthesis / GLS |
|-----------|---------------|----------------------|
| Design | RVMyth RTL | Synthesized Netlist |
| Simulation | Functional Simulation | Gate-Level Simulation |
| Testbench | RVMyth Testbench | Same Testbench |
| Functionality | Expected RTL behavior | Same functional behavior |

### Observation

The post-synthesis GLS waveform shows the same functional behavior as the pre-synthesis RTL simulation.

Small timing differences may occur because the GLS is performed using the synthesized gate-level netlist.

Therefore, the synthesized RVMyth design was successfully verified against the original RTL design.

---

## 6. Results

- RVMyth functional simulation was performed successfully.
- The RVMyth RTL design was synthesized using Yosys.
- A synthesized gate-level netlist was generated.
- Gate-Level Simulation was performed using the synthesized netlist.
- The GLS waveform was observed using GTKWave.
- Pre-synthesis and post-synthesis waveforms were compared.
- The synthesized design maintains the expected functionality of the RTL design.

---



## 9. Complete RTL-to-GLS Flow

```text
              RVMyth RTL
                  ↓
        Functional Simulation
                  ↓
               GTKWave
                  ↓
              Synthesis
                  ↓
        Synthesized Netlist
                  ↓
        Gate-Level Simulation
                  ↓
               GTKWave
                  ↓
      Pre vs Post Comparison
```

---

## Conclusion

The RVMyth design was successfully taken through the complete RTL-to-GLS flow.

Functional simulation was performed first to verify the RTL design. The design was then synthesized using Yosys to generate the gate-level netlist. The synthesized netlist was used for Gate-Level Simulation, and the resulting waveform was compared with the pre-synthesis functional simulation.

The comparison confirmed that the synthesized RVMyth design preserves the intended functionality of the RTL design.
