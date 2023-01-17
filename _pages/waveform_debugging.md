---
title: Waveform Debugging Tips
author: XXX
date: 09
layout: post
---

Waveform Debugging Tips

### PE tile important signals

### MEM tile important signals
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



### GLB important signals

Navigate to `/top/dut/global_buffer_W_inst0/global_buffer and select input/output siganls only`

**cgra_cfg_** signals are cgra configuration

**if_cfg_** signals are glb configuration 

**proc_** signals are read/writes to glb memories from the processor

**strm_g2f** and **strm_f2g** signals include data between glb and cgra, interrupt pulses, and flush

![GLB Waveform](/aha-wiki-page/assets/glb_waveform.png)
