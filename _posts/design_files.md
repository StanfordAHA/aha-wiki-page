---
title: Design files
author: XXX
date: 2023-01-13
layout: post
---

### design_top.json
`design_top.json` contains the fully mapped application with configured PE tiles, MEM tiles, IO tiles, and registers. It also contains all of the connections between these components. At this point, they are still abstract instances that are not yet mapped to the physical MEM and PE tiles in the CGRA array. design_top.json is a CoreIR file (https://github.com/rdaly525/coreir) and the fully mapped application is specified in `["namespaces"]["global"]["modules"]["<app_name>"]`. The `["instances"]` field has the MEM, PE, and IO instances along with their configurations. The `["connections"]` field lists the wirings between instance pins.

Example instances:
- an example memory instance is called `input_cgra_stencil$ub_input_cgra_stencil_BANK_0_garnet`
    - memory instances usually have `BANK_` in their instance names
    - the `metadata` field contains the memory tile configuration register values after pipelining.
- an example IO instance is called `io16in_input_host_stencil_clkwrk_0_op_hcompute_input_glb_stencil_read_0`
    - `glb2out` is from GLB to the CGRA array
    - `in2glb` is from the array to GLB
- an example PE instance is called `op_hcompute_output_cgra_stencil_1$inner_compute$c11`

### design.packed

`design.packed` conatins all of the information that gets sent to the place and route tool. It contains 3 dictionaries: netlist, ID to names, and netlist bus. 
- The netlist is a dictionary of edge IDs to connections in the application between tiles (PE/MEM/IO/reg tiles). The format of these entries is:
    - ```edge_id: (source_tile_id, source_tile_pin) (dest_tile1_id, dest_tile1_pin) (dest_tile2_id, dest_tile2_pin) ...```
- The ID to names dictionary maps the tile IDs to the tile names as defined in design_top.json.
- The netlist bus dicionary maps each edge ID to the bitwidth of the interconnection used to route that connection. 


### design.place 

`design.place` contains the name of the tile, X,Y location and tile id (used in io_placement.py, etc.).

### design.route

`design.route` contains the route for each edge. First we have the Net ID, then each net is composed of segments (# is "Segment Size"). Then each segment is described. 

SB: (track, x, y, side, input/output, bit_width)
PORT: port_name (x, y, bit_width]
REG:  reg_name (track, x, y, bit_width) (reg_name is track and direction)
RMUX: rmux_name (x, y, bit_width) (rmux name is track and direction)

(Side)
```
      3 
     ---
  2 |   | 0
     ---
      1
```
   
Source (https://github.com/Kuree/cgra_pnr/blob/master/cyclone/src/graph.cc#L109-L139)

### design_meta.json

`design_meta.json` contains the information needed by aha glb to configure and run on our CGRA. Most importantly this includes input data (file name, shape, and glb location) and glb configuration. 
