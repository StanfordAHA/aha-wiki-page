---
title: Mem tile
author: XXX
date: 2022-09-21
layout: post
---


# MEM tile

![Image](/assets/aha_wiki_page/mem_tile.PNG)

Figure above is the block diagram of a MEM tile. We use a wide-fetch 
single-port SRAM with three unified buffers inside. Two of them (AGG 
and TB) are small register files and the other is large single-port 
SRAM. 

## AGG
  - Size: 2*4 (each block is 16 bits)
  - SIPO
  - Buffer between input port and SRAM
  - Collect 4 words then pass to SRAM
  
## single-port SRAM
  - Size: 64*512 
  - Since the SRAM is single-port, we can **only do READ or WRITE**, one 
    action at a time. However, there are two AGGs and two TBs share the 
    SRAM port, so we use **time modulation**, each module use 1/4 port time.

> ###### Note
> 
> We have tried dual port SRAM before so that we can have four ports 
> and don't need AGG or TB to control. However, t turns out that 
> single-port SRAM still achieve better energy and area result. 
{: .block-tip }

## TB
  - Size: 2*4 (each block is 16 bits)
  - PISO
  - Buffer between SRAM and output port
  - Transpose the column set of 4 blocks and pass them serially to the 
  output port one at a time


## Affine controller
Each unified buffer (AGG, TB and SRAM) all has their own **affine controller**.
The controller contains three different modules:

### IterationDomain (ID)
ID corresponds to **for loop**, specifying the range of memory operations. We 
support up to maximum **6-D** loop nest. 

The reason why the controller is "affine" is that the access pattern must 
be repeated in certain pattern so that it can be written in the form of 
for-loop.   

For example, we can use `ID: [4,3]` to represent the 2-D for loop below:

    for x in [0,3):
        for y in [0,4):
            read MEM[x][y]

The first `stride_x` corresponds to the number of inner loop and the second `stride_y`
corresponds to the number of outer loop. 

In circuit implementation, ID would pass a **MUX_SEL signal** to AG and SG, specifying 
which dimension of stride should be added on the current value. 

  
### AddressGenerator (AG)
AG generate the **memory address** under the control of ID. 

Using the same for loop example above, assume we have a 3*4 memory, we want 
to iterate each horizontal row sequentially. We would access the four 
entries in the first row from MEM[0][0] to MEM[3][0], then continue to the 
second row from MEM[0][1] to MEM[3][1].

The AG expression would be like:

    offset = 0
    stride = [1,4]

`AG offset` is 0 since we directly start at the first entry MEM[0][0]. The `stride_x`
is the relative increment in the inner loop. While the definition of `stride_y` is different 
than `stride_x`, represent the absolute index where the next outer loop should start from.
At loop 0, the index start at `stride_y*0 + offset = 4*0 + 0 = 0`. And increment 
`stride_x = 1` for `ID_x - 1 = 4 - 1 = 3` times. Then the first inner loop finish and 
the index would jump to `stride_y*0 + offset = 4*1 + 0 = 4`. And increment `stride_x` 
and so on.


### ScheduleGenerator (SG)
SG generate the **cycle time schedule** under the control of ID. SG would 
pass a **READ/WRITE enable** to AG at specific cycle time. 

The compiler performs **static scheduling**. In other words, there is no 
VALID or READY signal. The compiler would computes a **cycle accurate schedule** 
for the controller, explicitly stating when each operation should perform. 
Each tile has one **cycle counter** to trace the cycle number, and a **FLUSH signal** 
to restart the counter.

Using the same for loop example above, assume we want start READ operation at cycle 1, 
the SG expression is similar to AG:

    offset = 1
    stride = [2,10]

`SG offset` is 1 since we the first operation occurs at `stride_y*0 + offset = 10*0 + 1 = cycle 1`. 
The `stride_x` is 2, thus we pass `READ enable` for the second entry at `1 + 2 = cycle 3`. And then, 
pass `READ enable` for the third entry at cycle 5 and fourth entry at cycle 7. Then the second 
outer loop would occur at `stride_y*1 + offset = 10*1 + 1 = cycle 11`. And the cycle time increment 
`stride_x` and so on. 





## Tool
Use [Kratos](https://kratos-doc.readthedocs.io/en/latest/index.html), which is a python library, to generate Lake RTL code.







