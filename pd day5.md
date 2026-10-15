# Day 10: Detailed Routing, Design Rule Checking, and TritonRoute

**Module:** RTL-to-GDS Physical Design Flow  
**Focus Areas:** Maze Routing | DRC | Global Routing | Detailed Routing | Power Distribution Network

## Overview

This module explores the physical routing stage that follows placement in the RTL-to-GDS flow. It introduces Lee's maze-routing algorithm, explains the manufacturing constraints checked by Design Rule Checking (DRC), and describes how TritonRoute converts global-routing guides into detailed interconnect geometry.

The module also covers the connectivity and topology strategies used by TritonRoute, followed by practical exercises involving Power Distribution Network (PDN) construction and routing.

## Contents

1. [Maze Routing Using Lee's Algorithm](#1-maze-routing-using-lees-algorithm)
2. [Limitations and Significance of Lee's Algorithm](#2-limitations-and-significance-of-lees-algorithm)
3. [Design Rule Checking](#3-design-rule-checking-drc)
4. [Global Routing, Detailed Routing, and TritonRoute](#4-global-routing-detailed-routing-and-tritonroute)
5. [TritonRoute Feature 1: Route Guide Handling](#5-tritonroute-feature-1-route-guide-handling)
6. [TritonRoute Features 2 and 3: Connectivity and Panel Routing](#6-tritonroute-features-2-and-3-connectivity-and-panel-routing)
7. [Connectivity Management in TritonRoute](#7-connectivity-management-in-tritonroute)
8. [Routing Topology Optimization](#8-routing-topology-optimization)
9. [Practical Lab: PDN Construction and Routing](#9-practical-lab-pdn-construction-and-routing)
10. [Conclusion](#10-conclusion)

---

## 1. Maze Routing Using Lee's Algorithm

### 1.1 Introduction

Lee's algorithm is a grid-based routing method that finds a shortest path between two points while avoiding obstacles. It uses a breadth-first search approach, commonly described as wavefront expansion.

The routing area is divided into a grid, and the algorithm explores available grid cells until it reaches the destination.

### 1.2 Working Procedure

The routing process follows these steps:

1. Represent the routing region as a uniform grid.
2. Select the source pin and assign its initial distance label.
3. Expand the search wave into neighboring cells that are available for routing.
4. Assign increasing distance values to the reachable cells.
5. Continue the expansion until the destination is reached or no further cells can be explored.
6. Trace backward from the destination by following cells with decreasing distance labels.
7. Recover the path from the source to the destination.

### 1.3 Important Characteristics

- Finds a shortest path in grid steps when a valid path exists.
- Supports routing around obstacles.
- Uses systematic wavefront expansion.
- Can require substantial processing time and memory for large routing regions.
- Provides the conceptual basis for more advanced routing methods.

The quality of the result depends on the grid representation and the legal routing locations available to the algorithm.

---

## 2. Limitations and Significance of Lee's Algorithm

Lee's algorithm is useful for understanding maze routing, but its computational cost can become significant in large VLSI designs.

### 2.1 Computational Complexity

For a grid containing \(N\) cells, the algorithm can require \(O(N)\) time and \(O(N)\) memory in the worst case, assuming each cell is explored at most once.

As the routing area grows, the number of cells that must be examined can increase considerably.

### 2.2 Modern Routing Improvements

Modern detailed routers use more advanced techniques to improve efficiency and routing quality, including:

- A* search.
- Multi-source and multi-sink routing.
- Rip-up-and-reroute operations.
- Parallel wavefront exploration.
- Congestion-aware path selection.

These techniques retain the fundamental idea of searching through available routing space while improving performance for larger designs.

### 2.3 Key Takeaway

Understanding Lee's algorithm helps explain how routing tools search for legal paths, avoid obstacles, and connect terminals. More sophisticated routers extend this foundation with additional optimization and constraint-handling methods.

---

## 3. Design Rule Checking (DRC)

### 3.1 What Is DRC?

Design Rule Checking verifies whether the physical geometry of a layout follows the manufacturing rules defined by the semiconductor process.

Routing geometry includes metal wires, vias, pins, and other shapes. Each shape must satisfy the applicable design constraints.

A route can be electrically connected and still be unsuitable for manufacturing if its geometry violates these rules.

### 3.2 Common Routing Design Rules

| Rule | Purpose |
|---|---|
| Minimum metal width | Ensures that wire width meets manufacturing requirements |
| Metal spacing | Maintains the required separation between neighboring wires |
| Via dimensions | Ensures that via geometry follows process requirements |
| Via spacing | Prevents vias from being placed too close together |
| Via enclosure | Ensures that surrounding metal adequately encloses the via |
| End-of-line spacing | Controls spacing near the ends of metal segments |
| Notch rules | Restrict narrow indentations in layout geometry |
| Antenna rules | Address manufacturing-related antenna effects during processing |

The precise values depend on the selected PDK and the relevant process layer.

### 3.3 Why DRC Matters

DRC is essential because physical geometry must satisfy manufacturing constraints, not just logical connectivity.

During detailed routing, the router must create connections while respecting the applicable design rules. Verification and cleanup stages help identify and resolve violations.

**Goal:** Obtain a connected layout that satisfies the required design rules.

---

## 4. Global Routing, Detailed Routing, and TritonRoute

### 4.1 Global Routing

Global routing determines the approximate path that each net should follow through the design.

The routing region is divided into global cells, often called GCells. The global router assigns a sequence of these regions to each net.

Its primary objectives include:

- Estimating wirelength.
- Managing routing congestion.
- Allocating routing resources.
- Producing route guides for detailed routing.

Global routing does not generate the final physical wire geometry.

### 4.2 Detailed Routing

Detailed routing converts the global-routing plan into actual metal wires and vias.

The detailed router must connect the required terminals and obey the physical constraints of the technology.

Its responsibilities include:

- Selecting legal routing tracks.
- Connecting pins and terminals.
- Choosing vias between metal layers.
- Respecting spacing and width requirements.
- Resolving local congestion and routing conflicts.
- Reducing design-rule violations.

### 4.3 TritonRoute

TritonRoute is the detailed-routing engine used in the OpenROAD flow.

It uses the global router's guides as input and generates the physical routing geometry needed to connect the design.

Important configuration considerations include:

- The available routing-layer range.
- Metal-layer preferred directions.
- Routing-track patterns.
- Legal via definitions.
- Congestion-driven rip-up-and-reroute.
- Design-rule checking and cleanup.

The exact configuration depends on the technology setup and the flow being executed.

### 4.4 Comparison

| Global Routing | Detailed Routing |
|---|---|
| Creates approximate routing paths | Creates physical wire and via geometry |
| Works with GCells and route guides | Works with detailed routing tracks and geometry |
| Focuses on congestion and estimated wirelength | Focuses on connectivity and design-rule compliance |
| Produces guides for the next stage | Produces the detailed interconnect solution |

---

## 5. TritonRoute Feature 1: Route Guide Handling

### 5.1 Guide-Aware Detailed Routing

TritonRoute attempts to honor the preprocessed route guides generated by the global router.

The guides indicate the intended regions through which a net should travel. They act as routing constraints while allowing the detailed router to handle local connectivity and geometry requirements.

### 5.2 Guide Preprocessing

The supplied module describes the following stages:

1. Initial route guides.
2. Guide splitting.
3. Guide merging.
4. Guide bridging.
5. Final preprocessed route guides.

Preprocessing organizes the guides before the detailed-routing stage.

### 5.3 Properties of Preprocessed Guides

According to the supplied notes, the preprocessed guides should:

- Have unit width.
- Follow the preferred direction of the corresponding metal layer.

For example, a layer may prefer vertical routing while another layer prefers horizontal routing. The actual preferred directions depend on the technology definition.

### 5.4 Why Guide Handling Is Important

Following the guides helps detailed routing remain consistent with the global-routing plan. The detailed router must still establish legal connections and produce geometry that satisfies the applicable physical constraints.

---

## 6. TritonRoute Features 2 and 3: Connectivity and Panel Routing

### 6.1 Inter-Guide Connectivity

Route guides for a net must provide a connected path between the required terminals.

The supplied notes describe two ways that guides can be considered connected:

- They are on the same metal layer and their edges touch.
- They are on neighboring metal layers and have a nonzero area of vertical overlap.

In addition, every unconnected terminal, such as a standard-cell pin, must have its pin shape overlapped by an appropriate route guide.

These requirements help the detailed router establish a connected routing solution from the guide information.

### 6.2 Intra-Layer Parallel Routing

TritonRoute uses a panel-based routing approach.

Intra-layer parallel routing means that panels on the same metal layer can be processed in parallel, subject to the router's constraints and implementation.

Parallel processing can improve routing throughput by allowing independent routing work to proceed simultaneously.

### 6.3 Inter-Layer Sequential Routing

Inter-layer sequential routing means that layers are processed in sequence rather than all being routed simultaneously.

The supplied notes describe alternating between even-indexed and odd-indexed panels during the process.

This strategy balances parallel processing within a layer with the ordering requirements associated with routing across multiple layers.

### 6.4 Summary of the Two Features

| Feature | Main Idea |
|---|---|
| Inter-guide connectivity | Ensures that route guides form valid connections between terminals |
| Intra-layer parallel routing | Processes eligible panels on the same layer in parallel |
| Inter-layer sequential routing | Processes routing layers in an ordered sequence |

---

## 7. Connectivity Management in TritonRoute

### 7.1 Routing Problem Definition

The connectivity problem described in the supplied notes can be summarized as follows.

**Inputs**

- LEF: physical cell and routing-layer information.
- DEF: design placement and physical-design information.
- Preprocessed route guides: regions assigned to the nets by global routing.

**Output**

A detailed-routing solution that connects the required terminals while optimizing routing cost, including wirelength and via count.

**Constraints**

- Respect route guides.
- Maintain required connectivity.
- Follow applicable physical design rules.
- Use legal routing tracks and vias.

### 7.2 Access Points

An Access Point (AP) is an on-grid routing location on the metal layer associated with a route guide.

Access points provide possible locations for establishing connections between:

- A route guide and a lower-layer segment.
- A route guide and a pin shape.
- A route guide and an upper-layer connection.
- A route guide and an I/O port.

The available access points depend on the geometry and routing constraints.

### 7.3 Access Point Clusters

An Access Point Cluster (APC) groups access points derived from the same relevant routing object.

The supplied notes identify access points associated with:

- Lower-layer segments.
- Upper-layer guides.
- Pin shapes.
- I/O ports.

Grouping related access points makes it easier to reason about how a net can connect its terminals through multiple routing layers.

### 7.4 Role in Detailed Routing

Access points identify possible locations where a connection can enter, leave, or change layers.

The router uses these candidates to construct connectivity between terminals while respecting the available geometry and routing constraints.

---

## 8. Routing Topology Optimization

### 8.1 Purpose of Routing Topology

After the access point clusters have been identified, the router must decide how the clusters should be connected.

A routing topology specifies the connection structure that links the relevant access point clusters before detailed wire geometry is generated.

The supplied module describes an approach based on a Minimum Spanning Tree (MST).

### 8.2 Algorithm: Optimize the Connection Topology

For a net containing access point clusters, the algorithm is described as follows.

**Step 1: Calculate pairwise costs**

For every pair of clusters \(i\) and \(j\), calculate their connection cost:

```text
cost[i,j] = dist(APC_i, APC_j)
```

**Step 2: Construct the Minimum Spanning Tree**

Use the pairwise costs to construct an MST:

```text
T = MST(APCs, COSTs)
```

**Step 3: Extract the selected connections**

Return the edges that belong to the tree:

```text
Return edges e[i,j] belonging to T
```

These edges form the selected routing topology.

### 8.3 Why Use an MST?

An MST connects all vertices in a graph while minimizing the sum of the selected edge weights, assuming the graph is connected and the edge weights represent the chosen costs.

In this context, the clusters represent routing connection groups, and the edge costs estimate the cost of connecting them.

The resulting topology provides a connection structure for the detailed-routing engine to realize using legal wires and vias.

**Note:** The MST description above follows the algorithm supplied in the original notes. It should not be interpreted as a complete description of every internal optimization performed by TritonRoute.

---

# 9. Practical Lab: PDN Construction and Routing

This section covers the practical work associated with power distribution and routing.

## Lab 1: Construct the Power Distribution Network

### Objective

Build the Power Distribution Network (PDN) that distributes the power and ground connections through the physical design.

The PDN provides the infrastructure needed to deliver power from the supply connections to the standard cells.

The supplied lab outline identifies the following activities:

- Configure the power-distribution structure.
- Create the required power connections.
- Inspect the generated PDN geometry.

---

## Lab 2: Connect Power Straps to Standard-Cell Rails

### Objective

Understand how the larger power-distribution structures connect to the local power rails that supply standard cells.

The lab covers the transition from the power straps toward the standard-cell power rails.

### Main Areas of Study

- Power straps.
- Standard-cell power rails.
- Power and ground connectivity.
- Physical inspection of the generated power network.


---

## Lab 3: Routing

### Objective

Perform the routing stage and inspect the generated interconnect geometry.

The routing stage converts the available routing plan into physical connections while attempting to meet the design's connectivity and manufacturing constraints.



---

## 10. Conclusion

Day 10 introduces the routing stage of the RTL-to-GDS physical-design flow, covering maze routing, manufacturing design rules, global routing, and detailed routing.

Lee's algorithm explains the basic shortest-path search used in maze routing. DRC establishes the geometric constraints that the physical layout must satisfy. TritonRoute builds on the global router's guides and addresses guide connectivity, access-point selection, panel routing, and topology construction.

The routing-topology section describes how a Minimum Spanning Tree can be used to select a low-cost connection structure between access point clusters before detailed geometry is generated.

The practical portion of the module focuses on building the Power Distribution Network, connecting power straps to standard-cell rails, and performing routing.

**Current progress:** The PDN construction and routing lab results are still pending. This README will be updated with the actual commands, screenshots, and results as the remaining Module 4 work is completed.

---

## Tools and Concepts Covered

- **OpenROAD / TritonRoute:** Detailed routing.
- **LEF:** Technology and physical-cell abstracts.
- **DEF:** Physical-design representation.
- **DRC:** Manufacturing geometry verification.
- **Lee's algorithm:** Grid-based maze routing.
- **MST:** Routing topology construction.
- **PDN:** Power Distribution Network construction.

## Repository Image Organization

Store your own screenshots in an `images` folder and ensure that their filenames match the Markdown references above.

```text
repository/
├── README.md
└── images/
    ├── pdn-construction.png
    ├── power-straps-standard-cell-rails.png
    └── detailed-routing-results.png
```


