Title: Coreboot Part 1
Date: 2020-01-09 06:15:00
Modified: 2020-01-09 06:15:00
Category: Firmware
Tags: blog, firmware, coreboot, intel
Slug: coreboot-part-1
Summary: Compiling  Coreboot 4.11 for Minnowmax with payload seabios or Tianocore

> Coreboot is an open source firmware project. It is focused on x86 processors but other processors, like Alpha, PPC, and ARM-based systems are supported. Coreboot is designed to do critical hardware initialization before passing control to a payload.
>
> [Embedded Firmware solutions][1]

How to Compile Coreboot for MinnowMax on ubuntu?
Here's a snippet which I use to compile Coreboot on Ubuntu 18.04.

First, we need the following files:

1. [Intel UEFI‌ firmware][4] for creating necessary binary files 

Intel Management Engine (ME)or(TXE)
[Intel Flash descriptor (IFD)][5]
Intel Gigabit Engine (GBE)
using `ifdtool` included in the Coreboot source tree.

2. FSP package file which is needed to extract [VGA.dat][3] and [BayTrailFSP.fd][2]

3. Microcode for E3800 CPUs  

we Downloaded these files and got in suitable path like this Makefile.



```sh
# Coreboot Varibales
COREBOOT_DIR=coreboot_minnowmax
COREBOOT_REPO=http://review.coreboot.org/coreboot.git
COREBOOT_COMMIT=4.11_branch
COREBOOT_MICROCODES=microcode/M023067221D.h microcode/M0130673321.h microcode/M0130679901.h

#
#FSP_BINARY_URL=https://github.com/coreboot/fsp/raw/BayTrail/BayTrailFspBinPkg/FspBin/BayTrailFSP.fd
#VGA_BIOS_URL=https://github.com/coreboot/fsp/raw/BayTrail/BayTrailFspBinPkg/Vbios/Vga.dat

# Optional UEFI ROM File
#UEFI_ROM_URL=https://software.intel.com/sites/default/files/managed/e5/fb/minnowboard-max-rel-1-01-firmware-images.zip
UEFI_ROM_FILE=MNW2MAX1.X64.0101.R01.1908091048.bin

#
#BOOT_ORDER=bootorder

# Dependencies
DEPENDS=git m4 bison flex g++ wget unzip tar zlib1g-dev

# Configurations
define GENERAL_CONFIG
CONFIG_VENDOR_INTEL=y
CONFIG_VGA_BIOS=y
CONFIG_VGA_BIOS_FILE="vga.dat"
CONFIG_HAVE_IFD_BIN=y
CONFIG_HAVE_ME_BIN=y
CONFIG_FSP_FILE="BayTrailFSPGold004.fd"
CONFIG_BOARD_INTEL_MINNOWMAX=y
CONFIG_ENABLE_BUILTIN_COM1=y
CONFIG_IFD_BIN_PATH="flashregion_0_flashdescriptor.bin"
CONFIG_ME_BIN_PATH="flashregion_2_intel_me.bin"
CONFIG_HAVE_FSP_BIN=y
CONFIG_CONSOLE_POST=y
CONFIG_CPU_MICROCODE_HEADER_FILES="$(COREBOOT_MICROCODES)"
CONFIG_MAINBOARD_SMBIOS_PRODUCT_NAME="E3800Firmware"
CONFIG_HAVE_GBE_BIN=y
CONFIG_GBE_BIN_PATH="flashregion_3_gbe.bin"
# CONFIG_PAYLOAD_UBOOT is not set
endef

define SEABIOS_CONFIG
#CONFIG_SEABIOS_BOOTORDER_FILE=""
CONFIG_PAYLOAD_SEABIOS=y
endef

define TIANOCORE_CONFIG
CONFIG_PAYLOAD_TIANOCORE=y
CONFIG_PAYLOAD_FILE="payloads/external/tianocore/tianocore/Build/UEFIPAYLOAD.fd"
endef

export GENERAL_CONFIG
export SEABIOS_CONFIG
export TIANOCORE_CONFIG

.PHONY: all depends checkout toolchain ifdtool extract binaries config seabios-config tianocore-config menuconfig build
all: depends checkout toolchain ifdtool extract binaries seabios-config menuconfig build
seabios: all
tianocore: depends checkout toolchain ifdtool extract binaries tianocore-config menuconfig build

depends:
	sudo apt install -y $(DEPENDS)

$(COREBOOT_DIR):
	git clone $(COREBOOT_REPO) $(COREBOOT_DIR)

checkout: $(COREBOOT_DIR)
	git -C $(COREBOOT_DIR) checkout $(COREBOOT_COMMIT)

toolchain: depends
	make -C $(COREBOOT_DIR) crossgcc-i386

ifdtool: $(COREBOOT_DIR)/util/ifdtool/ifdtool

$(COREBOOT_DIR)/util/ifdtool/ifdtool: 
	make -C $(COREBOOT_DIR)/util/ifdtool clean all

binaries:
	#wget -cNO $(COREBOOT_DIR)/fsp.dat $(FSP_BINARY_URL)
	#wget -cNO $(COREBOOT_DIR)/vga.dat $(VGA_BIOS_URL)
	#wget -cNO rom.zip $(UEFI_ROM_URL)
	#unzip -od rom rom.zip
	#find rom -name $(UEFI_ROM_FILE) | xargs -I {} cp {} $(COREBOOT_DIR)/
	#rm -rf rom

extract:
	$(COREBOOT_DIR)/util/ifdtool/ifdtool -x $(COREBOOT_DIR)/$(UEFI_ROM_FILE)
	mv *.bin $(COREBOOT_DIR)

config:
	echo "$${GENERAL_CONFIG}" > $(COREBOOT_DIR)/.config

seabios-config: config
	echo "$${SEABIOS_CONFIG}" >> $(COREBOOT_DIR)/.config

tianocore-config: config
	echo "$${TIANOCORE_CONFIG}" >> $(COREBOOT_DIR)/.config

menuconfig:
	make -C $(COREBOOT_DIR) menuconfig 

microcodes:
	#TODO: 

build:
	make -C $(COREBOOT_DIR) clean
	make -C $(COREBOOT_DIR) all

clean:
	# TODO: 
```

we can compile this makefile with this commands 


```sh
make 

# with payload seabios 
make seabios-config 

#with payload tianocore 

make tianocore-config

```


[1]: https://link.springer.com/book/10.1007/978-1-4842-0070-4
[2]: https://github.com/coreboot/fsp/raw/BayTrail/BayTrailFspBinPkg/FspBin/BayTrailFSP.fd
[3]:https://github.com/coreboot/fsp/raw/BayTrail/BayTrailFspBinPkg/Vbios/Vga.dat
[4]:https://software.intel.com/sites/default/files/managed/e5/fb/minnowboard-max-rel-1-01-firmware-images.zip
[5]:https://doc.coreboot.org/ifdtool/layout.html