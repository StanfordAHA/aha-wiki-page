---
title: CGRA Design Flow
author: XXX
date: 2022-01-02 (Date used for order should be 2022-01-XX.)
layout: post
---

Our CGRA design flow generates **both the CGRA-based hardware accelerator and the application compiler**. The main feature of AHA project is the **co-design** of accelerators and compilers, where the compiler updates automatically as the accelerator evolves.


# Accelerator
CGRAs generally consist of: **PEs**, **memories**, and an **interconnect**. Accordingly, three high-level **domain-specific hardware specification languages (DSLs)** are used to specify each component. Each DSL is used to generate both the RTL code for the CGRA and the collateral for generating the new compilers.
The three DSLs we use are:
- PEak for PEs
- Lake for memories
- Canal for interconnects

All of these DSL's are embedded in Python, and are based on a low-level hardware description DSL called **magma** which is also embedded in Python. 

## Usage
Inside the Docker container, we generate **verilog of the CGRA** using the following command. The `width` and `height` flag represent **the size of the array** we want to generate. Although the size might be flexible for different applications, it's still better to generate a larger array so that we can have more tiles to use. Note that every flag needs to be included in the command.

    aha garnet --width 32 --height 16 --verilog --use_sim_sram
    
When generating the verilog for physical design, remove the `--use_sim_sram` flag

After running the command, it would create the verilog file `/aha/garnet/garnet.v`.


# Compiler
We will now go through the steps to compile an application onto the CGRA hardware and simulate it.

In most of the experiments, we target applications in **dense linear algebra applications** domain. 

The application is written in high-level DSL called **Halide**, which is embedded in C++. The Halide application would go through a several stages before we can simulate it.

We will use Gaussian as an example to go through the three steps of the compiler: map, pnr, and test.

## aha map 
`aha map apps/gaussian` will first compile the halide application using the Halide to Hardware compiler. It takes in the gaussian_generator.cpp and process.cpp described in https://stanfordaha.github.io/aha-wiki-page/pages/03_h2h_files/. 

Next, `aha map` will use a tool called MetaMapper to map the compute of the application (described in the CoreIR file `bin/gaussian_compute.json`) to PEs. It will produce a CoreIR file called `bin/gaussian_compute_mapped.json`.

Finally, it will use the clockwork tool to schedule the application and map the storage and buffers of the application to memory tiles. This step takes in  `bin/gaussian_compute_mapped.json` and `bin/gaussian_memory.cpp` and produces a CoreIR file called `bin/design_top.json` (described in detail here: https://stanfordaha.github.io/aha-wiki-page/pages/08_design_files/).

## aha pnr 
Now we have both verilog file and fully mapped CoreIR file. The `aha pnr apps/gaussian --width 32 --height 16` command will now place and route the application, perform pipelining, and generate a bitstream used to configure the CGRA to execute your application. 

The `width` and `height` flag specify what portion of the array we like to map to. This can only be as large as the verilog that you generated using `aha garnet` but may be smaller if desired.

After running the command, the bitstream file would be saved in `./bin/gaussian.bs`. Additionally, several design files (described here: https://stanfordaha.github.io/aha-wiki-page/pages/08_design_files/) will be generated as well.

## aha test 
Then we can run `aha test apps/gaussian`, which will run a VCS functional simulation of your application running on your CGRA verilog. On the kiwi server, use `module load base vcs` to load vcs before running `aha test`.

You can optionally generate an fsdb waveform using the `--waveform` flag. This will require Verdi to be loaded: `module load verdi`. 

## aha sta
To run your mapped application through a critical path timing model, we can use `aha sta apps/gaussian`. The maximum frequency that you will be able to run the CGRA array at when running the application will be printed.

The `aha sta` command can also be used to generate a visualization of the critical path of the application using the `--visualize or -v` flag. It will produce `./bin/pnr_result_{width}.png`. See `aha/aha/util/sta.py` for more details. 
