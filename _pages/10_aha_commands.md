---
title: AHA Commands
author: XXX
date: 2022-01-10 (Date used for order should be 2022-01-XX.)
layout: post
---

# AHA Command Template

    aha [-h] [-v] [-d] [--dir AHA_DIR] <command>
    
We can use this command to generate verilog, compile applications, and test those applications. The `<command>` options include **'config', 'deps', 'docker', 'garnet', 'glb', 'halide', 'map', 'my_regress', 'pd', 'pipeline', 'regress', 'report', 'sta',** and **'test'**. Each cooresponds to a python file under `aha/util/` folder. 


## Regression
See [Design Flow](03_design_flow.md) doc. Four commonly used commands are `aha garnet`, `aha halide`, `aha pipeline`, and `aha glb`. All of these commands are run in sequence for a selection of applications using `aha regress`. 


## Garnet
The `aha garnet` command specified in `aha/util/garnet.py` and calls `garnet/garnet.py`. This command is used to generate the verilog for the CGRA array. 
The main function is `create_cgra()` specified in `garnet/cgra/util.py`. The file import modules from Peak, Lake, and Canal.


## Halide
The `aha halide` command is specified in `aha/util/halide.py`. It will find appications under `Halide-to-Hardware/apps/hardware_benchmarks/apps/` and store the map result for an `app` under `Halide-to-Hardware/apps/hardware_benchmarks/apps/{app}/bin/`. 

## Pipeline
The `aha pipeline` command specified in `aha/util/pipeline.py` runs pipelining, place and route, and bitstream generation. It also runs `aha/util/garnet.py`, but with options to compile the application, not generate the verilog for the CGRA. It produces design files and a bitstream located in `Halide-to-Hardware/apps/hardware_benchmarks/apps/{app}/bin/`.

## Glb
The `aha glb` command specified in `aha/util/glb.py` runs a SystemVerilog VCS simulation of an application on the CGRA. This simulates data movement into the CGRA, kernel processing, and checking against a gold model. It has the optional `--waveform` flag to dump an fsdb file. 

## Sta
The `aha sta` command specified in `aha/util/sta.py` calls `archipelago/sta.py`. This command is used to determine the maximum clock frequency of an application running on the CGRA. The `--visualize or -v` flag also generates a visualization of the application place and route result. It will produce png images of this visualization in `Halide-to-Hardware/apps/hardware_benchmarks/apps/{app}/bin/pnr_result.png`.
