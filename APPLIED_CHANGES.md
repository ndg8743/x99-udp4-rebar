# Applied DSDT Changes for X99-UD4P

## System
- Motherboard: Gigabyte GA-X99-UD4P-CF
- BIOS: F23c (AMI)
- CPU: Intel Xeon (Broadwell-EP)
- RAM: 128GB DDR4

## Changes Made

### 1. Added label to PCI0 QWordMemory (line 15573)
Changed:
```
,, , AddressRangeMemory, TypeStatic)
```
To:
```
,, _Y11, AddressRangeMemory, TypeStatic)
```

### 2. Modified PCI0 _CRS Method (line 15685)
Changed from simple return to dynamic 64-bit memory configuration:

```asl
Method (_CRS, 0, Serialized)
{
    CreateQWordField (P0RS, \_SB.PCI0._Y11._MIN, M2MN)
    CreateQWordField (P0RS, \_SB.PCI0._Y11._MAX, M2MX)
    CreateQWordField (P0RS, \_SB.PCI0._Y11._LEN, M2LN)

    M2MN = 0x0000002100000000  // 132GB (above 128GB RAM)
    M2MX = 0x00003FFFFFFFFFFF  // 46-bit limit
    M2LN = (M2MX - M2MN) + One

    Return (P0RS)
}
```

## Kernel Parameters Added
```
pci=realloc pci=nocrs pci=assign-busses
```

## Expected Result
After reboot, dmesg should show 64-bit memory region for large-BAR GPUs.
