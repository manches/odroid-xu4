# Odroid-XU4 System Image Repository

## Project Overview

This repository contains precompiled bootloader, kernel, and system components for the **Odroid-XU4** single-board computer. It provides a complete system image with:

- **U-Boot bootloader** and configuration
- **Linux kernel** (version 4.7.5+) with modules
- **Hardware-specific binary blobs** from Hardkernel
- **Device tree blobs** for hardware initialization
- **Boot scripts** for automated system startup

The primary purpose is to provide a ready-to-use system image for Odroid-XU4 development and deployment.

## Architecture

This is an **embedded system distribution** rather than a traditional software project:

### Key Components

1. **Bootloader Layer**
   - `u-boot-dtb.bin` - U-Boot bootloader with device tree
   - `boot.scr` - Boot script for automated startup
   - Binary blobs for low-level hardware initialization

2. **Kernel Layer**
   - `zImage` - Compressed Linux kernel image
   - `modules/4.7.5+/` - Kernel modules for hardware support
   - `exynos5422-odroidxu4.dtb` - Device tree blob for hardware description

3. **System Information**
   - `System.map` - Kernel symbol table for debugging
   - `Module.symvers` - Module version information

### Boot Process Flow

```
Power On → BL1 → BL2 → TrustZone → U-Boot → Boot Script → Linux Kernel
```

## Directory Structure

```
├── blobs/                    # Hardkernel proprietary bootloader binaries
│   ├── bl1.bin.hardkernel   # Primary bootloader
│   ├── bl2.bin.hardkernel   # Secondary bootloader
│   └── tzsw.bin.hardkernel  # TrustZone software
├── configs/                 # Configuration files
│   ├── linux.config        # Kernel configuration
│   ├── u-boot.config       # U-Boot configuration
│   └── u-boot_autoboot.cmd # Boot script source
├── modules/                 # Kernel modules
│   └── 4.7.5+/             # Modules for kernel version 4.7.5+
├── LICENSES/               # License files
├── *.bin                  # Compiled binaries
├── *.dtb                 # Device tree blobs
└── zImage               # Linux kernel image
```

## Key Files

### Critical System Files
- **`zImage`** - Main Linux kernel image (5.2MB)
- **`u-boot-dtb.bin`** - U-Boot bootloader with device tree (550KB)
- **`boot.scr`** - Compiled boot script for automated startup
- **`exynos5422-odroidxu4.dtb`** - Hardware description for Odroid-XU4

### Configuration Files
- **`configs/linux.config`** - Complete kernel build configuration
- **`configs/u-boot.config`** - U-Boot build configuration  
- **`configs/u-boot_autoboot.cmd`** - Source for boot script generation

### Hardware-Specific Blobs
- **`blobs/bl1.bin.hardkernel`** - Primary bootloader (15KB)
- **`blobs/bl2.bin.hardkernel`** - Secondary bootloader (14KB)
- **`blobs/tzsw.bin.hardkernel`** - TrustZone secure world software (256KB)

## System Specifications

- **Target Hardware**: Odroid-XU4 (Samsung Exynos5422 SoC)
- **Architecture**: ARM (32-bit)
- **Kernel Version**: 4.7.5+
- **Base Configuration**: Derived from Debian armhf config
- **Boot Method**: SD card / eMMC with U-Boot

## Development Context

### Build Information
- Binaries (except those in `blobs/`) are custom-built by the repository maintainer
- Kernel configuration based on Debian's armhf configuration
- Hardware blobs are proprietary binaries from Hardkernel

### Boot Script Generation
To regenerate the boot script from source:
```bash
# Requires U-Boot tools
./tools/mkimage -c none -A arm -T script -d configs/u-boot_autoboot.cmd boot.scr
```

### Environment Variables
The boot script requires these environment variables:
- **Platform dependent**: `boardname`, `fdtfile`, `console`
- **System dependent**: `mmcbootdev`, `mmcbootpart`, `mmcrootdev`, `mmcrootpart`, `rootfstype`

## Common Tasks

### Updating Kernel Modules
```bash
# Extract modules to target filesystem
cp -r modules/4.7.5+/* /target/lib/modules/4.7.5+/
depmod -a 4.7.5+
```

### SD Card Deployment
```bash
# Write bootloader to SD card
dd if=blobs/bl1.bin.hardkernel of=/dev/sdX seek=1 bs=512
dd if=blobs/bl2.bin.hardkernel of=/dev/sdX seek=31 bs=512
dd if=u-boot-dtb.bin of=/dev/sdX seek=63 bs=512
dd if=blobs/tzsw.bin.hardkernel of=/dev/sdX seek=1503 bs=512

# Copy boot files to boot partition
cp zImage /boot/
cp *.dtb /boot/
cp boot.scr /boot/
```

### Kernel Configuration Changes
```bash
# Use provided config as base
cp configs/linux.config /path/to/kernel/.config
cd /path/to/kernel
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- zImage modules dtbs
```

## Important Notes

### Hardware Compatibility
- **Odroid-XU4 specific** - binaries are not compatible with other ARM boards
- Device tree blob (`exynos5422-odroidxu4.dtb`) is hardware-specific
- Boot addresses and memory layout are tailored for Exynos5422 SoC

### Licensing
- Linux kernel components: GPL v2 (see `LICENSES/COPYING.linux`)
- U-Boot components: Various open source licenses (see `LICENSES/u-boot/`)
- Hardkernel blobs: Proprietary (usage restricted)

### Boot Process Requirements
- Requires properly partitioned SD card or eMMC
- Boot partition must be FAT32 formatted
- Root filesystem can be ext2/3/4 or other supported filesystem
- Boot script automatically detects kernel type (zImage, uImage, or Image.itb)

### System Limitations
- **No source code** - this is a binary distribution
- **No build system** - binaries are pre-compiled
- **Version locked** - kernel modules tied to specific kernel version 4.7.5+
- **ARM 32-bit only** - not compatible with 64-bit ARM systems

### File System Integration
The modules directory structure follows standard Linux conventions:
```
modules/4.7.5+/kernel/     # Kernel modules organized by subsystem
modules/4.7.5+/modules.*   # Module dependency and alias files
```

This repository serves as a complete system image for embedded development and deployment on Odroid-XU4 hardware.