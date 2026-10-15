
# Day 9: Pre-Layout Timing Analysis, Clock Trees, and Custom Cell Integration

**Topics:** Liberty Delay Tables | OpenSTA | Clock Tree Synthesis | Magic | SKY130 | OpenLane

## Project Overview

This README documents the timing-analysis concepts and practical physical-design exercises performed using the SKY130 PDK, Magic, OpenSTA, and OpenLane.

The work covers:

- Understanding Liberty cell-delay tables.
- Performing setup and hold timing analysis.
- Studying clock-tree synthesis and clock crosstalk.
- Inspecting routing tracks and alignment grids in Magic.
- Creating and labeling a custom inverter cell.
- Generating and integrating a LEF file.
- Running synthesis and placement in OpenLane.
- Examining pre-STA timing and defining timing constraints.

> **Note:** Labs 10 and 11 were marked as pending in the supplied notes. Their results have not been added or assumed.

## Table of Contents

1. [Understanding Liberty Delay Tables](#1-understanding-liberty-delay-tables)
2. [Setup Analysis with an Ideal Clock](#2-setup-analysis-with-an-ideal-clock)
3. [Clock Tree Synthesis and Signal Integrity](#3-clock-tree-synthesis-and-signal-integrity)
4. [Setup and Hold Analysis with a Real Clock](#4-setup-and-hold-analysis-with-a-real-clock)
5. [Practical Implementation Labs](#5-practical-implementation-labs)
6. [Summary](#6-summary)

---

## 1. Understanding Liberty Delay Tables

### 1.1 Purpose of Delay Tables

Liberty files (`.lib`) contain characterized timing information for standard cells. Static Timing Analysis (STA) tools use these tables to estimate cell delay and output transition time.

A delay table relates two input conditions:

- Input transition time, also called slew.
- Total output-net capacitance, which represents the load.

The corresponding table values describe the cell's propagation delay and output transition.

Using pre-characterized tables makes timing analysis practical for large digital designs without requiring a separate SPICE simulation for every cell instance.

### 1.2 Parameters Used in Delay Tables

| Parameter | Description |
|---|---|
| `input_net_transition` | Input signal transition time |
| `total_output_net_capacitance` | Total capacitance driven by the output |
| `cell_rise` | Cell propagation delay for a rising output |
| `cell_fall` | Cell propagation delay for a falling output |
| `rise_transition` | Output transition time for a rising signal |
| `fall_transition` | Output transition time for a falling signal |

When the actual input slew and output load fall between characterized table entries, the STA tool interpolates the values.

Separate timing models are provided for different PVT conditions:

- Process
- Voltage
- Temperature

These conditions affect cell delay and must be considered when checking whether a design meets its timing requirements.

---

## 2. Setup Analysis with an Ideal Clock

### 2.1 Setup Time

Setup time is the minimum duration for which data must remain stable before the active clock edge at a flip-flop's data input.

If this requirement is violated, the flip-flop may enter a metastable state.

The ideal-clock setup relationship can be represented as:

```text
Tclk >= Tck->q + Tlogic + Tsetup + Tskew
```

Where:

- `Tclk` — Clock period.
- `Tck->q` — Clock-to-Q delay of the launching flip-flop.
- `Tlogic` — Combinational logic delay.
- `Tsetup` — Setup time of the capturing flip-flop.
- `Tskew` — Clock skew between the launch and capture points.

### 2.2 Clock Jitter and Uncertainty

**Clock jitter** is the variation in the timing of clock edges. It may be caused by PLL/DLL noise and power-supply noise.

**Clock uncertainty** represents the timing margin used to account for effects such as:

- Clock jitter.
- Duty-cycle distortion.
- PLL phase error.

This margin prevents timing analysis from assuming that the clock is perfectly stable.

### 2.3 Understanding Slack

Slack indicates whether a timing path meets its required timing constraint.

```text
Slack = Required Time - Arrival Time
```

- Positive slack: the timing requirement is met.
- Zero slack: the path is exactly at the timing limit.
- Negative slack: the path violates the timing requirement.

---

## 3. Clock Tree Synthesis and Signal Integrity

### 3.1 Clock Tree Synthesis

Clock Tree Synthesis (CTS) constructs a clock distribution network between the clock source and sequential elements in the design.

The goal is to distribute the clock while controlling clock skew, insertion delay, and transition time.

An H-tree is one possible clock distribution structure. It divides the clock path into progressively smaller branches intended to provide similar path lengths and delays.

Buffers and inverters may be inserted to drive clock loads and control transition times.

Typical CTS objectives include:

- Reducing clock skew.
- Controlling clock insertion delay.
- Meeting maximum transition limits.
- Meeting clock-net capacitance limits.
- Distributing the clock to sequential elements.

### 3.2 Clock Crosstalk

Clock nets can be affected by coupling capacitance from neighboring signal wires.

Because clock signals are important to sequential timing, unwanted noise or changes in clock-edge timing can affect the design's timing behavior.

### 3.3 Clock Shielding

Shielding can reduce coupling between clock wires and nearby signal nets. Grounded shield wires or increased spacing may be used to reduce the effect of neighboring switching signals.

The effectiveness of shielding depends on the physical routing implementation.

---

## 4. Setup and Hold Analysis with a Real Clock

After CTS, the clock is distributed through a physical network. Launch and capture clock paths therefore have their own delays.

Timing analysis must account for the actual clock arrival times at the sequential elements.

### 4.1 Setup Check After CTS

The setup relationship is:

```text
Slack = Data Required Time - Data Arrival Time
```

For setup timing:

```text
Slack >= 0  -> Timing requirement met
Slack < 0   -> Setup violation
```

The data arrival time includes the launch-clock delay, clock-to-Q delay, and combinational logic delay.

The required time depends on the capture-clock arrival, clock period, setup requirement, and timing uncertainty.

**Useful skew** is an intentional adjustment of relative clock arrival times that can improve setup timing for selected paths. Its effect on hold timing must also be checked.

### 4.2 Hold Check After CTS

The hold relationship is:

```text
Slack = Data Arrival Time - Data Required Time
```

For hold timing:

```text
Slack >= 0  -> Timing requirement met
Slack < 0   -> Hold violation
```

Hold checks concern data stability around the relevant capture-clock edge and do not use a full clock period in the same way as setup checks.

A real clock network can introduce skew that improves setup timing but worsens hold timing on short data paths.

**Clock Reconvergence Pessimism Removal (CRPR)** accounts for common portions of launch and capture clock paths so their shared delay is not unnecessarily counted twice.

---

# 5. Practical Implementation Labs

## Lab 1: Inspect Routing Tracks and Configure the Magic Grid

Open the inverter layout using the SKY130A technology file:

```bash
magic -T sky130A.tech sky130_inv.mag &
```

Navigate to the OpenLane technology directory and inspect the routing-track information:

```bash
cd ~/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/openlane/sky130_fd_sc_hd
less tracks.info
```

### Sample Track Information

```text
li1 X 0.23 0.46
li1 Y 0.17 0.34
met1 X 0.17 0.34
met1 Y 0.17 0.34
...
met5 X 1.70 3.40
met5 Y 1.70 3.40
```

Configure the Magic grid from the tkcon console:

```text
grid 0.46um 0.34um 0.23um 0.46um
```

This aligns the layout grid with the specified routing-grid values.

### Screenshots

<img width="1082" height="532" alt="image" src="https://github.com/user-attachments/assets/135e40cc-429d-4853-b336-9486b5b85048" />



<img width="567" height="340" alt="image" src="https://github.com/user-attachments/assets/354174e5-4e87-416a-b823-7b32409d7c24" />



<img width="1055" height="547" alt="image" src="https://github.com/user-attachments/assets/5120060f-386c-44bb-91f6-3de0f0508413" />



<img width="1020" height="517" alt="image" src="https://github.com/user-attachments/assets/89058976-50e5-4c4c-972f-205d2c21301e" />


<img width="1077" height="527" alt="image" src="https://github.com/user-attachments/assets/4738047f-2853-42b7-aa8f-dfd93b18fdee" />

---

## Lab 2: Label the Custom `sky130_vsdinv` Cell

Open the inverter layout and inspect the input, output, and power connections.

Use **Edit → Text (label)** to create the required port labels.

| Label | Layer | Port Class | Port Use |
|---|---|---|---|
| `VPWR` | `metal1` | `inout` | `power` |
| `VGND` | `metal1` | `inout` | `ground` |
| `A` | `locali` | `input` | `signal` |
| `Y` | `locali` | `output` | `signal` |

Use the `what` command in tkcon to inspect the selected label.

Example:

```text
% what
Selected label(s):
    "VPWR" is attached to metal1 in cell def sky130_inv
% port class inout
% port use power
```

Repeat the appropriate port-class and port-use settings for the other labels.

Save the modified layout using a new cell name:

```text
save sky130_vsdinv.mag
```

### Screenshots


<img width="976" height="492" alt="image" src="https://github.com/user-attachments/assets/6a599c7a-d5c0-4253-a219-e0333f3d8cda" />



<img width="987" height="560" alt="image" src="https://github.com/user-attachments/assets/e44422f0-d9f4-4989-8453-a2a907b9155e" />


<img width="695" height="482" alt="image" src="https://github.com/user-attachments/assets/d0614ffb-bab3-48f9-96c0-78b03c722233" />

---

## Lab 3: Generate the LEF Abstract

With `sky130_vsdinv.mag` loaded, execute the following command in the Magic tkcon console:

```text
write lef
```

This generates the LEF abstract file:

```text
sky130_vsdinv.lef
```

The abstract contains the macro definition and physical information such as:

- Macro class and size.
- Pin definitions for `A` and `Y`.
- Power and ground pins `VPWR` and `VGND`.
- Pin geometry.
- Antenna-area information.

Check the files in the current directory:

```bash
ls -ltr
```

The supplied notes show:

```text
sky130_vsdinv.mag  (2716 bytes)
```

### Screenshots


<img width="625" height="512" alt="image" src="https://github.com/user-attachments/assets/41fade17-19fa-4bda-bf4f-f2be0dbb72c4" />



<img width="611" height="520" alt="image" src="https://github.com/user-attachments/assets/42446b15-7443-47eb-8b8a-5bb8097f9f45" />

<img width="990" height="487" alt="image" src="https://github.com/user-attachments/assets/e19f9021-65c2-42d2-932e-81cdf08943d8" />


---

## Lab 4: Integrate the Custom Cell into `picorv32a`

Copy the generated LEF file into the design's source directory:

```bash
cp sky130_vsdinv.lef /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src
```

Copy the fast, typical, and slow timing libraries into the same directory:

```bash
cd ~/Desktop/work/tools/openlane_working_dir/openlane/vsdstdcelldesign/libs

cp sky130_fd_sc_hd__* /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src
```

### Update `config.tcl`

Edit:

```text
designs/picorv32a/config.tcl
```

Add or configure the library paths and additional LEF setting:

```tcl
set ::env(LIB_SYNTH)   "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"
set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib"
set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib"
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"

set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
```

These settings allow OpenLane to access the custom LEF and the timing libraries required by the flow.




---

## Lab 5: Prepare and Synthesize the Design with OpenLane

Launch the OpenLane Docker container:

```bash
docker run -it \
  -v $PWD:/openLANE_flow \
  -v $PDK_ROOT:$PDK_ROOT \
  -e PDK_ROOT=$PDK_ROOT \
  -u $(id -u $USER):$(id -g $USER) \
  efabless/openlane:v0.21
```

Start the interactive flow:

```bash
bash-4.2$ ./flow.tcl -interactive
```

Prepare the design:

```text
% prep -design picorv32a -tag 11-09_13-36 -overwrite
```

The supplied log confirms that the custom LEF was merged:

```text
[INFO]: Merging the following extra LEFs: /openLANE_flow/designs/picorv32a/src/sky130_vsdinv.lef
[INFO]: Preparation complete
```

Run synthesis:

```text
% run_synthesis
```

### Synthesis Results

The supplied synthesis output contains:

```text
sky130_vsdinv                1554
Chip area for module '\picorv32a': 147712.918000
...
tns -711.59
wns -23.89
[INFO]: Synthesis was successful
```

The synthesized design contains **1,554 instances** of `sky130_vsdinv`.

The reported chip area is `147712.918000`, with TNS of `-711.59` and WNS of `-23.89`.

### Screenshots


<img width="946" height="547" alt="image" src="https://github.com/user-attachments/assets/855c0fc2-d6ce-46da-914f-eac714464a79" />



<img width="681" height="532" alt="image" src="https://github.com/user-attachments/assets/2471b818-a175-4a0b-86c6-3cb381fe7997" />


<img width="930" height="537" alt="image" src="https://github.com/user-attachments/assets/9b28cc47-0ae8-4e20-a438-8f25c13a317c" />

---

## Lab 6: Explore Synthesis Optimization Settings

Inspect and modify the synthesis strategy variables from the OpenLane interactive shell.

```text
% echo $::env(SYNTH_STRATEGY)
AREA 0

% set ::env(SYNTH_STRATEGY) 1

% echo $::env(SYNTH_BUFFERING)
1

% echo $::env(SYNTH_SIZING)
0

% set ::env(SYNTH_SIZING) 1

% echo $::env(SYNTH_DRIVING_CELL)
sky130_fd_sc_hd__inv_8
```

These environment variables control aspects of synthesis optimization, including area and delay strategy, buffering, cell sizing, and the driving cell used for primary inputs.

Different settings can affect the resulting area and timing reports. The actual effect should be confirmed by running synthesis and comparing the results.

### Screenshot


<img width="1077" height="537" alt="image" src="https://github.com/user-attachments/assets/1a590309-1583-485a-8872-e9a6a2dc673a" />

---

## Lab 7: Run Placement

Execute placement in the OpenLane interactive shell:

```text
% run_placement
```

### Placement Results

The supplied log reports the following statistics:

```text
total instances        21696
nets                    15444
design area             420473.3 u^2
utilization             36 %
utilization padded      54 %
rows                    238
row height              2.7 u

original HPWL           756230.0 u
legalized HPWL          769495.6 u
delta HPWL              2 %
[INFO DPL-0020] Mirrored 6226 instances
[INFO]: Taking a Screenshot of the Layout Using Klayout...
```

The report shows **21,696 instances**, **15,444 nets**, and **36% utilization**.

The half-perimeter wire length (HPWL) changes from `756230.0 u` to `769495.6 u` after legalization, with the reported delta of `2%`.

### Screenshot


<img width="1087" height="460" alt="image" src="https://github.com/user-attachments/assets/528f74a5-6b9c-4445-aa8e-72078355d6dc" />


---

## Lab 8: Inspect the Placed Design in Magic

Navigate to the placement results directory:

```bash
cd designs/picorv32a/runs/11-09_13-36/results/placement/
```

Open the placement DEF in Magic using the technology file and merged LEF:

```bash
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech \
  lef read ../../tmp/merged.lef \
  def read picorv32a.placement.def
```

The layout displays the placed standard-cell rows. Zooming in allows individual instances and their placement locations to be inspected.

Examples listed in the supplied notes include:

- `sky130_fd_sc_hd__o22a_1`
- `sky130_fd_sc_hd__tapvpwrvgnd_1`
- `PHY_3136`
- `_18745_`

### Screenshots

<img width="1057" height="542" alt="image" src="https://github.com/user-attachments/assets/c2261023-473f-45b1-ba02-baadfe2378bd" />



<img width="1080" height="532" alt="image" src="https://github.com/user-attachments/assets/a5adb9c2-3956-43f5-b56b-9b1a4cfd5670" />


---

## Lab 9: Examine Pre-STA Timing and Create Base Constraints

Run the pre-STA timing analysis and inspect the reported timing path.

### Initial Timing Results

The supplied timing report shows:

```text
data arrival time      48.32
data required time      4.71
slack (VIOLATED)      -43.62
wns -43.62
tns -11872.33
```

The negative slack indicates that the analyzed setup path does not meet the reported timing requirement.

### Create `my_base.sdc`

Create the file in:

```text
designs/picorv32a/src/my_base.sdc
```

Use the supplied clock and input/output timing settings:

```tcl
set ::env(CLOCK_PORT) clk
set ::env(CLOCK_PERIOD) 5.000
set ::env(SYNTH_DRIVING_CELL) sky130_fd_sc_hd__buf_8
set ::env(SYNTH_DRIVING_CELL_PIN) X
set ::env(SYNTH_CAP_LOAD) 17.65

create_clock [get_ports $::env(CLOCK_PORT)] -name $::env(CLOCK_PORT) -period $::env(CLOCK_PERIOD)

set IO_PCT 0.9892
set input_delay_value  [expr {$::env(CLOCK_PERIOD) * $IO_PCT}]
set output_delay_value [expr {$::env(CLOCK_PERIOD) * $IO_PCT}]

puts "\[INFO\]: Setting output delay to: $output_delay_value"
puts "\[INFO\]: Setting input delay to: $input_delay_value"

set clk_indx  [lsearch [all_inputs] [get_port $::env(CLOCK_PORT)]]
set rst_indx  [lsearch [all_inputs] [get_port resetn]]
set all_inputs_wo_clk     [lreplace [all_inputs] $clk_indx $clk_indx]
set all_inputs_wo_clk_rst [lreplace $all_inputs_wo_clk $rst_indx $rst_indx]

# correct resetn
set_input_delay $input_delay_value -clock [get_clocks $::env(CLOCK_PORT)] $all_inputs_wo_clk_rst
set_output_delay $output_delay_value -clock [get_clocks $::env(CLOCK_PORT)] [all_outputs]

set cap_load [expr {$::env(SYNTH_CAP_LOAD) / 1000.0}]
puts "\[INFO\]: Setting load to: $cap_load"
set_load $cap_load [all_outputs]
```

The file defines the clock period, driving cell, clock uncertainty-related I/O timing assumptions, and output load used by the supplied setup.

### Screenshots


<img width="891" height="547" alt="image" src="https://github.com/user-attachments/assets/2b80c4a4-cf53-4f6d-b523-047a0d7dfc7f" />

<img width="1025" height="542" alt="image" src="https://github.com/user-attachments/assets/b50f3b8b-c29e-4dbc-b9ac-5dfbb60b4ef8" />



---

## 6. Summary

This lab documents the following work:

- Studied Liberty delay tables and their role in static timing analysis.
- Reviewed setup timing, clock jitter, uncertainty, and slack.
- Examined CTS, clock skew, crosstalk, and shielding concepts.
- Inspected SKY130 routing tracks and configured the Magic grid.
- Labeled and saved the custom `sky130_vsdinv` layout.
- Generated the custom cell's LEF abstract.
- Integrated the custom cell and timing libraries into `picorv32a`.
- Ran OpenLane synthesis and inspected the synthesis statistics.
- Examined synthesis optimization variables.
- Ran placement and inspected the resulting design in Magic.
- Reviewed pre-STA timing and created `my_base.sdc`.
- Left CTS and post-CTS analysis pending because their results were not included in the source notes.

### Key Reported Values

| Metric | Recorded value |
|---|---:|
| Custom inverter instances | 1554 |
| Synthesis chip area | 147712.918000 |
| Synthesis WNS | -23.89 |
| Synthesis TNS | -711.59 |
| Placement instances | 21696 |
| Placement nets | 15444 |
| Placement utilization | 36% |
| Placement padded utilization | 54% |
| Pre-STA slack | -43.62 |
| Pre-STA WNS | -43.62 |
| Pre-STA TNS | -11872.33 |
| Clock period in SDC | 5.000 |
| Synthesis capacitance load | 17.65 |

---

## Tools Used

- **Magic** — layout inspection and custom-cell editing.
- **SKY130A PDK** — process design kit and standard-cell libraries.
- **OpenSTA** — static timing analysis.
- **OpenLane** — synthesis and physical-design flow.
- **Docker** — environment for running OpenLane.

## Screenshot Setup

Create an `images` directory in your GitHub repository and upload your own screenshots using the filenames referenced above. If your filenames differ, update the corresponding Markdown image links.

```text
your-repository/
├── README.md
└── images/
    ├── open-magic-cmd.png
    ├── inverter-layout.png
    ├── tracks-info-filw.png
    ├── docker-cmds.png
    ├── prep-complete.png
    └── ...
```

> 
