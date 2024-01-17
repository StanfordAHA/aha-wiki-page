---
title: Garnet Daemon
author: steveri@stanford.edu
date: 2024-01-16 (Date used for order should be 2022-01-XX.)
layout: post
---


# Garnet Daemon

`garnet.py` now has a `--daemon` option, designed to improve
turnaround time for garnet dense apps.

Testing a dense app test requires three phases: `mapping`, `pnr`, and
`testing`. The `pnr` phase uses an internal python representation of
the garnet circuit. Previously, this circuit object would be rebuilt
from scratch for every test, even though it is generally the same
structure each time.

The garnet circuit object is such that it cannot be "pickled" (saved
to an external file) for later reuse. So, we added the option to
launch `garnet.py` as a reusable background daemon, allowing
successive pnr/test invocations to reuse the same circuit over and
over.

## Using the Daemon

For aha regressions, you can use the `aha regress --daemon auto`
option to tell `pnr` to use the daemon (or launch one if it does not
yet exist).

When doing standalone pnr, you can do the same thing with `aha pnr
--daemon auto`. The pnr program will in turn pass along the `--daemon
auto` switch when it invokes `garnet.py`.

When using the `--daemon` switch, it is important to run your job in
the background, as the daemon is designed to remain running and
available for future jobs (see example below). To control the running daemon, use the
various `--daemon` switches available via `garnet.py` e.g.
```
    aha garnet --daemon help
    aha garnet --daemon kill
```

See "options" section below and/or `aha garnet --daemon help` for more
information about daemon options.

### EXAMPLE: Regression suite using daemon
```
  # Note: examples assume you are running from inside an aha docker instance
  $ source /aha/bin/activate
  $ aha regress pr --daemon auto
```

### EXAMPLE: Using the daemon across multiple pnr sessions
```
  # Launch the daemon. Background it so that it will persist.
  $ aha pnr apps/pointwise --width 4 --height 2 --daemon auto &

  # Wait for pnr job to complete (optional; can instead just watch for daemon to finish)
  $ aha garnet --daemon wait

  # Subsequent pnr jobs use existing daemon
  $ aha pnr tests/ushift --width 4 --height 2 --daemon auto
  $ aha pnr <app3>       --width 4 --height 2 --daemon auto
  $ aha pnr <app4>       --width 4 --height 2 --daemon auto
  $ ...
```

### EXAMPLE: App development using daemon
( Also see [daemon-test.sh](https://github.com/StanfordAHA/garnet/blob/master/daemon/daemon-test.sh) )
```
  $ source /aha/bin/activate

  ### Garnet Verilog build
  $ flags1="--width  4 --height  2 --verilog --use_sim_sram --rv --sparse-cgra --sparse-cgra-combined"
  $ aha garnet $flags1

  ### Kill existing daemon if one exists
  $ aha garnet --daemon kill

  ### Run first app (pointwise) and launch daemon
  ### Remember to background the daemon so that it will persist
  $ app=apps/pointwise
  $ (cd aha/Halide-to-Hardware/apps/hardware_benchmarks/$app; make clean)
  $ aha map $app --chain
  $ aha pnr $app --width 4 --height 2 --daemon auto &
  $ python garnet/garnet.py --daemon wait   # (optional)

  ### Run a second app (ushift) using the daemon
  ### No need to background pnr here, it uses existing background daemon
  $ app=tests/ushift
  $ (cd aha/Halide-to-Hardware/apps/hardware_benchmarks/$app; make clean)
  $ aha map $app --chain
  $ aha pnr $app --width 4 --height 2 --daemon auto
```

## Garnet daemon options
Use `--daemon help` to find the latest info about how to use the daemon.
```
$ aha garnet --daemon help

    DESCRIPTION:
      garnet.py can run as a daemon to save you time when generating
      bitstreams for multiple apps using the same garnet circuit. Use
      the "launch" command to build a circuit and keep state in the
      background. The "use-daemon" command reuses the background state
      to more quickly do pnr and bitstream generation.

          --daemon launch -> process args and launch a daemon
          --daemon use    -> use existing daemon to process args
          --daemon auto   -> "launch" if no daemon yet, else "use"
          --daemon wait   -> wait for daemon to finish processing args
          --daemon kill   -> kill the daemon
          --daemon status -> print daemon status and exit
          --daemon force  -> same as kill + launch

    EXAMPLE:
        garnet.py --daemon kill
        garnet.py --width 28 --height 16 --verilog ...
        garnet.py <app1-pnr-args> --daemon launch
        garnet.py <app2-pnr-args> --daemon use
        garnet.py <app3-pnr-args> --daemon use
        garnet.py <app4-pnr-args> --daemon use
        ...
        garnet.py --daemon kill

    NOTE! 'daemon.use' width and height must match 'daemon.launch'!!!
    NOTE 2: cannot use the same daemon for verilog *and* pnr (not sure why).
```

## Also See

For more details see [Garnet daemon README](https://github.com/StanfordAHA/garnet/blob/master/daemon/README.txt)

