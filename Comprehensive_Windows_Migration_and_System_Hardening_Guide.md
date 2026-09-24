## Comprehensive Windows Migration & System Hardening Guide

This guide details how to securely migrate a Windows operating system installation from an unstable, mechanical Hard Disk Drive (HDD) connected via SATA over to a high-speed NVMe M.2 SSD. It also covers post-migration optimization, space reclamation, user folder re-routing, weekly secondary backups, game library management, and system safety configurations using an uninterruptible power supply (UPS).

> **How to read this guide:** It's written as a complete worked example on one specific build — a **SIX X4000 512GB NVMe SSD**, an **ASRock A520M-HVS** motherboard, and a **CyberPower CP1500PFCLCD** UPS. The procedure applies to almost any Windows desktop; wherever a step depends on specific hardware, an **"Adapting to your hardware"** note tells you what to check on your own machine. Follow the example steps and substitute your own models.

> **Match your drive to your slot.** A drive runs at the *lower* of its own PCIe generation and the slot's. In this example the SIX X4000 is a PCIe **Gen3 x4** drive and the A520M-HVS M.2 slot is also Gen3 x4, so it runs at full speed with no bottleneck.
>
> *Adapting to your hardware:* Check your motherboard's M.2 slot generation (Gen3 / Gen4 / Gen5) against your SSD's spec. A higher-generation drive in a lower-generation slot still works — it just caps at the slot's speed. That's expected, not a fault.

---

## Phase 1: Hardware Pre-Flight & Integration

## 1\. Safety & Physical Installation

1. Shut down your PC completely. Flip the rocker switch on the back of your Power Supply Unit (PSU) to the OFF (O) position and unplug the main AC cable from the wall.  
2. Press and hold your case's physical power button for 10 seconds to fully discharge lingering electrical capacity on the motherboard.  
3. Open your computer side panel. Locate your motherboard's M.2 slot. On the A520M-HVS it's the Ultra M.2 slot (PCIe Gen3 x4), silkscreened **"M2_1"** — identify it by that label rather than by position, since exact placement varies. It is that board's single M.2 socket. *Adapting to your hardware: find the slot in your board's manual by its silkscreen label (M2_1, M2A_1, etc.). Boards with multiple M.2 slots usually reserve the fastest, CPU-connected slot for the boot drive — use that one.*  
4. Insert your SIX X4000 M.2 NVMe SSD at a slight 30-degree angle into the slot until it seats cleanly. Press the tail end down flat against the standoff and secure it down firmly using the specialized M.2 anchor screw packaged with your ASRock motherboard.  
5. Leave your original mechanical Windows HDD and any secondary storage HDDs plugged into their standard SATA ports. In this example, the A520M-HVS M2_1 slot is **CPU-connected** and is the board's only M.2 slot, so an NVMe drive there does **not** disable or share bandwidth with any of its four SATA ports.  

   > **Adapting to your hardware — check this before you assume your SATA drives stay active; it varies a lot between boards.** Many budget and mid-range motherboards *do* disable one or more SATA ports (or a second M.2 slot) when certain M.2 slots are populated — most often the chipset-connected ones. Look for a "lane sharing" or "shared with SATA" footnote in your motherboard manual's storage or specifications section. (Also note: a slot that accepts a *SATA-type* M.2 drive may borrow a SATA port for that drive — but only for a SATA M.2, not an NVMe one.)  
6. Reattach your case panel, reconnect cables, and boot the system normally into your existing Windows HDD workspace.

## 2\. Disk Initialization

1. Once booted, right-click the Start Menu icon and select Disk Management.  
2. A popup window should automatically detect the uninitialized drive. Select GPT (GUID Partition Table) as the partition style and click OK.  
3. Note: Leave the bar labeled "Unallocated Space" untouched. Do not create new volumes or format partitions yet; the sector cloning process will completely overwrite this space.

---

## Phase 2: OS Health Check & Capacity Optimization

Abrupt power losses cause mechanical HDD arms to snap back violently, causing logical sectors to mismatch. Compounding this, cloning directly onto a new drive will clone the file corruption along with it. Run these maintenance utilities first:

1. Click the Windows Start menu, type cmd, right-click Command Prompt, and select Run as Administrator.  
2. Input the volume validation utility (**chkdsk** — Check Disk) and press Enter:  
   chkdsk c: /f /r  
