PetaLinux Project source files
==============================

### How to build the PetaLinux projects

#### Requirements

* Windows or Linux PC with Vivado installed
* Linux PC or virtual machine with PetaLinux installed

#### Instructions

In order to make use of these source files, you must:

1. First generate the Vivado project hardware design(s) (the bitstream) and export the design(s) to SDK.
2. Launch PetaLinux by sourcing the `settings.sh` bash script, eg: `source <path-to-installed-petalinux>/settings.sh`
3. Build the PetaLinux project(s) by executing the `build-petalinux` script in Linux.

The script will generate a separate PetaLinux project for all of the generated and exported Vivado projects that
it finds in the Vivado directory of this repo.

### UNIX line endings

The scripts and files in the PetaLinux directory of this repository must have UNIX line endings when they are
executed or used under Linux. The best way to ensure UNIX line endings, is to clone the repo directly onto your
Linux machine. If instead you have copied the repo from a Windows machine, the files will have DOS line endings and
you must use the `dos2unix` tool to convert the line endings for UNIX.

1. Copy the cloned repository from your Windows machine to your Linux machine.
2. Use the `cd` command to navigate to the copied repository on your Linux machine.
3. Type `find . -type f -exec dos2unix --keepdate {} +` to convert all of the files
to the Unix format.

### How the script works

The PetaLinux directory contains a `build-petalinux` shell script which can be run in Linux to automatically
generate a PetaLinux project for each of the generated/exported Vivado projects in the Vivado directory.

When executed, the build script searches the Vivado directory for all projects containing a `.xsa` exported
hardware design file. Then for every exported project, the script does the following:

1. Verifies that the `.bit` file exists.
2. Determines the CPU type: Microblaze, Zynq or ZynqMP. It currently does this
by looking at the first 3 letters of the project name.
3. Creates a PetaLinux project, referencing the exported hardware design (.xsa).
4. Copies the relevant configuration files from the `src` directory into the created
PetaLinux project.
5. Builds the PetaLinux project.
6. Generates a BOOT.bin and image.ub files for the Zynq projects.

### Launch PetaLinux on hardware

#### Via JTAG

To launch the PetaLinux project on hardware via JTAG, connect and power up your hardware and then
use the following commands in a Linux command terminal:

1. Change current directory to the PetaLinux project directory:
`cd <petalinux-project-dir>`
2. Download bitstream to the FPGA:
`petalinux-boot --jtag --fpga --bitstream ./images/linux/system.bit`
If you don't use the --bitstream option to specify the bitstream, then PetaLinux will download the
./images/linux/download.bit bitstream containing the FSBL. We don't want to run the FSBL when
booting via JTAG.
3. Download the PetaLinux kernel to the FPGA:
`petalinux-boot --jtag --kernel`

#### Via SD card (Zynq and ZynqMP)

To launch the PetaLinux project on hardware via SD card, copy the following files to the root of the
SD card:

* `/<petalinux-project>/images/linux/BOOT.bin`
* `/<petalinux-project>/images/linux/image.ub`

Then connect and power your hardware.

### Kernel Start Address for AXI PCIe Gen3 Subsystem

The AXI PCIe Gen3 Subsystem requires it's CTL0 interface to be allocated 256MB on the address map.
During Linux boot, vmalloc is used to allocate virtual memory for this interface. This repo configures
the Kernel start address to 0xB0000000 from the default 0xC0000000, in order to create sufficient
virtual memory for the CTL0 interface. Without this modification, vmalloc fails during boot.

Find the modification here:

`PetaLinux/src/axi_pcie3/project-spec/meta-user/recipes-kernel/linux/linux-xlnx/kernel-options.cfg`

