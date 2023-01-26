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

## Hardware Generation
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

## aha regress
When testing architectural or compiler changes, its often useful to automate the compilation and testing of many applications. `aha regress` is the command that we use for this purpose. In this step, it will iterate through a list of applications and run `aha map`, `aha pnr`, and `aha test`. There are many regression application suites, each with different uses. 


- `aha regress fast` will run the smallest sparse and smallest dense image processing application and is intended to run in minutes.
- `aha regress daily` tests more complex applications including gaussian, harris, camera pipeline, unsharp, and resnet. It generally takes hours.
- `aha regress full` tests every application and takes several hours to complete.


To see a list of all regress suites and the applications they include, see `aha/util/regress.py`.

## Environmental Variables
To modify the compilation, scheduling, place-and-route, and pipelining of our applications, we use environmental variables. Here is a brief description of the important variables that we use:

- `HALIDE_GEN_ARGS` determines scheduling parameters used in the `{app}_generator.cpp` described here: https://stanfordaha.github.io/aha-wiki-page/pages/03_h2h_files. A valid assignment to this variable is a space separated list of `GeneratorParam` listed at the top of `{app}_generator.cpp`. For example if you want to run gaussian you could set `HALIDE_GEN_ARGS="mywidth=62 myunroll=2 schedule=3`.
- `PIPELINED` determines whether or not compute pipelining is turned on. Valid values are `PIPELINED=1` (default) compute pipelining turned on or `PIPELINED=0` compute pipelining turned off
- `DISABLE_GP` determines whether or not the global placement stage of place-and-route is done. `DISABLE_GP=1` (default) turns off global placement, `DISABLE_GP=0` turns on global placement
- `HL_TARGET` has two valid values `HL_TARGET=host-x86-64` or `HL_TARGET=host-x86-64-enable_ponds`. Use `HL_TARGET=host-x86-64-enable_ponds` do enable the memory mapper to use the register files (ponds) present in our PE tiles. By default, this is set to `HL_TARGET=host-x86-64` for image processing applications and `HL_TARGET=host-x86-64-enable_ponds` for machine learning applications.
- `PNR_PLACER_EXP` is a tuning knob for the placement stage of place-and-route. Generally, a higher value results in an application placement and routing that has a shorter critical path and may reduce the runtime of applications running on the array. By default, this variable is not set, and the place-and-route tool will choose the first value that routes successfully. 
- `SWEEP_PNR_PLACER_EXP` will tell the place-and-route tool to try every value of `PNR_PLACER_EXP` from 1 to 30 and choose the result with the shortest critical path. By default it is not set.



To avoid needing to set each of these environmental variables each time we run an application, we have included default values in `aha/util/application_parameters.json`. If you run any AHA flow command, it will use the `default` entry in the json file for the application you are compiling. If you would like to run the fastest (shortest critical path) version of the application you can run:
`aha halide apps/gaussian --env-parameters fastest`

Some machine learning applications, including ResNet, have significantly different schedules depending on the layer in the network. In the AHA flow, we use one application with different environmental parameters to choose which layer and which schedule to use. Default parameters for each layer are saved in `aha/util/application_parameters.json` and can be selected using the `--layer` flag. For example:

`aha map apps/resnet_output_stationary --layer conv1`

`aha pnr apps/resnet_output_stationary --width 32 --height 16 --layer conv1`
