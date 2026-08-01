# Digital VLSI SoC Design and Planning

> A 10 days digital VLSI SoC design and planning workshop covering the complete RTL2GDSII flow, organized by VSD in collaboration with NASSCOM.

## Table of Contents

- [Section 1 — Inception of Open-Source EDA, OpenLANE, and Sky130 PDK](#section-1--inception-of-open-source-eda-openlane-and-sky130-pdk)
- [Section 2 — Good Floorplan vs. Bad Floorplan and Introduction to Library Cells](#section-2--good-floorplan-vs-bad-floorplan-and-introduction-to-library-cells)
- [Section 3 — Designing a Library Cell Using Magic Layout and ngspice Characterization](#section-3--designing-a-library-cell-using-magic-layout-and-ngspice-characterization)
- [Section 4 — Pre-Layout Timing Analysis and the Importance of a Good Clock Tree](#section-4--pre-layout-timing-analysis-and-the-importance-of-a-good-clock-tree)
- [Section 5 — Final Steps for RTL2GDS Using TritonRoute and OpenSTA](#section-5--final-steps-for-rtl2gds-using-tritonroute-and-opensta)
- [Acknowledgements](#acknowledgements)

---

## Section 1 — Inception of Open-Source EDA, OpenLANE, and Sky130 PDK

### Theory — OpenLANE ASIC Design Flow

OpenLANE is an open-source digital ASIC implementation flow built around the SkyWater SKY130 Process Design Kit (PDK). It automates the complete RTL-to-GDSII physical design process by integrating several industry-standard open-source EDA tools into a unified workflow. Its primary goal is to transform a synthesized RTL design into a manufacturable integrated-circuit layout while satisfying timing, power, and design-rule requirements.

### OpenLANE Design Flow Stages

**1. RTL Synthesis**
The Register Transfer Level (RTL) design is synthesized into a gate-level netlist using standard cell libraries. Logic optimization is performed during this stage to reduce area, improve timing, and minimize power consumption.

**2. Floorplanning**
The physical dimensions of the chip are established. This includes:
- Defining the core area
- Creating the power distribution network (PDN)
- Placing macros
- Reserving routing resources

A good floorplan provides the foundation for successful placement and routing.

**3. Placement**
Standard cells are positioned inside the core region.
- **Global Placement:** Determines approximate cell locations while minimizing wire length.
- **Detailed Placement:** Eliminates overlaps and aligns cells to legal placement sites.

**4. Clock Tree Synthesis (CTS)**
A balanced clock distribution network is generated to ensure the clock reaches all sequential elements with minimal skew and latency. Buffers and clock routing are inserted to improve timing reliability.

**5. Routing**
- **Global Routing:** Determines routing paths.
- **Detailed Routing:** Generates the final metal wires while satisfying all design rules.

**6. Sign-off Verification**
- **Static Timing Analysis (STA):** Verifies timing constraints.
- **Design Rule Check (DRC):** Ensures the layout follows manufacturing rules.
- **Layout Versus Schematic (LVS):** Confirms the physical layout matches the synthesized netlist.
- **Antenna and Parasitic Checks:** Detect manufacturing and reliability issues.

**7. GDSII Generation**
After successful verification, the final GDSII layout is produced, containing all geometric information required for semiconductor fabrication.

### OpenLANE Workflow

```
RTL Design
    │
    ▼
Synthesis
    │
    ▼
Floorplanning
    │
    ▼
Placement
    │
    ▼
Clock Tree Synthesis
    │
    ▼
Routing
    │
    ▼
STA / DRC / LVS
    │
    ▼
GDSII Layout
```

### Key Features of OpenLANE

- Fully open-source RTL-to-GDSII flow
- Based on the SkyWater SKY130 PDK
- Automated physical design process
- Integrated timing and physical verification
- Produces fabrication-ready GDSII layouts
- Suitable for education, research, and ASIC prototyping

### Implementation

Section 1 tasks:
1. Run `picorv32a` design synthesis using the OpenLANE flow and generate the necessary outputs.
2. Calculate the flop ratio.

$$
\text{Flop Ratio} = \frac{\text{Number of D Flip-Flops}}{\text{Total Number of Cells}}
$$

$$
\text{Percentage of DFFs} = \text{Flop Ratio} \times 100
$$

#### 1. Run `picorv32a` design synthesis using the OpenLANE flow

```bash
# Change directory to the OpenLANE flow directory
cd Desktop/work/tools/openlane_working_dir/openlane

# alias docker='docker run -it -v $(pwd):/openLANE_flow -v $PDK_ROOT:$PDK_ROOT -e PDK_ROOT=$PDK_ROOT -u $(id -u $USER):$(id -g $USER) efabless/openlane:v0.21'
# Since we aliased the long command to 'docker', we can invoke the OpenLANE flow Docker subsystem with:
docker
```

```tcl
# Invoke the OpenLANE flow in interactive mode
./flow.tcl -interactive

# Load the required package for the OpenLANE flow to function properly
package require openlane 0.9

# Prep the design — creates the necessary files and directories for 'picorv32a'
prep -design picorv32a

# Run synthesis
run_synthesis

# Exit OpenLANE flow
exit

# Exit the OpenLANE flow Docker subsystem
exit
```

Screenshots of the commands running:

<img width="1133" height="555" alt="image" src="https://github.com/user-attachments/assets/03f92235-5cc6-4b1f-a2fc-afae49cdde74" />
<img width="1282" height="769" alt="Synthesis run" src="https://github.com/user-attachments/assets/d7eb084b-32a9-4191-a85d-8ab9e22f5063" />

#### 2. Calculate the flop ratio

Screenshots of the synthesis statistics report file with the required values highlighted:

<img width="1286" height="770" alt="Synthesis stats report - part 1" src="https://github.com/user-attachments/assets/85f95345-9791-4028-b853-2bc6f0285abb" />
<img width="1280" height="772" alt="Synthesis stats report - part 2" src="https://github.com/user-attachments/assets/33639fe0-7ba9-4d1c-adcc-447f3c587595" />

Calculation of flop ratio and DFF percentage from the synthesis statistics report:

$$
\text{Flop Ratio} = \frac{1613}{14876} = 0.108429685
$$

$$
\text{Percentage of DFFs} = 0.108429685 \times 100 = 10.84296854\%
$$

---

## Section 2 — Good Floorplan vs. Bad Floorplan and Introduction to Library Cells

### Implementation

Section 2 tasks:
1. Run `picorv32a` design floorplan using the OpenLANE flow and generate the necessary outputs.
2. Calculate the die area in microns from the values in the floorplan DEF.
3. Load the generated floorplan DEF in the Magic tool and explore it.
4. Run `picorv32a` congestion-aware placement using the OpenLANE flow and generate the necessary outputs.
5. Load the generated placement DEF in the Magic tool and explore it.

$$
\text{Area of die in microns} = \text{Die width in microns} \times \text{Die height in microns}
$$

#### 1. Run `picorv32a` design floorplan using the OpenLANE flow

```bash
cd Desktop/work/tools/openlane_working_dir/openlane
docker
```

```tcl
./flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a
run_synthesis

# Run floorplan
run_floorplan
```

Screenshots of the floorplan run:

<img width="1280" height="742" alt="Floorplan run - part 1" src="https://github.com/user-attachments/assets/06be4f2d-d58e-45ae-8fa3-01ab4b9c1ddb" />
<img width="1286" height="776" alt="Floorplan run - part 2" src="https://github.com/user-attachments/assets/f098d935-dcdb-4b7a-8f23-9bd89fdcc322" />

#### 2. Calculate the die area in microns from the values in the floorplan DEF

Screenshot of the floorplan DEF contents:

<img width="1280" height="742" alt="Floorplan DEF contents" src="https://github.com/user-attachments/assets/60e30436-7c90-4870-a3ef-577d6694b88e" />

According to the floorplan DEF:

$$
1000 \text{ unit distance} = 1 \text{ micron}
$$

$$
\text{Die width in unit distance} = 660685 - 0 = 660685
$$

$$
\text{Die height in unit distance} = 671405 - 0 = 671405
$$

$$
\text{Distance in microns} = \frac{\text{Value in unit distance}}{1000}
$$

$$
\text{Die width in microns} = \frac{660685}{1000} = 660.685 \text{ microns}
$$

$$
\text{Die height in microns} = \frac{671405}{1000} = 671.405 \text{ microns}
$$

$$
\text{Area of die in microns} = 660.685 \times 671.405 = 443{,}587.212425 \text{ square microns}
$$

#### 3. Load the generated floorplan DEF in Magic and explore it

```bash
# Change directory to the path containing the generated floorplan DEF
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/17-03_12-06/results/floorplan/

# Load the floorplan DEF in Magic
magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.floorplan.def &
```

Screenshots of the floorplan DEF in Magic:

<img width="1283" height="771" alt="Floorplan DEF in Magic" src="https://github.com/user-attachments/assets/bd80f877-5ffb-4f3c-b1a2-0495c9167c47" />

Equidistant placement of ports:

<img width="1282" height="746" alt="Equidistant port placement" src="https://github.com/user-attachments/assets/deba06ae-0f1e-4eaa-b6a1-8254b11b7540" />

Port layer as set through `config.tcl`:

<img width="1281" height="769" alt="Port layer configuration" src="https://github.com/user-attachments/assets/454016df-1b98-4438-86f9-08c1427d745f" />

#### 4. Run `picorv32a` congestion-aware placement using the OpenLANE flow

```tcl
# Congestion-aware placement is the default
run_placement
```

Screenshot of the placement run:

<img width="1288" height="744" alt="Placement run" src="https://github.com/user-attachments/assets/aed41fd3-620c-4e8d-ac08-f1d23ba01bf1" />

#### 5. Load the generated placement DEF in Magic and explore it

```bash
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/17-03_12-06/results/placement/

magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &
```

Screenshots of the placement DEF in Magic:

<img width="1223" height="690" alt="Placement DEF in Magic" src="https://github.com/user-attachments/assets/4baaf752-d4ee-4791-9ed3-91c80dc6639a" />

Standard cells legally placed:

<img width="1223" height="682" alt="Legally placed standard cells" src="https://github.com/user-attachments/assets/55315f4f-7f67-4770-b093-7448f9174210" />

```tcl
# Exit OpenLANE flow
exit

# Exit the OpenLANE flow Docker subsystem
exit
```

---

## Section 3 — Designing a Library Cell Using Magic Layout and ngspice Characterization

### Implementation

Section 3 tasks:
1. Clone the custom inverter standard-cell design from the GitHub repository: [Standard cell design and characterization using OpenLANE flow](https://github.com/nickson-jose/vsdstdcelldesign).
2. Load the custom inverter layout in Magic and explore it.
3. Perform SPICE extraction of the inverter in Magic.
4. Edit the SPICE model file for simulation-based analysis.
5. Run post-layout ngspice simulations.
6. Find and fix problems in the DRC section of the outdated Magic tech file for the SkyWater process.

#### 1. Clone the custom inverter standard-cell design

```bash
cd Desktop/work/tools/openlane_working_dir/openlane

git clone https://github.com/nickson-jose/vsdstdcelldesign
cd vsdstdcelldesign

# Copy the Magic tech file into the repo directory for easy access
cp /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech .

ls

# Open the custom inverter layout in Magic
magic -T sky130A.tech sky130_inv.mag &
```

#### 2. Load the custom inverter layout in Magic and explore it

Screenshot of the custom inverter layout in Magic:
<img width="1223" height="691" alt="Custom inverter layout" src="https://github.com/user-attachments/assets/b78461b2-a0db-454d-b475-37a7cdc06b68" />

NMOS and PMOS identified:
<img width="1221" height="667" alt="NMOS identified" src="https://github.com/user-attachments/assets/a95c51ee-20bc-4281-8325-a14d5a26a1b7" />
<img width="1222" height="671" alt="PMOS identified" src="https://github.com/user-attachments/assets/fdafaf5c-795a-4965-8459-b613cf914ff8" />

Output Y connectivity to the PMOS and NMOS drains verified:
<img width="1220" height="671" alt="Output Y connectivity verified" src="https://github.com/user-attachments/assets/f007f86d-1f3e-4292-8532-620d863c6bbf" />

PMOS source connectivity to VDD (VPWR) verified:
<img width="1221" height="669" alt="PMOS source to VPWR verified" src="https://github.com/user-attachments/assets/a1aa8502-1d9c-4d6a-baed-3b4b0e9352c1" />

NMOS source connectivity to VSS (VGND) verified:
<img width="1216" height="665" alt="NMOS source to VGND verified" src="https://github.com/user-attachments/assets/8aa4eeae-89bd-4955-9922-96876991affb" />

Deleting part of the layout to trigger a DRC error:
<img width="1217" height="662" alt="Layout edit to trigger DRC error" src="https://github.com/user-attachments/assets/e272cd31-fc5a-4aaa-a7e0-36f0b51bfde5" />

#### 3. SPICE extraction of the inverter in Magic

Run in the Magic `tkcon` window:

```tcl
# Check current directory
pwd

# Extract to .ext format
extract all

# Enable parasitic extraction before converting to SPICE
ext2spice cthresh 0 rthresh 0

# Convert .ext to SPICE
ext2spice
```

#### 4. Edit the SPICE model file for simulation-based analysis

Measuring unit distance in the layout grid:
<img width="1219" height="667" alt="Unit distance measurement" src="https://github.com/user-attachments/assets/1d5ce96d-774c-4ba5-b837-0dbd10ffcba5" />

#### 5. Post-layout ngspice simulations

```bash
# Load the SPICE file directly for simulation
ngspice sky130_inv.spice

# Once in ngspice, plot the results
plot y vs time a
```

Screenshots of the ngspice run:
<img width="1218" height="651" alt="ngspice run - part 1" src="https://github.com/user-attachments/assets/dbafdef6-e730-476a-880a-0692a4659502" />
<img width="1218" height="652" alt="ngspice run - part 2" src="https://github.com/user-attachments/assets/3bc72496-e9e2-4f16-92d9-ee23f754994b" />

Screenshot of the generated plot:
<img width="1221" height="691" alt="Generated ngspice plot" src="https://github.com/user-attachments/assets/9507ea71-b352-4be5-bdd4-f337f608ac7b" />

**Rise transition time**

$$
\text{Rise transition time} = \text{Time at 80\% output} - \text{Time at 20\% output}
$$

$$
20\% \text{ of output} = 660\text{ mV}, \quad 80\% \text{ of output} = 2.64\text{ V}
$$

$$
\text{Rise transition time} = 2.24638 - 2.18242 = 0.06396\text{ ns} = 63.96\text{ ps}
$$

**Fall transition time**

$$
\text{Fall transition time} = \text{Time at 20\% output} - \text{Time at 80\% output}
$$

$$
20\% \text{ of output} = 660\text{ mV}, \quad 80\% \text{ of output} = 2.64\text{ V}
$$

$$
\text{Fall transition time} = 4.0955 - 4.0536 = 0.0419\text{ ns} = 41.9\text{ ps}
$$

**Rise cell delay**

$$
\text{Rise cell delay} = \text{Time at 50\% output rise} - \text{Time at 50\% input fall}
$$

$$
50\% \text{ of } 3.3\text{ V} = 1.65\text{ V}
$$

$$
\text{Rise cell delay} = 2.21144 - 2.15008 = 0.06136\text{ ns} = 61.36\text{ ps}
$$

**Fall cell delay**

$$
\text{Fall cell delay} = \text{Time at 50\% output fall} - \text{Time at 50\% input rise}
$$

$$
50\% \text{ of } 3.3\text{ V} = 1.65\text{ V}
$$

$$
\text{Fall cell delay} = 4.07 - 4.05 = 0.02\text{ ns} = 20\text{ ps}
$$

#### 6. Find and fix DRC problems in the outdated SkyWater Magic tech file

Reference: [Sky130 Periphery Rules](https://skywater-pdk.readthedocs.io/en/main/rules/periphery.html)

```bash
cd

# Download the lab files
wget http://opencircuitdesign.com/open_pdks/archive/drc_tests.tgz

# Extract
tar xfz drc_tests.tgz
cd drc_tests
ls -al

# View the .magicrc file
gvim .magicrc

# Open Magic with better graphics
magic -d XR &
```

Screenshot of the commands running:
<img width="1220" height="672" alt="DRC test setup commands" src="https://github.com/user-attachments/assets/c3468c99-5b26-4636-9e27-6e71d813c3fb" />

**Fixing the incorrectly implemented `poly.9` spacing rule**

Screenshot of the poly rules:
<img width="1221" height="655" alt="Poly rules reference" src="https://github.com/user-attachments/assets/2e6ddf02-a7db-466d-9e44-f8d6d9330608" />

No DRC violation is flagged even though spacing is below 0.48 µm:
<img width="1220" height="688" alt="Poly.9 missing violation - part 1" src="https://github.com/user-attachments/assets/8b72c0b8-8907-4d80-9110-c96a9916c433" />
<img width="1219" height="691" alt="Poly.9 missing violation - part 2" src="https://github.com/user-attachments/assets/2ba15e61-dc64-42f3-8f1d-c4c6c40399fc" />

New rule commands inserted into `sky130A.tech` to fix DRC. Run in the `tkcon` window:

```tcl
# Load the updated tech file
tech load sky130A.tech

# Re-run the DRC check to see the updated errors
drc check

# Select the region showing new errors and print the error message
drc why
```

Screenshot of Magic with the rule fix applied:
<img width="1217" height="686" alt="Poly.9 rule fixed - part 1" src="https://github.com/user-attachments/assets/f7e6ef58-ffac-40cb-9595-820eb9ef1d5b" />
<img width="1218" height="683" alt="Poly.9 rule fixed - part 2" src="https://github.com/user-attachments/assets/d5538a85-b41c-4943-87d2-d6cf55988d3f" />

**Fixing the incorrectly implemented `difftap.2` spacing rule**

No DRC violation is flagged even though spacing is below 0.42 µm:
<img width="1216" height="688" alt="Difftap.2 missing violation" src="https://github.com/user-attachments/assets/0f766b05-9e7d-42c4-8109-c25f9c86d52f" />

New rule commands inserted into `sky130A.tech`. Run in the `tkcon` window:

```tcl
tech load sky130A.tech
drc check
drc why
```

```tcl
tech load sky130A.tech

# Switch to the full DRC style
drc style drc(full)

drc check
drc why
```

---

## Section 4 — Pre-Layout Timing Analysis and the Importance of a Good Clock Tree

### Implementation

Section 4 tasks:
1. Fix the remaining DRC errors and verify the design is ready to enter the flow.
2. Save the finalized layout under a custom name and reopen it.
3. Generate a LEF file from the layout.
4. Copy the new LEF and associated lib files into the `picorv32a` design's `src` directory.
5. Edit `config.tcl` to reference the new lib files and add the extra LEF to the OpenLANE flow.
6. Run OpenLANE synthesis with the newly inserted custom inverter cell.
7. Reduce the timing violations introduced by the custom inverter cell by tuning design parameters.
8. Once synthesis accepts the custom inverter, run floorplan and placement and verify the cell is accepted in the PnR flow.
9. Perform post-synthesis timing analysis with OpenSTA.
10. Make timing ECO fixes to remove violations.
11. Replace the old netlist with the ECO-fixed netlist, then run floorplan, placement, and CTS.
12. Perform post-CTS OpenROAD timing analysis.
13. Explore post-CTS OpenROAD timing analysis after removing `sky130_fd_sc_hd__clkbuf_1` from the `CTS_CLK_BUFFER_LIST` variable.

#### 1. Fix DRC errors and verify the design is ready for the flow

Conditions to verify before proceeding with the custom cell layout:
- **Condition 1:** The standard cell's input and output ports must lie on the intersection of the vertical and horizontal tracks.
- **Condition 2:** The standard cell's width must be an odd multiple of the horizontal track pitch.
- **Condition 3:** The standard cell's height must be an even multiple of the vertical track pitch.

```bash
cd Desktop/work/tools/openlane_working_dir/openlane/vsdstdcelldesign

magic -T sky130A.tech sky130_inv.mag &
```

Set the grid to match the locali layer tracks (run in the `tkcon` window):

```tcl
# View grid command syntax
help grid

# Set grid values
grid 0.46um 0.34um 0.23um 0.17um
```

Condition 1 verified:
<img width="1216" height="679" alt="Condition 1 - port alignment" src="https://github.com/user-attachments/assets/c6900b5b-eb2e-47d2-90ef-322faa1fb16f" />

Condition 2 verified:

$$
\text{Horizontal track pitch} = 0.46\ \mu m
$$

<img width="1217" height="684" alt="Condition 2 - horizontal track pitch" src="https://github.com/user-attachments/assets/ed551d28-0b0c-4b62-a84c-4df4713c3b0e" />

$$
\text{Width of standard cell} = 1.38\ \mu m = 0.46 \times 3
$$

Condition 3 verified:

$$
\text{Vertical track pitch} = 0.34\ \mu m
$$

<img width="1216" height="684" alt="Condition 3 - vertical track pitch" src="https://github.com/user-attachments/assets/e538ebdf-750f-45ed-aca4-07b3b7049ac8" />

$$
\text{Height of standard cell} = 2.72\ \mu m = 0.34 \times 8
$$

#### 2. Save the finalized layout under a custom name

```tcl
save sky130_vsdinv.mag
```

```bash
magic -T sky130A.tech sky130_vsdinv.mag &
```

Screenshot of the newly saved layout:
<img width="1218" height="687" alt="Saved layout as sky130_vsdinv" src="https://github.com/user-attachments/assets/7843a235-d8d4-4776-b2fd-a957580f35c2" />

#### 3. Generate a LEF file from the layout

```tcl
lef write
```

#### 4. Copy the new LEF and lib files into the `picorv32a` `src` directory

```bash
cp sky130_vsdinv.lef ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/
ls ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/

cp libs/sky130_fd_sc_hd__* ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/
ls ~/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/src/
```

Screenshot of the commands running:
<img width="1210" height="359" alt="Copying LEF and lib files" src="https://github.com/user-attachments/assets/13510c6d-f897-4b57-890d-c30f34244b66" />

#### 5. Update `config.tcl` to reference the new lib files and extra LEF

```tcl
set ::env(LIB_SYNTH) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"
set ::env(LIB_FASTEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__fast.lib"
set ::env(LIB_SLOWEST) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__slow.lib"
set ::env(LIB_TYPICAL) "$::env(OPENLANE_ROOT)/designs/picorv32a/src/sky130_fd_sc_hd__typical.lib"

set ::env(EXTRA_LEFS) [glob $::env(OPENLANE_ROOT)/designs/$::env(DESIGN_NAME)/src/*.lef]
```

#### 6. Run OpenLANE synthesis with the custom inverter cell

```bash
cd Desktop/work/tools/openlane_working_dir/openlane
docker
```

```tcl
./flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a

# Include the newly added LEF in the flow
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs

run_synthesis
```

#### 7. Reduce the timing violations introduced by the custom inverter cell

```tcl
# Re-prep the design to refresh variables
prep -design picorv32a -tag 24-03_10-03 -overwrite

# Re-include the custom LEF
set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs

# View and update SYNTH_STRATEGY
echo $::env(SYNTH_STRATEGY)
set ::env(SYNTH_STRATEGY) "DELAY 3"

# View SYNTH_BUFFERING (check whether it's enabled)
echo $::env(SYNTH_BUFFERING)

# View and update SYNTH_SIZING
echo $::env(SYNTH_SIZING)
set ::env(SYNTH_SIZING) 1

# View SYNTH_DRIVING_CELL to confirm it's the correct cell
echo $::env(SYNTH_DRIVING_CELL)

run_synthesis
```

#### 8. Run floorplan and placement with the custom inverter cell

```tcl
run_floorplan
```

`run_floorplan` produced an unexplained error, so the individual commands it wraps were run instead (sourced from `scripts/tcl_commands/floorplan.tcl` and the `Floorplan Commands` section of `docs/source/OpenLANE_commands.md`):

```tcl
init_floorplan
place_io
tap_decap_or
```

```tcl
run_placement
```

```bash
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/24-03_10-03/results/placement/

magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.placement.def &
```

Screenshot of the placement DEF in Magic:
<img width="1216" height="687" alt="Placement DEF with custom inverter" src="https://github.com/user-attachments/assets/475bfd47-24ea-47b8-8044-d0d45a71ac77" />

Custom inverter inserted in the placement DEF with proper abutment:
<img width="1214" height="684" alt="Custom inverter abutment" src="https://github.com/user-attachments/assets/bd88de31-f3cc-456b-becc-780f41f80307" />

View the internal layers of the cells (run in `tkcon`):

```tcl
expand
```

Abutment of power pins with adjacent library cells clearly visible:
<img width="1219" height="683" alt="Power pin abutment" src="https://github.com/user-attachments/assets/afb178c9-99c6-4f6a-bfe1-eb264f62d711" />

#### 9. Post-synthesis timing analysis with OpenSTA

Since the tuned run produced 0 WNS, timing analysis was instead performed on the initial (unoptimized) synthesis run, which contains numerous violations.

```bash
cd Desktop/work/tools/openlane_working_dir/openlane
docker
```

```tcl
./flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a

set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs

set ::env(SYNTH_SIZING) 1

run_synthesis
```

A `pre_sta.conf` file was created in the `openlane` directory, and `my_base.sdc` was created in `openlane/designs/picorv32a/src`, based on `openlane/scripts/base.sdc`. Run STA in a separate terminal:

```bash
cd Desktop/work/tools/openlane_working_dir/openlane
sta pre_sta.conf
```

High fanout was found to be increasing delay, so `SYNTH_MAX_FANOUT` was reduced and synthesis re-run:

```tcl
prep -design picorv32a -tag 25-03_18-52 -overwrite

set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs

set ::env(SYNTH_SIZING) 1
set ::env(SYNTH_MAX_FANOUT) 4

echo $::env(SYNTH_DRIVING_CELL)

run_synthesis
```

```bash
cd Desktop/work/tools/openlane_working_dir/openlane
sta pre_sta.conf
```

#### 10. Timing ECO fixes to remove violations

An OR gate of drive strength 2 was found driving 4 fanouts:
<img width="1210" height="688" alt="OR gate drive strength 2 with 4 fanouts" src="https://github.com/user-attachments/assets/b8f9c0c1-70d8-4c07-b7ef-f896ad5e85d0" />

```tcl
# Report all connections to a net
report_net -connections _11672_

# Check the syntax of replace_cell
help replace_cell

# Replace with a higher drive-strength cell
replace_cell _14510_ sky130_fd_sc_hd__or3_4

# Generate a custom timing report
report_checks -fields {net cap slew input_pins} -digits 4
```

Result: slack reduced.
<img width="1213" height="679" alt="Slack reduced after ECO fix" src="https://github.com/user-attachments/assets/45588629-260f-4427-b665-0cb0cb02bd0c" />

The same pattern (an OR gate of drive strength 2 driving 4 fanouts, or driving an OA gate with excess delay) recurred at several other nets and was fixed the same way:

```tcl
report_net -connections _11675_
replace_cell _14514_ sky130_fd_sc_hd__or3_4
report_checks -fields {net cap slew input_pins} -digits 4
```

```tcl
report_net -connections _11643_
replace_cell _14481_ sky130_fd_sc_hd__or4_4
report_checks -fields {net cap slew input_pins} -digits 4
```

```tcl
report_net -connections _11668_
replace_cell _14506_ sky130_fd_sc_hd__or4_4
report_checks -fields {net cap slew input_pins} -digits 4
```

Verifying that instance `_14506_` was replaced with `sky130_fd_sc_hd__or4_4`:

```tcl
report_checks -from _29043_ -to _30440_ -through _14506_
```

Screenshot of the replaced instance:
<img width="1210" height="646" alt="Replaced instance verification" src="https://github.com/user-attachments/assets/19a0fb37-75b5-458e-bee5-16248a7c763e" />

ECO fixes started at a WNS of −23.9000 ns and ended at −22.6173 ns — a reduction of roughly 1.2827 ns of violation.

#### 11. Replace the old netlist with the ECO-fixed netlist and continue the flow

Before overwriting, a backup of the old netlist was made:

```bash
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/25-03_18-52/results/synthesis/
ls

cp picorv32a.synthesis.v picorv32a.synthesis_old.v
ls
```

```tcl
help write_verilog

# Overwrite the current synthesis netlist
write_verilog /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/25-03_18-52/results/synthesis/picorv32a.synthesis.v

# Exit OpenSTA — timing analysis complete
exit
```

The netlist overwrite was verified by confirming instance `_14506_` was replaced with `sky130_fd_sc_hd__or4_4`. To continue with the cleaner (0-violation) design, the flow proceeded with the earlier run instead:

```tcl
prep -design picorv32a -tag 24-03_10-03 -overwrite

set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs

set ::env(SYNTH_STRATEGY) "DELAY 3"
set ::env(SYNTH_SIZING) 1

run_synthesis

init_floorplan
place_io
tap_decap_or

run_placement

# In case of an error
unset ::env(LIB_CTS)

run_cts
```

#### 12. Post-CTS OpenROAD timing analysis

```tcl
openroad

read_lef /openLANE_flow/designs/picorv32a/runs/24-03_10-03/tmp/merged.lef
read_def /openLANE_flow/designs/picorv32a/runs/24-03_10-03/results/cts/picorv32a.cts.def

write_db pico_cts.db
read_db pico_cts.db

read_verilog /openLANE_flow/designs/picorv32a/runs/24-03_10-03/results/synthesis/picorv32a.synthesis_cts.v
read_liberty $::env(LIB_SYNTH_COMPLETE)
link_design picorv32a

read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc
set_propagated_clock [all_clocks]

help report_checks

report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4

exit
```

Screenshots of the commands and the generated timing report:
<img width="1212" height="688" alt="Post-CTS timing analysis - part 1" src="https://github.com/user-attachments/assets/6f84f7de-8cb3-4b06-af1e-05aef8d603a1" />
<img width="1221" height="709" alt="Post-CTS timing analysis - part 2" src="https://github.com/user-attachments/assets/aeb2f933-0f26-46eb-b9a8-22525be03396" />

#### 13. Post-CTS analysis after removing `sky130_fd_sc_hd__clkbuf_1` from `CTS_CLK_BUFFER_LIST`

```tcl
echo $::env(CTS_CLK_BUFFER_LIST)

set ::env(CTS_CLK_BUFFER_LIST) [lreplace $::env(CTS_CLK_BUFFER_LIST) 0 0]

echo $::env(CTS_CLK_BUFFER_LIST)
echo $::env(CURRENT_DEF)

set ::env(CURRENT_DEF) /openLANE_flow/designs/picorv32a/runs/24-03_10-03/results/placement/picorv32a.placement.def

run_cts

echo $::env(CTS_CLK_BUFFER_LIST)

openroad

read_lef /openLANE_flow/designs/picorv32a/runs/24-03_10-03/tmp/merged.lef
read_def /openLANE_flow/designs/picorv32a/runs/24-03_10-03/results/cts/picorv32a.cts.def

write_db pico_cts1.db
read_db pico_cts.db

read_verilog /openLANE_flow/designs/picorv32a/runs/24-03_10-03/results/synthesis/picorv32a.synthesis_cts.v
read_liberty $::env(LIB_SYNTH_COMPLETE)
link_design picorv32a

read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc
set_propagated_clock [all_clocks]

report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4

report_clock_skew -hold
report_clock_skew -setup

exit

echo $::env(CTS_CLK_BUFFER_LIST)

# Restore 'sky130_fd_sc_hd__clkbuf_1' at the start of the list
set ::env(CTS_CLK_BUFFER_LIST) [linsert $::env(CTS_CLK_BUFFER_LIST) 0 sky130_fd_sc_hd__clkbuf_1]

echo $::env(CTS_CLK_BUFFER_LIST)
```

Screenshot of the commands and the generated timing report:
<img width="1212" height="686" alt="CTS buffer list experiment results" src="https://github.com/user-attachments/assets/1f7144ce-f244-4bdf-9d18-c506e9b598af" />

---

## Section 5 — Final Steps for RTL2GDS Using TritonRoute and OpenSTA

### Implementation

Section 5 tasks:
1. Generate the Power Distribution Network (PDN) and explore the PDN layout.
2. Perform detailed routing using TritonRoute.
3. Perform post-route parasitic extraction using the SPEF extractor.
4. Perform post-route OpenSTA timing analysis using the extracted parasitics.

#### 1. Generate the Power Distribution Network (PDN)

```bash
cd Desktop/work/tools/openlane_working_dir/openlane
docker
```

```tcl
./flow.tcl -interactive
package require openlane 0.9
prep -design picorv32a

set lefs [glob $::env(DESIGN_DIR)/src/*.lef]
add_lefs -src $lefs

set ::env(SYNTH_STRATEGY) "DELAY 3"
set ::env(SYNTH_SIZING) 1

run_synthesis

init_floorplan
place_io
tap_decap_or

run_placement

unset ::env(LIB_CTS)

run_cts

# Generate the power distribution network
gen_pdn
```

Screenshot of the PDN generation run:
<img width="1209" height="684" alt="PDN generation run" src="https://github.com/user-attachments/assets/e81bc420-3f02-4e06-9f38-61dcfce476b7" />

```bash
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/26-03_08-45/tmp/floorplan/

magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read 14-pdn.def &
```

Screenshot of the PDN DEF:
<img width="1201" height="666" alt="PDN DEF in Magic" src="https://github.com/user-attachments/assets/ecdd201b-77f8-49d1-97c7-dad03c52bf2b" />

#### 2. Perform detailed routing with TritonRoute

```tcl
echo $::env(CURRENT_DEF)
echo $::env(ROUTING_STRATEGY)

run_routing
```

Screenshot of the routing run:
<img width="978" height="639" alt="Routing run" src="https://github.com/user-attachments/assets/31455c6e-41d8-4b91-979f-558091be3265" />

```bash
cd Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/26-03_08-45/results/routing/

magic -T /home/vsduser/Desktop/work/tools/openlane_working_dir/pdks/sky130A/libs.tech/magic/sky130A.tech lef read ../../tmp/merged.lef def read picorv32a.def &
```

Screenshot of the routed DEF:
<img width="1206" height="640" alt="Routed DEF in Magic" src="https://github.com/user-attachments/assets/fcaaa247-1000-4d0f-bfde-d8de3e263ec1" />

A fast-route guide is also available in `openlane/designs/picorv32a/runs/26-03_08-45/tmp/routing`.

#### 3. Post-route parasitic extraction using the SPEF extractor

```bash
cd Desktop/work/tools/SPEF_EXTRACTOR

python3 main.py /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/26-03_08-45/tmp/merged.lef /home/vsduser/Desktop/work/tools/openlane_working_dir/openlane/designs/picorv32a/runs/26-03_08-45/results/routing/picorv32a.def
```

#### 4. Post-route OpenSTA timing analysis with extracted parasitics

```tcl
openroad

read_lef /openLANE_flow/designs/picorv32a/runs/26-03_08-45/tmp/merged.lef
read_def /openLANE_flow/designs/picorv32a/runs/26-03_08-45/results/routing/picorv32a.def

write_db pico_route.db
read_db pico_route.db

read_verilog /openLANE_flow/designs/picorv32a/runs/26-03_08-45/results/synthesis/picorv32a.synthesis_preroute.v
read_liberty $::env(LIB_SYNTH_COMPLETE)
link_design picorv32a

read_sdc /openLANE_flow/designs/picorv32a/src/my_base.sdc
set_propagated_clock [all_clocks]

read_spef /openLANE_flow/designs/picorv32a/runs/26-03_08-45/results/routing/picorv32a.spef

report_checks -path_delay min_max -fields {slew trans net cap input_pins} -format full_clock_expanded -digits 4

exit
```

Screenshot of the commands and the generated timing report:
<img width="1155" height="634" alt="Post-route timing analysis" src="https://github.com/user-attachments/assets/6567a759-1ebc-40a4-8fd5-197c20a3ef41" />

---

## Acknowledgements

- [Kunal Ghosh](https://github.com/kunalg123) — Co-founder, VSD Corp. Pvt. Ltd.
- [Nickson P Jose](https://github.com/nickson-jose) — Physical Design Engineer, Intel Corporation.
- [R. Timothy Edwards](https://github.com/RTimothyEdwards) — Senior Vice President of Analog and Design, efabless Corporation.
