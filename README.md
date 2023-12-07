[JoSIM]: https://github.com/JoeyDelp/josim.git
[pyjosim]: https://github.com/JoeyDelp/pyjosim.git
[poetry]: https://github.com/sdispater/poetry

# JoSIM Tools

JoSIM Tools is a set of tools that performs analysis on Superconducting Single Flux Quantum Circuits. 
JoSIM Tools utilizes [JoSIM] subroutines through a [pyjosim] which creates a python bindings for the C++ library. 
The tools include verification, margin analysis, yield analysis, and optimization routines.

The full documentation can be found on the [JoSIM Tools Github Pages](https://joeydelp.github.io/josim-tools/)

## Project Goal

A tool which does analysis and optimization of SFQ circuits while being:

* Reasonably efficient
* Configurable
* Programmatically extendable

## Requirements
- Python 3.6+
- [pyjosim]
- [poetry]

## Installation
```bash
$ git clone https://github.com/JoeyDelp/josim-tools
$ cd josim-tools
$ poetry build --format=wheel
$ pip uninstall josim-tools # Just in case of older versions
$ pip install ./dist/josim_tools-1.1.6-py3-none-any.whl
```

## Usage

JoSIM Tools requires the following 3 files to function:
1. A circuit file
2. A specification file generated using [sp_generator.py](https://github.com/JoeyDelp/josim-tools/blob/master/scripts/sp_generator.py)
3. A configuration file in TOML format

#### Circuit file
The circuit file needs to be an operational simulation with components of interest abstracted to parameters.
#### Specification file
This is a file in a tabular format that identifies the time point and $2\pi$ multiple of phase for each JJ in the circuit file.
The provided script automatically creates this file when given the simulation result that contains the phase output of each JJ.
#### Configuration file
This configuration file in TOML language specifies the type of operation (verify, margin, yield, optimize) that JoSIM tools needs to perform.
It also contains the path to the circuit file, specification file and the threshold for verification of this circuit operation.

More detail on each of these files can be found [here](https://joeydelp.github.io/josim-tools/)

JoSIM Tools is called through the following command:
```bash
$ josim-tools configuration.toml
```

#### Alternatives in this space

* [PSCAN2/COWBoy](alternatives.md#pscan2cowboy)
* [MALT](alternatives.md#malt)
* [xopt](alternatives.md#xopt)
* [Cadance AAO](alternatives.md#cadence-aao)

#### License

This software is licensed under the BSD-2-Clause license. See [LICENSE.md](https://github.com/JoeyDelp/josim-tools/LICENSE.md) for more details.

## Changelog

### V1.1.6 - 12/07/2023

- Added compatibility for JoSIM v2.6.8

### v1.1.4 - 31/01/2022
- Altered to work with the latest release of JoSIM v2.6
- Updated documentation

## Acknowledgements
- Original author Dr Paul le Roux