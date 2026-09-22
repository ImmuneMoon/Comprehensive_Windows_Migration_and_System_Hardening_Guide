# ASRock A520M-HVS — BIOS/UEFI Setup Checklist

A companion to the *Comprehensive Windows Migration & System Hardening Guide*. This is one pass through the UEFI to lock in the boot drive and the security settings after your NVMe migration. Work top to bottom — the order matters (some toggles depend on earlier ones).

---

## Before you start

- [ ] **Back up your BitLocker/Device Encryption recovery key.** If drive encryption is on (common on Windows 11 24H2/25H2), changing the fTPM or Secure Boot settings below will demand the 48-digit key at next boot. View and save it from https://aka.ms/myrecoverykey **before** touching anything. Skipping this can lock you out of your own drive.
- [ ] Know your keys: **F2** or **Delete** at boot to enter UEFI · **F7** to toggle Advanced Mode · **F10** to save and exit.

---

## Entering the BIOS

- [ ] Restart and tap **F2 / Delete** repeatedly until the ASRock UEFI appears.
- [ ] Press **F7** to switch from EZ Mode into **Advanced Mode**.

---

## 1. Boot drive (from Phase 4 of the guide)

- [ ] Go to the **Boot** tab.
- [ ] Set **Boot Option #1** → your **SIX X4000 NVMe SSD** (not the old HDD).
- [ ] If the old HDD keeps hijacking boot, use the isolation method in the guide (physically unplug its SATA cable for the first boot).

---

## 2. Storage sanity check

- [ ] **Advanced → Storage Configuration** (or the SATA/NVMe section): confirm the **SIX X4000 NVMe** is detected and listed.
- [ ] Confirm **SATA Mode = AHCI** (the default — leave it; do not switch to RAID).

---

## 3. TPM — required for Windows 11 & encryption

- [ ] **Advanced → CPU Configuration → AMD fTPM switch** → set to **AMD CPU fTPM**.
- [ ] (This enables the on-board TPM 2.0 that Windows 11 and drive encryption depend on.)

---

## 4. UEFI-only boot & Secure Boot

Secure Boot needs pure-UEFI booting, so disable the legacy compatibility layer first.

- [ ] Find **CSM (Compatibility Support Module)** → set to **Disabled**. (Usually under Boot or Security. Your Windows is already installed in UEFI/GPT mode, so this is safe.)
- [ ] Go to the **Security** tab → **Secure Boot** → set to **Enabled**.
- [ ] If Secure Boot won't enable, select **Install Default Secure Boot Keys** / **Restore Factory Keys** first, then enable it.

---

## 5. Memory — run your RAM at its rated speed

- [ ] **OC Tweaker** tab → find the memory profile (labeled **DOCP** or **XMP** on this DDR4 board).
- [ ] Select **DOCP/XMP Profile 1** so your RAM runs at its advertised speed instead of the slow JEDEC default. (Skip if you've had stability issues at the rated speed.)

---

## 6. Optional physical-access hardening

- [ ] **Security** tab → set a **Supervisor/Administrator Password** to stop anyone with physical access from changing boot order or disabling Secure Boot.
- [ ] **Write this password down somewhere safe** — there is no easy recovery if you forget it.

---

## 7. Save & exit

- [ ] Press **F10** → **Yes** to save all changes and reboot.

---

## Verify inside Windows

Once booted back to the desktop, confirm everything took:

- [ ] Press **Win + R**, type **`tpm.msc`**, Enter → should read **"The TPM is ready for use"** with **Specification Version 2.0**.
- [ ] Press **Win + R**, type **`msinfo32`**, Enter → **System Summary** → **Secure Boot State** should say **On**, and **BIOS Mode** should say **UEFI**.
- [ ] Optionally re-check Settings → Privacy & security → Device encryption to confirm its status (and that your recovery key is saved).

---

## Firmware upkeep

- [ ] Check the **[ASRock A520M-HVS support page](https://www.asrock.com/mb/AMD/A520M-HVS/index.asp#BIOS)** now and then for a newer **BIOS version** — updates patch security flaws and AGESA/AMD microcode bugs. Only flash if there's a version newer than what's installed (check your current version on the UEFI main screen).
