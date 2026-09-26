# Day 8: Device-Level Characterization — SPICE Modeling of a CMOS Inverter and the 16-Mask Fab Flow

## Overview

Day 8 is the entry point into Module 3 of the Sky130 VSD program. Having spent Day 7 on floorplanning and placement at the digital level, the focus now drops all the way down to individual transistors and how they're actually manufactured. The day splits into two theory tracks plus a connected lab sequence:

- **Track 1** builds up SPICE-based characterization of a CMOS inverter from first principles — how a SPICE deck is put together, and how it's used to pull out both the inverter's **static** behavior (the switching threshold, V_M) and its **dynamic** behavior (rise and fall delay).
- **Track 2** walks the **16-mask CMOS fabrication sequence** end to end, tracing how a blank silicon wafer becomes a functioning CMOS inverter one masking step at a time.
- The **labs** turn both tracks into hands-on work: tweaking an IO placer parameter in the floorplan, opening the actual Sky130 `sky130_inv` standard cell in Magic, pulling a SPICE netlist straight out of that layout, patching the netlist so it points at real Sky130 device models, running ngspice transient sims to pull out rise/fall delay and transition-time numbers, and finally poking at the DRC rule deck for the Metal3 and Poly layers.

---

## 1. Anatomy of a SPICE Deck

Any SPICE netlist boils down to three ingredients:

1. **Connectivity** — how each component's terminals wire together via shared nodes.
2. **Values** — resistance, capacitance, supply voltages, transistor W/L, and so on.
3. **Model references** — which device model a given instance is supposed to use (e.g., a specific PMOS or NMOS process model).

Take a textbook CMOS inverter built from a matched PMOS/NMOS pair (W = 0.375 µm, L = 0.25 µm in the example used to introduce this). The netlist declares both transistors, a load cap on the output node, the supply and input sources, plus whatever simulation directive is needed — `.op` for an operating-point solve, `.dc` for a sweep, `.tran` for transient analysis.

Now compare that to a netlist **pulled straight out of a Magic layout**, which is what the labs actually do. The same three ingredients are still there, but the values come from the drawn geometry itself (width, length, diffusion area/perimeter) instead of being typed in by hand, and the model names default to whatever Magic's internal naming scheme uses rather than a course-supplied model library — which is precisely the mismatch that the netlist-fixing lab later has to resolve.

---

## 2. Static Behavior: Where Does the Inverter Flip? (V_M)

Sweep the input of a CMOS inverter from 0 to V_DD and plot the output against it, and you get a characteristic S-curve. Along that sweep, the inverter passes through five distinct operating regions:

1. PMOS linear, NMOS cut off
2. PMOS linear, NMOS saturated
3. Both devices saturated
4. PMOS saturated, NMOS linear
5. PMOS cut off, NMOS linear

**V_M, the switching threshold**, is the point on that curve where V_in equals V_out. Because both transistors sit in saturation at that exact point, it's also where the PMOS and NMOS currents balance out (I_dsP = −I_dsN) — making V_M essentially a measure of how evenly matched the two halves of the inverter are.

V_M isn't a fixed number — it moves with relative device sizing. With a matched pair (W_n/L_n = W_p/L_p = 1.5), V_M lands close to the middle of the supply rail (~0.98 V at 2.5 V). Make the PMOS wider relative to the NMOS (say, W_p/L_p = 3.75 with W_n/L_n held at 1.5), and V_M shifts upward (~1.2 V) — the beefier PMOS is able to hold the output high across a wider input range before the NMOS overpowers it. This is exactly why PMOS devices are conventionally drawn wider than NMOS in a balanced design: holes (the PMOS carrier) move more slowly than electrons (the NMOS carrier), so the extra width is there to compensate for that mobility gap.

<img width="940" height="547" alt="image" src="https://github.com/user-attachments/assets/7747a056-d665-47a8-b1b9-d5883a9c0989" />


---

## 3. Dynamic Behavior: How Fast Does It Switch?

If V_M is the DC story, the transient response tells you how quickly the inverter reacts — measured by driving a pulse into the input and watching the output in a `.tran` simulation.

Two related-but-different timing metrics fall out of this:

- **Transition time (rise/fall time)** — how long an output edge takes to cross through its own "middle," usually measured between the 20% and 80% points of the full voltage swing.
- **Propagation delay (cell rise/fall delay)** — how long the output takes to respond to a change on the input, usually measured from the 50% crossing of the input edge to the 50% crossing of the matching output edge.

These aren't interchangeable: transition time is about how clean an edge looks to whatever stage receives it next (sluggish edges burn extra power and can cause downstream timing headaches), while propagation delay is the number that actually feeds directly into static timing analysis of a digital path.

