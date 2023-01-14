---
title: Design files
author: XXX
date: 2023-01-13
layout: post
---




Describe each of
-design.place

The design.place contains the name of the tile, X,Y location and tile id (used in io_placement.py, etc.).

-design.route

-design.packed

The design.packed has every edge in the application graph from point to point (tile id, port).

-design_top.json

-design_meta.json

The design_meta.json contains the information needed by aha glb to configure and run on our CGRA. Most importantly this includes input data (file name, shape, and glb location) and glb configuration. 
