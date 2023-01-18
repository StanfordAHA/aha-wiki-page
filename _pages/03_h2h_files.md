---
title: H2H Description
author: XXX
date: 2022-01-03 (Date used for order should be 2022-01-XX.)
layout: post
---

Under each application folder, there are two files that describe the application in Halide.

`<AHA_path>/aha/Halide-to-Hardware/apps/hardware_benchmarks/apps/<app>/<app>_generator.cpp`

`<AHA_path>/aha/Halide-to-Hardware/apps/hardware_benchmarks/apps/<app>/process.cpp`

In `<app>_generator.cpp`, there are several knobs that we can control to determine how the images are streamed. The schedule, mywidth, and myunroll are three arguments that we usually focus on. We can change the arguments either by setting `HALIDE_GEN_ARGS` (e.g. `export HALIDE_GEN_ARGS="mywidth=62 myunroll=2 schedule=3"` for gaussian) or setting the arguments in both `<app>_generator.cpp` and `process.cpp`. To quickly know the `HALIDE_GEN_ARGS` that work for dense apps, we could check in `/aha/aha/util/regress.py`.
  
# Change the Application Unrolling
In order to change the utilization of a certain application, we can change its unrolling by setting `myunroll`. This would be useful if we want to exploit the maximum utilization of the CGRA array or use low unrolling duplication to get higher frequency. However, there are certain rules needed to be honored when changing the unrolling.

1. `output width % unroll == 0`
2. `input bank width < 20 || input bank width % 4 == 0`

The variables can be calculated as follows:
- The `output width` is the `mywidth` set in `<app>_generator.cpp` and `process.cpp`.
- `input bank width = ceiling(input width / unroll)`.
- The `input width` depends on which application you are running. For most dense applications (e.g. gaussian, harris_color, unsharp), `input width = output width + filter size - 1`. The `filter size` could be found in `<app>_generator.cpp`. Take the gaussian for example, `filter size = 3`.

# Change the Tile Size
In most cases, the images are not streamed to the CGRA at one time, but are divided into several tiles first and then streamed tile by tile. When you run the bitstream generation and the rtl simulation test, you only get the estimated frequency, cycle delays, and simulation result for one tile. We can change the tile size by adjusting the tileWidth and tileHeight in `<app>_generator.cpp` for specific purposes. For example, if we want to accelerate the spin time for CI test, we can set a smaller tile size by reducing `mywidth` in `HALIDE_GEN_ARGS`. If we want to make sure that the whole image is processed correctly, we can set the size to the whole image size to test the functionality of an application.
