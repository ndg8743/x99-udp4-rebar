# Patched BIOS for Gigabyte GA-X99-UD4P

## GIGABYTE.BIN

Patched F23c BIOS with 64GB BAR limit removed from PciBus module.

### Changes Applied
- **Module**: PciBus (GUID: 3C1DE39F-D207-408A-AACC-731CFB7F1DD7)
- **Original**: `mov rax, 0x8FFFFFFFF` (64GB limit)
- **Patched**: `mov rax, 0xFFFFFFFFFFFFFFFF` (no limit)

### Checksums
| File | MD5 |
|------|-----|
| Original F23c | d61aa388b6fb7617d70ffbde8bd13aff |
| GIGABYTE.BIN (patched) | b1e532e1a0069a61888421dad7c260ba |

### How to Flash
1. Copy `GIGABYTE.BIN` to a FAT32 USB drive root
2. Boot into BIOS (Press DEL during POST)
3. Use Q-Flash utility to flash from USB
4. Select GIGABYTE.BIN and confirm

### WARNING
Flashing a modified BIOS carries risk. Keep the original BIOS backup and a way to recover (dual BIOS, hardware flasher, etc.).
