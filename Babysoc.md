# VSDBabySoC Implementation
### RTL Verification | Synthesis | Static Timing Analysis | Physical Design

## Overview

This project implements the VSDBabySoC design using the SKY130 HD technology platform. It covers the complete digital implementation workflow, starting from RTL simulation and continuing through synthesis, gate-level verification, timing analysis, and physical design using OpenROAD.

The design integrates the `rvmyth` RISC-V processor with the `avsdpll` PLL and `avsddac` DAC.

**Design details**
- Top module: `vsdbabysoc`
- Technology: SKY130 HD
- Processor: `rvmyth`
- Clock block: `avsdpll`
- Data converter: `avsddac`
- Simulation tools: Icarus Verilog and GTKWave
- Synthesis tool: Yosys
- Timing analysis: OpenSTA
- Physical design: OpenROAD-flow-scripts

---

## Workflow

| Step | Activity | Tool |
|---|---|---|
| 1 | RTL simulation | Icarus Verilog |
| 2 | Waveform inspection | GTKWave |
| 3 | Logic synthesis | Yosys |
| 4 | Gate-level simulation | Icarus Verilog |
| 5 | Static timing analysis | OpenSTA |
| 6 | Multi-corner timing evaluation | OpenSTA |
| 7 | Floorplanning and placement | OpenROAD |
| 8 | Clock-tree synthesis | OpenROAD |
| 9 | Global and detailed routing | OpenROAD |
| 10 | Post-route timing analysis | OpenROAD / OpenSTA |

---

## 1. RTL Simulation

RTL simulation is used to observe the design's functional behavior before technology mapping. Icarus Verilog compiles the testbench and design sources, and GTKWave displays the generated waveform.

```bash
mkdir -p baby_soc
cd baby_soc

git clone https://github.com/Subhasis-Sahu/BabySoC_Simulation
cd BabySoC_Simulation
```

Compile and run the simulation:

```bash
iverilog -o pre_synth_sim.out \
  -DPRE_SYNTH_SIM \
  src/module/testbench.v \
  -I src/include/ \
  -I src/module/

./pre_synth_sim.out
gtkwave pre_synth_sim.vcd
```

The following signals can be inspected in GTKWave:

- `CLK`
- `reset`
- `OUT`
- `RV_TO_DAC[9:0]`

### RTL waveform

<img width="1600" height="598" alt="image" src="https://github.com/user-attachments/assets/e5bb51d1-6608-4459-86ce-2d47d791f458" />


---

## 2. Logic Synthesis Using Yosys

Yosys converts the RTL description into a gate-level representation and maps the design to standard cells available in the SKY130 HD library.

The synthesis process includes reading Verilog sources, loading library models, synthesizing the top module, mapping sequential and combinational logic, and exporting the resulting netlist.

```tcl
read_verilog src/module/vsdbabysoc.v
read_verilog -I src/include/ src/module/rvmyth.v
read_verilog -I src/include/ src/module/clk_gate.v

read_liberty -lib src/lib/avsdpll.lib
read_liberty -lib src/lib/avsddac.lib
read_liberty -lib src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

synth -top vsdbabysoc
dfflibmap -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
opt
abc -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
setundef -zero
clean -purge
rename -enumerate
stat
write_verilog -noattr baby_soc_net.v
```

### Synthesis output

The `stat` command reports the mapped cell statistics. The generated `baby_soc_net.v` file is used for subsequent gate-level simulation.
<img width="1600" height="402" alt="image" src="https://github.com/user-attachments/assets/da736a54-c269-45d0-a3d0-27a86e498e46" />

### Netlist visualization

<img width="1567" height="817" alt="image" src="https://github.com/user-attachments/assets/f94d08f1-819b-4150-90df-bfeda9180cdf" />

---

## 3. Gate-Level Simulation

Gate-level simulation checks the behavior of the synthesized design using the mapped netlist and standard-cell functional models.

