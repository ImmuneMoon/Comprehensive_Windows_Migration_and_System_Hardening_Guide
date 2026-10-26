# Comprehensive Windows Migration & System Hardening Guide

![Windows 11](https://img.shields.io/badge/Windows-11-0078D4?logo=windows&logoColor=white)
![Migration: HDD to NVMe SSD](https://img.shields.io/badge/Migration-HDD%20%E2%86%92%20NVMe%20SSD-6f42c1)
![Hardening: BitLocker, Secure Boot, TPM](https://img.shields.io/badge/Hardening-BitLocker%20%7C%20Secure%20Boot%20%7C%20TPM-d73a49)
[![Last updated](https://img.shields.io/github/last-commit/ImmuneMoon/Comprehensive_Windows_Migration_and_System_Hardening_Guide?label=updated)](https://github.com/ImmuneMoon/Comprehensive_Windows_Migration_and_System_Hardening_Guide/commits/main)
[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey)](LICENSE)

A step-by-step walkthrough for moving a Windows install off an old mechanical hard drive onto a new NVMe M.2 SSD — safely, without a reinstall — then optimizing, backing up, and hardening the fresh system. It's written as a complete worked example on one real build, with **"Adapting to your hardware"** notes throughout so it applies to almost any Windows desktop.

## Read the guide

| File | What it is |
|---|---|
| [Comprehensive_Windows_Migration_and_System_Hardening_Guide.md](Comprehensive_Windows_Migration_and_System_Hardening_Guide.md) | The full guide. Start here. |
| [ASRock_A520M-HVS_BIOS_Setup_Checklist.md](ASRock_A520M-HVS_BIOS_Setup_Checklist.md) | Companion one-pass BIOS/UEFI checklist for the example board (ASRock A520M-HVS). Board-specific, but easy to adapt to yours. |

## What's covered

- **Phase 1 — Hardware pre-flight:** safe physical install of the M.2 SSD and disk initialization (GPT).
- **Phase 2 — Health check:** repairing the old drive *before* cloning (CHKDSK, DISM, SFC) and confirming your data will fit.
- **Phase 3 — Migration:** a bit-level OS clone to the new SSD using a free, drive-agnostic tool.
- **Phase 4 — Boot & drive letters:** setting the SSD as the boot device, plus the disk-signature isolation trick.
- **Phase 5 — Space reclamation:** disabling hibernation / Fast Startup and re-routing user folders to a secondary drive.
- **Phase 6 — Backups:** automated weekly local backups (SSD → HDD).
- **Phase 7 — Cleanup & hardening:** decommissioning the old drive, UPS auto-shutdown, and a full Windows 11 security pass — TPM/Secure Boot, Defender, drive encryption, ransomware resilience, and attack-surface reduction.
- **Phase 8 — Gaming:** keeping game libraries off the boot SSD and optional Storage Spaces pooling.

## Who it's for

Anyone doing an HDD-to-SSD migration on a Windows desktop who wants to keep their existing install rather than start over. The example build is a **SIX X4000 512GB NVMe** on an **ASRock A520M-HVS** with a **CyberPower CP1500PFCLCD** UPS, but every hardware-specific step is flagged with what to check on your own machine, so the procedure carries over to other builds.


## ☕ Support the Project

If you find this project helpful and want to support further development by Fulllion Creative Works, consider leaving a tip!

* [Donate via PayPal](https://www.paypal.com/donate/?hosted_button_id=LCDZX75HR4CLC)
* [Support on Ko-fi](https://ko-fi.com/fulllion)

---
© 2026 Fulllion Creative Works. Released under the MIT License.