3. The prompt will state that the volume is locked and ask to schedule validation upon the next restart. Type Y, press Enter, and immediately restart your computer. Let the black loading screen completely finish scanning and repairing disk sectors. On a large mechanical drive the /r surface scan can take a while — let it run to completion rather than interrupting it.  
4. Once back on your desktop, re-open an Administrator Command Prompt and first repair the Windows component store with **DISM** (Deployment Image Servicing and Management). Input the following and press Enter:  
   DISM /Online /Cleanup-Image /RestoreHealth  
   This repairs the underlying store of known-good system files (using Windows Update as the source). Running it first matters, because the next command draws its repairs from this store — if the store itself is corrupt, the repair cannot work.  
5. When DISM reports "The operation completed successfully," run the **System File Checker (SFC)** to scan and repair protected Windows system files, and press Enter:  
   sfc /scannow  
6. Allow the scanner to reach 100%. If it finds corrupted files, it will overwrite and fix them using the now-repaired local component store.  
7. Open This PC via File Explorer and confirm your active C: drive consumption metrics. Your current data must fit on the new SSD with room to spare — aim to leave **at least ~10–15% of the SSD free** for performance and overhead. (On the 512GB example drive, that means keeping usage under ~450 GB.) Manually cut and paste non-essential storage folders (such as heavy game directories, media archives, or downloads) onto your alternate secondary storage drive if required.

---

## Phase 3: Bit-Level Operating System Migration

Some SSDs (including the SIX X4000 in this example) don't include a dedicated cloning suite. If yours does — Samsung Data Migration, Crucial's Acronis edition, WD/SanDisk Dashboard, and so on — you can use it. Otherwise, use a platform-agnostic tool like **DiskGenius** or **Hasleo Disk Clone**.

> **Which free tool?** There's a meaningful difference between the two for this job:
> - **Hasleo Disk Clone** clones whole disks and partitions for free, but its **OS/System Transfer feature is paywalled** — migrating *just the OS* to another drive requires the paid version.
> - **DiskGenius** can do the OS migration on its **free tier**, and during the process it **offers to automatically reassign the C: drive letter** to the new SSD for you. Taking that option can save you the manual boot/drive-letter juggling in Phase 4.

1. Fire up your cloning program of choice.  
2. Select the utility labeled System Migration or OS Migration.  
3. Verify the selection profiles accurately match your components:  
   * Source Drive: Your old structural HDD containing the current Windows OS footprint.  
   * Target Drive: Your unallocated SIX X4000 NVMe SSD.  
4. When prompted by the execution wizard, pick Hot Migration. This isolates active file system hooks into a local shadow state, allowing Windows to safely clone itself in the background without locking you out of your workstation.  
5. Keep your PC completely idle and do not launch extra background applications until the completion prompt flags a success status.

---

## Phase 4: UEFI Boot Selection & C-Drive Assignment

Windows assigns the C: drive letter at runtime to the hardware device that handles the initial system boot. To shift your target SSD to primary position:

> **Adapting to your hardware:** BIOS access keys and menu layout differ by vendor. This example uses ASRock (**F2 / Delete** to enter, **F7** for Advanced Mode, **F10** to save). ASUS, MSI, and Gigabyte also generally use **Delete**; some prebuilts and laptops use **F2**, **F10**, or **Esc**. The boot splash usually shows the key for a second, or check your board/PC manual.