```bash
iverilog -DPOST_SYNTH_SIM -DFUNCTIONAL \
  -I src/include/ \
  -I ../../sky130RTLDesignAndSynthesisWorkshop/my_lib/verilog_model/ \
  -I src/module/ \
  src/module/testbench.v

./a.out
gtkwave post_synth_sim.vcd
```

The simulated outputs can be compared with the RTL waveform to examine whether the observed signal behavior remains consistent after synthesis.

### Gate-level waveform

<img width="1191" height="606" alt="image" src="https://github.com/user-attachments/assets/d97026ca-1034-47bd-9348-d0c21551d617" />


### RTL and gate-level comparison

<img width="1191" height="587" alt="image" src="https://github.com/user-attachments/assets/aa005a25-5f1c-4c7e-9569-c2b41c2ab302" />
---

## 4. Static Timing Analysis with OpenSTA

Static timing analysis evaluates the timing behavior of a digital circuit using its netlist, timing libraries, and constraints.

The initial OpenSTA exercise uses a small example circuit to demonstrate library loading, design linking, clock creation, input-delay constraints, and timing-report generation.

### Launch OpenSTA

```bash
cd OpenSTA

docker build \
  --file Dockerfile.ubuntu22.04 \
  --tag opensta .

docker run -i -v "$HOME:/data" opensta
```

An alternative environment used in the recorded run was:

```bash
docker run -it \
  -v /home/vsduser/Desktop/OpenSTA:/data \
  efabless/openlane:v0.21 sta
```

### Example timing analysis

```tcl
read_liberty /data/examples/nangate45_slow.lib
read_verilog /data/examples/example1.v
link_design top

create_clock -name clk -period 10 {clk1 clk2 clk3}
set_input_delay -clock clk 0 {in1 in2}

report_checks
```

To inspect both minimum- and maximum-delay paths:

```tcl
report_checks -path_delay min_max
```

The recorded example report showed:
- Data arrival time: `0.42 ns`
- Setup slack: `9.43 ns`

These values belong to the example circuit, not the VSDBabySoC design.


<img width="912" height="417" alt="image" src="https://github.com/user-attachments/assets/d7958f26-dc68-4d00-b8e1-5e8bc2d6011c" />

---

## 5. Pre-Route Timing Analysis

The pre-route timing analysis evaluates the synthesized VSDBabySoC netlist using the SKY130 HD timing library, PLL and DAC libraries, and synthesis constraints.

Example Tcl commands:

```tcl
set base_path /data/Desktop/OpenSTA/vsdbabysoc/
set lib_name sky130_fd_sc_hd__tt_025C_1v80.lib

read_liberty -min $base_path/lib/$lib_name
read_liberty -max $base_path/lib/$lib_name

read_liberty -min $base_path/lib/avsdpll.lib
read_liberty -max $base_path/lib/avsdpll.lib

read_liberty -min $base_path/lib/avsddac.lib
read_liberty -max $base_path/lib/avsddac.lib

read_verilog /data/Desktop/OpenSTA/vsdbabysoc/vsdbabysoc.synth.v
link_design vsdbabysoc

read_sdc /data/Desktop/OpenSTA/vsdbabysoc/sdc/vsdbabysoc_synthesis.sdc

report_checks
```

The file paths should correspond to the actual locations in the working environment.


<img width="662" height="357" alt="image" src="https://github.com/user-attachments/assets/26382002-c48a-4398-ad51-1ce9ad91c06a" />

<img width="767" height="535" alt="image" src="https://github.com/user-attachments/assets/5b7a6d63-81b8-49c5-bde4-f2d6369ad5b1" />



---

## 6. Multi-Corner Timing Analysis

Timing analysis was performed across **16 process-voltage-temperature (PVT) corners** using fast (`ff`), typical (`tt`), and slow (`ss`) process models.

The following table preserves the recorded timing values.

