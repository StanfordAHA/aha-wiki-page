---
title: Waveform Debugging Tips
author: XXX
date: 2022-09-09
layout: post
---

Waveform Debugging Tips

### PE tile important signals

### MEM tile important signals

### GLB important signals

Navigate to /top/dut/global_buffer_W_inst0/global_buffer and select input/output siganls only

**cgra_cfg_** signals are cgra configuration

**if_cfg_** signals are glb configuration 

**proc_** signals are read/writes to glb memories from the processor

**strm_g2f** and **strm_f2g** signals include data between glb and cgra, interrupt pulses, and flush

![GLB Waveform](/aha-wiki-page/assets/glb_waveform.png)
