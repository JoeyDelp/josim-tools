# Configuration File

The command line format for calling JoSIM Tools includes a required configuration file.

```bash
$ josim-tools configuration.toml
```
Reference: [Introduction](index.md#getting-started)

The configuration file is in [TOML](https://github.com/toml-lang/toml ) format with a specific table structure. The configuration file instructs JoSIM Tools which mode to execute and the relevant settings for that mode.

JoSIM Tools also requires a fully operational circuit netlist and a verification file. These files are specified inside the configuration file as discussed in the [verify](#verify) section below.

## Modes

A mode line is required. The following execution modes are supported:

 * **[verify](#verify)** - Verify if a circuit works
 * **[margin](#margin)** - Perform a margin analysis on a circuit
 * **[yield](#yield)** - Perform a yield analysis on a circuit
 * **[optimize](#margin)** - Optimize a circuit

Example:

```toml
mode = "verify"
```

## Verify

Verify mode simulates the circuit, compares the output against the provided verification file, and reports whether they match (success) or not (fail).

A [verify](#verify-table) table is required.

### Verify Table

The first item required in the verification table is a method of verification. At present only the *specification file (spec)* method is supported. This method compares the simulation results of the input circuit to that of the specified verification *spec* file.

Example:

```toml
mode="verify"

[verify]
method="spec"
```

#### Specification File Verification

Verification using a specification file requires:

1. Netlist (.cir file) with a fully operational circuit. Note that all parameters in the [parameter table](#parameter-table) must be in the main design is not within a subcircuit.
2. A verification file in the SP (specification file) format.

Example:

```toml
mode="verify"

[verify]
method="spec"
circuit="path_to_circuit_file.cir"
spec_file="path_to_spec_file.sp"
```

#### SP File Format

An SP file consists of a table that contains time points and the integer number of 2\(\pi\) phase jumps that each specified Josephson junction (JJ) has experienced by that time point. 

The column headers must be `time` followed by each JJ label name as declared in the circuit file. The number of spaces between labels and numbers is arbitrary.

The first row following the header should be a time close to the start of the simulation at which no JJ has switched, and all JJ phases have reached stable values. This time point is used to provide for each JJ a phase offset to be subtracted the phase value measured at subsequent times (rows).

The remaining lines each include a time and the integer number of 2\(\pi\) phase shifts expected for each JJ.

A script to automate the generation of the specification file can be found [here](https://github.com/JoeyDelp/josim-tools/blob/master/scripts/sp_generator.py).

Example spec file:

```
time        B01TX1      B01TX2      B1          B2          B3          B03|X0RX    B03|X0RX2
4.00E-011   0           0           0           0           0           0           0
1.47E-010   1           1           1           1           1           0           0
1.66E-010   1           1           1           1           1           1           1
2.36E-010   2           2           2           2           2           1           1
2.55E-010   3           3           3           3           3           2           2
2.74E-010   3           3           3           3           3           3           3
```

Example visualization:

![Specification File Visualization](images/spec_file_plot.svg)

#### Optional Verification Settings

The optional **threshold** verification setting allows adjustment of the acceptable range within which the measured number of phase jumps is accepted as matching the expected integer number. The phase measured at the first time in the spec file is used to provide an offset or reference value. For example, if the measured phase is 6.51 radians and the offset measured at the first time in the spec file is 0.79 radians, the measured number of phase jumps is (6.51 - 0.79)/2pi = 0.91. If the number of phase jumps expected is 1, the absolute value of the difference is abs(0.91 - 1) = 0.09. This difference is larger than the default **threshold** value (0.05), so the measured output would not be considered to match the expected output. Setting a higher **threshold** value allows for ringing or other effects that can cause larger differences. Note that a better name for this quantity might be tolerance or range.

 * **threshold** - Range within which the measured number of phase jumps must be to the expected integer number to be considered a match. Default: **0.05**.

Example:

```toml
threshold=0.35
```

## Margin

A margin analysis tests the margins individually for each specified parameter and reports these margins along with the critical margin. The configuration file for a margin analysis requires at least 2 tables since all the settings in the [margin table](#margin-table) are optional. The required two tables are the [verify table](#verify-table) and a [parameter table](#parameter-table).

Example:

```toml
mode="margin"

[verify]
method="spec"
circuit="path_to_circuit_file.cir"
spec_file="path_to_spec_file.sp"
```

### Margin Table

All margin analysis settings are optional.

 Uncertainties in the upper and lower margin search are given by:
 ```
 δx_upper = (max_search - 1)/(s*(2**b))
 δx_lower = (1 - min_search)/(s*(2**b))
 ```
 where s = **scan_steps** and b = **binary_search_steps**. With max_search = 1.9, s = 4, and b = 3, 4, 5, or 8; δx = 2.8%, 1.4%, 0.7%, or 0.09%.

 * **max_search** - Multiplier for the nominal value to calculate the upper boundary for the margin search. Default: **1.9** (+90%).
 * **min_search** - Multiplier for the nominal value to calculate the lower boundary for the margin search. Default: **0.1** (-90%).
 * **scan_steps** - A positive integer specifying the number of scanning steps the margin analysis takes between the nominal and the max or min value. The scan identifies the rough range within which the margin lies. Default: **4**.
 * **binary_search_steps** - A positive integer specifying the number of binary search steps the margin analysis takes to more accurately locate the margin after scanning. Default: **3**.

Example:

```toml
[margin]
max_search=1.8
min_search=0.2
scan_steps=5
binary_search_steps=4
```

## Parameter Table

The parameter table is used by [margin](#margin), [yield](#yield) and [optimization](#optimization) modes. The parameter names must match parameter labels in the circuit file. 

Note that names of components with non-parameterized values cannot be used. For example, L01 will not work as a parameter if given in the netlist as:
    L01 23 24 1.60e-12
However, L01 will work as a parameter for the following netlist:
    .param L01=1.60e-12
    L01 23 24 L01

All parameters require a **nominal** value. **Min** or **max** values are optional.

```toml
[parameters]
B01={nominal=2.0}
L01={nominal=2E-12,min=1.8E-12,max=2.2E-12}
L02={nominal=2E-12,min=1.4E-12}
IB01={nominal=140E-6,max=180E-6}
```

Notes:
1. Only the parameters in this table will be used when calculating margins, optimization, or yield. Circuit component values not dependent on these parameters are assumed to not vary or affect the margins.
2. Exponential notation is supported (examples above). 
3. Not allowed are unit prefixes or '+' signs (examples: 1.2p, 1.2E+2).

## Yield

Yield analysis calculates the probable yield of a circuit as a percentage. The degree of confidence dependent on the number of samples. Each sample includes probabilistic variation of circuit parameters, margin analysis, and computation of the probability that the circuit will work. A normal (Gaussian) probability distribution is assumed. 

Yield analysis requires [verify](#verify-table), [yield](#yield-table) and [parameters](#parameters-table) tables. The **nominal** parameter specifies the mean value of the probability distribution. An additional requirement for yield analysis is that each of the **parameters** in the table must have a standard deviation (**sd**) to model the expected variation. 

Example:

```toml
mode="yield"

[verify]
method="spec"
circuit="path_to_circuit_file.cir"
spec_file="path_to_spec_file.sp"

[parameters]
B_mult={nominal=1.0,sd=0.03}
I_mult={nominal=1.0,sd=0.01}
L_mult={nominal=1.0,sd=0.02}
B01_0={nominal=2.0,sd=0.02}
B02_0={nominal=0.5,sd=0.04}
IB01_0={nominal=140E-6,sd=5.0E-6}
L01_0={nominal=2.57E-12,sd=1.0E-12}
L02_0={nominal=1.42E-12,sd=1.0E-12}
```
The above example is for component values calculated as:
.param B01=B01_0*B_mult
.param B02=B02_0*B_mult
.param IB01=IB01_0*I_mult
.param L01=L01_0*L_mult
.param L02=L02_0*L_mult

### Yield table

The yield table has one required setting:

 * **num_samples** - A positive integer specifying the number of samples. A larger number improves the statistical confidence but takes longer. 

Example:

```toml
[yield]
num_samples=1000
```

## Optimize

Optimization runs consist of multiple iterations. Each iteration includes adjustment of [parameters](#parameters-table), [verify](#verify), [margin analysis](#margin), and comparison of the new and initial critical margins. Optimization requires [verify](#verify-table) and [parameters](#parameters-table) tables. A [margin](#margin-table) table is optional.

Example:

```toml
mode="optimize"

[verify]
method="spec"
circuit="path_to_circuit_file.cir"
spec_file="path_to_spec_file.sp"

[parameters]
B_mult={nominal=1.0,min=1.0,max=1.0}
B01={nominal=2.0}
L01={nominal=2E-12,min=1.8E-12,max=2.2E-12}
L02={nominal=2E-12,min=1.4E-12}
IB01={nominal=14E-5,max=18E-5}
```

Note: Setting nominal, max, and min to the same values (as done above with B_mult) allows the parameter to be included in margin analyses while keeping the value constant during circuit optimization.

### Optimize table

#### Hybrid optimization

* **method** - Only **hybrid** optimization is supported at present. Reference: 
P. le Roux and C. J. Fourie, “Distance-to-failure-maximization optimization algorithm for SFQ logic cells,” IEEE Trans. Appl. Supercond., vol. 30, no. 7, Art. no. 1301405, Oct. 2020, doi: 10.1109/TASC.2020.2994453.

Optional settings:

 * **search_radius** - Relative distance from the current best value to search for a better value. Default: **0.05**. <!--- Add how to choose. What are consequences for values too large or too small? -->
 * **converge** - Difference between estimated and analyzed guess scores used to define convergence. A smaller difference is harder to achieve. Default: **0.01**.
 * **max_iterations** - A positive integer specifying the maximum number of attempts the optimization routine will make before terminating. Default: **500**.
 * **output** - Update the original circuit netlist with optimized values and save as a file with the specified name. An interim version is saved after each iteration with ‘.current’ appended to the file name. Note that the path to the file must exist. Default: no output file.

Example:

```toml
[optimize]
method="hybrid"
search_radius=0.1
converge=0.2
max_iterations=200
output="output.cir"
```