| PVT Corner | Setup WS (ns) | Setup WNS (ns) | Setup TNS (ns) | Hold WS (ns) |
|---|---:|---:|---:|---:|
| ff_100C_1v65 | 8.196 | 0 | 0 | 0.259 |
| ff_100C_1v95 | 9.059 | 0 | 0 | 0.205 |
| ff_n40C_1v56 | 7.052 | 0 | 0 | 0.303 |
| ff_n40C_1v65 | 7.758 | 0 | 0 | 0.267 |
| ff_n40C_1v76 | 8.384 | 0 | 0 | 0.235 |
| ff_n40C_1v95 | 9.072 | 0 | 0 | 0.198 |
| ss_100C_1v40 | -3.310 | -3.310 | -188.345 | 0.922 |
| ss_100C_1v60 | 1.754 | 0 | 0 | 0.653 |
| ss_n40C_1v28 | -28.657 | -28.657 | -13598.716 | 1.836 |
| ss_n40C_1v35 | -15.829 | -15.829 | -5921.846 | 1.361 |
| ss_n40C_1v40 | -10.325 | -10.325 | -2880.216 | 1.140 |
| ss_n40C_1v44 | -7.159 | -7.159 | -1573.228 | 1.000 |
| ss_n40C_1v60 | 0.197 | 0 | 0 | 0.673 |
| ss_n40C_1v76 | 3.618 | 0 | 0 | 0.502 |
| tt_025C_1v80 | 7.031 | 0 | 0 | 0.321 |
| tt_100C_1v80 | 7.108 | 0 | 0 | 0.326 |

### Timing observations

- Hold timing passed at all 16 recorded corners.
- Setup timing passed at the listed fast and typical corners.
- Three slow-process corners passed setup timing.
- Setup violations remained at five slow-process corners.
- The lowest recorded setup WNS was `-28.657 ns` at `ss_n40C_1v28`.
- The recorded critical paths included processor register-write logic, with half-adder and mux-related logic.

A successful analysis run does not necessarily mean that all timing constraints passed.


<img width="1336" height="630" alt="image" src="https://github.com/user-attachments/assets/8aebe26a-d57a-4db8-bd5a-2c8b52b23a64" />


<img width="1087" height="537" alt="image" src="https://github.com/user-attachments/assets/e508688b-7d95-40fc-81f7-db95d54cb2bd" />

<img width="1017" height="532" alt="image" src="https://github.com/user-attachments/assets/f6836ff3-d122-4f6c-b955-928101675a81" />

<img width="967" height="540" alt="image" src="https://github.com/user-attachments/assets/23bb463a-7d9a-4059-9858-7898e3be7561" />

<img width="847" height="515" alt="image" src="https://github.com/user-attachments/assets/5ddb314e-460b-4bcc-bef0-fab48b35c40b" />



<img width="717" height="445" alt="image" src="https://github.com/user-attachments/assets/b1ba607d-1656-47f2-9048-1ebadc87ceea" />


<img width="1222" height="675" alt="image" src="https://github.com/user-attachments/assets/140e3a0f-2164-4df9-ab6c-3b6c65edd21a" />

<img width="1282" height="677" alt="image" src="https://github.com/user-attachments/assets/2e103c02-9910-4a03-95f9-7f2c11b7d9e7" />

<img width="1172" height="695" alt="image" src="https://github.com/user-attachments/assets/fa6a5d84-9496-49df-b2c3-b3bec7b54248" />

<img width="1237" height="691" alt="image" src="https://github.com/user-attachments/assets/f95eb43a-0c75-4f3c-95ec-94d2a15ae669" />


---

## 7. OpenROAD Physical Design

The physical implementation uses OpenROAD-flow-scripts with the `sky130hd` platform.

The design configuration is organized under:

```text
flow/
└── designs/sky130hd/vsdbabysoc/
    ├── config.mk
    ├── config.tcl
    ├── vsdbabysoc_synthesis.sdc
    ├── lef/
    └── lib/
```

The configuration includes the technology libraries and macro information required for the PLL and DAC.

### Implementation stages

