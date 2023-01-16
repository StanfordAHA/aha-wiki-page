---
title: Design files
author: XXX
date: 2023-01-13
layout: post
---

### design.place 

The design.place contains the name of the tile, X,Y location and tile id (used in io_placement.py, etc.).

### design.route

The design.route contains the route for each edge. First we have the Net ID, then each net is composed of segments (# is "Segment Size"). Then each segment is described. 

SB: (track, x, y, side, input/output, bit_width)
PORT: port_name (x, y, bit_width]
REG:  reg_name (track, x, y, bit_width) (reg_name is track and direction)
RMUX: rmux_name (x, y, bit_width) (rmux name is track and direction)

(Side)

       3
       
      ---
      
    2 | | 0
    
      ---
      
       1
   
   
Source (https://github.com/Kuree/cgra_pnr/blob/master/cyclone/src/graph.cc#L109-L139)

### design.packed

The design.packed has every edge in the application graph from point to point (tile id, port).

### design_top.json

### design_meta.json

The design_meta.json contains the information needed by aha glb to configure and run on our CGRA. Most importantly this includes input data (file name, shape, and glb location) and glb configuration. 
