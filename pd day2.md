# PicoRV32A Layout Exploration with Magic VLSI

## 1. Project Overview

This work focuses on examining the physical implementation of the PicoRV32A RISC-V processor using **Magic VLSI** with the **SKY130A** technology.

After digital synthesis and physical implementation, the processor is represented using standard cells, interconnects, physical structures, and technology-specific layers. Magic VLSI provides a way to inspect these structures at different levels of detail.

The purpose of this exercise is to connect the concepts of digital design with their corresponding physical representation on an ASIC layout.

---

## 2. Aim of the Experiment

The main goals of this practical are:

- To become familiar with physical IC layout.
- To inspect the implemented PicoRV32A design in Magic VLSI.
- To identify standard-cell instances in the layout.
- To examine physical cells and signal labels.
- To understand the purpose of different layout layers.
- To study the placement of cells in the chip area.
- To observe the design at different zoom levels.
- To relate the synthesized circuit to its physical implementation.
- To gain practical exposure to the Magic VLSI environment.

---

## 3. Software and Technology Stack

| Component | Application |
|---|---|
| PicoRV32A | RISC-V processor design |
| Magic VLSI | Physical layout visualization |
| SKY130A | CMOS technology and PDK |
| Standard Cell Library | Digital circuit implementation |
| DRC | Layout rule verification |
| LVS | Connectivity verification |
| GDSII | Physical design data format |

---

## 4. Understanding PicoRV32A

PicoRV32A is a small 32-bit processor based on the RISC-V instruction-set architecture.

The design begins as an RTL description. During the ASIC implementation flow, this RTL is synthesized and mapped to standard cells. These cells are then physically arranged and interconnected to form the final layout.

Some of the logic elements that may appear in the implemented design are:

- Flip-flops
- Multiplexers
- Buffers
- Inverters
- NOR gates
- AND gates
- Other combinational logic cells

Magic VLSI allows these elements and their physical connections to be inspected directly.

---

## 5. SKY130A Technology

The **SKY130A** technology is a 130 nm CMOS technology used for ASIC design and implementation.

The PDK contains information required by the physical design tools, including:

- Process layers
- Standard-cell definitions
- Design rules
- Physical dimensions
- Cell geometries
- Electrical characteristics
- Timing-related information

Magic VLSI uses the technology information from the PDK to display the layout according to the selected fabrication technology.

---

## 6. Exploring the Layout in Magic VLSI

Magic VLSI is used to open and inspect the physical representation of the PicoRV32A processor.

During layout exploration, the following elements can be examined:

- Top-level PicoRV32A block
- Standard-cell instances
- Physical cells
- Signal labels
- Metal layers
- Cell boundaries
- Power structures
- Interconnections
- Layout geometries

The Magic VLSI interface also indicates the technology being used for the layout.

### My Magic VLSI Layout

<img width="1157" height="717" alt="image" src="https://github.com/user-attachments/assets/59887f60-f1bd-47f0-bc28-d00bd4c2b55c" />

---

# 7. Layout Examination at Different Zoom Levels

Different zoom levels provide different information about the physical implementation. A complete-chip view gives an idea of the overall area, while zooming into smaller regions makes individual cells and signals easier to identify.

## 7.1 Overall Layout View

At a high-level view, the PicoRV32A implementation appears as a large rectangular physical region containing numerous repeated structures.

At this scale, individual standard cells are difficult to distinguish because of the large number of geometries present in the design.

### What can be observed

- Overall layout boundary
- General physical dimensions
- Density of the implemented design
- Repeated physical structures
- Overall organization of the chip area

<img width="1166" height="680" alt="image" src="https://github.com/user-attachments/assets/d2c9937d-b99a-4a57-96dd-c91334518cda" />


---

## 7.2 Inspecting Signals and Cell Labels

A closer view of the layout makes signal names and physical-cell identifiers easier to recognize.

Examples of signal labels visible in the design include:

```text
pcpi_insn[30]
eoi[24]
```

Some physical-cell identifiers can also be seen:

```text
PHY_252
PHY_250
PHY_248
PHY_246
PHY_244
PHY_242
```

The top-level design name `picorv32a` can also be identified.

### Observation