---

## 4. Building a CMOS Inverter: The 16-Mask Process

Getting from a bare silicon wafer to a working CMOS inverter (or any CMOS logic, for that matter) means building up transistors and wiring one lithographic mask at a time. The full sequence breaks down into eight major stages spanning 16 masks total.

### 4.1 Choosing the Substrate

Everything starts with a **p-type silicon wafer** — chosen for its high resistivity (~10¹⁵ cm⁻³ doping) and `<100>` crystal orientation. That particular orientation wins out over the alternatives because it leaves fewer interface traps at the silicon–oxide boundary, which matters directly for transistor performance.

### 4.2 Defining the Active Region (Mask 1)

A stack of roughly 40 nm SiO₂, 80 nm Si₃N₄, and 1 µm of photoresist goes down on the wafer and gets patterned using Mask 1. The nitride/oxide combination acts as a barrier for the oxidation step that follows. Field oxide is then grown in the exposed areas via **LOCOS** (Local Oxidation of Silicon) — a process notorious for producing the "bird's beak" effect, where oxide tapers and encroaches laterally under the edge of the nitride mask. Once that's done, the nitride is stripped away with hot phosphoric acid.
<img width="922" height="531" alt="image" src="https://github.com/user-attachments/assets/492431c6-63d7-4250-b0b1-851d388577a4" />

### 4.3 Building the N-Well and P-Well (Mask 2)

Mask 2 opens up two separate implants side by side — boron for the p-well, phosphorus for the n-well — a twin-tub setup that lets both NMOS and PMOS live on the same substrate. A **high-temperature drive-in diffusion** in a furnace follows, pushing those implanted dopants deeper into the substrate and activating them to finish off the well regions.

<img width="906" height="552" alt="image" src="https://github.com/user-attachments/assets/1aeed497-7089-4b71-b2e1-7088901dbfd6" />

<img width="941" height="512" alt="image" src="https://github.com/user-attachments/assets/dc7f349c-9362-48e7-ab7a-11fb743fc4db" />


> **Note:** the exact drive-in temperature couldn't be cross-checked against a slide in this batch of screenshots — flagged as an open item below.

### 4.4 Forming the Gate (Mask 6)

Threshold-voltage tuning happens at this stage, governed by substrate doping (N_A) and gate oxide capacitance (C_ox). A sacrificial oxide layer is stripped off with dilute HF, and the gate region is defined through two doping steps — Mask 4 (boron) and Mask 5 (arsenic) — ahead of the actual gate patterning. Polysilicon goes down next and is doped n-type to keep resistance low, and Mask 6 is what finally defines and etches the gate shape out of that poly layer.

<img width="957" height="542" alt="image" src="https://github.com/user-attachments/assets/ae1650cd-ca2d-457e-b225-3ea047ffb3c0" />

<img width="1082" height="530" alt="image" src="https://github.com/user-attachments/assets/5a38c1d1-5e29-4feb-ad00-36547500d35e" />


> **Note:** the specific implant energies (mask 4 at roughly 60 kV boron, mask 5 arsenic) weren't independently verified against a slide in this batch — flagged below.

### 4.5 Adding the Lightly Doped Drain (Masks 7 & 8)

Ahead of the full-strength source/drain implant, a lighter dose gets placed right next to the gate edge — the LDD. Its job is to hold off two effects that get worse as devices shrink:

- **Hot-electron effect** — carriers accelerated by the strong field near the drain can pick up enough energy (crossing roughly a 3.2 eV barrier between silicon and the SiO₂ conduction band) to break Si–Si bonds, degrading the device over time.
- **Short-channel effect** — the drain's field reaching far enough into the channel to undermine the gate's control over it.

Mask 7 implants phosphorus and Mask 8 implants boron, forming the lightly doped N⁻/P⁻ regions. **Sidewall spacers** — roughly 0.1 µm of Si₃N₄/SiO₂, deposited and then plasma-etched anisotropically — go up alongside the gate. These spacers are what physically push back the heavier source/drain implant that comes next, creating the graded "lightly doped" profile at the drain edge.
<img width="961" height="542" alt="image" src="https://github.com/user-attachments/assets/b503bf14-4619-42bb-b0f0-90b7374e536c" />


### 4.6 Forming Source and Drain (Masks 9 & 10)

A thin screen oxide is grown first to stop channeling during implant. Two more implants — arsenic on the NMOS/p-well side, boron on the PMOS/n-well side — lay down the full-strength source/drain regions, followed by a high-temperature furnace anneal to activate the dopants and repair implant-induced damage.

