# Triple-Boot Portable Multi-OS M.2 SSD: Windows 11, Kali Linux & Hiren's BootCD PE

A complete, production-grade, zero-skip guide to building a high-speed **triple-boot portable NVMe M.2 SSD**. When connected to any compatible host PC, this single external drive will present a **custom graphical GRUB2 boot menu** allowing you to instantly launch:

1. **Portable Windows 11 (Windows To Go):** Your full daily desktop productivity & application environment.
2. **Kali Linux:** Your specialized penetration testing and security auditing workspace.
3. **Hiren's BootCD PE:** Your advanced Windows PE emergency recovery, partition management, and diagnostics toolkit.

Your host PC’s internal storage and bootloader remain 100% untouched and secure.

---

## 📋 Table of Contents

1. [Hardware & Software Prerequisites](#hardware--software-prerequisites)
2. [Phase 1: Partitioning the External M.2 SSD](#phase-1-partitioning-the-external-m2-ssd)
3. [Phase 2: Deploying Portable Windows 11 (Windows To Go)](#phase-2-deploying-portable-windows-11-windows-to-go)
4. [Phase 3: Deploying Hiren's BootCD PE](#phase-3-deploying-hirens-bootcd-pe)
5. [Phase 4: Installing Kali Linux in Native Layout](#phase-4-installing-kali-linux-in-native-layout)
6. [Phase 5: Configuring the Custom Unified GRUB2 Multi-Boot Menu](#phase-5-configuring-the-custom-unified-grub2-multi-boot-menu)
7. [Phase 6: Verification & Universal Boot Safety](#phase-6-verification--universal-boot-safety)
8. [Troubleshooting & Pro Tips](#troubleshooting--pro-tips)

---

## 🛠️ Hardware & Software Prerequisites

### Hardware Requirements
* **External M.2 NVMe SSD:** 512 GB or 1 TB capacity recommended (Recommended layout: ~250 GB for Windows, ~120 GB for Kali Linux, ~15 GB for Hiren's & ISO storage).
* **High-Speed Enclosure:** A USB 3.2 Gen 2 (10Gbps) or Type-C NVMe enclosure (e.g., Sabrent, UGREEN, or ORICO).
* **Installer USB Flash Drive:** 1 separate 8 GB+ USB stick to hold the Kali Linux installer.
* **Host PC:** Running Windows 11 with **Secure Boot disabled** in the motherboard BIOS/UEFI.

### Software Downloads
* **Windows 11 ISO:** Download directly from [Microsoft Official Page](https://www.microsoft.com/software-download/windows11).
* **Kali Linux Installer ISO:** Download the 64-bit Installer version from [Kali Official Downloads](https://www.kali.org/get-kali/#kali-installer-images).
* **Hiren's BootCD PE ISO:** Download from [Hiren's BootCD Official Website](https://www.hirensbootcd.org/download/).
* **Rufus:** Portable utility from [rufus.ie](https://rufus.ie/).

---

## Phase 1: Partitioning the External M.2 SSD

To accommodate three distinct operating systems without data collision, we will partition the external M.2 drive using a clean GPT layout.

1. Plug your external M.2 SSD into your host PC via the high-speed USB enclosure.
2. Press `Win + X` and select **Disk Management**.
3. Locate your external M.2 SSD by verifying its total storage size (ensure you do not target your internal drive, e.g., Disk 0).
4. If there are existing partitions on the external SSD, right-click each volume, select **Delete Volume**, until the entire drive shows as unallocated black space.
5. Right-click the unallocated space, select **New Simple Volume**, and create:
   * **Partition 1 (Windows & EFI):** Formatted as **NTFS** (or left unassigned for Rufus to configure during Windows To Go creation).
6. Leave the remaining space unallocated for Kali Linux and Hiren's. Close Disk Management.

---

## Phase 2: Deploying Portable Windows 11 (Windows To Go)

We will use Rufus to deploy a fully functional, self-contained Windows 11 environment onto the primary partition of the external M.2 SSD.

1. Open **Rufus** with administrator privileges.
2. Click the **Settings/Gear icon** at the bottom of Rufus ("Show advanced drive properties") and check **List USB Hard Drives**.
3. Select your external M.2 SSD from the **Device** dropdown.
4. Click **SELECT** and choose your downloaded **Windows 11 ISO**.
5. Change the **Image option** dropdown from *Standard Windows installation* to **Windows To Go**.
6. Ensure **Partition scheme** is `GPT` and **Target system** is `UEFI (non CSM)`.
7. Click **START**. When the Windows Version selection prompt appears, choose your desired edition (e.g., Windows 11 Pro).
8. When the Rufus warning prompt appears regarding data erasure, click **OK**.
9. **Critical Step:** When the Windows To Go properties dialog appears, check **Prevent Windows To Go from accessing internal disks**. This ensures your internal Windows 11 installation remains isolated and safe.
10. Wait for Rufus to finish flashing (status bar displays green **READY**).

---

## Phase 3: Deploying Hiren's BootCD PE

Hiren's BootCD PE can be booted directly as an ISO image using GRUB loopback capabilities, saving space and allowing clean file maintenance.

1. Open File Explorer and navigate to your external M.2 SSD's data partition.
2. Create a folder named exactly `boot` and inside it create a subfolder named `isos` (`/boot/isos/`).
3. Copy your downloaded `HBCD_PE_x64.iso` file directly into the `boot/isos/` directory on your external M.2 SSD.

---

## Phase 4: Installing Kali Linux in Native Layout

We will install Kali Linux into the remaining unallocated space on the external M.2 SSD using a secondary USB installer.

1. Plug in your separate **8 GB+ USB flash drive**.
2. Open **Rufus**, select your USB flash drive under **Device**, select the **Kali Linux Installer ISO**, and click **START** to create the bootable installer.
3. Keep both the external M.2 SSD and the Kali Installer USB plugged in. Restart your PC, mash your motherboard boot menu key (`F12`, `F11`, `Del`, or `Esc`), and select the **Kali USB Installer** (ensure Secure Boot is Disabled).
4. Select **Graphical install** from the boot menu.
5. Step through language, location, keyboard layout, and user account creation prompts.
6. When you reach the **Partition disks** screen:
   * Select **Manual** partitioning mode (or *Guided - use the largest continuous free space*, but **verify** your target drive model).
   * **MANDATORY CHECK:** Verify that the target storage device listed is your **External M.2 SSD** (e.g., `/dev/sdb` or `/dev/sdc`), **NOT** your internal NVMe drive (`/dev/nvme0n1`).
   * Select your unallocated free space on the external SSD, create an `ext4` filesystem partition mapped to mount point `/`, and a secondary swap partition if desired.
   * Finish partitioning and write changes to disk.
7. Complete software installation (leave default Xfce desktop and standard toolsets checked) and finish the installer.

---

## Phase 5: Configuring the Custom Unified GRUB2 Multi-Boot Menu

To unify all three operating systems under a single interactive menu when you boot from your external M.2 SSD, we will configure the custom GRUB script in Kali Linux.

1. Boot into your newly installed **Kali Linux** from the external M.2 SSD.
2. Open your terminal and run the following command to check your partition UUIDs and drive layout:
   ```bash
   lsblk -f
   ```
   *Note down the UUID of your Windows EFI/NTFS partition, your Kali root (`ext4`) partition, and verify the path to your Hiren's ISO file.*

3. Open the custom GRUB script template in nano editor:
   ```bash
   sudo nano /etc/grub.d/40_custom
   ```

4. Replace or append the contents with the following robust configuration script:

   ```bash
   #!/bin/sh
   exec tail -n +3 $0

   # ==========================================
   # CUSTOM MULTI-BOOT MENU FOR PORTABLE SSD
   # ==========================================

   # 1. Portable Windows 11 Entry
   menuentry "Portable Windows 11 (Windows To Go)" --class windows {
       insmod part_gpt
       insmod ntfs
       insmod chain
       search --no-floppy --fs-uuid --set=root YOUR_WINDOWS_NTFS_UUID
       chainloader /EFI/Microsoft/Boot/bootmgfw.efi
   }

   # 2. Kali Linux Pentesting Entry
   menuentry "Kali Linux (Penetration Testing)" --class kali {
       insmod part_gpt
       insmod ext4
       search --no-floppy --fs-uuid --set=root YOUR_KALI_EXT4_UUID
       linux /boot/vmlinuz-* root=UUID=YOUR_KALI_EXT4_UUID ro quiet splash panic=10
       initrd /boot/initrd.img-*
   }

   # 3. Hiren's BootCD PE Recovery Toolkit Entry
   menuentry "Hiren's BootCD PE (Emergency Repair Toolkit)" --class windows {
       insmod part_gpt
       insmod ntfs
       insmod fat
       set isofile="/boot/isos/HBCD_PE_x64.iso"
       search --no-floppy --set=root --file $isofile
       loopback loop $isofile
       linux (loop)/sources/boot.wim
       # Fallback direct loopback chain for WinPE environments
   }
   ```

5. Save the file by pressing `Ctrl + O`, hitting `Enter`, and exiting with `Ctrl + X`.

6. Make the script executable and update GRUB across the external drive:
   ```bash
   sudo chmod +x /etc/grub.d/40_custom
   sudo update-grub
   ```

7. Install the GRUB bootloader into the external drive's EFI partition with the `--removable` flag so it registers as a universal fallback boot option on any PC:
   ```bash
   sudo grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=PORTABLE_OS --removable
   ```

---

## Phase 6: Verification & Universal Boot Safety

1. Shut down your PC completely.
2. Unplug your Kali installer USB stick.
3. Power on your PC and press your motherboard’s Boot Menu key (`F12`, `F11`, or `Esc`).
4. Select your **External M.2 SSD** from the UEFI boot device list.
5. You will be greeted by your **Custom GRUB2 Multi-Boot Menu** listing all three operating systems. Test selecting each one to confirm seamless booting!

---

## 💡 Troubleshooting & Pro Tips

* **Windows Time Drift:** Linux reads hardware clocks as UTC while Windows reads them as local time. To fix time discrepancies when switching between Windows and Kali, run this command in your Kali terminal:
  ```bash
  timedatectl set-local-rtc 1 --adjust-system-clock
  ```
* **Secure Boot Restrictions:** Because Hiren's BootCD PE and custom GRUB configurations use unsigned loopback hooks, **Secure Boot must remain Disabled** in your target PC's BIOS/UEFI settings whenever booting from this external SSD.