1. Restart your machine and immediately spam the F2 or Delete key until you pull up the UEFI BIOS Menu (ASRock's, on the example board).  
2. Press F7 to transition out of standard mode and enter Advanced Mode.  
3. Navigate with your directional keys to the Boot configuration tab.  
4. Scroll down to line item Boot Option \#1 and modify it from your legacy mechanical HDD over to your new NVMe SSD (the SIX X4000, in this example).  
5. Save your alterations and close out the BIOS environment by pressing F10, then select Yes.

## Critical Drive Collision Isolation Step:

After a clone, the SSD and the old HDD carry an identical disk signature, which is exactly what causes the board to boot the wrong drive. If your motherboard defaults back to pulling data paths from your old HDD bootloader record during the reboot sequence, use this absolute isolation method:

1. Power down the system completely and detach the main power cord from the wall.  
2. Temporarily pull the physical SATA data cable connection clean out of your original Windows HDD. (Leave your other auxiliary storage drives attached).  
3. Connect power and start your machine. Windows will be forced to map the NVMe SSD as the undisputed primary device, hard-coding its identifier as the C: drive.  
4. Once you boot up successfully to your new blazing-fast desktop layout, execute a standard shutdown sequence, pull the power cord, and plug your old Windows SATA HDD right back into its lane.  
5. Restart your PC. Windows will hold fast to the newly established SSD architecture as C:, and safely assign your legacy mechanical drive to a secondary position down the alphabet line (such as D: or E:).  
6. **If the old drive does not appear:** because of the shared disk signature, Windows may show the reconnected HDD as "Offline" on this first boot. Open Disk Management, right-click the offline disk, and select **Online**. Windows resolves the signature collision and the drive returns with its own letter — this is normal and does not mean the drive has failed.

---

## Phase 5: SSD Space Reclamation & User Path Routing

## 1\. Disable Hibernation & Fast Startup (Reclaim ~40% of your RAM in space)

By default, Windows creates a hidden system file (hiberfil.sys) sized at roughly **40% of your installed RAM** to handle hybrid sleep and Fast Startup. Disabling it frees that space on your SSD (about 6–7 GB on a 16 GB system, 12–13 GB on 32 GB, and so on) and ensures every shutdown is a true, full shutdown rather than a hibernated kernel state that can carry stale data across hardware changes.

1. Right-click the Start menu button and select Terminal (Admin) or Command Prompt (Admin).  
2. Paste the following command and hit Enter:  
   powercfg \-h off  
3. This instantly deletes the hibernation file, drops your SSD usage metrics, and disables Windows Fast Startup—ensuring every shutdown is a true, clean reset.

## 2\. Relocate Windows User Paths to Secondary HDDs

To protect your new SSD from filling up with massive download pipelines, temporary desktop assets, or document folders, natively route your user directories onto an alternative drive. (Note: these folders will live on a slower mechanical drive, which trades a little speed for a lot of free SSD space — a reasonable choice for bulky, infrequently-accessed files.)

1. Open File Explorer, go to This PC, and enter your secondary storage drive (e.g., D: or E:).  
2. Create a folder named exactly after your profile (e.g., D:\\Storage\_HDD). Inside it, create three subfolders: Downloads, Documents, and Desktop.  
3. Navigate to your primary C:\\Users\\YourUsername folder.  
4. Right-click the Downloads folder and select Properties.  
5. Move to the Location tab at the top and select Move...  
6. Browse to your new target subfolder (e.g., D:\\Storage\_HDD\\Downloads), choose Select Folder, and click Apply.  
7. Confirm the security prompt with Yes to transfer all active data. Repeat this exact process for your Documents and Desktop folders.

---

## Phase 6: Automated Weekly Backups (SSD to HDD)

Since you have extra drives inside the computer case, configuring automated local snapshot utilities keeps your fresh user data heavily redundant without costing cloud subscription fees.

1. Press the Windows Key, type Control Panel, and select it.  
2. Under the System and Security column, click on Backup and Restore (Windows 7).  
3. On the right side of the window, click Set up backup.  
4. Select one of your secondary storage HDDs as your backup destination and click Next.  
5. When asked what to back up, choose Let me choose so you can keep the backup file targeted and compact. Click Next.  
6. Uncheck "Include a system image" (since your Windows is safely configured now, you only need to protect data files). Check the boxes for your user folders and libraries.  
7. Click Change schedule right above the save options.  
8. Set the frequency to Weekly, select your preferred day (e.g., Sunday), and pick a time when your PC is typically turned on. Click OK.  
9. Click Save settings and run backup. Windows will seamlessly build an automated schedule that keeps your files backed up locally.

> **Modern alternative — File History:** "Backup and Restore (Windows 7)" still works in Windows 11 but is a legacy tool that Microsoft has deprecated, and it can be occasionally flaky. If you'd rather use the current supported method for backing up user files, enable **File History** (Settings → search "File History") and point it at one of your secondary HDDs. It pairs naturally with the folder redirection you set up in Phase 5 and can keep versioned copies more frequently than once a week. Either approach works — this is offered as an option, not a required change.

---

## Phase 7: Post-Migration Cleanup & System Hardening

## 1\. Legacy HDD Decommissioning

* Staging Window: Leave your historical Windows folders intact on the secondary mechanical HDD partition for roughly 3 to 5 days to confirm the long-term integrity of your fresh NVMe architecture.  
* Format Command: Once you verify total system stability, harvest any leftover text archives or personalized folders out of the old user pathways. Open up Disk Management, identify the secondary disk partitions belonging to that old OS drive, right-click, and select Format to reclaim the full capacity of that spinning drive as pure secondary storage.

## 2\. CyberPower UPS Software Configuration

To completely safeguard your system from crashing or corrupting during a storm blackout, link your CyberPower CP1500PFCLCD UPS to interface with your operating system dynamically:

> **Adapting to your hardware:** These steps use CyberPower's PowerPanel software. For a different UPS brand, install *its* monitoring app instead — **APC → PowerChute**, **Eaton → Intelligent Power Manager**, **Tripp Lite → PowerAlert**. The concept is identical: connect the data cable, install the vendor app, and set it to trigger an automatic Windows shutdown after a set time on battery.

1. Connect the bundled **USB A-to-B communication cable** (packaged inside the UPS box) — the flat Type-A end goes into an open USB port on the back of your motherboard, and the square Type-B end goes into the communication port on the rear of the UPS.  
2. Download and install **PowerPanel Personal**, CyberPower's free consumer software for a single home PC. (Your CP1500PFCLCD is HID-compliant, so it works with it out of the box. CyberPower's site also offers *PowerPanel Business*, but that is aimed at IT/server setups and is overkill here — stick with Personal.)  
3. Open the PowerPanel interface dashboard. The software will display your utility input power status, battery charge level, and expected emergency runtime.  
4. Go to the software's power/shutdown settings. The exact menu labels vary between software versions, so follow the on-screen wording — you are looking for the option that governs behavior on battery power (formerly shown as **Settings > Runtime**).  
5. Choose the mode that preserves battery power / triggers an automatic shutdown on outage (formerly labeled **Preserve Battery Power**).  
6. Set the shutdown timer delay to about 5 minutes.  
7. How it works: If a storm breaks the electric mains at your house, the UPS battery bank takes the full load without your PC stuttering. After ~5 minutes on battery power, the software automatically closes your active applications, saves open work, and performs an orderly, fully protected Windows shutdown before the battery depletes.