<img width="997" height="531" alt="image" src="https://github.com/user-attachments/assets/f39e605d-2e3a-4e80-90a7-4b491d0eb71d" />

> **Note:** which mask number (9 vs. 10) corresponds to which implant species wasn't independently confirmed against a slide in this batch — flagged below, as originally suspected during dictation.

### 4.7 Local Interconnect and Contact Formation (Mask 11)

The thin oxide sitting over the source/drain/gate regions gets etched away in HF to reveal bare silicon, titanium is sputtered across the wafer, and the whole thing is annealed at **650–700 °C in an N₂ ambient for 60 seconds**. Wherever titanium touched bare silicon directly, this reaction produces **titanium nitride (TiN)**, used specifically for **local interconnect** — short-hop connections rather than full-chip routing. Mask 11 then defines exactly where those local contact plugs get etched and filled.

<img width="1002" height="591" alt="image" src="https://github.com/user-attachments/assets/81a509dc-7c34-47d6-b518-944833bba94a" />

### 4.8 Metal Stack-Up (Mask 12 through Mask 16)

From here, building each additional metal level is a repeating cycle of dielectric deposition, planarization, and metal patterning:

- Roughly 1 µm of PSG/BPSG dielectric goes down and gets planarized with CMP.
- A TiN barrier plus a blanket tungsten fill forms the contact plugs (Mask 12), followed by another CMP pass.
- Aluminum is deposited and plasma-etched into the metal1 pattern.
- Another SiO₂ layer is deposited and CMP-flattened, then Mask 14 opens new contact holes, again filled with TiN/tungsten.
- Mask 15 defines the next metal layer's pattern.
- A final Si₃N₄ passivation layer covers the whole stack, and **Mask 16 — the last mask in the entire flow** — opens the final contact holes through that passivation to expose the bond pads.

What comes out the other end — the full stack from substrate to top metal — is a fabrication-ready device, with every transistor's source, gate, and drain terminal now reachable from outside the chip.

<img width="965" height="582" alt="image" src="https://github.com/user-attachments/assets/985d5170-31cd-45ce-9261-4d8afc991568" />

<img width="1007" height="512" alt="image" src="https://github.com/user-attachments/assets/b6b2c80e-0d8d-48b1-bf3b-13a4c444e01f" />

<img width="1090" height="567" alt="image" src="https://github.com/user-attachments/assets/1bb0cd38-6dfc-45ae-b348-7f62a7401a63" />

---

## Labs

### Lab 1 — Tweaking the Floorplan's IO Placer Setting

Re-ran the floorplan stage with a different IO placer mode to compare pin placement outcomes.

<img width="1920" height="983" alt="io placer equi distance" src="https://github.com/user-attachments/assets/4a7ae227-5ad5-4bae-8821-903fcf729087" />
<img width="1920" height="983" alt="zoomed io placer equi distance" src="https://github.com/user-attachments/assets/2830dee4-8a77-4da5-bdb0-2d0dc4596937" />
<img width="1920" height="983" alt="io placer after mode set to 2" src="https://github.com/user-attachments/assets/9695ad32-b6e5-4119-9c88-1dd5ca9215f8" />
<img width="1920" height="983" alt="zoomed io placer after mode set to 2" src="https://github.com/user-attachments/assets/405ed811-699e-4f9f-bb3a-9daa36eea458" />

In its default (equidistant) mode, the IO placer spaces pins evenly along each edge of the die. Switching the mode parameter to `2` swaps in a different distribution strategy — the resulting pin clustering along the boundary looks noticeably different from the uniform baseline.

### Lab 2 — Getting Magic Set Up with the Sky130 Standard-Cell Repo

<img width="1920" height="983" alt="git clonned vsdstdcelldesign" src="https://github.com/user-attachments/assets/db652168-1e2e-464b-a044-94ae4585dcf8" />
<img width="1920" height="983" alt="copied sky130A tech to the directory clonned" src="https://github.com/user-attachments/assets/ca24c556-4172-4685-b192-f6d37c6c6c66" />
<img width="1920" height="983" alt="magic cmd" src="https://github.com/user-attachments/assets/673c71f0-e71d-4e68-a58f-b3e796b77f13" />
<img width="1920" height="983" alt="inverter layout" src="https://github.com/user-attachments/assets/ce662f30-6b04-4f39-a24c-7b8f46578e84" />

Cloned the `vsdstdcelldesign` repo (home to the reference `sky130_inv.mag` layout) into the OpenLane working directory, dropped a copy of the `sky130A.tech` technology file from the PDK's `libs.tech/magic` folder into that repo so Magic could find it locally, and launched Magic with:

