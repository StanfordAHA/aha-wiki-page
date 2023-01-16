---
title: Why using CGRA
author: XXX
date: 2022-09-09
layout: post
---


Why configuration is needed?
-------------
Both FPGA and CGRA are SRAM-based. Since SRAM is volatile, FPGA/CGRA lose their programming each time the power is removed. The configuration occurs on power-up, while it can also occur on demand.


Why re-configurable hardware is needed?
-------------
There are two conventional computing methods to execute an algorithm.

1. The first is to use hard-wired technology to custom-design the IC for a particular application. Since computation is performed by a circuit rather than by executing instructions, they are very fast and efficient (e.g. ASICs). However, the circuit cannot be altered after fabrication. This force a redesign and refabrication of the chip if any part of the circuit need modification, which would be a very expensive and time-consuming process.
   
2. The second method is to use software-programmed microprocessors to execute a set of instructions. This would be a very flexible solution since we can modify the instructions to alter the functionality of the system without changing the hardware. However, the performance is far below that of an ASIC. The processor must read each instruction from memory, decode its meaning, and only then execute it.

Thus, reconfigurable computing is intended to combine the flexibility of software with the speed of hardware. For example, FPGA contain an array of logic blocks, whose functionality is determined through multiple programmable configuration bits. The interconnection between these blocks are also programmable. In this way, it still achieve high performance by using the configured circuit to perform computation while maintaining the programmable flexibility.


Why using CGRA?
-------------
There are two classes of reconfigurable fabrics:

### Fine-grained
FPGAs allow data manipulation at bit-level for both processing and communication. The configuration of FPGAs generally requires long bitstreams since it controls the operation on a per LUT and per routing channel basis. Thus, the reconfiguration time is verg long.

### Coarse-grained
GRAs manipulate groups of bits via complex functional units (PEs) and the interconnect networks are organized at word-level. Since the larger units require less configuraion state, the amount of configuration data of CGRAs are smaller, which would reduce reconfiguration time substantially. CGRAs also have better performance and energy efficiency since PEs have specialized arithmetic units that operate at larger and more specialized units.


## Reference
[1] [https://people.ece.uw.edu/hauck/publications/ConfigCompute.pdf][1]

[2] [https://indico.cern.ch/event/78644/attachments/1059010/1510107/dyn_reconf.handouts.pdf][2]

[3] [https://link.springer.com/chapter/10.1007/978-3-319-78890-6_53][3]



[1]: https://people.ece.uw.edu/hauck/publications/ConfigCompute.pdf
[2]: https://indico.cern.ch/event/78644/attachments/1059010/1510107/dyn_reconf.handouts.pdf
[3]: https://link.springer.com/chapter/10.1007/978-3-319-78890-6_53
