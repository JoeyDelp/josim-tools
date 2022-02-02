# Example Usage

To demonstrate the usage of JoSIM Tools in all four modes of operation we choose a simple [RSFQ splitter cell](https://github.com/JoeyDelp/josim-tools/blob/master/examples/data/example_splitter.cir)

This file has the following [specification file](https://github.com/JoeyDelp/josim-tools/blob/master/examples/data/example_splitter.sp):

```ceylon
time        B01TX1      B01TX2      B1          B2          B3          B03|X0RX    B03|X0RX2
4.00E-011   0           0           0           0           0           0           0
1.47E-010   1           1           1           1           1           0           0
1.66E-010   1           1           1           1           1           1           1
2.36E-010   2           2           2           2           2           1           1
2.55E-010   3           3           3           3           3           2           2
2.74E-010   3           3           3           3           3           3           3
```

The examples are run directly out of the [examples](https://github.com/JoeyDelp/josim-tools/blob/master/examples) directory.

## Verify

```toml
mode = "verify"

[verify]
method = "spec_file"
circuit = "data/example_splitter.cir"
file = "data/example_splitter.sp"
```

```bash
$ josim-tools ./verify/simple_spec_file.toml
JoSIM Tools 1.1.4
SUCCESS
```

## Margin Analysis

```toml
mode = "margin"

[parameters]
#Globals
Btotal = {"nominal" = 1}
Ltotal = {"nominal" = 1}
Itotal = {"nominal" = 1}
# Junctions
B01rx2_unscaled = {"nominal" = 1}
B01tx1_unscaled = {"nominal" = 2}
B1_unscaled = {"nominal" = 3.25}
B2_unscaled = {"nominal" = 2.5}
# Currents
IB01rx2_unscaled = {"nominal" = 155e-6}
IB01tx1_unscaled = {"nominal" = 130e-6}
IB1_unscaled = {"nominal" = 569e-6}
# Inductors
L01_rx2_unscaled = {"nominal" = 0.2e-12}
L01_tx1_unscaled = {"nominal" = 2.5e-12}
L02_tx1_unscaled = {"nominal" = 3.3e-12}
L03_tx1_unscaled = {"nominal" = 8e-12}
L1_unscaled = {"nominal" = 1.4e-12}
L2_unscaled = {"nominal" = 2e-12}
L3_unscaled = {"nominal" = 0.4e-12}
L4_unscaled = {"nominal" = 1.9e-12}
L5_unscaled = {"nominal" = 2e-12}

[verify]
method = "spec_file"
circuit = "data/example_splitter.cir"
file = "data/example_splitter.sp"

```

```bash
$ josim-tools ./margin/simple_margin_analysis_all.toml
JoSIM Tools 1.1.4
Btotal          : 15.5 [                                #####|####                                 ] 12.7
Ltotal          : 54.8 [                 ####################|##########################           ] 71.7
Itotal          : 18.3 [                               ######|######                               ] 18.3
B01rx2_unscaled : 90.0 [    #################################|#############################        ] 80.2
B01tx1_unscaled : 49.2 [                   ##################|##################                   ] 49.2
B1_unscaled     : 32.3 [                          ###########|###############                      ] 40.8
B2_unscaled     : 38.0 [                       ##############|######                               ] 18.3
IB01rx2_unscaled: 57.7 [                #####################|##################                   ] 49.2
IB01tx1_unscaled: 90.0 [    #################################|##############                       ] 38.0
IB1_unscaled    : 23.9 [                             ########|###########                          ] 32.3
L01_rx2_unscaled: 90.0 [    #################################|#################################    ] 90.0
L01_tx1_unscaled: 90.0 [    #################################|#################################    ] 90.0
L02_tx1_unscaled: 90.0 [    #################################|#################################    ] 90.0
L03_tx1_unscaled: 90.0 [    #################################|#################################    ] 90.0
L1_unscaled     : 90.0 [    #################################|#################################    ] 90.0
L2_unscaled     : 85.8 [      ###############################|#################################    ] 90.0
L3_unscaled     : 90.0 [    #################################|#################################    ] 90.0
L4_unscaled     : 90.0 [    #################################|#################################    ] 90.0
L5_unscaled     : 90.0 [    #################################|#################################    ] 90.0
Critical margin: 12.7 % ['Btotal+']
```

Here we see the critical margin is `12.7%` and is towards the upper side of `Btotal`

## Yield Analysis

```toml
mode = "yield"

[yield]
num_samples = 128

[parameters]
Btotal  = {nominal = 1, variance = 0.1}
Ltotal  = {nominal = 1, variance = 0.1}
Itotal  = {nominal = 1, variance = 0.1}

[verify]
method = "spec_file"
circuit = "data/example_splitter.cir"
file = "data/example_splitter.sp"
```

```bash
$ josim-tools ./yield/simple_yield_analysis.toml
JoSIM Tools 1.1.4
Yield: 90 / 128 = 70.3 %
```

## Optimization

We will now attempt to improve the critical margin identified by the [margin analysis](#margin-analysis).

```toml
mode = "optimize"

[optimize]
method = "hybrid"
search_radius = 0.15
converge = 0.0005
max_iterations = 1000
output = "output/simple_hybrid_optimize.cir"

[margin]
max_search = 1.5
min_search = 0.5
scan_steps = 1
binary_search_steps = 15

[parameters]
Btotal = {nominal = 1, min=0.5, max=1.5}
Ltotal = {nominal = 1, min=0.5, max=1.5}
Itotal = {nominal = 1, min=0.5, max=1.5}

[verify]
method = "spec_file"
circuit = "data/example_splitter.cir"
file = "data/example_splitter.sp"
```

```bash
$ josim-tools ./optimize/advanced_hybrid_optimize.toml
JoSIM Tools 1.1.4
Starting value optimization
Analyzing point
.
.
.
Verifying next guess
Analyzing point
  Adding point to list of guesses
  Doing margin analysis of point:

Btotal:  21.1 [                              #######|#######                              ] 21.6
Ltotal:  37.7 [                        #############|#################                    ] 50.0
Itotal:  25.7 [                            #########|########                             ] 24.5
Critical margin: 21.1 % ['Btotal-']

  Adding all margin boundaries to points of failure
  Finished analyzing point
Guess score estimation error: 0.0
Convergence reached
Optimized:
  point: [0.52176166 1.49948778 0.50816279]
  score: 18.85875836212533
```

Here we can see that we have managed to improve the critical margin (`Btotal+`) of `12.7%` to a new critical margin `Btotal-` of `21.1%`.