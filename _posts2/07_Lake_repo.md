---
title: Lake repo explain
author: XXX
date: 2022-09-28
layout: post
---

From [Github repo Correspondence](05_repo_correspondence.md), MEM tile is specified in [`./strg_ub_vec.py`](https://github.com/StanfordAHA/lake/blob/f7f2b501e91ac4764e0f94a9247079adf0eb3d99/lake/modules/strg_ub_vec.py). And Lake architecture overview is explained [here](04_Lake.md). When looking into .py file, we can see lots of module names, so let's try to link those together. 

## StrgUBVec

    input  [1:0] [16:0] chain_data_in
    input  [1:0] [16:0] data_in,
    output [1:0] [16:0] data_out
    output [1:0] accessor_output

Every MEM tile has its own cycle counter, `cycle_count` in code, which would be used by SG to decide when READ/WRITE enable signal.
Each MEM tile contains two AGGs, one SRAM and two TBs. We devide them into five partitions. 

    StrgUBVec
    +-- StrgUBAggOnly
    +-- StrgUBAggSRAMShared
    +-- StrgUBSRAMOnly
    +-- StrgUBSRAMTBShared
    +-- StrgUBTBOnly


### StrgUBAggOnly

    StrgUBVec
    +-- StrgUBAggOnly
        +-- AddrGen *2
        +-- SchedGen *2 
        +-- ForLoop *2
        +-- agg *2 (register file) 

`AggOnly` contains two AGGs, each AGG is **a register file of size 2*4**, to aggrate words for 2 input ports. This `AggOnly` modules also contains the affine controller between input ports and the register file. Since there are two input ports, it contains **two AGs**, **two SGs** and **two IDs**, specifying the address and enable signal to READ/WRITE AGG. 

AGG is SIPO interface, so the input would be **one 16-bit data word** for each port. After it collects four 16-bit data words, AGG would **output four 16-bit data word** to SRAM at a time. 

    input  [1:0] [15:0] data_in
    output [1:0] [3:0] [15:0] agg_data_out


The **ID module** is specified in [`./for_loop.py`](https://github.com/StanfordAHA/lake/blob/f7f2b501e91ac4764e0f94a9247079adf0eb3d99/lake/modules/for_loop.py), **AG** is in [`./addr_gen.py`](https://github.com/StanfordAHA/lake/blob/f7f2b501e91ac4764e0f94a9247079adf0eb3d99/lake/modules/addr_gen.py), and **SG** is in [`./spec/sched_gen.py`](https://github.com/StanfordAHA/lake/blob/f7f2b501e91ac4764e0f94a9247079adf0eb3d99/lake/modules/spec/sched_gen.py).


#### AG
AG generate **the READ/WRITE address**. For example, AG under `AggOnly` would calculate which address in AGG to be READ/WRITE. Since AGG is a small memory of size 2*4 data-words, we need 3 bits for its WRITE address (capability: 8 data-words), 1 bit for its READ address (capability: 2 pairs of 4-data-word). But if the AG is used for SRAM, which can store 512 pairs of 4-data-word, it would need 9 bits for both its WRITE/READ address. 

We utilize the recurrence relation of ID and **replace multipliers by an adder, a register, and a multiplexer**. AG gets the input `mux_sel` signal from ID, deciding which stride to take to increment the running address. Lake supports up to **6-D loop nest**, so we would need 3 bits for the MUX and we would have 6 stride choices to choose from.

    input  [8:0] starting_addr 
    input  [5:0] [8:0] strides
    input  [2:0] mux_sel
    output [8:0] addr_out


#### SG
SG generate **the READ/WRITE enable signal**. under the control of ID. SG would use the same AG architecture to calculate **the next cycle schedule**. SG would also gets `cycle_count` input from ID, recording **the current cycle number**. It can then compare the two result, and **generate enable signal (`valid_output`)** when the two results match.  

    input  [15:0] cycle_count
    input  [15:0] sched_addr_gen_starting_addr
    input  [5:0] [15:0] sched_addr_gen_strides
    input  [2:0] mux_sel
    output valid_output


#### ID
ID corresponds to **for-loop**. It would output the **`mux_sel` signal** needed by AG and SG. The `ranges` is the **boundary of each loop**, corresponding to **`extent` attribute** in config file. 

    input  [5:0] [9:0] ranges
    input  [3:0] dimensionality
    output [2:0] mux_sel_out


> ###### Note
> 
> If the `input` was added additional attribute `ConfigRegAttr`, it means that this `input` signal **could be reconfigured** directly by the compiler. This is where *reconfigurable* array comes from.
{: .block-tip }


### StrgUBAggSRAMShared

    StrgUBVec
    +-- StrgUBAggSRAMShared
        +-- AggSramSharedAddrGen *2
        +-- AggSramSharedSchedGen *2

`AggSRAMShared` only contains the affine controller between AGGs and SRAM. It contains **two AGs** and **two SGs**. The SG here is used to generate **READ enable from AGG**, which is also the **WRITE enable for SRAM**. The AG here is used to generate both the **WRITE address for SRAM** and the **READ address for AGG**, SRAM address is 9-bit and we would just take its LSB as the AGG READ address since AGG depth is 2. 

The `agg_read_out` signal is the WRITE enable signal from SRAM, and the `agg_sram_shared_addr_out` signal is the WRITE address for SRAM. 

    output [1:0] agg_read_out
    output [1:0] [8:0] agg_sram_shared_addr_out


#### AggSramSharedSchedGen
Now SG is optimize to 4-to-1, in other words, it collects 4 data-words than pass READ enable signal to SRAM. But what if the range is not multiplicant of 4? 
For example, in `resnet_tiny.json`, the range (the `extent` flag) is 14 now. AGG starts collecting data-word from cycle 0, SG would send out READ enable signal after collecting 4 data-words (`cycle_stride[0] = 4`) on cycle 4 (pass data from cycle 0,1,2,3), 8 (pass data from cycle 4,5,6,7), 12 (pass data from cycle 8,9,10,11). But the data-words collected at cycle 12 and cycle 13 are writen in AGG but haven't pass to SRAM yet. So we would specify `agg_read_padding` flag for `AggSramSharedSchedGen` to start counting at cycle 13 and generate the READ enable signal at cycle 16. 
In the mean time, data-words still coming in for the second loop start with cycle 14. So there would also READ enable on cycle 18 (pass data from cycle 14,15,16,17), 22 (pass data from 18,19,20,21), etc. 
    
    "in2agg_0":{
        "cycle_starting_addr":[0],
        "extent":[14],
        "cycle_stride":[1,14]
        ...

    "agg2sram_0":{
        "agg_read_padding":[3],
        "cycle_starting_addr":[4],
        "cycle_stride":[4,14],
        ...


### StrgUBSRAMOnly

    StrgUBVec
    +-- StrgUBSRAMOnly
        +-- AddrGen *2

`SRAMOnly` only contains the affine controller between SRAM and TBs. It contains **two AGs**, which are used for generate READ address for SRAM. The `t_read` signal is the READ enable signal of SRAM, receive from SG in `SRAMTBShared`.

    input  [1:0] t_read
    output [1:0] [8:0] sram_read_addr_out,


### StrgUBSRAMTBShared

    StrgUBVec
    +-- StrgUBSRAMTBShared
        +-- SchedGen *2 
        +-- ForLoop *2

`SRAMTBShared` also only contains the affine controller between SRAM and TBs. It contains **two SGs** and **two IDs**. SG here is used for generate READ enable from SRAM. The `t_read_out` signal is the **READ enable from SRAM**, and if delay 1 cycle it would be the **WRITE enable for TB**. The `loops_sram2tb_mux_sel` signal is generated from ID. We support 6-D so mux sel signal is 3 bits.

    output [1:0] t_read_out
    output [1:0] [2:0] loops_sram2tb_mux_sel


### StrgUBTBOnly

    StrgUBVec
    +-- StrgUBTBOnly
        +-- AddrGen *4
        +-- SchedGen *2 
        +-- ForLoop *2
        +-- tb *2 (register file) 

`TBOnly` contains two TBs, each TB is  **a register file of size 2*4**, for 2 output ports, act as a transposed buffer between TB and output port. This `TBOnly` modules also contains the affine controller between TBs and output port. It contains **four AGs**, **two SGs** and **two IDs**. Two AGs generate the WRITE address for TB and the other two AGs generate the READ address from TB. SG is used to generate READ enable of TB. 

TB is PISO interface, so the input would be **four 16-bit data word**, and the output would be **one 16-bit data word** for each port. 

    input  [3:0] [15:0] sram_read_data
    output [1:0] [15:0] data_out