## 3\. Firmware & Boot Security (BIOS/UEFI)

Your board supports the security features Windows 11 relies on. Set these once in the ASRock UEFI (F2 or Delete at boot, then F7 for Advanced Mode):

> **Adapting to your hardware:** The settings below are universal Windows 11 requirements, but these menu paths are ASRock/AMD-specific (and match the companion BIOS checklist). On Intel boards the TPM is called **Intel PTT** rather than AMD fTPM; on ASUS, MSI, and Gigabyte the wording and menu locations differ. Search your board manual for "TPM" and "Secure Boot" if the paths don't match.

* **AMD fTPM:** Advanced → CPU Configuration → **AMD fTPM switch** → set to "AMD CPU fTPM." This is the on-board TPM 2.0 module Windows 11 (and drive encryption) uses.  
* **Secure Boot:** Security tab (or Boot tab) → **Secure Boot** → Enabled. This blocks unsigned bootloaders and rootkits from loading before Windows. It works because you installed to a GPT disk in UEFI mode (Phase 1).  
* **Keep the BIOS updated:** Check ASRock's A520M-HVS support page for the latest BIOS every so often; firmware updates patch security flaws and AGESA-level bugs.  
* **Optional — UEFI supervisor password:** Under the Security tab you can set a password that stops anyone with physical access from changing boot order or disabling Secure Boot. Write it down somewhere safe; there is no easy recovery if you forget it.

> **Critical warning before you touch fTPM:** On Windows 11 24H2/25H2, drive encryption is often turned on automatically (see section 7.6). Clearing the TPM or flipping the fTPM switch **invalidates the encryption keys** and will demand your 48-digit recovery key at the next boot. Confirm you have that key saved (section 7.6) *before* changing any TPM or Secure Boot setting, or you can lock yourself out of your own drive.

## 4\. Windows Update & Driver Currency

An unpatched system is the single most common way home PCs get compromised. After the migration:

1. Go to Settings → Windows Update → **Check for updates**, install everything, and reboot until it comes back clean.  
2. Turn on **Get the latest updates as soon as they're available** (the toggle on the same screen).  
3. Update your chipset and GPU drivers from the vendors directly — AMD chipset drivers from AMD's site, and your graphics driver from NVIDIA/AMD — rather than relying only on Windows Update.

## 5\. Accounts & Sign-In

