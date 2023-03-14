---
title: Waveform Debugging Tips
author: XXX
date: 2022-01-09 (Date used for order should be 2022-01-XX.)
layout: post
---

# PE tile important signals
Navigate to `/top/dut/Interconnect_inst0/Tile_X00_Y01/PE_inst0/PE_inner_W_inst0/PE_inner/mem_ctrl_PE_onyx_flat/PE_onyx_inst/onyxpeintf` (replace the X and Y coordinates with PE you are interested in)

This PE module has the following important inputs:
- **ASYNCRESET and CLK**
- **inst** is the instruction to the PE. It is a concatenation of a bunch of different configuration within the PE, so it is unreadable at this level. If you want to know configurations of PE internals, look at the submodules of the PE.
- **data0[15:0], data1[15:0], data2[15:0]** are the three 16 bit data inputs into the PE. Arithmetic operations will use these data inputs.
- **bit0, bit1, bit2** are the three bit inputs into the PE. Bit operations will use these inputs.
- **O0[15:0]** is the data output, usually from an aritchmetic operation.
- **O1** is the bit output, usually from the condition code generator or the LUT.
- **O2[15:0], O3[15:0], O4[15:0]** are the values stored in the three input registers in the PE. 


# MEM tile important signals
The memory macro is located under `/top/dut/Interconnect_inst0/Tile_X<#>_Y<#>/MemCore_inst0/MemCore_inner_W_inst0/MemCore_inner/
`. You can monitor memory read and write requests sent from the unified buffer.

The unified buffer (UB) is located under `/top/dut/Interconnect_inst0/Tile_X<#>_Y<#>/MemCore_inst0/MemCore_inner_W_inst0/MemCore_inner/mem_ctrl_strg_ub_vec_flat/`
- **chain_data_in_f_0[0:0][16:0]** are the chaining data input for port 0
- **chain_data_in_f_1[0:0][16:0]** are the chaining data input for port 1
- **data_in_f_0[0:0][16:0]** are the UB data input for port 0 (going in to AGG0)
- **data_in_f_0[0:0][16:0]** are the UB data input for port 1 (going in to AGG1)
- **data_out_f_0[0:0][16:0]** are the UB data output for port 0 (going out of TB0)
- **data_out_f_1[0:0][16:0]** are the UB data output for port 1 (going out of TB1)

It further lists all the UB configuration registers (starting_addr, strides, ranges, dim...), such as `strg_ub_vec_inst_agg_only_agg_write_addr_gen_0_starting_addr[2:0]`. You can probe them to see if they match the configurations in `design_top.json`. Note when comparing to  `design_top.json`, `extent` registers in hardware are `-2`. Each `strides` in hardware becomes the relative difference to the next loop dimension (as an optimization), rather than the absolute stride number shown in `design_top.json`.



# GLB important signals

Navigate to `/top/dut/global_buffer_W_inst0/global_buffer and select input/output siganls only`

**cgra_cfg_** signals are cgra configuration

**if_cfg_** signals are glb configuration 

**proc_** signals are read/writes to glb memories from the processor

**strm_g2f** and **strm_f2g** signals include data between glb and cgra, interrupt pulses, and flush

![GLB Waveform](/aha-wiki-page/assets/glb_waveform.png)