Zooming into the layout helps establish the connection between logical signal names and their corresponding physical structures.

<img width="1152" height="692" alt="image" src="https://github.com/user-attachments/assets/93298176-a93d-40b9-a5cc-4ca7c6ab737d" />

---

## 7.3 Inspecting the Internal Regions

Further zooming into the processor reveals smaller physical regions and individual structures.

For example, the signal:

```text
trace_data[26]
```

can be identified in the layout.

The view also makes rectangular regions, cell structures, and their physical organization more visible.

### Observation

A detailed view provides a better understanding of how the internal portions of the processor are physically organized.

<!-- Add your detailed-layout screenshot here -->

---

## 7.4 Complete Design View

Another complete-layout view shows the PicoRV32A block as a dense physical region containing a large number of structures.

The label:

```text
picorv32a
```

can be identified within the layout.


---

# 8. Identifying Standard-Cell Instances

The physical implementation uses cells from the SKY130 standard-cell library.

Some library names follow patterns such as:

```text
sky130_fd_sc_hd__dff...
sky130_fd_sc_hd__mux...
sky130_fd_sc_hd__buf...
sky130_fd_sc_hd__inv...
sky130_fd_sc_hd__nor...
```

These cells perform different logical functions required by the processor.

| Cell Type | Function |
|---|---|
| DFF | Stores sequential data |
| MUX | Selects between input signals |
| BUF | Provides signal buffering |
| INV | Performs logical inversion |
| NOR | Implements NOR logic |

The cells are physically placed and connected to implement the required processor functionality.



# 9. Physical Cell Identification

Apart from logical standard cells, the layout also contains physical structures identified by names such as:

```text
PHY_252
PHY_250
PHY_248
PHY_246
PHY_244
PHY_242
```

These identifiers represent physical cells or structures used within the implemented layout.

Their presence shows that the physical design contains additional implementation structures along with the logical standard cells.



---

# 10. Signal Labels in the Layout

Signal names provide a way to identify electrical connections within the processor.

Some signals that can be observed include:

```text
pcpi_insn[30]
eoi[24]
trace_data[26]
```

These labels help locate particular signals and understand how different regions of the design are interconnected.



---

# 11. Standard-Cell Placement

Placement determines the physical locations of the synthesized standard cells inside the available chip area.

The cells are arranged in structured rows to make efficient use of the physical region.

A simplified representation of the placement process is:

```text
Synthesized Cells
       ↓
Global Placement
       ↓
Detailed Placement
       ↓
Final Cell Arrangement
```

Efficient placement is important because it affects:

- Interconnect length
- Routing congestion
- Timing
- Area utilization
- Routing complexity

---

## 11.1 Dense Placement View

A detailed placement view shows a large number of standard cells and physical geometries distributed throughout the design.

Library identifiers containing:

```text
sky130_fd_sc_hd__
```

indicate cells belonging to the SKY130 standard-cell library.

Examples include:

```text
dff
mux
buf
inv
nor
```

### Placement Observation

The repeated structures visible in the layout indicate the physical arrangement of standard cells within the processor area.
<img width="1145" height="636" alt="image" src="https://github.com/user-attachments/assets/3be1be9c-ad3f-4318-b52a-7895f171b374" />



---

## 11.2 Detailed Cell and Layer Inspection

A further zoomed view provides more information about the individual physical structures.

The following elements can be identified:

- Standard-cell boundaries
- Cell names
- Physical layers
- Vertical structures
- Horizontal structures
- Connections between cells

Different colors and patterns correspond to different physical layers used by the technology.

<img width="1117" height="652" alt="image" src="https://github.com/user-attachments/assets/6caf8a8e-eae4-49ba-ab46-321649509d2e" />

---

# 12. Routing of the Design

Once cells have been positioned, physical connections must be established between them.

Routing uses different metal layers to create these electrical connections.

The routing sequence can be represented as:

```text
Placed Cells
     ↓
Global Routing
     ↓
Detailed Routing
     ↓
Final Physical Connections
```

The wires and metal geometries visible in detailed Magic VLSI views represent the physical interconnections between circuit elements.


---

# 13. Understanding Physical Layout Layers

