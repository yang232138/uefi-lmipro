# UEFI agent guide

This repository owns the lmi board package for Mu-Silicium: the DSC/FDF, the memory
and configuration maps, the BootShim, and the device configuration. It does not own
the shared Mu-Silicium tree or the vendor binaries.

## Read first

- [Firmware integration](https://unixnu.com/zh/porting/uefi-firmware), especially
  "required integration" and "payload discovery"
- [Payload](https://unixnu.com/zh/concepts/payload)
- [Boot image](https://unixnu.com/zh/guide/boot)
- The Mu-Silicium `Building.md` at the pinned baseline commit

## Rules

- Keep the upstream file headers. Every Mu-Silicium derived file is BSD-2-Clause-Patent.
- Do not add `Binaries/` content to Git. Vendor DXE modules come from the handset's own
  XBL and are recorded by hash in `docs/firmware-inputs.md`.
- `lmi.dsc` and `lmi.fdf` must keep the four UniXNU modules listed before `BdsDxe`:
  `UniXNURamDiskDxe`, `UniXNUDxe`, `UniXNU`, `DarwinBoot`.
- The memory map is board data. Change it only with an observation from the device
  (bootloader memory map or UniXNU firmware log), and record the observation.
- `Resources/Configs/lmi.toml` values are compile inputs for BootShim and `unixnu
  bootimg`; a wrong FD base or size produces an image the bootloader rejects.

## Verify

```sh
./unixnu build firmware --uefi ../uefi-lmipro
```

A successful run reports the FD path and its size. Compare the size with
`[uefi_fd] size` in `Resources/Configs/lmi.toml`; `unixnu bootimg` checks it again.