```
magic -T sky130A.tech sky130_inv.mag &
```

This brought up the reference Sky130 inverter layout — VPWR/VGND rails, the **A** input pin, the **Y** output pin, and the PMOS/NMOS diffusion areas all visible in the layout window.

### Lab 3 — Telling the Transistors Apart

<img width="1920" height="983" alt="nmos" src="https://github.com/user-attachments/assets/b2d1f6ce-cf62-4498-9578-d6fdd7928c6d" />
<img width="1920" height="983" alt="pmos" src="https://github.com/user-attachments/assets/1da18003-1631-48df-83e6-467e6f751bcf" />

Clicked into each transistor region and ran `what` in Magic's `tkcon` console to check which mask layer sat under the cursor — confirming the left-hand device as `nmos` and the right-hand device as `pmos`.

### Lab 4 — Pulling a SPICE Netlist Out of the Layout

<img width="1920" height="983" alt="extraxct all cmd in tcon" src="https://github.com/user-attachments/assets/a13e8bb1-95c8-4c88-b90f-752aae26b395" />
<img width="1920" height="983" alt="exttospicecmd" src="https://github.com/user-attachments/assets/9b0e9537-3224-4d31-afc3-7088e776dfc9" />
<img width="1920" height="983" alt="created successfully" src="https://github.com/user-attachments/assets/56897573-d67f-413a-b404-fc2f65a53d76" />
<img width="1920" height="983" alt="sky130_inv spice file" src="https://github.com/user-attachments/assets/e51a20f6-d6d2-4c9b-9573-727daf11e123" />

With the reference inverter still loaded, ran:

```
extract all
ext2spice cthresh 0 zthresh 0
ext2spice
```

This generated `sky130_inv.ext` and then `sky130_inv.spice`. Opening that SPICE file showed the transistors referenced by Magic's raw internal model names (`sky130_fd_pr__nfet_01v8` / `sky130_fd_pr__pfet_01v8`) — not yet mapped to any local model library, and still missing the stimulus and analysis statements needed to actually run a simulation.

### Lab 5 — Tracking Down the Sky130 Device Models

<img width="1920" height="983" alt="lib files" src="https://github.com/user-attachments/assets/a8557a22-1863-4bec-8dee-74c971469eff" />
<img width="1920" height="983" alt="nshort_model file" src="https://github.com/user-attachments/assets/c51becb8-baca-4dc8-b443-e322123f5cda" />
<img width="1920" height="983" alt="pshort_model file" src="https://github.com/user-attachments/assets/005f68d0-989e-4661-a4f9-6c5d46bbd45f" />

Poked around the `libs` folder inside `vsdstdcelldesign`, which holds `pshort.lib` and `nshort.lib` — BSIM4 models for the short-channel PMOS and NMOS devices used in the standard-cell library — alongside the fast/typical/slow corner libraries for the full `sky130_fd_sc_hd` cell set. Opening these files confirmed the model names the netlist actually needed: **`pshort_model.0`** for PMOS and **`nshort_model.0`** for NMOS.

### Lab 6 — Patching the Netlist and Getting a First Simulation Running

<img width="1920" height="983" alt="ngspice run cmd" src="https://github.com/user-attachments/assets/6c881ec6-429d-4d53-b621-a372972a3194" />
<img width="1920" height="983" alt="modified spice deck file" src="https://github.com/user-attachments/assets/71186fc0-2da7-4fc6-8b96-f5d5acefde21" />

Running the raw extracted netlist in ngspice as-is failed immediately with:

```
Error: unknown subckt: x0 y a vgnd vgnd pshort_model.0 ...
```

since the deck wasn't actually pointing at the model libraries. Fixing this meant adding `.include ./libs/pshort.lib` and `.include ./libs/nshort.lib`, writing a proper `.subckt sky130_inv A Y VPWR VGND` block with M0/M1 correctly instantiating `pshort_model.0` and `nshort_model.0`, adding a `Va` PULSE source on the input along with the extracted parasitic capacitances, and appending a `.tran`/`.control` block. With those changes in place, the simulation ran clean — ngspice printed the initial transient solution with no errors.

### Lab 7 — Pulling Delay and Transition-Time Numbers from the Transient Sim