1. Use a **strong, unique password** on your Windows account, and add a **Windows Hello PIN** (Settings → Accounts → Sign-in options). The PIN is tied to this specific device's TPM, so it's safer than a reusable password.  
2. Leave **User Account Control (UAC)** at its default level. Do **not** disable it — that prompt before admin actions is a real barrier against silent malware installs.  
3. For best practice, consider using a **Standard** (non-administrator) account for everyday use and keeping a separate Administrator account for installs. This limits the damage any single program can do. Optional, but it's the biggest single hardening step for daily safety.

## 6\. Microsoft Defender & Drive Encryption

**Antivirus (built in, no third-party needed):** Open **Windows Security** → Virus & threat protection → Manage settings, and confirm these are **On**: Real-time protection, Cloud-delivered protection, and **Tamper Protection** (Tamper Protection stops malware from switching your defenses off). Then under App & browser control → Reputation-based protection, make sure **SmartScreen** is on. Run one **Full scan** after the migration to establish a clean baseline.

**Drive encryption:** Check Settings → Privacy & security → **Device encryption** (or search "BitLocker"). 

* If encryption is already **On** (common on newer Windows 11 installs), immediately **back up your recovery key** — sign in at https://aka.ms/myrecoverykey to view it, and also save a copy somewhere offline. Without it, a BIOS/TPM change or a failed boot can permanently lock your data.  
* If it's **Off** and you want it on: full **BitLocker** management requires Windows 11 **Pro**; Windows 11 **Home** offers the simpler automatic "Device Encryption" when the hardware qualifies. For a **desktop that never leaves your house**, drive encryption is optional — its main benefit is protecting data if the machine is physically stolen. If you do enable it, save the recovery key first.

## 7\. Ransomware Resilience

1. **Keep one backup offline.** The in-case HDD backup from Phase 6 is always connected, which means ransomware could encrypt it too. Rotate an **external drive that you unplug** between backups, or add a cheap cloud copy, so you always have one copy malware can't reach. This is the single most effective ransomware protection.  
2. **Optional — Controlled Folder Access.** Windows Security → Virus & threat protection → Ransomware protection lets you lock your Documents/Desktop folders so only approved apps can write to them. Be aware it can block games and apps from saving files until you whitelist them, so enable it only if you're willing to manage the occasional "allow this app" prompt.

## 8\. Reduce the Attack Surface

1. **Remote Desktop / Remote Assistance:** Confirm Settings → System → **Remote Desktop is Off** unless you specifically use it. Turn off Remote Assistance in the classic System Properties → Remote tab.  
2. **Startup & installed apps:** Open Task Manager → Startup apps and disable anything you don't recognize or need; review Settings → Apps → Installed apps and uninstall bloat. Fewer running programs means fewer things that can be exploited.  
3. **Core Isolation / Memory Integrity:** Windows Security → Device security → Core isolation → **Memory integrity**. It blocks a class of driver-based attacks. It can conflict with some older drivers or a few games/anti-cheat systems and cost a little performance, so enable it and test your games — if something breaks, turn it back off.

---

## Phase 8: Appendix \- Gaming Layout & Volume Management

## 1\. Re-Routing Game Launchers Off the SSD

To keep your 512GB C: drive clear of heavy modern games, configure your launchers to install to your secondary storage drives by default:

* Steam Configuration: Open Steam \> Go to Settings \> Storage. Click the drop-down menu at the top, select Add Drive, and choose your secondary storage partition. Set this secondary drive as your Default location. You can also click on existing games here and choose Move to shift them instantly without re-downloading.  
* Epic Games Launcher: When installing any game, click Browse on the installation prompt and choose a folder on your secondary HDD (e.g., D:\\Games\\EpicGames).  
* Xbox / Microsoft Store App: Open the Xbox App \> Click your profile name \> Settings \> Install Options. Change the default drive under "Change where this app installs games by default" to your secondary HDD.

## 2\. Windows Storage Spaces (Optional Multi-Drive Pooling)

If you have two secondary storage HDDs and want to combine them into one seamless folder structure rather than managing separate drive letters, you can use a native Windows Storage Pool:

1. Open the Start Menu, search for Storage Spaces, and select it.  
2. Click Create a new pool and storage space.  
3. Select the check boxes for the two secondary HDDs you want to merge and click Create pool.  
4. Warning: This action completely formats those specific drives. Make sure your data is cleared off them first.  
5. Choose a drive letter, then choose your resiliency type: **Two-way mirror** (safest — keeps a duplicate copy, but halves your total usable capacity) or **Simple** (no resiliency — gives you the maximum pooled space of both drives combined, but a single drive failure loses everything). Click Create storage space. They will now appear as a single large drive in File Explorer.

---
