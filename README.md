---
layout: home
title: Readme page
permalink: /
---


### AHA Quickstart Guide ###


This website is for onboarding new students into the AHA project. The website describes how to use docker and the basic AHA application flow. To learn more about the project (people, publications, etc.) navigate to https://aha.stanford.edu/. 

### Table of Contents

<i>=> HELPME: Surely there's a better way to organize this list?</i>

* **[Docker Setup](01_docker.md)** for AHA development environment

* **[AHA Git Tips](10_aha_git_tips.md)**

* **[Waveform Debugging Tips](09_waveform_debugging.md)**

* **[CGRA Design Flow](02_design_flow.md)**

* **[Design files](08_design_files.md)**
hold metadata used by tools in the design flow

* **[Mem tile](04_lake.md)**
-- An overview of the CGRA memory tile architecture.

* **[Lake repo explain](05_lake_repo.md)**
-- `Lake` is the design-specific language we use to generate memory tiles (right?)

* **[PnR](07_pnr.md)**
The AHA place-and-route tool, for mapping apps onto our CGRA.

* **[Garnet Daemon](11_daemon.md)**
Use `garnet.py --daemon` for faster turnaround on dense-app PNR.

* **[H2H Description](03_h2h_files.md)**
Apps are written in the `Halide` language. `Halide-to-Hardware` automatically generates custom hardware for running the app.