An integrated-circuit layout is composed of multiple technology-specific layers.

These layers are used for different physical purposes, including:

- Device formation
- Contacts
- Interconnects
- Metal routing
- Power distribution

Magic VLSI represents these layers using different colors and patterns.

The layer-selection area in the Magic VLSI interface can be used to identify and inspect the available technology layers.


---

# 14. Power Distribution

Power delivery is an important part of physical implementation.

The primary supply connections are:

```text
VDD → Power Supply
VSS → Ground
```

The physical power structures distribute the required supply throughout the standard-cell region.

A suitable power network helps provide reliable supply connections to the implemented circuit.


---

# 15. Physical Hierarchy of PicoRV32A

The top-level design visible in the layout is:

```text
picorv32a
```

The top-level block contains numerous logical and physical structures.

A simplified representation is:

```text
picorv32a
    |
    +── Standard-Cell Instances
    |
    +── Physical Structures
    |
    +── Signal Interconnections
    |
    +── Power Structures
```

Understanding this hierarchy makes it easier to navigate a large physical design.

---

# 16. Physical Verification

After physical implementation, the layout needs to be checked to ensure that it satisfies the required physical and connectivity rules.

## 16.1 Design Rule Check

**DRC (Design Rule Check)** verifies whether the layout follows the manufacturing rules associated with the selected technology.

Typical checks include:

- Minimum width
- Minimum spacing
- Via requirements
- Layer restrictions
- Enclosure rules

## 16.2 Layout Versus Schematic

**LVS (Layout Versus Schematic)** compares the connectivity represented by the physical layout against the intended circuit or netlist.

A simplified verification sequence is:

```text
Physical Layout
      ↓
     DRC
      ↓
     LVS
      ↓
Verified Design
```


---

# 17. From RTL to Physical Implementation

The complete transformation from the processor RTL to the physical layout can be summarized as:

```text
RTL Design
    ↓
Logic Synthesis
    ↓
Gate-Level Netlist
    ↓
Standard-Cell Mapping
    ↓
Cell Placement
    ↓
Physical Routing
    ↓
ASIC Layout
    ↓
DRC / LVS
    ↓
GDSII
```

The Magic VLSI screenshots mainly represent the later physical-implementation stage, where the synthesized logic has been converted into actual layout geometries.

---

# 18. Key Layout Observations

During the inspection of the PicoRV32A layout, the following points were observed:

- The complete design occupies a rectangular physical region.
- A large number of standard-cell instances are present.
- Standard cells are arranged in organized regions.
- SKY130 library cell names can be identified.
- Physical identifiers such as `PHY_252`, `PHY_250`, and others are visible.
- Signal labels such as `pcpi_insn[30]`, `eoi[24]`, and `trace_data[26]` can be observed.
- Different technology layers are represented using different visual patterns.
- Zooming in allows individual structures and connections to be examined.
- Zooming out provides an understanding of the overall chip area and layout density.
- The layout represents the physical realization of the synthesized digital design.

---

# 19. Overall Physical Design Flow

The complete concept studied in this practical can be summarized as:

```text
PicoRV32A RTL
       ↓
Synthesis
       ↓
Gate-Level Netlist
       ↓
Standard-Cell Mapping
       ↓
Floorplanning
       ↓
Placement
       ↓
Clock Distribution
       ↓
Routing
       ↓
Physical Layout
       ↓
DRC / LVS
       ↓
GDSII
```

---

# 20. What I Learned

This practical helped me understand how a processor design moves from a logical description to an actual physical layout.

The major concepts covered were:

- Physical ASIC layout
- SKY130 technology
- Magic VLSI
- Standard-cell implementation
- Cell placement
- Physical routing
- Technology layers
- Power and ground structures
- Physical hierarchy
- DRC
- LVS
- RTL-to-GDSII flow

---

# Conclusion

The Magic VLSI inspection of PicoRV32A provided practical exposure to the physical side of ASIC design.

By examining the layout at different zoom levels, standard cells, physical structures, signal labels, routing geometries, and technology layers could be identified.

This exercise helped establish the relationship between the synthesized gate-level circuit and its physical representation, providing a foundation for further study of ASIC physical design and the complete RTL-to-GDSII flow.
