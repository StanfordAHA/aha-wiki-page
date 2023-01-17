---
title: CGRA Design Flow
author: XXX
date: 2022-09-01
layout: post
---

In order to update the design, we need to update **both the CGRA-based hardware accelerator and the compiler**. The main feature of AHA project is the **co-design** of accelerators and compilers, where the compiler updates automatically as the accelerator evolves.


Accelerator
-------------
CGRAs consist of three types of modules: **PEs**, **memories**, and **interconnect**. Accordingly, three high-level **domain-specific hardware specification languages (DSLs)** is used to represent formal specification for each component. Each DSL would be used to generate both the RTL code for accelerators and the collateral rewrite rules for compilers.

- PEak for PEs
- Lake for memories
- Canal for interconnects

All of these DSL programs are based on a low-level DSL called **magma** which is embedded in Python. 

> ##### Note
> 
> In other words, we **use Python instead of verilog** to design our circuit. The process has already been packed, so we can use one command to set up all these, and generate the verilog we need.
{: .block-tip }


### Usage
Inside the Docker container, we generate **verilog of the CGRA** using the following command. The `width` and `height` flag represent **the size of the array** we want to generate. Although the size might be flexible for different applications, it's still better to generate a larger array so that we can have more tiles to use. Note that every flag needs to be included in the command.

    aha garnet --width 32 --height 16 --verilog --use_sim_sram --rv --sparse-cgra --sparse-cgra-combined
    
    for test:
        aha garnet --width 32 --height 16 --verilog --rv --sparse-cgra --sparse-cgra-combined

After running the command, it would save verilog file in `/aha/garnet/garnet.v`.


Compiler
-------------
Since now we have everything prepared, we want to use the compiler to **lower an application onto the CGRA hardware**.

In most of the experiments, we target applications in **dense linear algebra applications** domain. 

> ##### Note
> 
> Comparing to FPGA which are designed to handle applications in several different domains, 
> CGRA can target to accelerate only one domain of applications.
{: .block-tip }

The application is written in high-level DSL called **Halide**, which is embedded in C++. The Halide application would go through the scheduling phase and output a dataflow graph of logical operations in CoreIR format.


### Usage 
Inside the Docker container, we use Gaussian application as example. 

In `/aha/aha/util/regress.py` we can see there are different setting suggestion for different applications. For example, the suggested `HALIDE_GEN_ARGS` is `"mywidth=62 myunroll=2 schedule=3"` for gaussian application. We need to **set HALIDE_GEN_ARGS before compiling the apps**. 

    cd /aha/Halide-to-Hardware/apps/hardware_benchmarks/apps/gaussian
    export HALIDE_GEN_ARGS="mywidth=62 myunroll=2 schedule=3" 
    aha halide apps/gaussian

After running the command, it would save CoreIR file in `./bin/design_top.json`. 


### Mapping 
Now we have both verilog file and CoreIR file. Then the CoreIR graph would be mapped, placed, and routed onto physical hardware units to produce CGRA bitstream. The CGRA bitstream acts just the same as FPGA bitstream, which contains configuration data and can be used to load the design onto the hardware.

We can use the following command to map the CoreIR graph onto the CGRA. The `width` and `height` flag specify what portion of the array we like to map to. The size must be smaller than the original setting of the CGRA verilog. We need to **set DISABLE_GP before compiling the apps**.

    export PIPELINED=1   // we can still map without this variable. But the frequency would be very low.
    export DISABLE_GP=1
    aha pipeline apps/gaussian --width 32 --height 16 --input-broadcast-branch-factor 2 --input-broadcast-max-leaves 32 --rv --sparse-cgra --sparse-cgra-combined

After running the command, the bitstream file would be saved in `./bin/gaussian.bs`. 

The placement file would be saved in `./bin/design.place`. We can see the x and y coordinates and the functions of each tile. The connection edges among each tile for the application is in `./bin/design.packed`. And `./bin/design.route` would show the actual track and `./bin/design.freq` wold show the frequency in MHz unit.


### Testing 
Then we can run glb test, if the thing goes well, it should show `glb mapping success` as well as the simulation time, cpu time and the data structure size. 

    module load base vcs verdi
    aha glb apps/gaussian (optionally use --waveform to dump fsdb)


To generate the log file about critical path, we can use `aha sta` command. The log file would be saved at `/aha/Halide-to-Hardware/apps/hardware_benchmarks/apps/gaussian/log/aha_sta.log`.

    aha sta apps/<app name> --log


### Visualization
The `aha sta` command would call `/aha/archipelago/visualize.py`. It would produce `./bin/pnr_result_{width}.png` for visualization. See `aha/aha/util/sta.py` for more details. 

    python3 -m pip install --upgrade pip
    python3 -m pip install --upgrade Pillow
    aha sta apps/<app name> --visualize








