# uefi-lmipro

Mu-Silicium UEFI firmware port for the Xiaomi Redmi K30 Pro / K30 Pro Zoom Edition
(`lmi` / `lmipro`, Qualcomm SM8250 / kona), for UniXNU's `/zh/porting/uefi-firmware`
chain.

This repository is a **board overlay**, not a fork of Mu-Silicium. It carries only
the files a device adds on top of the shared tree:

| Path | Contents |
| --- | --- |
| `Platforms/Xiaomi/lmiPkg/` | `lmi.dsc`, `lmi.fdf`, `lmi.dec`, `Include/*.inc`, `Library/MemoryMapLib`, `Library/ConfigurationMapLib` |
| `BootShim/BootShim.S` | the shim that copies the FD to its load address |
| `Resources/Configs/lmi.toml` | FD base/size and Android boot image parameters |

## Baseline

| Item | Value |
| --- | --- |
| Mu-Silicium | `e51cd877e31c8fbb322032dc4cb6715e83d25ad3` |
| Mu_Basecore | `cfb93724a06ff86ff5ba971413368a7809ee42a5` |
| SOC_TYPE | `0` (SM8250) |
| FD base / size | `0x9FC00000` / `0x300000` (historical lmi values, see limits) |

The lmi platform files were taken from the public lmi UEFI overlay and keep their
BSD-2-Clause headers. `MemoryMapLib.c` and `ConfigurationMapLib.c` still say
"firmware/runtime validation is pending"; that is accurate for this repository too.

## Assembly

```sh
git clone https://github.com/Project-Silicium/Mu-Silicium uefi-lmipro
git -C uefi-lmipro checkout e51cd877e31c8fbb322032dc4cb6715e83d25ad3
# then copy this repository's Platforms/, BootShim/ and Resources/ over that tree
```

`Binaries/` in the Mu-Silicium tree is a submodule; for this board it must contain the
modules extracted from the handset's own XBL, see
[docs/firmware-inputs.md](docs/firmware-inputs.md). This repository never ships them.

## UniXNU integration

`lmi.dsc` and `lmi.fdf` add what UniXNU requires on top of the stock lmi package:

- `[Components]`: `UniXNURamDiskDxe`, `UniXNUDxe`, `UniXNU`, `DarwinBoot`
- `[LibraryClasses]`: `HobLib|MdePkg/Library/DxeHobLib/DxeHobLib.inf`,
  `FdtLib|MdePkg/Library/BaseFdtLib/BaseFdtLib.inf`
- `lmi.fdf`: the four modules are placed before `Include/DXE.inc`, so they are in the
  volume before `BdsDxe` starts BDS

DiskIoDxe, PartitionDxe, Fat and EnglishDxe are already in `Include/DXE.inc`
(the first two from `Binaries/lmi/MdeModulePkg/`), so the payload FAT volume is
readable without adding duplicates.

## Build

From the UniXNU checkout, with this repository as `--uefi`:

```sh
./unixnu build firmware --uefi ../uefi-lmipro
```

`board.toml` in the device repository expects
`Build/lmiPkg/{target}_CLANGPDB/FV/SILICIUM_UEFI.fd`, which matches the DSC's
`OUTPUT_DIRECTORY`.

## Status and limits

- **Not built, not booted.** No FD has been produced from this layout.
- `MemoryMapLib`/`ConfigurationMapLib` are the historical lmi map, not a measurement on
  this handset; confirm the reserved ranges and the FD address from the device.
- No `Binaries/lmi` content is present; the firmware cannot build until the XBL modules
  are extracted.
- `Resources/Configs/lmi.toml` uses the historical FD base/size and boot image header
  parameters; confirm them from the phone's own boot image before flashing.

## License

BSD-2-Clause-Patent for the Mu-Silicium derived files; each file keeps its original
header.
