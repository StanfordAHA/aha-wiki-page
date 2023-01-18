---
title: Lake waveform
author: XXX
date: 2022-01-06 (Date used for order should be 2022-01-XX.)
layout: post
---

The waveform contains three types of task using the same MEM tile architecture. We can go through each task one at a time. 

    TOP
    +-- conv_stencil
    +-- hw_input_global_wrapper_stencil
    +-- hw_kernel_global_wrapper_stencil

# Kernel Stencil

    hw_kernel_global_wrapper_stencil
    +-- ub_hw_kernel_global_wrapper_stencil_BANK_0
        +-- LakeTop_flat_inst
            +-- LakeTop
                +-- mem_ctrl_strg_ub_vec_flat




extent = range
