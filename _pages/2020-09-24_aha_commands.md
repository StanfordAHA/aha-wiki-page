---
title: AHA Commands
author: XXX
date: 2022-09-24
layout: post
---


There are so many github repositories under AHA project. Let's have a quick overview of each correspondence. 

## AHA Command Template

    aha [-h] [-v] [-d] [--dir AHA_DIR] <command>
    
We use `aha <command>` to run most of the task. We can use this command to generate verilog, compile applications, mapping for example. The `<command>` task can be chosen from **'config', 'deps', 'docker', 'garnet', 'glb', 'halide', 'map', 'my_regress', 'pd', 'pipeline', 'regress', 'report', 'sta',** and  **'test'**. Each cooresponds to each python file under `/aha/aha/util/` folder. 


### Regression
See [Design Flow](03_design_flow.md) doc. Four commonly used commands are `aha garnet`, `aha halide`, `aha pipeline`, and `aha glb`. All of them are wrapped under `aha regress`. Thus, if we want to type each command manually, please refer to **`/aha/aha/util/regress.py`** to include every flag needed. 


### Garnet
The `aha garnet` command specified in `/aha/aha/util/garnet.py` would call `/aha/garnet/garnet.py`. This command is used to implement the CGRA using our generator infrastructure and generate the verilog code. 
The main function is called `create_cgra()` specified in `/aha/garnet/cgra/util.py`. The file import modules from Peak, Lake and Canal.


### Halide
The `aha halide` command is specified in `/aha/aha/util/halide.py`. It would find appications under `Halide-to-Hardware/apps/hardware_benchmarks/apps/` and store the map result under `./bin/map_result/` folder in the application folder. 


### Pipeline
TBD

### Glb
The `aha glb` command specified in `/aha/aha/util/glb.py` runs a SystemVerilog VCS simulation of a kernel. This simulates data movement into the CGRA, kernel processing, and gold checking. It has the optional `--waveform` flag to dump an fsdb. 

### Sta
The `aha sta` command specified in `aha/aha/util/sta.py` would call `/aha/archipelago/visualize.py`. This command is used to generate log file and visualization of the final CGRA. 
