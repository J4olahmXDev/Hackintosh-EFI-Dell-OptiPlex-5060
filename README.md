# 🖥️ Hackintosh EFI — Dell OptiPlex 5060
> OpenCore EFI for macOS Tahoe on Dell OptiPlex 5060 (Coffee Lake)

![macOS](https://img.shields.io/badge/macOS-Tahoe_26.4-white?style=flat-square&logo=apple)
![OpenCore](https://img.shields.io/badge/OpenCore-Latest-blue?style=flat-square)
![Status](https://img.shields.io/badge/Status-Working-brightgreen?style=flat-square)

---

## 🧰 Hardware Specifications

| Component | Details |
|-----------|---------|
| **Model** | Dell OptiPlex 5060 |
| **CPU** | Intel Core i5-8500 (Coffee Lake, 6C/6T) |
| **GPU** | AMD Radeon Pro WX 2100 — Spoofed as RX 560 (`device-id: FF67`) |
| **RAM** | 8 GB DDR4 |
| **Storage** | M.2 NVMe Gen3 256 GB + HDD 1 TB |
| **WiFi / BT** | USB WiFi — RtWlanU + RtWlanU1827 · BT via BlueToolFixup |
| **LAN** | Intel I219 — IntelMausi.kext |
| **Audio** | AppleALC — layout-id: 66 (`alcid=66`) |
| **SMBIOS** | `Macmini8,1` |

---

## ✅ What's Working

| Feature | Status |
|---------|--------|
| CPU Power Management (XCPM + CPUFriend) | ✅ Working |
| GPU Acceleration (WX 2100 → RX 560 spoof) | ✅ Working |
| Audio (AppleALC layout-id 66) | ✅ Working |
| WiFi (USB Realtek) | ✅ Working |
| Bluetooth (BlueToolFixup) | ✅ Working |
| LAN (IntelMausi) | ✅ Working |
| NVMe Trim (NVMeFix) | ✅ Working |
| Sleep / Wake | ✅ Working |
| USB Ports (USBMap) | ✅ Working |
| iCloud / iMessage / FaceTime | ✅ Working (requires own SMBIOS) |

---

## ❌ What's Not Working / Notes

| Feature | Status |
|---------|--------|
| AirDrop / Handoff | ⚠️ Depends on USB WiFi adapter |
| DRM (Apple TV+, Netflix Safari) | ⚠️ Partial via OCLP |

---

## 🔧 GPU Device-ID Spoof (WX 2100 → RX 560)

WX 2100 ไม่มี native driver ใน macOS Tahoe — ต้อง spoof เป็น RX 560

**DeviceProperties:**

| Property | Value | Notes |
|----------|-------|-------|
| `device-id` | `ff67` | Spoof เป็น RX 560 |
| `@0,name` | `ATY,Acre` | GPU name injection |
| `model` | `AMD Radeon Pro WX 2100` | Display name |

**PCI Path:** `PciRoot(0x0)/Pci(0x1,0x0)/Pci(0x0,0x0)`

---

## 📦 ACPI

**SSDT (Active):**

| File | Status | Description |
|------|--------|-------------|
| `SSDT_ALL_For_DellOptiplex5060_With_Wx2100.aml` | ✅ ON | SSDT หลัก: EC, PLUG, USBX, PMCR, MCHC, SBUS, HPET, WX2100 spoof |
| `DSDT_ForD5060_Wx2100.aml` | ❌ OFF | DSDT แบบ patch โดยตรง (สำรอง) |

**Binary Patches:**

| Patch | Status |
|-------|--------|
| HPET `_STA` → `XSTA` Rename | ✅ Enabled |
| HPET `_CRS` → `XCRS` Rename | ✅ Enabled |

---

## 🔑 Kexts

| Kext | Status | หน้าที่ |
|------|--------|--------|
| Lilu | ✅ | Base patcher framework |
| VirtualSMC | ✅ | SMC emulation |
| WhateverGreen | ✅ | GPU patch + spoof |
| AppleALC | ✅ | Audio (layout-id 66) |
| IntelMausi | ✅ | Intel I219 LAN |
| NVMeFix | ✅ | NVMe power + trim fix |
| RestrictEvents | ✅ | CPU branding + `revpatch` |
| CPUFriend | ✅ | CPU frequency scaling |
| CPUFriendDataProvider | ✅ | CPUFriend data |
| AMFIPass | ✅ | AMFI bypass สำหรับ OCLP |
| BlueToolFixup | ✅ | Bluetooth fix (macOS 12+) |
| RtWlanU | ✅ | USB WiFi Realtek |
| RtWlanU1827 | ✅ | USB WiFi Realtek firmware |
| USBMap | ✅ | USB port mapping |
| XHCI-unsupported | ✅ | XHCI controller fix |
| SMCProcessor | ✅ | CPU temperature |
| SMCSuperIO | ✅ | Fan/sensor data |
| USBInjectAll | ❌ Off | แทนด้วย USBMap |
| USBToolBox / UTBMap | ❌ Off | ไม่ใช้ในชุดนี้ |

---

## 🖥️ iGPU (Headless)

| Property | Value | Notes |
|----------|-------|-------|
| `AAPL,ig-platform-id` | `0300913E` | Headless iGPU (Coffee Lake) |

**PCI Path:** `PciRoot(0x0)/Pci(0x2,0x0)`

> iGPU ถูกตั้งเป็น headless เพราะใช้ WX 2100 เป็น primary display output

---

## 🔊 Audio

| Property | Value |
|----------|-------|
| **layout-id** | `66` (boot-arg `alcid=66`) |
| **PCI Path** | `PciRoot(0x0)/Pci(0x17,0x0)` |
| **compatible** | `pci8086,a182` |

---

## 🥾 Boot Arguments

```
-v alcid=66 watchdog=0 agdpmod=pikera dk.e1000=0 e1000=0 radpg=15 amfi=0x80 -no_compat_check revpatch=sbvmm,asset,f16c
```

| Argument | หน้าที่ |
|----------|--------|
| `alcid=66` | AppleALC layout ID |
| `agdpmod=pikera` | GPU display fix สำหรับ AMD |
| `radpg=15` | AMD GPU power gating fix |
| `amfi=0x80` | AMFI flag สำหรับ AMFIPass |
| `-no_compat_check` | Bypass macOS compatibility check |
| `revpatch=sbvmm,asset,f16c` | RestrictEvents patches |
| `dk.e1000=0 e1000=0` | Disable e1000 driver conflict |
| `watchdog=0` | Disable watchdog timer |

---

## ⚙️ Kernel Quirks

| Quirk | Status |
|-------|--------|
| AppleXcpmCfgLock | ✅ |
| CustomSMBIOSGuid | ✅ |
| DisableIoMapper | ✅ |
| DisableLinkeditJettison | ✅ |
| DisableRtcChecksum | ✅ |
| PanicNoKextDump | ✅ |
| PowerTimeoutKernelPanic | ✅ |
| SetApfsTrimTimeout | `-1` |

## ⚙️ Booter Quirks

| Quirk | Status |
|-------|--------|
| AvoidRuntimeDefrag | ✅ |
| DevirtualiseMmio | ✅ |
| EnableSafeModeSlide | ✅ |
| EnableWriteUnprotector | ✅ |
| ProtectUefiServices | ✅ |
| ProvideCustomSlide | ✅ |
| SetupVirtualMap | ✅ |
| SyncRuntimePermissions | ✅ |

---

## ⚙️ BIOS Settings

**Disable:**
- Secure Boot
- Fast Boot
- CFG Lock (ใช้ `AppleXcpmCfgLock` quirk แทน)
- VT-d (ใช้ `DisableIoMapper` quirk แทน)

**Enable:**
- UEFI Boot Mode
- SATA Mode: AHCI
- Above 4G Decoding

---

## 🚀 Installation

1. สร้าง macOS Tahoe installer USB ด้วย `createinstallmedia`
2. วาง EFI folder นี้บน EFI partition ของ USB
3. **Generate SMBIOS ใหม่** ด้วย [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) — SMBIOS: `Macmini8,1`
4. Boot จาก USB → เลือก macOS Installer
5. หลัง install เสร็จ mount EFI ของ NVMe และวาง EFI folder

---

## ⚠️ Disclaimer

> กรุณา **Generate ค่าต่อไปนี้ใหม่เสมอ** ก่อนนำ EFI ไปใช้:
> - `SystemSerialNumber`
> - `MLB`
> - `SystemUUID`
> - `ROM`
>
> ใช้ [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) หรือ [OCAuxiliaryTools](https://github.com/ic005k/OCAuxiliaryTools)
>
> การใช้ Serial Number ซ้ำกันอาจทำให้ iCloud / iMessage ถูก ban ได้

---

## 📚 References

- [Dortania — Desktop Coffee Lake Guide](https://dortania.github.io/OpenCore-Install-Guide/config.plist/coffee-lake.html)
- [OpenCore Legacy Patcher (OCLP)](https://github.com/dortania/OpenCore-Legacy-Patcher)
- [Acidanthera Kexts](https://github.com/acidanthera)
- [RestrictEvents](https://github.com/acidanthera/RestrictEvents)

---

*Made with ❤️ by [suanaph](https://github.com/J4olahmXDev) · Bangkok, TH*