<img width="1920" height="983" alt="transcient analysis plot of cmps inverter (modified code)" src="https://github.com/user-attachments/assets/587558a9-e4dc-4152-a208-e33f39c9a55b" />
<img width="1917" height="987" alt="rise time transistion" src="https://github.com/user-attachments/assets/37a15742-305a-4bd8-89e2-1a381692c4b1" />
<img width="1917" height="981" alt="fall time transistion" src="https://github.com/user-attachments/assets/97e1ec5d-88f7-47c5-82e9-6ed0a2652d99" />
<img width="1912" height="982" alt="cell rise delay" src="https://github.com/user-attachments/assets/12064c53-e62a-4c83-96f4-507ee86bde61" />
<img width="1917" height="977" alt="cell fall delay" src="https://github.com/user-attachments/assets/6c18e544-6a78-42d6-820b-047769db1b27" />

Plotting `y` (output) against `time a` (input) in ngspice's transient viewer produced several clean inverter switching events. Using the plot's cursor tool to measure between the relevant threshold crossings gave:

| Metric | Method | Result |
|---|---|---|
| Rise transition time | 20%–80% on a rising edge | 0.06194 ns |
| Fall transition time | 20%–80% on a falling edge | 0.04267 ns |
| Cell rise delay | 50% input → 50% output, rising | 0.05953 ns |
| Cell fall delay | 50% input → 50% output, falling | 0.05941 ns |

These map directly onto the dynamic-behavior definitions from the theory section above — only now measured off a netlist extracted from an actual layout rather than a hand-typed textbook circuit.

### Lab 8 — Poking at the DRC Rule Deck (Metal3 and Poly)

<img width="1920" height="983" alt="met3" src="https://github.com/user-attachments/assets/f83f8c06-707d-4c40-b04d-28f64313001d" />

Loaded up a reference DRC test deck holding a set of labeled Metal3 (m3) structures — m3.1 through m3.7 — split between correctly designed cells (m3.4) and a deliberately broken one (m3.7), plus two larger multi-via structures (m3.3c, m3.3d).

<img width="1920" height="983" alt="drc why met2" src="https://github.com/user-attachments/assets/22ed60bf-4c47-4821-bb7d-90b772be695d" />
<img width="1920" height="983" alt="drc why met 3 2" src="https://github.com/user-attachments/assets/19cee2bb-670f-4174-9ddd-6691324c6712" />

Running `drc why` on the flagged regions returned the exact rule each one broke:

- `Metal3 spacing < 0.3um (met3.2)`
- `Metal3 minimum area < 0.24um^2 (met3.6)`

<img width="1920" height="983" alt="cif see VIA2 cmd full view" src="https://github.com/user-attachments/assets/3094af63-0b52-43de-b6fc-360b89278dde" />
<img width="1920" height="983" alt="cif see VIA2 cmd" src="https://github.com/user-attachments/assets/2a803c93-5686-4559-b2bb-1710e3dc4f14" />
<img width="1920" height="983" alt="measurement" src="https://github.com/user-attachments/assets/2afd3f72-2eeb-40d3-9e84-3d3d3682c10a" />

Zoomed into the m3.3c structure — a grid of via/contact cuts inside a met3 region flagged with DRC=22 total violations — and used `paint m3contact` followed by `cif see VIA2` to isolate the VIA2 mask layer on its own. Querying a selected via/contact region with `box` reported dimensions of 0.190 × 0.200 µm (0.038 µm² area).

<img width="1920" height="983" alt="rule violation" src="https://github.com/user-attachments/assets/d80dfd77-e13a-40bb-be33-398927d40db2" />

Switched the active editing layer to **poly** (DRC count dropped to DRC=10 on this layer) and queried a selected poly shape, which came back at 0.335 × 0.210 µm (0.070 µm² area) — labeled "poly.9" in the reference deck.

---

## Conclusion

Day 8 tied the transistor-level theory of Module 3 to two closely linked hands-on threads. On the fabrication side, stepping through all 16 masks made it clear how every structural feature of a finished CMOS inverter — wells, gate, LDD, source/drain, local interconnect, and the metal stack above it — traces back to a specific masking and processing step, and why techniques like LOCOS and self-aligned TiN interconnect exist to solve concrete process problems (controlling the bird's beak, separating local from global routing) rather than being arbitrary design choices.

On the characterization side, carrying a reference Magic layout all the way to a working SPICE netlist — extracting it, discovering it pointed at the wrong model names, fixing the deck against the real `pshort_model.0` / `nshort_model.0` BSIM4 models, and finally pulling clean rise/fall delay and transition-time numbers out of an actual transient simulation — turned the abstract V_M and delay definitions from the lecture into something concrete and measured. The DRC labs rounded things out by showing what happens when a layout falls short of the rules those same masks are built around, and how Magic's `drc why` command turns an abstract violation into a specific, actionable spacing or area number.
