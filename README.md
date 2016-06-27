# RISC-V port to Parallella Board

## A Google Summer of Code 2016 Project

With this project I hope to benefit the open-source hardware enthusiast
community with work related to the incredible Parallella board used by thousands
of students and hobbyists around the world. This project will focus on the
integration of the RISC-V rocket core, inside the Zynq FPGA device of Parallella.
The RISC-V rocket core is an implementation of the RISV-V ISA that has gotten a
lot of attention and support due to being clean, modular and power efficient.

This project will allow owners of Parallella boards to write and execute RISC-V
programs with minimal effort from their side. The system will work out of the
box with a prebuilt binary image ready to be placed in an SD card and users
will be able to re-build it with minimal effort. Moreover, a tutorial document
will be created to aid inexperienced users make the most of this work and allow
them to modify it for their own needs and purposes with custom hardware and /
or software code.

## Build Instructions

### Main Build Script
    
You can build everything (bitstream and software) in one step using the master build script:

```bash
./scripts/build.sh
```

This script automates the dependency setup and builds an FPGA bitstream along with the rest of the
needed output files (boot) for the chosen board. Feel free to edit it if want to skip something, until
we have a final makefile with the "sections" of this script. You can also edit `scripts/set.env.sh`
if you need to change the target board (to e.g `zedboard` instead of the default `parallella`) or the
number of jobs your machine can simultaneously handle while building (default is `8`).

### Setup Dependencies

This repository uses the following Git submodules. You can initialize each one separately if needed.

* [Parallella Open Hardware](https://github.com/parallella/oh)
  - Mapped to `root_dir/parallella/oh/`
  - Run in root dir: `git submodule update --init -- parallella/oh`
  - Needed to build the bitstream for Parallella or Zedboard (dev)
    inside the `root_dir/$board/fpga/` folder

* [Rocket-Chip Generator](https://github.com/ucb-bar/rocket-chip)
  - Mapped to `root_dir/ip/rocket-chip/`
  - Run in root dir: `git submodule update --init -- ip/rocket-chip`
  - Needed to update the generated RISC-V Rocket Core RV64G IP
    inside the `root_dir/ip/RISCV_Rocket_Core_RV64G_1.0/` folder

In order to build the boot images (U-Boot, Linux Kernel) for Parallella board target:

* [Parallella U-Boot Bootloader](https://github.com/parallella/parallella-uboot)
  - Mapped to `root_dir/boot/parallella-uboot`
  - Run in root dir: `git submodule update --init -- boot/parallella-uboot`
  - Needed to build the Parallella U-Boot bootloader (u-boot.elf)
    inside the `root_dir/$board/output/boot/` folder

* [Parallella Linux Kernel](https://github.com/parallella/parallella-linux)
  - Mapped to `root_dir/boot/parallella-linux`
  - Run in root dir: `git submodule update --init -- boot/parallella-linux`
  - Needed to build the Zynq ARM Linux kernel of Parallella (uImage)
    inside the `root_dir/$board/output/final/` folder

In order to build the boot images (U-Boot, Linux Kernel) for ZedBoard board target (development only):

* [Xilinx U-Boot Bootloader](https://github.com/Xilinx/u-boot-xlnx)
  - Mapped to `root_dir/boot/u-boot-xlnx`
  - Run in root dir: `git submodule update --init -- boot/u-boot-xlnx`
  - Needed to build the ZedBoard U-Boot bootloader (u-boot.elf)
    inside the `root_dir/$board/output/boot/` folder

* [Xilinx Linux Kernel](https://github.com/Xilinx/linux-xlnx)
  - Mapped to `root_dir/boot/linux-xlnx/`
  - Run in root dir: `git submodule update --init -- boot/linux-xlnx`
  - Needed to build the ZedBoard Linux kernel (uImage)
    inside the `root_dir/$board/output/final/` folder

* [Device Tree Compiler](https://git.kernel.org/cgit/utils/dtc/dtc.git)
  - Mapped to `root_dir/boot/dtc`
  - Run in root dir: `git submodule update --init -- boot/dtc`
  - Needed to build the ZedBoard U-Boot bootloader (u-boot.elf)
    inside the `root_dir/board/output/boot/` folder and possibly
    a new device tree blob (devicetree.dtb) inside the
    `root_dir/$board/output/final/` folder

### Build Bitstream

In order to only build a bitstream (no software), first populate the Parallella OH submodule (see above)
and then run the following:

* **Parallella**

```bash
cd parallella
./build.sh
```
You can view / edit the Parallella design by opening the `root_dir/parallella/fpga/parallella_riscv/system.xpr` Vivado project.

* **Zedboard** (Only for development and testing)

```bash
cd zedboard
./build.sh
```
You can view / edit the Zedboard design by opening the `root_dir/zedboard/fpga/zedboard_riscv/system.xpr` Vivado project.

## Design

![Vivado Block Diagram](doc/images/vivado.parallella.riscv.bd.png)

### Parallella Base Component
This design contains the Parallella Base component connected to the ARM cores via AXI4.
The Parallella Base component contains the `E-Link` needed for communication with the `Epiphany` chip on-board Parallella
along with `GPIO` single ended passthrough (PL <-> PS connections) and `I2C` bus connection to the on-board power regulators
that power mangage the Epiphany chip. All these mean that the bitstreams produced here have identical functionality with
those provided by Parallella plus the RISC-V core (see below).

### RISC-V RV64G Core
The design contains a `RISC-V RV64G` (**G** = **IMAFD** = **I**nteger, **M**ultiply/Division, **A**tomic,
**F**loating Point with Single or **D**ouble Precision extensions) Core produced by
[Rocket Chip Generator](https://github.com/ucb-bar/rocket-chip).

It communicates with the rest of the ARM SoC of the Zynq FPGA device using AXI4 interfaces:

* **AXI Master**: RV64G Core to DDR3 via ARM  (memory access of the core)

* **AXI Slave**:  ARM to HostIO of RV64G Core (boot / control the core)

### Timing
The Parallella Base component runs with a 100 MHz clock and the RISC-V RV64G core runs with a 25 MHz clock.
It can be synthesized and implemented with the latest Xilinx Vivado tools (tested with 2015.4 and 2016.1).

You can select your version of the Xilinx tools by editing the `VIVADO` variable in `root_dir/scripts/set.env.sh`
before running the build scripts to produce a bitstream (see above).

## Links

[FOSSi](http://www.fossi-foundation.org)

[Parallella](https://www.parallella.org)

[RISC-V](http://riscv.org)

[Google Summer of Code](https://developers.google.com/open-source/gsoc/)

[FOSSi GSoC'16 Projects](https://summerofcode.withgoogle.com/organizations/5516229267685376/#projects)

## Contributors

### Code

- Elias Kouskoumvekakis ([Blog](http://eliaskousk.teamdac.com))

### GSoC Mentors

- Olof    Kindgren ([OpenRISC](http://openrisc.io))
- Andreas Olofsson ([Adapteva](http://www.adapteva.com))
