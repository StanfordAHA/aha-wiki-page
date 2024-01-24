---
layout: home
title: Readme page
permalink: /
---


### AHA Quickstart Guide ###

This website is for onboarding new students into the AHA project. The website describes how to use docker and the basic AHA application flow. To learn more about the project (people, publications, etc.) navigate to https://aha.stanford.edu/. 


### Table of Contents
<i>Note TOC links only work on the
[Pages]([https://stanfordaha.github.io/aha-wiki-page/)
site!</i>


#### Development and Debugging

* **[Docker Setup](01_docker.md)** for AHA development environment

* **[AHA Git Tips](10_aha_git_tips.md)**

* **[CGRA Design Flow](02_design_flow.md)**

* **[Waveform Debugging Tips](09_waveform_debugging.md)**


#### Hardware Architecture

* **[Mem tile](04_lake.md)**
-- An overview of the CGRA memory tile architecture.


#### Software Architecture

* **[H2H Description](03_h2h_files.md)**
-- Apps are written in the `Halide` language.<br/>
`Halide-to-Hardware` (H2H) automatically generates custom hardware for running the app.

* **[Design files](08_design_files.md)**
hold metadata used by tools in the design flow

* **[Lake repo explain](05_lake_repo.md)**
-- `Lake` is the design-specific language we use to generate memory tiles (right?)<br/>
<i>Should this maybe just be a pointer to Lake's README file?</i>

* **[PnR](07_pnr.md)**
-- The AHA place-and-route tool, for mapping apps onto our CGRA.

* **[Garnet Daemon](11_daemon.md)**
-- Use `garnet.py --daemon` for faster turnaround on dense-app PNR.


#### Documentation

* TBD: How To Wiki (yes please contribute to the wiki!)
* TBD: Github Pages vs. Github Wiki