| Stage | Database | Description |
|---|---|---|
| Floorplanning | `2_floorplan.odb` | Core geometry and initial physical setup |
| Placement | `3_place.odb` | Standard-cell placement |
| Clock-tree synthesis | `4_cts.odb` | Clock distribution network |
| Global routing | `5_1_grt.odb` | Routing estimation and congestion analysis |
| Detailed routing | `5_route.odb` | Detailed interconnect routing |
| Finalization | `6_final.odb` | Final physical database |
| Parasitic extraction | `6_final.spef` | Extracted interconnect parasitics |

### Run the physical-design flow

From the OpenROAD-flow-scripts `flow` directory:

```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk
```

Individual stages can also be run with:

```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk floorplan

make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk place

make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk cts

make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk route

make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk final
```

To open the final database in the GUI:

```bash
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_final
```

### Clock-tree configuration

The recorded CTS configuration included:

```make
export CTS_BUF_DISTANCE = 600
export SKIP_GATE_CLONING = 1
```


<img width="965" height="497" alt="image" src="https://github.com/user-attachments/assets/b4c756cf-8fa5-49c4-9e81-033ae53d6e41" />

---

## 8. Global-Routing Congestion Analysis

During the initial global-routing attempt, OpenROAD reported `GRT-0116` due to residual congestion. The reported overflow was small and localized to `met5`.

| Layer | Usage | Max Horizontal Overflow | Max Vertical Overflow | Total Overflow |
|---|---:|---:|---:|---:|
| met1 | 1.67% | 0 | 0 | 0 |
| met2 | 2.01% | 0 | 0 | 0 |
| met3 | 0.53% | 0 | 0 | 0 |
| met4 | 0.98% | 0 | 0 | 0 |
| met5 | 0.02% | 1 | 0 | 2 |
| **Total** | **1.40%** | **1** | **0** | **2** |

### Routing configuration changes

The following settings were recorded in `config.mk`:

```make
ROUTING_LAYER_ADJUSTMENT = 0.3
export GLOBAL_ROUTE_ARGS = -allow_congestion
```

A lower core-utilization value was also considered:

```make
# export CORE_UTILIZATION = 0.1
```

The DAC LEF geometry was edited during congestion troubleshooting. The exact geometry change should be documented alongside the corresponding screenshot.

**Note:** Allowing congestion does not by itself prove that the final design is congestion-free or DRC-clean.


<img width="922" height="526" alt="image" src="https://github.com/user-attachments/assets/fe0f2be8-ac9b-4763-98ba-89c11fb9b021" />


<img width="1032" height="485" alt="image" src="https://github.com/user-attachments/assets/ec13e0b2-cdf0-4609-b532-5c76db20ab2b" />



<img width="1087" height="235" alt="image" src="https://github.com/user-attachments/assets/362f4970-9ba3-420b-b7f5-6b80cf040205" />

---

## 9. Detailed Routing Results

After the recorded configuration adjustments, global and detailed routing completed.

The reported routing statistics were:

| Metric | Recorded Value |
|---|---:|
| Routed nets | 6,379 |
| Total wirelength | 334,256 µm |
| Final vias | 48,218 |
| Final 3D usage | 177,398 |

The final database can be inspected using the OpenROAD GUI to examine the routed signals, metal layers, power structures, and macro connections.


<img width="941" height="492" alt="image" src="https://github.com/user-attachments/assets/eca51a47-b733-406c-9afe-dc72c849e4e4" />


---

## 10. Post-Route Timing Analysis

Post-route timing analysis uses the final physical database and extracted parasitics.

| Input | Purpose |
|---|---|
| `6_final.odb` | Final physical database |
| `6_final.spef` | Extracted parasitics |
| `vsdbabysoc_synthesis.sdc` | Timing constraints |
| SKY130 Liberty files | Cell timing models |

The recorded analysis script was located at:

```text
designs/sky130hd/post_route/run.tcl
```

Run the script from the OpenROAD-flow-scripts `flow` directory:

```bash
cd ~/Desktop/OpenROAD-flow-scripts/flow

openroad -no_splash -exit designs/sky130hd/post_route/run.tcl
```

