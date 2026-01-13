# X99-UD4P BIOS Modding Guide for Large BAR GPUs

## Overview

This guide covers multiple methods to enable large BAR support (>4GB) for Tesla V100/M40/P40 GPUs on Gigabyte GA-X99-UD4P.

## Methods (In Order of Preference)

### Method 1: DSDT Override (Linux Only, No Flash) ✅ DONE
- Modify ACPI tables via GRUB bootloader
- Already implemented in `dsdt-modified/`
- Safe, reversible, no BIOS flash required

### Method 2: UEFIPatch (Flash Required)
- Patch BIOS to extend MMIO space to 512GB
- Works with existing BIOS, just patched

### Method 3: ReBarDxe Module (Flash Required)
- Add DXE driver for full Resizable BAR support
- Requires Above 4G Decoding to work optimally

---

## Method 2: UEFIPatch Instructions

### Step 1: Dump Current BIOS
```bash
# Install flashrom
apt install flashrom

# Dump current BIOS (backup!)
flashrom -p internal -r bios_backup.bin

# Make a working copy
cp bios_backup.bin bios_patched.bin
```

### Step 2: Apply Haswell/Broadwell Patches
```bash
cd /path/to/uefi-patches

# Apply HswAbove4G patches (expands MMIO to 512GB)
../uefi-tools/UEFIPatch bios_patched.bin HswAbove4G.txt

# Check output - should say patches applied
```

### Step 3: Apply BAR Size Limit Patches
```bash
# Apply general patches for BAR size limits
../uefi-tools/UEFIPatch bios_patched.bin patches.txt
```

### Step 4: Verify Patches
```bash
# Use UEFITool to verify changes
../uefi-tools/uefitool bios_patched.bin

# Compare file sizes - patched should be same size
ls -la bios_backup.bin bios_patched.bin
```

### Step 5: Flash Patched BIOS
```bash
# WARNING: This can brick your system!
# Have recovery method ready (CH341A programmer, etc)

flashrom -p internal -w bios_patched.bin
```

---

## Method 3: Add ReBarDxe Module

### Step 1: Extract DXE Volume
Use UEFITool GUI to:
1. Open BIOS file
2. Find DXE Volume (usually largest)
3. Right-click → Insert after... → ReBarDxe.ffs

### Step 2: Flash and Configure
```bash
# Flash modified BIOS
flashrom -p internal -w bios_with_rebar.bin

# After reboot, set ReBAR size (on Linux)
./ReBarState 32  # Unlimited BAR size
```

---

## Patches Explained

### HswAbove4G.txt
| Patch | Effect |
|-------|--------|
| NBPEI 36→39 bit | Extends memory addressing from 64GB to 512GB |
| PciHostBridge MMIO | Expands MMIO window to use full 512GB space |

### patches.txt
| Patch | Effect |
|-------|--------|
| <4GB BAR limit | Removes artificial 4GB BAR limit |
| <16GB BAR limit | Allows BARs up to 16GB |
| <64GB BAR limit | Allows BARs up to 64GB |
| 64-bit downgrade prevention | Stops BIOS from converting 64-bit BARs to 32-bit |

---

## Recovery Methods

### If BIOS Bricked:
1. **Dual BIOS** - X99-UD4P has backup BIOS, switch and reflash
2. **CH341A Programmer** - Direct SPI flash programming
3. **Q-Flash Plus** - USB recovery (if supported)

### Rollback DSDT Override:
```bash
rm /boot/DSDT.aml
rm /etc/grub.d/01_acpi
cp /etc/default/grub.backup /etc/default/grub
update-grub
reboot
```

---

## References

- [ReBarUEFI Wiki](https://github.com/xCuri0/ReBarUEFI/wiki)
- [DSDT Patching Guide](https://github.com/xCuri0/ReBarUEFI/wiki/DSDT-Patching)
- [UEFIPatch Guide](https://github.com/xCuri0/ReBarUEFI/wiki/Using-UEFIPatch)
- [Miyconst X99 Tutorial](https://www.youtube.com/watch?v=vcJDWMpxpjE)
