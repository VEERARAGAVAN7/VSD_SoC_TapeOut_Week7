# Week 7 Task – BabySoC Physical Design & Post-Route SPEF Generation

### VSD Hardware Design Program

## OpenROAD-Flow-Scripts: Physical Design, Post-Route SPEF and Verilog Netlist Generation for VSDBabySoC

### 📚 Contents
- [Contents of Config.mk](#contents-of-config.mk)
- [Synthesis to Route of VSDBabySoC Design](#synthesis-to-route-of-vsdbabysoc-design)
  - [Run Synthesis](#run-synthesis)
  - [Run Floorplan](#run-floorplan)
  - [Run Placement](#run-placement)
  - [Run CTS](#run-cts)
  - [Run Routing](#run-routing)
  - [🔄 Convert `.odb` to `.def` in OpenROAD](#convert-odb-to-def-in-openroad)
- [VSDBabySoC post_route SPEF generation](#vsdbabysoc-post_route-spef-generation)
  - [Step 1: Launch OpenROAD](#step-1-launch-openroad)
  - [Step 2: Load Design and Technology Files](#step-2-load-design-and-technology-files)
  - [Step 3: RC Extraction and Output Generation](#step-3-rc-extraction-and-output-generation)
    - [1. Define Process Corner](#1-define-process-corner)
    - [2. Extract Parasitics](#2-extract-parasitics)
    - [3. Write SPEF File](#3-write-spef-file)
    - [4. Write Post-Placement Verilog Netlist](#4-write-post-placement-verilog-netlist)

---

### `Contents of Config.mk`

<details> <summary><strong>config.mk</strong></summary>

```
    # Design and Platform Configuration
   export DESIGN_NICKNAME = vsdbabysoc
   export DESIGN_NAME = vsdbabysoc
   export PLATFORM    = sky130hd

  # Design Paths
  export vsdbabysoc_DIR = /home/veeraragavan/OpenROAD-flow-scripts/flow/designs/sky130hd/$(DESIGN_NICKNAME)

  # Explicitly list Verilog files for synthesis
   export VERILOG_FILES = /home/veeraragavan/OpenROAD-flow-scripts/flow/designs/src/vsdbabysoc/vsdbabysoc.v \
                         /home/veeraragavan/OpenROAD-flow-scripts/flow/designs/src/vsdbabysoc/rvmyth.v \
                         /home/veeraragavan/OpenROAD-flow-scripts/flow/designs/src/vsdbabysoc/clk_gate.v


  # Include Directory for Verilog Header Files
   export VERILOG_INCLUDE_DIRS = $(vsdbabysoc_DIR)/include

  # Constraints File
    export SDC_FILE = $(vsdbabysoc_DIR)/vsdbabysoc_synthesis.sdc

  # Additional GDS Files
    export ADDITIONAL_GDS = $(vsdbabysoc_DIR)/gds/avsddac.gds \
                            $(vsdbabysoc_DIR)/gds/avsdpll.gds

  # Additional LEF Files
   export ADDITIONAL_LEFS = $(vsdbabysoc_DIR)/lef/avsddac.lef \
                            $(vsdbabysoc_DIR)/lef/avsdpll.lef

  # Additional LIB Files
   export ADDITIONAL_LIBS = $(vsdbabysoc_DIR)/lib/avsddac.lib \
                            $(vsdbabysoc_DIR)/lib/avsdpll.lib

 # Pin Order and Macro Placement Configurations
 export FP_PIN_ORDER_CFG = $(vsdbabysoc_DIR)/pin_order.cfg
 export MACRO_PLACEMENT_CFG = $(vsdbabysoc_DIR)/macro.cfg
 #ADDED BY VEERA
 #export PRE_GLOBAL_ROUTE_TCL = $(vsdbabysoc_DIR)/pre_gr.tcl

 # Clock Configuration
 export CLOCK_PORT = CLK
 export CLOCK_NET  = $(CLOCK_PORT)
 export CLOCK_PERIOD = 11.00

# Floorplanning Configuration
 export DIE_AREA   = 0 0 1600 1600
 export CORE_AREA  = 20 20 1590 1590 
  
  
# Core Area Usage and How close the cells can be placed
 export PLACE_DENSITY = 0.6


# Placement Configuration
export PLACE_PINS_ARGS = -exclude left:0-600 -exclude left:1000-1600 -exclude right:* -exclude top:* -exclude bottom:*

# Tuning for Timing and Buffers
  export TNS_END_PERCENT     = 100
  export REMOVE_ABC_BUFFERS  = 1
  export CTS_BUF_DISTANCE    = 600
  export SKIP_GATE_CLONING   = 1

 # Magic Tool Configuration
   export MAGIC_ZEROIZE_ORIGIN = 0
   export MAGIC_EXT_USE_GDS    = 1
   
   
#ADDE BY VEERA
# Macro placement & channel
export MACRO_PLACE_HALO    = 30 30
export MACRO_PLACE_CHANNEL = 40 40
export MACRO_BLOCKAGE_HALO = 0
export DISABLE_MACRO_PLACEMENT = 1

```
</details>

### `Synthesis to Route of VSDBabySoC Design`

### `run synthesis`

Before running the updated flow, make sure to remove any previously generated results, logs, and intermediate files. Use the following command:

```shell

cd OpenROAD-flow-scripts/flow
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk clean_all
```

```shell
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk synth
```

![Alt Text](Screenshots/synth1.png)

**Synthesis Stats**

```shell
gvim reports/sky130hd/vsdbabysoc/base/synth_stat.txt
```

![Alt Text](Screenshots/synth2.png)


**Synthesis netlist**

```shell
gvim results/sky130hd/vsdbabysoc/base/1_2_yosys.v
```
![Alt Text](Screenshots/synth3.png)

**Synthesis Check**

```shell
gvim reports/sky130hd/vsdbabysoc/base/synth_check.txt
```

![Alt Text](Screenshots/synth4.png)


### `run floorplan`

```shell
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk floorplan
```

![Alt Text](Screenshots/fp1.png)

![Alt Text](Screenshots/fp2.png)

**Floorplan Result (GUI)**

```shell
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_floorplan
```

This image shows the floorplan view in OpenROAD where you can see two macros placed: **DAC** and **PLL** after the floorplanning step.

![Alt Text](Screenshots/fp3.png)


### `run placement`

```shell
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk place
```

![Alt Text](Screenshots/p1.png)

![Alt Text](Screenshots/p2.png)

**Placement Result (GUI)**

```shell
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_place
```

This image shows the placement stage in OpenROAD with the **placement density** heat map enabled.

![Alt Text](Screenshots/p3.png)

This image shows a zoomed-in view of the Placement Density Heatmap after the placement stage:

- **Red regions** indicate areas with higher cell density, approaching 100%.
- **Green and blue regions** indicate moderate to low cell density.
- The highlighted instance (`_07289_`) displays details such as origin coordinates, site count, site spacing, and bounding box dimensions.

❗**Note:** In the floorplan stage, you do not see any placement density heat maps because standard cells have not yet been placed. The heat map will only appear after the placement step.

<ins>The placement density percentage is calculated as:</ins>

**Placement Density (%) = (Area Occupied by Cells ÷ Total Placement Area) × 100**

![Alt Text](Screenshots/p4.png)

This image shows the **Pin Density Heatmap** after the placement stage.

<ins>The pin density percentage is calculated as:</ins>

**Pin Density (%) = (Number of Pins in a Region ÷ Total Area of that Region) × 100**

![Alt Text](Screenshots/p5.png)


### `run cts`

```shell
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk cts
```

![Alt Text](Screenshots/cts1.png)

![Alt Text](Screenshots/cts2.png)

**CTS Result (GUI)**

```shell
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_cts
```
This image shows the **Clock Tree Viewer after Clock Tree Synthesis (CTS)**, illustrating the hierarchical structure of the clock network. The root node at the top represents the clock source (`pll/CLK`), and the branches show the inserted clock buffers used to distribute the clock signal across the design. The vertical axis represents the **clock arrival times (in nanoseconds)** at each stage. The endpoints at the bottom represent the registers (clock sinks), where the clock reaches after passing through multiple buffer levels. The balanced branching and closely aligned arrival times indicate **low clock skew and a well-optimized clock tree**.

![Alt Text](Screenshots/cts3.png)

This image shows the **Setup Timing Report**, presenting a list of timing paths with key metrics such as:

- **Required Time**
- **Arrival Time**
- **Slack**
- **Skew**
- **Logic Delay**
- **Logic Depth**
- **Fanout**

All paths have **positive slack**, confirming that the design meets **setup timing requirements**.
![Alt Text](Screenshots/cts4.png)


This image displays the **Hold Timing Report**, showing timing paths with details such as:

- **Required Time**
- **Arrival Time**
- **Slack**
- **Skew**
- **Logic Delay**
- **Fanout**

All paths listed have **positive slack**, indicating that the design meets **hold timing requirements** and is free from hold violations.
![Alt Text](Screenshots/cts5.png)

This image shows the **Setup Slack Histogram** after CTS. The histogram represents the distribution of endpoint slack values, all of which are positive, indicating that there are no setup timing violations.

![Alt Text](Screenshots/cts6.png)

This image shows the **Hold Slack Histogram** after CTS. The histogram represents the distribution of hold slack values for all endpoints. All values are positive, confirming that the design meets hold timing requirements without any violations.

![Alt Text](Screenshots/cts7.png)


Zoomed-in view of the design after CTS, showing inserted clock buffers and routing connections.
![Alt Text](Screenshots/cts8.png)
**CTS final report:**

```shell
gvim /home/veeraragavan/OpenROAD-flow-scripts/flow/reports/sky130hd/vsdbabysoc/base/4_cts_final.rpt
```

<details> <summary><strong>4_cts_final.rpt</strong></summary>

```

==========================================================================
cts final report_tns
--------------------------------------------------------------------------
tns max 0.00

==========================================================================
cts final report_wns
--------------------------------------------------------------------------
wns max 0.00

==========================================================================
cts final report_worst_slack
--------------------------------------------------------------------------
worst slack max 6.06

==========================================================================
cts final report_clock_min_period
--------------------------------------------------------------------------
clk period_min = 4.94 fmax = 202.47

==========================================================================
cts final report_clock_skew
--------------------------------------------------------------------------
Clock clk
   0.79 source latency core.CPU_src2_value_a3[11]$_DFF_P_/CLK ^
  -0.87 target latency core.CPU_src1_value_a3[18]$_DFF_P_/CLK ^
   0.00 CRPR
--------------
  -0.08 setup skew


==========================================================================
cts final report_checks -path_delay min
--------------------------------------------------------------------------
Startpoint: core.CPU_inc_pc_a2[3]$_DFF_P_
            (rising edge-triggered flip-flop clocked by clk)
Endpoint: core.CPU_inc_pc_a3[3]$_DFF_P_
          (rising edge-triggered flip-flop clocked by clk)
Path Group: clk
Path Type: min

Fanout     Cap    Slew   Delay    Time   Description
-----------------------------------------------------------------------------
                          0.00    0.00   clock clk (rise edge)
                          0.00    0.00   clock source latency
     1    0.31    0.00    0.00    0.00 ^ pll/CLK (avsdpll)
                                         CLK (net)
                  0.05    0.03    0.03 ^ clkbuf_0_CLK/A (sky130_fd_sc_hd__clkbuf_16)
     8    0.22    0.23    0.27    0.30 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_0_CLK (net)
                  0.23    0.00    0.30 ^ clkbuf_3_0__f_CLK/A (sky130_fd_sc_hd__clkbuf_16)
    12    0.22    0.23    0.33    0.62 ^ clkbuf_3_0__f_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_3_0__leaf_CLK (net)
                  0.23    0.00    0.63 ^ clkbuf_leaf_3_CLK/A (sky130_fd_sc_hd__clkbuf_16)
     9    0.03    0.06    0.20    0.82 ^ clkbuf_leaf_3_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_leaf_3_CLK (net)
                  0.06    0.00    0.83 ^ core.CPU_inc_pc_a2[3]$_DFF_P_/CLK (sky130_fd_sc_hd__dfxtp_1)
     1    0.00    0.04    0.30    1.13 ^ core.CPU_inc_pc_a2[3]$_DFF_P_/Q (sky130_fd_sc_hd__dfxtp_1)
                                         core.CPU_inc_pc_a2[3] (net)
                  0.04    0.00    1.13 ^ core.CPU_inc_pc_a3[3]$_DFF_P_/D (sky130_fd_sc_hd__dfxtp_1)
                                  1.13   data arrival time

                          0.00    0.00   clock clk (rise edge)
                          0.00    0.00   clock source latency
     1    0.31    0.00    0.00    0.00 ^ pll/CLK (avsdpll)
                                         CLK (net)
                  0.05    0.03    0.03 ^ clkbuf_0_CLK/A (sky130_fd_sc_hd__clkbuf_16)
     8    0.22    0.23    0.27    0.30 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_0_CLK (net)
                  0.23    0.00    0.30 ^ clkbuf_3_0__f_CLK/A (sky130_fd_sc_hd__clkbuf_16)
    12    0.22    0.23    0.33    0.62 ^ clkbuf_3_0__f_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_3_0__leaf_CLK (net)
                  0.23    0.00    0.63 ^ clkbuf_leaf_110_CLK/A (sky130_fd_sc_hd__clkbuf_16)
     9    0.06    0.08    0.21    0.84 ^ clkbuf_leaf_110_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_leaf_110_CLK (net)
                  0.08    0.00    0.84 ^ core.CPU_inc_pc_a3[3]$_DFF_P_/CLK (sky130_fd_sc_hd__dfxtp_1)
                          0.00    0.84   clock reconvergence pessimism
                         -0.03    0.82   library hold time
                                  0.82   data required time
-----------------------------------------------------------------------------
                                  0.82   data required time
                                 -1.13   data arrival time
-----------------------------------------------------------------------------
                                  0.31   slack (MET)



==========================================================================
cts final report_checks -path_delay max
--------------------------------------------------------------------------
Startpoint: core.CPU_src2_value_a3[13]$_DFF_P_
            (rising edge-triggered flip-flop clocked by clk)
Endpoint: core.CPU_src2_value_a3[24]$_DFF_P_
          (rising edge-triggered flip-flop clocked by clk)
Path Group: clk
Path Type: max

Fanout     Cap    Slew   Delay    Time   Description
-----------------------------------------------------------------------------
                          0.00    0.00   clock clk (rise edge)
                          0.00    0.00   clock source latency
     1    0.31    0.00    0.00    0.00 ^ pll/CLK (avsdpll)
                                         CLK (net)
                  0.05    0.03    0.03 ^ clkbuf_0_CLK/A (sky130_fd_sc_hd__clkbuf_16)
     8    0.22    0.23    0.27    0.30 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_0_CLK (net)
                  0.23    0.00    0.30 ^ clkbuf_3_6__f_CLK/A (sky130_fd_sc_hd__clkbuf_16)
    11    0.18    0.19    0.30    0.60 ^ clkbuf_3_6__f_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_3_6__leaf_CLK (net)
                  0.19    0.00    0.60 ^ clkbuf_leaf_70_CLK/A (sky130_fd_sc_hd__clkbuf_16)
    10    0.04    0.06    0.19    0.80 ^ clkbuf_leaf_70_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_leaf_70_CLK (net)
                  0.06    0.00    0.80 ^ core.CPU_src2_value_a3[13]$_DFF_P_/CLK (sky130_fd_sc_hd__dfxtp_1)
     3    0.05    0.46    0.60    1.40 ^ core.CPU_src2_value_a3[13]$_DFF_P_/Q (sky130_fd_sc_hd__dfxtp_1)
                                         core.CPU_src2_value_a3[13] (net)
                  0.46    0.00    1.40 ^ _05535_/A (sky130_fd_sc_hd__inv_1)
     1    0.00    0.09    0.09    1.49 v _05535_/Y (sky130_fd_sc_hd__inv_1)
                                         _00102_ (net)
                  0.09    0.00    1.49 v _10833_/B (sky130_fd_sc_hd__ha_1)
    13    0.06    0.50    0.62    2.11 ^ _10833_/SUM (sky130_fd_sc_hd__ha_1)
                                         _00104_ (net)
                  0.50    0.00    2.11 ^ _05459_/D (sky130_fd_sc_hd__nand4_1)
     5    0.03    0.33    0.37    2.49 v _05459_/Y (sky130_fd_sc_hd__nand4_1)
                                         _01142_ (net)
                  0.33    0.00    2.49 v _08027_/C (sky130_fd_sc_hd__nor4_1)
     1    0.01    0.52    0.56    3.04 ^ _08027_/Y (sky130_fd_sc_hd__nor4_1)
                                         _03108_ (net)
                  0.52    0.00    3.04 ^ _08028_/C1 (sky130_fd_sc_hd__o311ai_0)
     1    0.00    0.19    0.26    3.30 v _08028_/Y (sky130_fd_sc_hd__o311ai_0)
                                         _03109_ (net)
                  0.19    0.00    3.30 v place284/A (sky130_fd_sc_hd__buf_4)
     2    0.02    0.04    0.21    3.51 v place284/X (sky130_fd_sc_hd__buf_4)
                                         net283 (net)
                  0.04    0.00    3.52 v _08206_/A1 (sky130_fd_sc_hd__a311oi_1)
     3    0.02    0.57    0.48    3.99 ^ _08206_/Y (sky130_fd_sc_hd__a311oi_1)
                                         _03283_ (net)
                  0.57    0.00    3.99 ^ _08214_/A2 (sky130_fd_sc_hd__o211ai_1)
     1    0.01    0.19    0.23    4.22 v _08214_/Y (sky130_fd_sc_hd__o211ai_1)
                                         _03291_ (net)
                  0.19    0.00    4.22 v _08215_/C_N (sky130_fd_sc_hd__nor3b_2)
     2    0.01    0.12    0.22    4.44 v _08215_/Y (sky130_fd_sc_hd__nor3b_2)
                                         _03292_ (net)
                  0.12    0.00    4.44 v _08216_/B (sky130_fd_sc_hd__nand2_1)
     1    0.00    0.08    0.09    4.53 ^ _08216_/Y (sky130_fd_sc_hd__nand2_1)
                                         _03293_ (net)
                  0.08    0.00    4.53 ^ _08224_/B1 (sky130_fd_sc_hd__a2bb2o_2)
     2    0.04    0.20    0.29    4.81 ^ _08224_/X (sky130_fd_sc_hd__a2bb2o_2)
                                         _03301_ (net)
                  0.20    0.00    4.82 ^ _09898_/B1_N (sky130_fd_sc_hd__a21boi_0)
     2    0.03    0.82    0.70    5.51 ^ _09898_/Y (sky130_fd_sc_hd__a21boi_0)
                                         _04589_ (net)
                  0.82    0.00    5.52 ^ _10251_/A1 (sky130_fd_sc_hd__o21a_1)
     1    0.00    0.05    0.22    5.74 ^ _10251_/X (sky130_fd_sc_hd__o21a_1)
                                         core.CPU_src2_value_a2[24] (net)
                  0.05    0.00    5.74 ^ core.CPU_src2_value_a3[24]$_DFF_P_/D (sky130_fd_sc_hd__dfxtp_1)
                                  5.74   data arrival time

                         11.00   11.00   clock clk (rise edge)
                          0.00   11.00   clock source latency
     1    0.31    0.00    0.00   11.00 ^ pll/CLK (avsdpll)
                                         CLK (net)
                  0.05    0.03   11.03 ^ clkbuf_0_CLK/A (sky130_fd_sc_hd__clkbuf_16)
     8    0.22    0.23    0.27   11.30 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_0_CLK (net)
                  0.23    0.00   11.30 ^ clkbuf_3_4__f_CLK/A (sky130_fd_sc_hd__clkbuf_16)
    17    0.24    0.25    0.34   11.64 ^ clkbuf_3_4__f_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_3_4__leaf_CLK (net)
                  0.25    0.00   11.64 ^ clkbuf_leaf_91_CLK/A (sky130_fd_sc_hd__clkbuf_16)
    12    0.04    0.06    0.21   11.85 ^ clkbuf_leaf_91_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_leaf_91_CLK (net)
                  0.06    0.00   11.85 ^ core.CPU_src2_value_a3[24]$_DFF_P_/CLK (sky130_fd_sc_hd__dfxtp_1)
                          0.00   11.85   clock reconvergence pessimism
                         -0.05   11.80   library setup time
                                 11.80   data required time
-----------------------------------------------------------------------------
                                 11.80   data required time
                                 -5.74   data arrival time
-----------------------------------------------------------------------------
                                  6.06   slack (MET)



==========================================================================
cts final report_checks -unconstrained
--------------------------------------------------------------------------
Startpoint: core.CPU_src2_value_a3[13]$_DFF_P_
            (rising edge-triggered flip-flop clocked by clk)
Endpoint: core.CPU_src2_value_a3[24]$_DFF_P_
          (rising edge-triggered flip-flop clocked by clk)
Path Group: clk
Path Type: max

Fanout     Cap    Slew   Delay    Time   Description
-----------------------------------------------------------------------------
                          0.00    0.00   clock clk (rise edge)
                          0.00    0.00   clock source latency
     1    0.31    0.00    0.00    0.00 ^ pll/CLK (avsdpll)
                                         CLK (net)
                  0.05    0.03    0.03 ^ clkbuf_0_CLK/A (sky130_fd_sc_hd__clkbuf_16)
     8    0.22    0.23    0.27    0.30 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_0_CLK (net)
                  0.23    0.00    0.30 ^ clkbuf_3_6__f_CLK/A (sky130_fd_sc_hd__clkbuf_16)
    11    0.18    0.19    0.30    0.60 ^ clkbuf_3_6__f_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_3_6__leaf_CLK (net)
                  0.19    0.00    0.60 ^ clkbuf_leaf_70_CLK/A (sky130_fd_sc_hd__clkbuf_16)
    10    0.04    0.06    0.19    0.80 ^ clkbuf_leaf_70_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_leaf_70_CLK (net)
                  0.06    0.00    0.80 ^ core.CPU_src2_value_a3[13]$_DFF_P_/CLK (sky130_fd_sc_hd__dfxtp_1)
     3    0.05    0.46    0.60    1.40 ^ core.CPU_src2_value_a3[13]$_DFF_P_/Q (sky130_fd_sc_hd__dfxtp_1)
                                         core.CPU_src2_value_a3[13] (net)
                  0.46    0.00    1.40 ^ _05535_/A (sky130_fd_sc_hd__inv_1)
     1    0.00    0.09    0.09    1.49 v _05535_/Y (sky130_fd_sc_hd__inv_1)
                                         _00102_ (net)
                  0.09    0.00    1.49 v _10833_/B (sky130_fd_sc_hd__ha_1)
    13    0.06    0.50    0.62    2.11 ^ _10833_/SUM (sky130_fd_sc_hd__ha_1)
                                         _00104_ (net)
                  0.50    0.00    2.11 ^ _05459_/D (sky130_fd_sc_hd__nand4_1)
     5    0.03    0.33    0.37    2.49 v _05459_/Y (sky130_fd_sc_hd__nand4_1)
                                         _01142_ (net)
                  0.33    0.00    2.49 v _08027_/C (sky130_fd_sc_hd__nor4_1)
     1    0.01    0.52    0.56    3.04 ^ _08027_/Y (sky130_fd_sc_hd__nor4_1)
                                         _03108_ (net)
                  0.52    0.00    3.04 ^ _08028_/C1 (sky130_fd_sc_hd__o311ai_0)
     1    0.00    0.19    0.26    3.30 v _08028_/Y (sky130_fd_sc_hd__o311ai_0)
                                         _03109_ (net)
                  0.19    0.00    3.30 v place284/A (sky130_fd_sc_hd__buf_4)
     2    0.02    0.04    0.21    3.51 v place284/X (sky130_fd_sc_hd__buf_4)
                                         net283 (net)
                  0.04    0.00    3.52 v _08206_/A1 (sky130_fd_sc_hd__a311oi_1)
     3    0.02    0.57    0.48    3.99 ^ _08206_/Y (sky130_fd_sc_hd__a311oi_1)
                                         _03283_ (net)
                  0.57    0.00    3.99 ^ _08214_/A2 (sky130_fd_sc_hd__o211ai_1)
     1    0.01    0.19    0.23    4.22 v _08214_/Y (sky130_fd_sc_hd__o211ai_1)
                                         _03291_ (net)
                  0.19    0.00    4.22 v _08215_/C_N (sky130_fd_sc_hd__nor3b_2)
     2    0.01    0.12    0.22    4.44 v _08215_/Y (sky130_fd_sc_hd__nor3b_2)
                                         _03292_ (net)
                  0.12    0.00    4.44 v _08216_/B (sky130_fd_sc_hd__nand2_1)
     1    0.00    0.08    0.09    4.53 ^ _08216_/Y (sky130_fd_sc_hd__nand2_1)
                                         _03293_ (net)
                  0.08    0.00    4.53 ^ _08224_/B1 (sky130_fd_sc_hd__a2bb2o_2)
     2    0.04    0.20    0.29    4.81 ^ _08224_/X (sky130_fd_sc_hd__a2bb2o_2)
                                         _03301_ (net)
                  0.20    0.00    4.82 ^ _09898_/B1_N (sky130_fd_sc_hd__a21boi_0)
     2    0.03    0.82    0.70    5.51 ^ _09898_/Y (sky130_fd_sc_hd__a21boi_0)
                                         _04589_ (net)
                  0.82    0.00    5.52 ^ _10251_/A1 (sky130_fd_sc_hd__o21a_1)
     1    0.00    0.05    0.22    5.74 ^ _10251_/X (sky130_fd_sc_hd__o21a_1)
                                         core.CPU_src2_value_a2[24] (net)
                  0.05    0.00    5.74 ^ core.CPU_src2_value_a3[24]$_DFF_P_/D (sky130_fd_sc_hd__dfxtp_1)
                                  5.74   data arrival time

                         11.00   11.00   clock clk (rise edge)
                          0.00   11.00   clock source latency
     1    0.31    0.00    0.00   11.00 ^ pll/CLK (avsdpll)
                                         CLK (net)
                  0.05    0.03   11.03 ^ clkbuf_0_CLK/A (sky130_fd_sc_hd__clkbuf_16)
     8    0.22    0.23    0.27   11.30 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_0_CLK (net)
                  0.23    0.00   11.30 ^ clkbuf_3_4__f_CLK/A (sky130_fd_sc_hd__clkbuf_16)
    17    0.24    0.25    0.34   11.64 ^ clkbuf_3_4__f_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_3_4__leaf_CLK (net)
                  0.25    0.00   11.64 ^ clkbuf_leaf_91_CLK/A (sky130_fd_sc_hd__clkbuf_16)
    12    0.04    0.06    0.21   11.85 ^ clkbuf_leaf_91_CLK/X (sky130_fd_sc_hd__clkbuf_16)
                                         clknet_leaf_91_CLK (net)
                  0.06    0.00   11.85 ^ core.CPU_src2_value_a3[24]$_DFF_P_/CLK (sky130_fd_sc_hd__dfxtp_1)
                          0.00   11.85   clock reconvergence pessimism
                         -0.05   11.80   library setup time
                                 11.80   data required time
-----------------------------------------------------------------------------
                                 11.80   data required time
                                 -5.74   data arrival time
-----------------------------------------------------------------------------
                                  6.06   slack (MET)



==========================================================================
cts final report_check_types -max_slew -max_cap -max_fanout -violators
--------------------------------------------------------------------------

==========================================================================
cts final max_slew_check_slack
--------------------------------------------------------------------------
0.21897073090076447

==========================================================================
cts final max_slew_check_limit
--------------------------------------------------------------------------
1.4979510307312012

==========================================================================
cts final max_slew_check_slack_limit
--------------------------------------------------------------------------
0.1462

==========================================================================
cts final max_fanout_check_slack
--------------------------------------------------------------------------
1.0000000150474662e+30

==========================================================================
cts final max_fanout_check_limit
--------------------------------------------------------------------------
1.0000000150474662e+30

==========================================================================
cts final max_capacitance_check_slack
--------------------------------------------------------------------------
0.01424864586442709

==========================================================================
cts final max_capacitance_check_limit
--------------------------------------------------------------------------
0.021067000925540924

==========================================================================
cts final max_capacitance_check_slack_limit
--------------------------------------------------------------------------
0.6763

==========================================================================
cts final max_slew_violation_count
--------------------------------------------------------------------------
max slew violation count 0

==========================================================================
cts final max_fanout_violation_count
--------------------------------------------------------------------------
max fanout violation count 0

==========================================================================
cts final max_cap_violation_count
--------------------------------------------------------------------------
max cap violation count 0

==========================================================================
cts final setup_violation_count
--------------------------------------------------------------------------
setup violation count 0

==========================================================================
cts final hold_violation_count
--------------------------------------------------------------------------
hold violation count 0

==========================================================================
cts final report_checks -path_delay max reg to reg
--------------------------------------------------------------------------
Startpoint: core.CPU_src2_value_a3[13]$_DFF_P_
            (rising edge-triggered flip-flop clocked by clk)
Endpoint: core.CPU_src2_value_a3[24]$_DFF_P_
          (rising edge-triggered flip-flop clocked by clk)
Path Group: clk
Path Type: max

  Delay    Time   Description
---------------------------------------------------------
   0.00    0.00   clock clk (rise edge)
   0.00    0.00   clock source latency
   0.00    0.00 ^ pll/CLK (avsdpll)
   0.30    0.30 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
   0.31    0.60 ^ clkbuf_3_6__f_CLK/X (sky130_fd_sc_hd__clkbuf_16)
   0.20    0.80 ^ clkbuf_leaf_70_CLK/X (sky130_fd_sc_hd__clkbuf_16)
   0.00    0.80 ^ core.CPU_src2_value_a3[13]$_DFF_P_/CLK (sky130_fd_sc_hd__dfxtp_1)
   0.60    1.40 ^ core.CPU_src2_value_a3[13]$_DFF_P_/Q (sky130_fd_sc_hd__dfxtp_1)
   0.09    1.49 v _05535_/Y (sky130_fd_sc_hd__inv_1)
   0.62    2.11 ^ _10833_/SUM (sky130_fd_sc_hd__ha_1)
   0.37    2.49 v _05459_/Y (sky130_fd_sc_hd__nand4_1)
   0.56    3.04 ^ _08027_/Y (sky130_fd_sc_hd__nor4_1)
   0.26    3.30 v _08028_/Y (sky130_fd_sc_hd__o311ai_0)
   0.21    3.51 v place284/X (sky130_fd_sc_hd__buf_4)
   0.48    3.99 ^ _08206_/Y (sky130_fd_sc_hd__a311oi_1)
   0.23    4.22 v _08214_/Y (sky130_fd_sc_hd__o211ai_1)
   0.22    4.44 v _08215_/Y (sky130_fd_sc_hd__nor3b_2)
   0.09    4.53 ^ _08216_/Y (sky130_fd_sc_hd__nand2_1)
   0.29    4.81 ^ _08224_/X (sky130_fd_sc_hd__a2bb2o_2)
   0.70    5.51 ^ _09898_/Y (sky130_fd_sc_hd__a21boi_0)
   0.22    5.74 ^ _10251_/X (sky130_fd_sc_hd__o21a_1)
   0.00    5.74 ^ core.CPU_src2_value_a3[24]$_DFF_P_/D (sky130_fd_sc_hd__dfxtp_1)
           5.74   data arrival time

  11.00   11.00   clock clk (rise edge)
   0.00   11.00   clock source latency
   0.00   11.00 ^ pll/CLK (avsdpll)
   0.30   11.30 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
   0.34   11.64 ^ clkbuf_3_4__f_CLK/X (sky130_fd_sc_hd__clkbuf_16)
   0.21   11.85 ^ clkbuf_leaf_91_CLK/X (sky130_fd_sc_hd__clkbuf_16)
   0.00   11.85 ^ core.CPU_src2_value_a3[24]$_DFF_P_/CLK (sky130_fd_sc_hd__dfxtp_1)
   0.00   11.85   clock reconvergence pessimism
  -0.05   11.80   library setup time
          11.80   data required time
---------------------------------------------------------
          11.80   data required time
          -5.74   data arrival time
---------------------------------------------------------
           6.06   slack (MET)



==========================================================================
cts final report_checks -path_delay min reg to reg
--------------------------------------------------------------------------
Startpoint: core.CPU_inc_pc_a2[3]$_DFF_P_
            (rising edge-triggered flip-flop clocked by clk)
Endpoint: core.CPU_inc_pc_a3[3]$_DFF_P_
          (rising edge-triggered flip-flop clocked by clk)
Path Group: clk
Path Type: min

  Delay    Time   Description
---------------------------------------------------------
   0.00    0.00   clock clk (rise edge)
   0.00    0.00   clock source latency
   0.00    0.00 ^ pll/CLK (avsdpll)
   0.30    0.30 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
   0.33    0.62 ^ clkbuf_3_0__f_CLK/X (sky130_fd_sc_hd__clkbuf_16)
   0.20    0.82 ^ clkbuf_leaf_3_CLK/X (sky130_fd_sc_hd__clkbuf_16)
   0.00    0.83 ^ core.CPU_inc_pc_a2[3]$_DFF_P_/CLK (sky130_fd_sc_hd__dfxtp_1)
   0.30    1.13 ^ core.CPU_inc_pc_a2[3]$_DFF_P_/Q (sky130_fd_sc_hd__dfxtp_1)
   0.00    1.13 ^ core.CPU_inc_pc_a3[3]$_DFF_P_/D (sky130_fd_sc_hd__dfxtp_1)
           1.13   data arrival time

   0.00    0.00   clock clk (rise edge)
   0.00    0.00   clock source latency
   0.00    0.00 ^ pll/CLK (avsdpll)
   0.30    0.30 ^ clkbuf_0_CLK/X (sky130_fd_sc_hd__clkbuf_16)
   0.33    0.62 ^ clkbuf_3_0__f_CLK/X (sky130_fd_sc_hd__clkbuf_16)
   0.22    0.84 ^ clkbuf_leaf_110_CLK/X (sky130_fd_sc_hd__clkbuf_16)
   0.00    0.84 ^ core.CPU_inc_pc_a3[3]$_DFF_P_/CLK (sky130_fd_sc_hd__dfxtp_1)
   0.00    0.84   clock reconvergence pessimism
  -0.03    0.82   library hold time
           0.82   data required time
---------------------------------------------------------
           0.82   data required time
          -1.13   data arrival time
---------------------------------------------------------
           0.31   slack (MET)



==========================================================================
cts final critical path target clock latency max path
--------------------------------------------------------------------------
0

==========================================================================
cts final critical path target clock latency min path
--------------------------------------------------------------------------
0

==========================================================================
cts final critical path source clock latency min path
--------------------------------------------------------------------------
0

==========================================================================
cts final critical path delay
--------------------------------------------------------------------------
5.7372

==========================================================================
cts final critical path slack
--------------------------------------------------------------------------
6.0609

==========================================================================
cts final slack div critical path delay
--------------------------------------------------------------------------
105.642125

==========================================================================
cts final report_power
--------------------------------------------------------------------------
Group                  Internal  Switching    Leakage      Total
                          Power      Power      Power      Power (Watts)
----------------------------------------------------------------
Sequential             4.37e-03   4.41e-04   9.26e-09   4.81e-03  40.3%
Combinational          8.42e-04   1.89e-03   9.76e-09   2.74e-03  22.9%
Clock                  2.50e-03   1.90e-03   2.05e-09   4.40e-03  36.8%
Macro                  0.00e+00   0.00e+00   0.00e+00   0.00e+00   0.0%
Pad                    0.00e+00   0.00e+00   0.00e+00   0.00e+00   0.0%
----------------------------------------------------------------
Total                  7.71e-03   4.23e-03   2.11e-08   1.19e-02 100.0%
                          64.6%      35.4%       0.0%
```
</details>


### `run routing`

```shell
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk route
```

![Alt Text](Screenshots/rt1.png)
![Alt Text](Screenshots/rt2.png)

**Routing Result (GUI)**

```shell
make DESIGN_CONFIG=./designs/sky130hd/vsdbabysoc/config.mk gui_route
```
This image shows the **post-routing stage**. The highlighted net `OUT` is fully routed, with its details such as signal type, wire type, and bounding box displayed in the Inspector.

![Alt Text](Screenshots/rt3.png)

![Alt Text](Screenshots/rt4.png)

This image shows the **Routing Congestion Heatmap** after the routing stage. Areas with higher congestion are highlighted in **red**, while green regions indicate lower congestion. The highlighted net `_01595_` is fully routed, and its properties such as bounding box and connectivity details are shown in the Inspector.

![Alt Text](Screenshots/rt5.png)

#### <ins>Formula for Congestion:</ins>

The routing congestion percentage is calculated as:

**Congestion (%) = (Used Routing Tracks ÷ Available Routing Tracks) × 100**

Where:
- **Used Routing Tracks** = Number of tracks occupied by wires in a specific region.
- **Available Routing Tracks** = Total routing capacity of that region.

![Alt Text](Screenshots/rt6.png)

### 🔄 Convert `.odb` to `.def` in OpenROAD

Follow the steps below to export a DEF file from an existing OpenDB (`.odb`) database.

```shell
cd ~/OpenROAD-flow-scripts
source env.sh
cd flow
openroad
# Load the .odb database file
read_db /home/veeraragavan/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/5_route.odb
# Write out the DEF file
write_def /home/veeraragavan/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/5_route.def
```
![Alt Text](Screenshots/odb2def1.png)

```shell
gvim /home/veeraragavan/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/5_route.def
```
![Alt Text](Screenshots/odb2def2.png)

### `VSDBabySoC post_route SPEF generation`

This section covers the step-by-step procedure to generate the **post-route Standard Parasitic Exchange Format (SPEF)** and **post-placement Verilog netlist** for the `VSDBabySoC` design using OpenROAD. These outputs are essential for accurate timing analysis and signoff after the routing stage. The SPEF file captures parasitic RC effects from the physical layout, while the updated Verilog reflects the final net connections post-placement and routing.

### `Step 1: Launch OpenROAD`

Before starting OpenROAD, set up the environment and navigate to the flow directory:

```bash
cd ~/OpenROAD-flow-scripts
source env.sh
cd flow/
openroad
```

![Alt Text](Screenshots/openroad.png)

### `Step 2: Load Design and Technology Files`

Once inside the OpenROAD shell, run the following commands in sequence to load the required design and technology data for VSDBabySoC:

These files describe the physical dimensions and metal/via layers for standard cells and macros:
```shell
read_lef /home/veeraragavan/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/lef/sky130hd.lef
read_lef /home/veeraragavan/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/lef/avsdpll.lef
read_lef /home/veeraragavan/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/lef/avsddac.lef
```

This file contains timing and power data for the standard cells:
```shell
read_liberty /home/veeraragavan/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/lib/avsddac.lef
```

The DEF file represents the post-route physical layout of the design:
```shell
read_def /home/veeraragavan/OpenROAD-flow-scripts/flow/results/sky130hd/vsdbabysoc/base/5_2_route.def
```

![Alt Text](Screenshots/loadtech.png)

### `Step 3: RC Extraction and Output Generation`

After loading the LEF, Liberty, and DEF files, run the following commands to define the process corner and extract parasitics using the available `.calibre`-format model:

#### 🔹 1. Define Process Corner
Set the process corner using the available Calibre-based extraction rules file:

```tcl
define_process_corner -ext_model_index 0 /home/veeraragavan/OpenROAD-flow-scripts/external-resources/open_pdks/sky130/openlane/rules.openrcx.sky130A.nom.calibre
```

#### 🔹 2. Extract Parasitics

Run parasitic extraction using the same file:

```shell
extract_parasitics -ext_model_file /home/veeraragavan/OpenROAD-flow-scripts/external-resources/open_pdks/sky130/openlane/rules.openrcx.sky130A.nom.calibre
```

#### 🔹 3. Write SPEF File
Save the extracted parasitics:

```shell
write_spef /home/veeraragavan/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/vsdbabysoc.spef
```

#### 🔹 4. Write Post-Placement Verilog Netlist
Save the netlist after placement and routing:

```shell
write_verilog /home/veeraragavan/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/vsdbabysoc_post_place.v
```

![Alt Text](Screenshots/spef1.png)

The Standard Parasitic Exchange Format (SPEF) file captures the resistance and capacitance (RC) parasitics of interconnects extracted from the routed layout. This file is essential for accurate post-route static timing analysis (STA) as it models real-world wire delays caused by metal layers and vias. Tools like OpenSTA read the SPEF file to compute timing paths that reflect true physical behavior after routing. Generating and inspecting the SPEF ensures that your design is signoff-ready with precise timing estimates.

```shell
gvim /home/veeraragavan/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/vsdbabysoc.spef
```

![Alt Text](Screenshots/spef2.png)

The post-placement Verilog netlist represents the logical connectivity of the design after placement and routing have been completed. This version of the netlist includes any modifications made by optimization or physical synthesis during the backend flow and ensures consistency with the final layout. It is used in downstream verification flows and enables correlation between logical simulation and physical implementation. Writing this netlist is crucial for timing closure and for validating the final connectivity of the design.

```shell
gvim /home/veeraragavan/OpenROAD-flow-scripts/flow/designs/sky130hd/vsdbabysoc/vsdbabysoc_post_place.v
```

![Alt Text](Screenshots/v1.png)

