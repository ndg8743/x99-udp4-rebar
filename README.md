# X99-UD4P Large BAR / ReBAR Support for Tesla GPUs

Enabling 16GB+ BAR allocation for Tesla V100/M40/P40 GPUs on Gigabyte GA-X99-UD4P motherboard which lacks native Above 4G Decoding support.

## System Configuration

| Component | Specification |
|-----------|---------------|
| **Motherboard** | Gigabyte GA-X99-UD4P-CF Rev 1.0 |
| **BIOS** | F23c (AMI) |
| **CPU** | Intel Xeon E5-2697A v4 (16C/32T, Broadwell-EP) |
| **RAM** | 128GB DDR4 2133MHz |
| **Target GPU** | NVIDIA Tesla V100 16GB |
| **OS** | Proxmox VE (Debian-based Linux) |

## The Problem

- Tesla V100 requires 16GB BAR allocation above the 4GB boundary
- Gigabyte X99 consumer boards don't have Above 4G Decoding option
- Without proper MMIO allocation, V100 won't POST or shows `<ignored>` memory regions

## Solutions Implemented

### 1. DSDT Override (Primary Method) - NO BIOS FLASH
Modifies ACPI tables via GRUB to expose 64-bit MMIO region above RAM.

**Files:**
- `dsdt-original/` - Original DSDT from system
- `dsdt-modified/` - Patched DSDT with 64-bit memory region

**Changes Made:**
- Added `_Y11` label to PCI0 QWordMemory
- Modified `_CRS` method to expose memory from 132GB to 64TB (46-bit limit)
- Kernel parameters: `pci=realloc pci=nocrs pci=assign-busses`

### 2. UEFI Patching (Alternative - Requires BIOS Flash)
Patch BIOS directly to extend MMIO space to 512GB.

**Files:**
- `uefi-tools/` - UEFIPatch and UEFITool binaries
- `uefi-patches/` - Haswell/Broadwell patches from ReBarUEFI

### 3. ReBarDxe Module (Full ReBAR - Requires BIOS Flash)
DXE driver for complete Resizable BAR support.

**Files:**
- `uefi-patches/ReBarDxe.ffs` - Ready to insert into BIOS

## Quick Start - DSDT Override

```bash
# Copy modified DSDT to boot
cp dsdt-modified/DSDT_modified.aml /boot/DSDT.aml

# Create GRUB ACPI loader script
echo '#!/bin/sh
if [ -f /boot/DSDT.aml ]; then
    echo "acpi /boot/DSDT.aml"
fi' > /etc/grub.d/01_acpi
chmod +x /etc/grub.d/01_acpi

# Add kernel parameters to GRUB
# Edit /etc/default/grub and add to GRUB_CMDLINE_LINUX_DEFAULT:
# pci=realloc pci=nocrs pci=assign-busses

# Update GRUB and reboot
update-grub
reboot
```

## Verify After Reboot

```bash
# Check for 64-bit memory region
dmesg | grep "root bus resource"
# Should show: [mem 0x2100000000-0x3fffffffffff window]

# Check GPU BAR allocation (after installing GPU)
lspci -vvv | grep -A5 "NVIDIA"
```

## File Structure

```
x99-udp4-rebar/
├── README.md
├── APPLIED_CHANGES.md          # Detailed DSDT modification notes
├── BIOS_MODDING_GUIDE.md       # Full UEFI patching guide
├── dsdt-original/
│   ├── DSDT.aml                # Original binary
│   └── DSDT.dsl                # Original decompiled
├── dsdt-modified/
│   ├── DSDT_modified.aml       # Patched binary
│   └── DSDT_modified.dsl       # Patched source
├── uefi-tools/
│   ├── UEFIPatch               # BIOS patching tool
│   └── uefitool                # BIOS analysis tool
└── uefi-patches/
    ├── HswAbove4G.txt          # Haswell/Broadwell MMIO patches
    ├── patches.txt             # BAR size limit patches
    └── ReBarDxe.ffs            # ReBAR DXE module
```

## References

- [ReBarUEFI](https://github.com/xCuri0/ReBarUEFI) - Main project for ReBAR on older systems
- [DSDT Patching Wiki](https://github.com/xCuri0/ReBarUEFI/wiki/DSDT-Patching)
- [Miyconst X99 Tutorial](https://www.youtube.com/watch?v=vcJDWMpxpjE)

## Credits

- [@xCuri0](https://github.com/xCuri0) for ReBarUEFI project and patches
- [@godcrying](https://github.com/xCuri0/ReBarUEFI/issues) for the DSDT modification concept
