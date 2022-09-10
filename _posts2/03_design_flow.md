---
title: CGRA Design Flow
author: XXX
date: 2022-09-09
layout: post
---

In order to update the design, we need to update both the CGRA-based hardware accelerator and the compiler. The main feature of AHA project is the co-design of accelerators and compilers, where the compiler updates automatically as the accelerator evolves.


Accelerator
-------------
CGRAs consist of three types of modules: PEs, memories, and interconnect. Accordingly, three high-level domain-specific hardware specification languages (DSLs) is used to represent formal specification for each component. Each DSL would be used to generate both the RTL code for accelerators and the collateral rewrite rules for compilers.

- PEak for PEs
- Lake for memories
- Canal for interconnects

All of these DSL programs are based on a low-level DSL called magma which is embedded in Python. 

> ##### Note
> 
> In other words, we use Python instead of verilog to design our circuit. The process has already been packed, so we can use one command to set up all these, and generate the verilog we need.
{: .block-tip }


Compiler
-------------
Since now we have everything prepared, we want to use the compiler to lower an application onto the CGRA hardware.

In most of the experiments, we target applications in dense linear algebra applications domain. 

> ##### Note
> 
> Comparing to FPGA which are designed to handle applications in several different domains, 
> CGRA can target to accelerate only one domain of applications.
{: .block-tip }

The application is written in high-level DSL called Halide, which is embedded in C++. The Halide application would go through the scheduling phase and output a dataflow graph of logical operations in CoreIR format.

Then the CoreIR graph would be mapped, placed, and routed onto physical hardware units to produce CGRA bitstream. 

> ##### WARNING
>
> The size in mapping command must fit the setting of our chip.
{: .block-warning }

The CGRA bitstream acts just the same as FPGA bitstream, which contains configuration data and can be used to load the design onto the hardware.
