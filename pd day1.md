# OpenLane Physical Design – PicoRV32A

##  Overview

This project focuses on implementing the **PicoRV32A processor** through the ASIC physical design flow using **OpenLane** and the **Sky130 PDK**.

The objective is to understand how a digital design moves from RTL description to a physical layout through different stages of the ASIC flow.

The major stages covered in this project are:

- OpenLane setup
- Design preparation
- Logic synthesis
- Gate-level netlist generation
- Floorplanning
- Power planning
- Placement
- Clock Tree Synthesis (CTS)
- Routing
- Static Timing Analysis (STA)
- Physical verification
- Signoff

---

## PicoRV32A

**PicoRV32A** is a compact RISC-V processor core based on the RV32I instruction set architecture.

In this project, PicoRV32A is used as the design for understanding the complete physical design process using OpenLane.

The RTL design is progressively transformed into a gate-level netlist and then into a physical implementation.

---

## Sky130 PDK

The **Sky130 PDK** is an open-source Process Design Kit for the SkyWater 130 nm CMOS technology.

It provides the technology information required by the EDA tools during ASIC implementation.

The PDK contains:

- Standard-cell libraries
- Timing libraries
- Technology files
- Design rules
- Physical library information

The Sky130 technology is used as the target technology for the PicoRV32A physical design.

---

## OpenLane

**OpenLane** is an open-source automated RTL-to-GDSII flow.

It combines different EDA tools to perform synthesis and physical implementation of digital designs.

The general OpenLane flow used in this project is:

```text
RTL Design
    ↓
Design Preparation
    ↓
Logic Synthesis
    ↓
Gate-Level Netlist
    ↓
Floorplanning
    ↓
Power Planning
    ↓
Placement
    ↓
Clock Tree Synthesis
    ↓
Routing
    ↓
Static Timing Analysis
    ↓
Physical Verification
    ↓
Signoff
    ↓
GDSII
----
# OpenLane Setup

## Launching OpenLane

The OpenLane flow is started from the OpenLane working directory using interactive mode.

```bash
./flow.tcl -interactive
```

This command starts the OpenLane interactive shell.

## Loading the OpenLane Package

```tcl
package require openlane 0.9
```

This makes the OpenLane flow commands available in the interactive environment.

## Preparing the PicoRV32A Design

Before running the physical design stages, the PicoRV32A design needs to be prepared.

```tcl
prep -design picorv32a
```

The `prep` command initializes the design and prepares the required configuration and working directories for the OpenLane run.

## Running Synthesis

```tcl
run_synthesis
```

This converts the RTL description into a gate-level representation using the selected standard-cell library.

## OpenLane Setup Result

<img width="970" height="522" alt="image" src="https://github.com/user-attachments/assets/6c2924bf-7bbf-4430-9551-041bae31bb57" />

# Logic Synthesis

## Synthesis Overview

Logic synthesis converts the PicoRV32A RTL design into a gate-level netlist that can be implemented using the target technology library.

The synthesis process includes:

- RTL elaboration
- Logic optimization
- Technology mapping
- Standard-cell selection
- Gate-level netlist generation

## Synthesis Command

```tcl
run_synthesis
```

## Synthesis Statistics

| Parameter | Value |
|---|---:|
| Total Wires | 22,926 |
| Wire Bits | 25,811 |
| Public Wires | 1,162 |
| Public Wire Bits | 1,972 |
| Memories | 0 |
| Memory Bits | 0 |
| Processes | 0 |
| Total Cells | 18,036 |
| Flip-Flops | 1,613 |

## Flip-Flop Percentage

The percentage of flip-flops among the total synthesized cells is:

```text
(1613 / 18036) × 100 ≈ 8.94%
```

Therefore, approximately **8.94% of the total synthesized cells are flip-flops**.

<img width="725" height="397" alt="image" src="https://github.com/user-attachments/assets/696e5221-d399-4f78-a26c-e7b8b054fdcd" />

# Gate-Level Netlist

After synthesis, the RTL design is converted into a gate-level netlist consisting of technology-specific standard cells.

The generated netlist contains different types of cells such as:

- Logic gates
- Multiplexers
- Flip-flops
- Buffers
- Inverters

<img width="967" height="502" alt="image" src="https://github.com/user-attachments/assets/0b417f53-57eb-4d19-a46a-ddd19650dd74" />

# Floorplanning

Floorplanning defines the physical organization of the design before placement and routing.

The main aspects considered during floorplanning are:

- Core dimensions
- Die dimensions
- Standard-cell placement area
- I/O locations
- Overall chip organization

<img width="977" height="572" alt="image" src="https://github.com/user-attachments/assets/10cfe0c8-492f-4a59-8981-e9f29dd43eb5" />

# Power Planning

Power planning establishes the power distribution network required to supply the cells throughout the design.

The main power connections are:

- **VDD** — Power supply
- **VSS** — Ground

A proper power distribution network helps provide stable power to the standard cells.

# Placement

During placement, the synthesized standard cells are positioned inside the available core area.

Placement considers:

- Cell density
- Timing
- Wire length
- Routing congestion
- Available area

<img width="1022" height="512" alt="image" src="https://github.com/user-attachments/assets/d6e2f704-332f-43bb-a73e-e03735c6ad01" />

# Clock Tree Synthesis

Clock Tree Synthesis (CTS) creates a clock distribution network that delivers the clock signal to sequential elements across the design.

The main objectives are:

- Reduce clock skew
- Control clock delay
- Distribute the clock properly
- Improve timing behavior

# Routing

Routing establishes the physical connections between the placed cells.

The routing process consists mainly of:

- **Global Routing** — Determines the overall routing paths.
- **Detailed Routing** — Creates the final physical connections using available metal layers.

# Area Analysis

Area analysis is used to evaluate the physical size and utilization of the implemented design.

Important parameters include:

- Physical design size
- Core utilization
- Cell utilization
- Overall implementation efficiency

<img width="1020" height="575" alt="image" src="https://github.com/user-attachments/assets/008b8ea9-524b-40d9-9eab-f8dba0534ebd" />

# Static Timing Analysis

Static Timing Analysis (STA) checks whether the implemented design satisfies its timing requirements.

The analysis considers:

- Setup time
- Hold time
- Data path delay
- Clock delay
- Slack



# Physical Signoff

Physical signoff is the final verification stage before the design is considered ready for fabrication.

The major checks include:

- Design Rule Check (DRC)
- Layout Versus Schematic (LVS)
- Static Timing Analysis (STA)
- Antenna checks
- Physical verification

# OpenLane Command Summary

| Operation | Command |
|---|---|
| Start OpenLane | `./flow.tcl -interactive` |
| Load OpenLane package | `package require openlane 0.9` |
| Prepare design | `prep -design picorv32a` |
| Run synthesis | `run_synthesis` |

## Initial OpenLane Commands

```bash
./flow.tcl -interactive
```

```tcl
package require openlane 0.9
prep -design picorv32a
run_synthesis
```

# Synthesis Summary

| Parameter | Value |
|---|---:|
| Total Wires | 22,926 |
| Wire Bits | 25,811 |
| Public Wires | 1,162 |
| Public Wire Bits | 1,972 |
| Memories | 0 |
| Memory Bits | 0 |
| Processes | 0 |
| Total Cells | 18,036 |
| Flip-Flops | 1,613 |

<img width="1007" height="557" alt="image" src="https://github.com/user-attachments/assets/e90b13d8-796f-4889-a1f5-530dbfcba1c1" />

# Physical Design Flow Summary

The complete implementation flow can be represented as:

```text
PicoRV32A RTL
      ↓
OpenLane Setup
      ↓
Design Preparation
      ↓
Logic Synthesis
      ↓
Gate-Level Netlist
      ↓
Floorplanning
      ↓
Power Planning
      ↓
Standard-Cell Placement
      ↓
Clock Tree Synthesis
      ↓
Routing
      ↓
Static Timing Analysis
      ↓
Physical Verification
      ↓
Signoff
      ↓
GDSII
```

# Tools and Technologies Used

- **OpenLane** — RTL-to-GDSII implementation flow
- **Yosys** — Logic synthesis
- **OpenROAD** — Physical design implementation
- **Sky130 PDK** — Technology and standard-cell library
- **PicoRV32A** — RISC-V processor core
- **Docker** — Containerized OpenLane environment
- **Ubuntu / Linux** — Development environment

# Key Learnings

Through this implementation, the following concepts were studied:

- RTL-to-GDSII implementation
- Logic synthesis
- Gate-level netlist generation
- Standard-cell mapping
- Floorplanning
- Power distribution
- Standard-cell placement
- Clock Tree Synthesis
- Physical routing
- Static Timing Analysis
- Physical verification
- Final design signoff

# Conclusion

The PicoRV32A design was taken through the major stages of the OpenLane physical design flow. This process provided practical understanding of how an RTL design is progressively transformed into a gate-level implementation and finally into a physical ASIC layout ready for signoff.
