# Firmware inputs from the handset

Mu-Silicium cannot build this board without the Qualcomm DXE modules that ship inside
the phone's XBL. They are not published here and must be extracted from the user's own
device. This page lists what to extract and where each file goes.

## Source

| Item | Value |
| --- | --- |
| XBL container | `BOOT.XF.3.2-00304-SM8250-2` (the lmi boot chain) |
| Extract with | the public `UEFIReader` tool, or the `inventory_firmware.py` / `audit-lmi-firmware.py` pair in the public lmi UEFI project |
| Destination | `<Mu-Silicium>/Binaries/lmi/` |

Do not reuse another phone's `Binaries` tree, even when the version string matches: the
modules are built per device, and the audit tools reject a mismatched set.

## Modules referenced by `Include/DXE.inc`

`QcomPkg/Drivers/`:

```text
TzDxe/ScmDxeLA.inf              TzDxe/TzDxeLA.inf
ChipInfoDxe                     SmemDxe              ULogDxe
NpaDxe                          CmdDbDxe             PwrUtilsDxe
RpmhDxe                         PdcDxe               VcsDxe
ClockDxe                        SPMIDxe              ShmBridgeDxe
TLMMDxe                         ResetRuntimeDxe      PmicDxe/PmicDxeLa.inf
DALSYSDxe                       HALIOMMUDxe          HWIODxe
GpiDxe                          I2CDxe               SPIDxe
SdccDxe                         UFSDxe               FeatureEnablerDxe
PlatformInfoDxe                 ButtonsDxe           ChargerExDxe
QcomChargerDxe/QcomChargerDxeLA.inf                  UsbPwrCtrlDxe
AdcDxe                          TsensDxe             LimitsDxe
DDRInfoDxe                      UsbfnDwc3Dxe         XhciPciEmulationDxe
XhciDxe                         UsbMsdDxe            UsbDeviceDxe
UsbConfigDxe                    UsbInitDxe           HashDxe
CipherDxe                       MacDxe               RNGDxe
PILDxe                          PILProxyDxe          SPSSDxe
EnvDxe
```

`MdeModulePkg/Universal/Disk/`:

```text
DiskIoDxe/DiskIoDxe.inf         PartitionDxe/PartitionDxe.inf
```

Other paths used by `APRIORI.inc` / `DXE.inc` (for example
`Binaries/lmi/MdeModulePkg/...`) come from the same tree.

## Record

For every extracted file, record its SHA-256 next to the module name in this page once a
set is known to belong to the handset. A blob that appears in the firmware volume but
cannot be identified must not be added.

## Checks before flashing

1. `./unixnu build firmware --uefi ../uefi-lmipro` produces the FD at the path in
   `board.toml` (`Build/lmiPkg/<target>_CLANGPDB/FV/SILICIUM_UEFI.fd`).
2. Its size equals `[uefi_fd] size` in `Resources/Configs/lmi.toml`.
3. `unixnu bootimg` accepts the volume and reports a `ramdisk_bytes` greater than zero
   when the payload is present.
