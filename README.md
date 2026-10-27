# OpenCore EFI for the Acer Nitro 5 AN515-57

My working OpenCore EFI and kexts for the **Acer Nitro 5 AN515-57** (i5-11400H, RTX 3060 Laptop
GPU, Intel Wi-Fi, ALC295) running **macOS 26.7.1 (Tahoe) with OpenCore 1.0.9 (DEBUG)**. Kept as
a backup and shared for other AN515-57 owners.

> [!IMPORTANT]
> **Only tested on the AN515-57.** The ACPI tables (including a patched `DSDT.aml`), USB map and
> kext choices are specific to this model. Do not use it on another laptop as is.

For fan control, keyboard RGB and the rest of NitroSense on macOS, see
[NitroSense-macOS](https://github.com/ptnzzn/NitroSense-macOS).

## Layout

```
EFI/
  BOOT/BOOTx64.efi
  OC/
    OpenCore.efi, config.plist
    ACPI/      DSDT.aml and SSDTs (SSDT-NITRO.aml is NitroSense's)
    Drivers/   HfsPlus.efi, OpenRuntime.efi, ResetNvramEntry.efi
    Kexts/     see the table below
    Resources/ OpenCanopy theme files
    Tools/     NullMothSafe.efi
LibraryExtensions/   NullMoth NVIDIA kexts for /Library/Extensions (+ licences)
```

## Before you boot it

- **SMBIOS is removed.** Generate your own `MacBookPro16,2` serial, MLB and UUID with
  [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS), fill `PlatformInfo > Generic`
  (`SystemSerialNumber`, `MLB`, `SystemUUID`) and set `ROM` to your Ethernet MAC address. Never
  reuse someone else's serial.
- `boot-args` carry NullMoth (`nvfb=1 nvaccel=1 ...`), AMFI and debug flags. Drop the `nv*` ones
  and disable `Misc > Tools > NullMothSafe.efi` if you do not use NullMoth.
- `csr-active-config` relaxes SIP so the unsigned kexts in `/Library/Extensions` can load.
- `itlwm.kext` ships with an empty `WiFiConfig`; `AirportItlwm` is the one enabled.
- Test from a USB stick first and keep a working EFI around.

## Kexts (`EFI/OC/Kexts`)

Listed in the order of `Kernel > Add` in `config.plist`.

| Kext | Version | Enabled | Kernel range | Upstream |
| --- | --- | --- | --- | --- |
| Lilu | 1.7.3 | yes | | [acidanthera/Lilu](https://github.com/acidanthera/Lilu) |
| BTProcFixup | 4.0.0 | yes | 21– | local build (`local.btprocfixup`) |
| PC711Probe | 1.8.1 | yes | 20– | stationk9 (`com.stationk9.driver.PC711Probe`) |
| AMFIPass | 1.4.1 | yes | 20– | dhinakg (`com.dhinakg.AMFIPass`, shipped with OCLP) |
| AppleALC | 1.9.9 | yes | | [acidanthera/AppleALC](https://github.com/acidanthera/AppleALC), **custom build** with ALC295 mic patches |
| SimpleHDA | 1.0.0 | yes | 25–25.99 | laobamac (`cn.laobamac.SimpleHDA`) |
| BlueToolFixup | 2.7.3 | no | 21– | [acidanthera/BrcmPatchRAM](https://github.com/acidanthera/BrcmPatchRAM) |
| BrightnessKeys | 1.0.4 | yes | | [acidanthera/BrightnessKeys](https://github.com/acidanthera/BrightnessKeys) |
| IntelBTPatcher | 2.5.0 | yes | 21– | [OpenIntelWireless/IntelBluetoothFirmware](https://github.com/OpenIntelWireless/IntelBluetoothFirmware) |
| IntelBluetoothFirmware | 2.5.0 | yes | | [OpenIntelWireless/IntelBluetoothFirmware](https://github.com/OpenIntelWireless/IntelBluetoothFirmware) |
| NVMeFix | 1.1.4 | yes | 18– | [acidanthera/NVMeFix](https://github.com/acidanthera/NVMeFix) |
| RealtekRTL8111 | 2.4.2 | yes | | [Mieze/RTL8111_driver_for_OS_X](https://github.com/Mieze/RTL8111_driver_for_OS_X) |
| RestrictEvents | 1.1.7 | yes | | [acidanthera/RestrictEvents](https://github.com/acidanthera/RestrictEvents) |
| VirtualSMC + SMCBatteryManager, SMCLightSensor, SMCProcessor, SMCSuperIO | 1.3.9 | yes | | [acidanthera/VirtualSMC](https://github.com/acidanthera/VirtualSMC) |
| USBToolBox + UTBMap | 1.2.0 / 1.0 | yes | | [USBToolBox/kext](https://github.com/USBToolBox/kext); `UTBMap` is the AN515-57 port map |
| UTBDefault | 1.0 | not in config | | USBToolBox default map (unused) |
| VoodooI2C (+ VoodooInput, VoodooI2CServices, VoodooGPIO) | 2.9.1 | yes | | [VoodooI2C/VoodooI2C](https://github.com/VoodooI2C/VoodooI2C) |
| VoodooI2CHID | 1.0 | yes | | [VoodooI2C/VoodooI2C](https://github.com/VoodooI2C/VoodooI2C) |
| VoodooPS2Controller (Keyboard, Mouse, Trackpad; its own VoodooInput off) | 2.3.8 | yes | | [acidanthera/VoodooPS2](https://github.com/acidanthera/VoodooPS2) |
| WhateverGreen | 1.7.2 | yes | | [acidanthera/WhateverGreen](https://github.com/acidanthera/WhateverGreen) |
| XHCI-unsupported | 0.9.2 | yes | | [RehabMan/OS-X-USB-Inject-All](https://github.com/RehabMan/OS-X-USB-Inject-All) |
| itlwm | 2.3.0 | no | | [OpenIntelWireless/itlwm](https://github.com/OpenIntelWireless/itlwm) |
| AirportItlwm | 2.4.0 | yes | 25–25.99 | [OpenIntelWireless/itlwm](https://github.com/OpenIntelWireless/itlwm) (Tahoe build) |

## NVIDIA (`LibraryExtensions/`)

NullMoth NVIDIA driver 2.5.5 kexts for the RTX 3060 Laptop GPU: `NVRM`, `NVRMFB`, `NVRMAGDC`,
`NVAccel`, plus `EFI/OC/Tools/NullMothSafe.efi`. These are only the kernel parts; Metal
acceleration also needs the rest of the NullMoth package (`/Library/NullMoth`, its LaunchDaemon
and bundles), so install it with NullMoth's own installer rather than copying these by hand.

Licensed under the **PolyForm Noncommercial 1.0.0** licence (non-commercial use only), with
NVIDIA open GPU kernel modules (MIT), NVIDIA firmware, Mesa and metal2vulkan components; the
licence texts are in [`LibraryExtensions/licenses`](LibraryExtensions/licenses).

Required Notice: Copyright (c) 2026 NullMoth Systems (https://nullmothsystems.com)

## Licences

Everything here is third-party (OpenCore, acidanthera, OpenIntelWireless, VoodooI2C,
USBToolBox, NullMoth and others) and keeps its upstream licence; see the links above. This repo
only collects them with my configuration. Acer and Nitro are trademarks of Acer Inc.; this
project is not affiliated with Acer or Apple.