The recorded run discovered 16 timing libraries, completed all 16 analyses, and reported no tool-run failures.

### Post-route timing results

| PVT Corner | Setup WS (ns) | Setup WNS (ns) | Setup TNS (ns) | Hold WS (ns) |
|---|---:|---:|---:|---:|
| ff_100C_1v65 | 8.196 | 0 | 0 | 0.259 |
| ff_100C_1v95 | 9.059 | 0 | 0 | 0.205 |
| ff_n40C_1v56 | 7.052 | 0 | 0 | 0.303 |
| ff_n40C_1v65 | 7.758 | 0 | 0 | 0.267 |
| ff_n40C_1v76 | 8.384 | 0 | 0 | 0.235 |
| ff_n40C_1v95 | 9.072 | 0 | 0 | 0.198 |
| ss_100C_1v40 | -3.310 | -3.310 | -188.345 | 0.922 |
| ss_100C_1v60 | 1.754 | 0 | 0 | 0.653 |
| ss_n40C_1v28 | -28.657 | -28.657 | -13598.716 | 1.836 |
| ss_n40C_1v35 | -15.829 | -15.829 | -5921.846 | 1.361 |
| ss_n40C_1v40 | -10.325 | -10.325 | -2880.216 | 1.140 |
| ss_n40C_1v44 | -7.159 | -7.159 | -1573.228 | 1.000 |
| ss_n40C_1v60 | 0.197 | 0 | 0 | 0.673 |
| ss_n40C_1v76 | 3.618 | 0 | 0 | 0.502 |
| tt_025C_1v80 | 7.031 | 0 | 0 | 0.321 |
| tt_100C_1v80 | 7.108 | 0 | 0 | 0.326 |

### Analysis summary

- All 16 timing analyses completed in the recorded run.
- Hold timing passed at all listed corners.
- Setup violations remained at five slow-process corners.
- The most negative setup WNS was `-28.657 ns` at `ss_n40C_1v28`.
- The critical paths were reported through processor register-write logic.

Successful report generation indicates that the timing analysis completed; it does not mean every setup constraint passed.


<img width="1082" height="382" alt="image" src="https://github.com/user-attachments/assets/c10539d9-a994-4c7c-b7f7-26248caac335" />



---

## 11. Project Findings

- RTL simulation and gate-level simulation showed matching behavior for the observed signals during the recorded simulation window.
- Yosys generated a SKY130-mapped netlist for subsequent simulation and timing analysis.
- The 16-corner timing evaluation showed that setup timing was most challenging under certain slow-process, low-voltage conditions.
- The OpenROAD flow reached detailed routing and generated a final database with extracted parasitics.
- Global routing initially reported a small overflow on `met5`; routing configuration adjustments were recorded.
- Post-route STA continued to show setup violations at five slow corners, identifying timing optimization as a possible next step.

---

## 12. Screenshots Directory

Create an `images` directory in your GitHub repository and add your own screenshots. The filenames below are examples; change the paths in this README if your image names differ.

```text
images/
├── rtl-simulation-waveform.png
├── yosys-synthesis-summary.png
├── netlist-hierarchy.png
├── netlist-mapped.png
├── gate-level-waveform.png
├── rtl-gls-comparison.png
├── opensta-example-report.png
├── preroute-sta-report.png
├── pvt-corner-comparison.png
├── setup-path-report.png
├── hold-path-report.png
├── cts-layout.png
├── global-route-congestion.png
├── dac-lef-adjustment.png
├── routing-config.png
├── final-routed-layout.png
├── postroute-pvt-summary.png
└── postroute-critical-path.png
```

---

## Tools Used

- **Icarus Verilog** — RTL and gate-level simulation
- **GTKWave** — waveform visualization
- **Yosys** — logic synthesis and technology mapping
- **OpenSTA** — static timing analysis
- **OpenROAD-flow-scripts** — physical implementation
- **SKY130 HD** — standard-cell technology platform

---

