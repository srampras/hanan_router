# Hanan Grid Router
This is a rectilinear detail router useful for routing and visualizing signal nets on few blocks (both digital and analog).
It uses a modified version of [A* search aglorithm](https://en.wikipedia.org/wiki/A*_search_algorithm) at its core with the routes on the [Hanan grid](https://doi.org/10.1137/0114025).

For now, the following sets of back-end of line design rules are honored:
* Manhattan spacing between metal and via shapes on each layer.
* Metal widths per layer
* Single/mult-cut vias
* User-specified non-default rules at a block level or net level. User can override the following
  - metal spacing and widths in any layer
  - restrict the use of certain routing layers for certain nets
  - net/block specific routing obstacle


# Building the router
Requirements:
  * A C++ compiler with support for C++14 or higher
  * `make` system
  
Steps to clone and build:
  ```
  git clone https://github.com/srini229/hanan_router.git
  cd hanan_router
  make -j4
  ```
The build commands have been verified on: Ubuntu 18.04 (GCC 6), Fedora  36 (GCC 12), Debian 10 on WSL (GCC 8), and Mac OS 10.15 (Clang 11).
  
 Make generates a binary `hanan_router` on the cloned directory. 
 The repository has a frozen copy of the [`nlohmann/JSON`](https://github.com/nlohmann/json) parser packaged with it to parse the netlist, layer information and user constraints.
 It has a small version of LEF parser that understands basic LEF syntax for now. The router uses `RTree` data structure to quickly accessing the geometric structures from memory.
 It has a copy of the [`RTree`](https://superliminal.com/sources/RTreeTemplate.zip). 
 Migration of the JSON/RTree APIs to [Boost](https://www.boost.org/) libraries is a work in progress. This will eliminate the need for these separate thirdparty libraries.
 
 The JSON file formats reuse the syntax from the [ALIGN](https://github.com/ALIGN-analoglayout/ALIGN-public) project. A generic version to use LEF/DEF files of the netlist/design rules is a work in progress.

# Interface
The router syntax is:
```
usage : hanan_router
	-d <layers.json> ; # Abstracted information for each layer metal/via layer used in a technology
	-p <placement file> ; # Placement information in JSON format
	-l <lef file> ; # LEF file containing the pin/blockage information for each leaf cell
	-s <lef scaling> ; # units used in the LEF file
	-uu <user units scaling> ; # user units scaling for placement file (e.g. nm/um)
	-ndr <ndr constraints.json> -o <output dir> ; # user specified non-default rules
```
The router generates one LEF and one DEF for each hierarchy in the placement. The LEF file has the suffix `_interim_hier.lef`.
There is also a `route.log` created which has a detailed log of the routers activity.

There is also a GDSII file generator available in the `bin/` directory.
It is written in python and requires the the following packages `numpy`, `gdspy`, `json`, and `argparse`.
Any missing package can be installed using the command `pip install <package>`.
The syntax to generate the GDSII files using the LEF/DEF files is:
```
usage: gen_rt_gds.py [-h] [-p PL_FILE] [-g GDS_DIR] [-t TOP_CELL] [-u UNITS] [-s SCALE] [-l LAYERS] [-d DEFF]

options:
  -h, --help            show this help message and exit
  -p PL_FILE, --pl_file PL_FILE
                        <filename.placement_verilog.json>
  -g GDS_DIR, --gds_dir GDS_DIR
                        <dir with all leaf gds files>
  -t TOP_CELL, --top_cell TOP_CELL
                        <top cell>
  -u UNITS, --units UNITS
                        <units in m>
  -s SCALE, --scale SCALE
                        <scale>
  -l LAYERS, --layers LAYERS
                        <layers.json>
  -d DEFF, --deff DEFF  <route def file>

```

# Test directory
There is a small unit test directory `test/`.
To run the test:
```
./hanan_router -d layers.json -p test.placement_verilog.json -l test.lef

```
The `ndr1.json`, `ndr2.json` and `ndr3.json` demonstrate different user constraints. To run the router with the user constraints:
```
./hanan_router -d layers.json -p test.placement_verilog.json -l test.lef.

```
The LEF/DEF outputs generated by the router can be visualized using [`klayout`](https://klayout.de/).

To generate GDSII file from the LEF/DEF files use:
```
../bin/gen_rt_gds.py -p test.placement_verilog.json -g . -t TEST_CONC_0  -l layers.json -d TEST_CONC_0.def 
```
The GDSII files can be visualized in klayout as well. The GDSII file can be imported into commercial PnR tools using the stream in function.