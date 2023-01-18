---
title: PnR stuff
author: XXX
date: 2022-01-07 (Date used for order should be 2022-01-XX.)
layout: post
---

This is a link to a good book which provide a very clear explanation for the concept and algorithms about different stages in PnR. 

[VLSI Physical Design: From Graph Partitioning to Timing Closure](https://link.springer.com/book/10.1007/978-90-481-9591-6)


# CGRA PnR

[cgra_pnr](https://github.com/Kuree/cgra_pnr/tree/multiple-regs-per-tile) is the AHA PnR tool.

- `cgra_pnr/cyclone` is the routing engine, which works quite well now.
- `cgra_pnr/thunder` is the placement engine , which might have some problems that could be solved.

PnR tool is written in C++ for faster runtime, but it needs to combine with python in order to be used by the AHA flow. 

Under thunder, `example/placer.cc` is the top module. `placer.cc` would be make into a binary executable called `/aha/cgra_pnr/thunder/build/example/placer`. When using the `aha pipeline` command during design flow, the executable which would be run by `archipelago/archipelago/place.py`. **Archipelago** is the **"glue function"** between C++ PnR tool and python AHA flow. **Canal** do the packing part before PnR step, it would output the CoreIR graph to be the input of the PnR tool.

Inside `Garnet.py`, it would call 
```
   	placement, routing, id_to_name = archipelago.pnr(self.interconnect, (netlist, bus),
                                                   	load_only=load_only,
                                                   	cwd=app_dir,
                                                   	id_to_name=id_to_name,
                                                   	fixed_pos=fixed_io,
                                                   	compact=compact,
                                                   	harden_flush=self.harden_flush,
                                                   	pipeline_config_interval=self.pipeline_config_interval)
```


`placer.cc` got three input files:

- `cgra.layout`

   Include flag arrays for PE (P), Pond (M), MEM (m), input (I), output (i) and register (r), specifying where it exists in each tile in the CGRA.  

   => what we care about for multi-register issue is the “r” array, originally there is only one register in each tile. (1 in the array, hardware could support up to 20*)

- `netlist.packed`

   Include three sections: Netlist (sink to multi source), Names (what P0, P1 stands for) and Bus bitwidth (1 bit/ 17 bit)

- `result.place`
  
	The output file, contain tile coordinates for each PE or MEM.



`placer.cc` call functions in `src/`
	
- `global.cc`
  
   Currently is problematic, use `export DISABLE_GP=1` to skip and cluster all nodes into one cluster and do only detail placement. 
  
- `detailed.cc`

   child constructor of `SimAnneal` (in `anneal.cc`).
  
	- `DetailedPlacer::compute_reg_no_pos`
      
   Generate the linked nets starting from the register source. `reg_no_pos` is the mapping between registers and the connected non-registers, vice versa. We would use this information to make sure that if a register and a non-register are connected by some nets, they won't be placed in the same tile. (Need to double check with Jack)

	- `DetailedPlacer::legalize_reg`
  
   In the master branch, this is where only one register can be placed in one tile.

   - `this->curr_energy`

   The sum of HPWL for every net in the netlist.

   - `DetailedPlacer::set_bounds`
  
   Detailed placement is plain annealing. Currently, there is no window size limit. There is some potential improvement to set the boundaries for each moves, but searching a replacement candidate can be slow since it need to maintain a data structure.


- `multiplace.cc`
  
   Call `DetailedPlacer` (defined in `detailed.cc`) for each cluster.
  
- `anneal.cc`
  

   > ##### Note
   > 
   > Simulated annealing add randomness so that it can reach the minimum point instead of staying at local minimum.
   {: .block-tip }
   

# PnR steps

## Packing 

Constant, register, LUT can be packed into one tile. Constant can be stored in the register inside PE tile. And small operation results can be stored in LUT inside the tile. 
This is done in the mapping step to group multiple operators into a single PE tile. 

After packing, the input of PnR would be the mapped CoreIR graph. Each node in the graph would be corresponding to one PE/MEM tile. And PnR goal is to place each tile on to CGRA. 

   > ##### Comparison
   > 
   > The input when running ASIC PnR step is **netlist**, each node is a **gate**. While now we are running PnR on CGRA, the input would be **mapped CoreIR graph**, each node is a **tile**.
   {: .block-warning }

## Partitioning

The partition goal is to divide netlist into small modules. The cost function could be the number or weight of cut edge, size balance of partition, number of tiles needed, number of interconnect between tiles.

- Use **logic replication** to deal with pin limitation
- Group highly connected subcircuits into **cluster** to reduce problem complexity
- **Multi-level partitioning** from coarse to fine

   > ##### Note
   > 
   > In this case, the clustering purpose of the `partition_netlist` phase during placement is to partition PE tiles into clusters, so that we can assign the computation-related tiles closer to each other. 
   {: .block-tip }

   > ##### Comparison
   > 
   > When running partition on ASIC, we divide **netlist** into small **modules**. And for the CGRA partitioning, we divide **tiles** into small **cluster**.
   {: .block-warning }



## Floorplanning

Floorplanning place the macros/blocks on the chip, determine **block outlines** and **pin locations** and set the routing area between them. We would assign each modules to different blocks in this phase using wire length objective. For example, wire length, routing congestion, signal delay etc.

   > ##### Comparison
   > 
   > When doing floorplanning on ASIC, we determine the **shape and arrangement** of each module. And for the CGRA floorplanning, we determine the **arrangement** of each cluster. Each tile is essentially a square, so the sizing of each module during floorplanning is to decide the topology of placing this group of tiles. For example, in *4\*3 way* or *6\*2 way*.
   > 
   > The power and ground routing are already connected to each tile in the CGRA, so it would be fixed.
   {: .block-warning }

   > ##### Note
   > 
   > Although both floorplan and placement deal with the location and arrangement of the modules, 
   > - Floorplan is about placing the macros/blocks in the chip at the upper level, determining block outlines and pin locations, leaving uniform space for the std cells and setting the routing areas between them. 
   > - Placement is about placing std cells within each block (the shape of the std cell is already fixed).
   {: .block-tip }


## Global placement

Global placement assign **general location** to movable objects within each block, determine **overall density distribution**. The goal would be interconnect minimization and cell overlap removal. Use estimate of routing quality metrics as the cost function. For example, total weighted wirelength, cut size, wire congestion and max signal delay.


## Legalization

Legalization align object locations to **legal cell sites** (rows and columns), **remove overlap**, and minimize displacement from GP. Unlike spreading in global placement, legalization assumes all cells are distributed well throughout the region and have small overlap. Legalization ensures that design rules & constraints are satisfied: 

  - All cells are in rows 
  - Cells align with routing tracks 
  - Cells connect to power & ground rails 
  - Additional constraints are often considered, e.g., maximum cell density



## Detailed placement

Detailed placement **refines individual locations** by local optimization while preserve legality.


## Variable format

| Variables   |      Format      |  Example             |
|:------------|:-----------------|:---------------------|
| Netlist     | <net id, blk ids> = <str, str vector> | e10 = (m10, r8, p55) |
| Cluster     | <cluster id, blk ids> = <int, str set>  | x6: (p17, p18, p21, r16, r19) |
| Position    | <blk id, coordinate> = <str, int pair>  | I0 = (3 0) |
| GP_result   | <blk type, coordinates> = <str, int set>  | m : (3 1), (3 3), (4 2), (7 7), ...   |
| DP_result   | <blk id, coordinate> = <str, int set>   | m10 = (19 8) |


## Recompile

After making some change under `thunder` or `cyclone`, we need to recompile to make a new executable. 

Under `/aha/cgra_pnr`:
   
      $ ./install.sh
      $ pip uninstall pycyclone -y && pip uninstall pythunder -y
      $ pip install -e thunder && pip install -e cyclone

Then we can run the below script to test **only** the placer:

      #!/bin/bash
      APP=gaussian
      APP_PATH=/aha/Halide-to-Hardware/apps/hardware_benchmarks/apps/$APP/bin
      PLACER=/aha/cgra_pnr/thunder/build/example/placer
      $PLACER $APP_PATH/design.layout $APP_PATH/design.packed $APP_PATH/design.place

# Future Work

## Current Issues

1. Multiple register in one tile
2. Global placement
3. Placement cost function => could be more analytical
4. Multi hop pass (L1, L3, L7)
5. Hierarchical placement (cluster routing)
6. Network on Chip (NoC) 
7. Fully connected cluster, then the upper level is also fully connected

## Multi-Register Tiles

During the pipeline stage, multiple registers might be installed between PE tile and MEM tile to balance the delay from different paths. But originally each tile only supported one register, so we would need to activate more tiles to support these registers. This kind of active tile is only used for register, it neither functions as PE nor MEM, but still consumes power. So we want to allow multiple registers with one tile. 

There is a switch box in each tile, and then mux to decide whether the input is connected to the PE core or MEM core. So we have to pass through one tile to arrive at the neighboring tile. Currently it is single-hop, so the signal might need to pass through multiple tiles along the path to reach the destination. 

Inside the tile, there is a register before every output port, so the tile hardware can actually support up to 20 registers. But at the same time, only one register can be used from one input port to the output port of the switch box. If both the input port and the output port are different, then multiple registers can be used in the same switch box. 

## Future Issues 

1. Auto mapping of larger BW to smaller BW
2. Hierarchical routing to solve routing congestion
3. superPE use two SB inside the tile?
4. Multiple MEM sharing same PE
5. Every SB have 20 MUX, most are not used
6. Dynamic Partial Reconfiguration => need reg to store config
7. Post PnR pipeline: find longest path and do branch delay match
8. Current ML application only have single layer
   => need to care Inter layer placement for the full NN
9. Flexibility to add latency
10. Add register at output of PE/MEM tile
11. Reduce the number of tracks in SB
12. Iterative PnR
13. Register resource

## Reference 

[1] [VLSI Physical Design: from Graph Partitioning to Timing Closure](https://link.springer.com/book/10.1007/978-90-481-9591-6)

[2] [VPR: A New Packing, Placement and Routing Tool for FPGA Research](https://janders.eecg.utoronto.ca/1387/readings/fpl97_vpr.pdf)

[3] [Timing Driven Placement](https://www.cerc.utexas.edu/utda/publications/book_tdp.pdf)

[4] [Leiden algorithm](https://arxiv.org/pdf/1810.08473.pdf)





