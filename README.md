# Triple-Boot Portable Multi-OS M.2 SSD: Windows 11, Kali Linux & Hiren's BootCD PE

A complete, production-grade, zero-skip guide to building a high-speed **triple-boot portable NVMe M.2 SSD**. When connected to any compatible host PC, this single external drive will present a unified boot menu allowing you to instantly launch:

1. **Portable Windows 11 (Windows To Go):** Your full daily desktop productivity & application environment.
2. **Kali Linux:** Your specialized penetration testing and security auditing workspace.
3. **Hiren's BootCD PE:** Your advanced Windows PE emergency recovery, partition management, and diagnostics toolkit.

Your host PC’s internal storage and bootloader remain 100% untouched and safe.

---

## 📋 Table of Contents
1. [Hardware & Software Prerequisites](#hardware--software-prerequisites)
2. [Partition Sizing & Layout Strategy (Exact MB/GB Specifications)](#partition-sizing--layout-strategy-exact-mbgb-specifications)
3. [Phase 1: Preparing the Drive & Partition Table](#phase-1-preparing-the-drive--partition-table)
4. [Phase 2: Deploying Portable Windows 11 (Windows To Go)](#phase-2-deploying-portable-windows-11-windows-to-go)
5. [Phase 3: Deploying Hiren's BootCD PE & ISO Storage Partition](#phase-3-deploying-hirens-bootcd-pe--iso-storage-partition)
6. [Phase 4: Installing Kali Linux in Native Layout](#phase-4-installing-kali-linux-in-native-layout)
7. [Phase 5: Configuring the Custom Unified GRUB2 Multi-Boot Menu](#phase-5-configuring-the-custom-unified-grub2-multi-boot-menu)
8. [Phase 6: Verification & Universal Boot Safety](#phase-6-verification--universal-boot-safety)
9. [Troubleshooting & Pro Tips](#troubleshooting--pro-tips)

---

## 🛠️ Hardware & Software Prerequisites

### Hardware Requirements
* **External M.2 NVMe SSD:** 512 GB or 1 TB capacity recommended. (Calculations below are optimized for a **512 GB SSD**; scale proportionally if using 1 TB).
* **High-Speed Enclosure:** A USB 3.2 Gen 2 (10Gbps) or USB-C NVMe enclosure (e.g., Sabrent, UGREEN, or ORICO).
* **Installer USB Flash Drive:** 1 separate 8 GB+ USB stick to hold the Kali Linux installer.
* **Host PC:** Running Windows 11 with **Secure Boot disabled** in the motherboard BIOS/UEFI.

### Software Downloads
* **Windows 11 ISO:** Download directly from the [Microsoft Official Page](https://www.microsoft.com/software-download/windows11).
* **Kali Linux Installer ISO:** Download the 64-bit Installer version from [Kali Official Downloads](https://www.kali.org/get-kali/#kali-installer-images).
* **Hiren's BootCD PE ISO:** Download from the [Hiren's BootCD Official Website](https://www.hirensbootcd.org/download/).
* **Rufus:** Portable utility from [rufus.ie](https://rufus.ie/).

---

## 📏 Partition Sizing & Layout Strategy (Exact MB/GB Specifications)

To ensure zero conflicts, proper bootloader isolation, and ample workspace for each environment on a standard **512 GB M.2 SSD** (approx. `476,940 MB` usable space), we will use the following strict GPT partition layout:

| Partition Name | File System | Size in MB | Size in GB | Purpose / Contents |
| :--- | :--- | :--- | :--- | :--- |
| **1. EFI System Partition** | FAT32 | `500 MB` | `0.5 GB` | Universal UEFI bootloader storage for GRUB and Windows boot managers. |
| **2. Windows To Go (OS)** | NTFS | `307,200 MB` | `300 GB` | Main system partition for Portable Windows 11 and user applications. |
| **3. Ventoy / ISO Storage**| NTFS / exFAT | `30,000 MB` | `30 GB` | Dedicated storage container for Hiren's BootCD PE ISO and utility tools. |
| **4. Kali Root (`/`)** | EXT4 | `120,000 MB` | `117 GB` | Linux root filesystem for Kali Linux system files and pentesting tools. |
| **5. Kali Swap Partition** | Linux-swap | `8,192 MB` | `8 GB` | Virtual memory paging space for Kali Linux compiling & memory overflow. |
| **Unallocated / Buffer** | None | `~11,048 MB` | `~10.8 GB` | Drive alignment buffer & wear-leveling space. |
| **Total Drive Space** | **GPT** | **476,940 MB** | **476.9 GB** | **Complete 512 GB M.2 NVMe Allocation** |

---

## Phase 1: Preparing the Drive & Partition Table

1. Plug your external M.2 SSD into your host PC via the high-speed USB enclosure.
2. Press `Win + X` and select **Disk Management**.
3. Locate your external M.2 SSD by verifying its exact total storage size (e.g., `476940 MB` for a 512GB drive). **Double-check that you are not selecting your internal boot drive.**
4. If there are existing partitions on the external SSD, right-click each volume and select **Delete Volume** until the entire disk shows as a single block of unallocated black space.
5. Leave the disk as unallocated space so Rufus and the Linux installer can precisely map the partitions defined above. Close Disk Management.

---

## Phase 2: Deploying Portable Windows 11 (Windows To Go)

We will use Rufus to deploy Portable Windows 11 to the first primary partition chunk (`300 GB`).

1. Open **Rufus** with administrator privileges.
2. Click the **Settings/Gear icon** at the bottom of Rufus ("Show advanced drive properties") and check **List USB Hard Drives**.
3. Select your external M.2 SSD from the **Device** dropdown.
4. Click **SELECT** and choose your downloaded **Windows 11 ISO**.
5. Change the **Image option** dropdown from *Standard Windows installation* to **Windows To Go**.
6. Ensure **Partition scheme** is `GPT` and **Target system** is `UEFI (non CSM)`.
7. Click **START**. When the Windows Version selection prompt appears, choose your desired edition (e.g., Windows 11 Pro).
8. When the Rufus warning prompt appears regarding data erasure, click **OK**.
9. **Critical Step:** When the Windows To Go properties dialog appears, check **Prevent Windows To Go from accessing internal disks**. This protects your host PC's internal drive from cross-contamination.
10. Wait for Rufus to finish flashing (status bar displays green **READY**).

---

## Phase 3: Deploying Hiren's BootCD PE & ISO Storage Partition

We will create a dedicated data partition to store diagnostic ISOs like Hiren's BootCD PE.

1. Open **Disk Management** again. Locate your external M.2 SSD.
2. Right-click the unallocated space immediately following your Windows partition, select **New Simple Volume**, and enter `30000` MB (`30 GB`) in the size box. Format it as **NTFS**, label it `ISOS`, and click **Finish**.
3. Open File Explorer, navigate to your newly created `ISOS` drive, create a folder named `boot`, and inside it create a subfolder named `isos` (`/boot/isos/`).
4. Copy your downloaded `HBCD_PE_x64.iso` file directly into that directory (`D:\boot\isos\HBCD_PE_x64.iso`, assuming `D:` is your ISO partition).

---

## Phase 4: Installing Kali Linux in Native Layout

We will install Kali Linux into the remaining free space using exact size allocations (`117 GB` root + `8 GB` swap).

1. Plug in your separate **8 GB+ USB flash drive**.
2. Open **Rufus**, select your USB flash drive under **Device**, select the **Kali Linux Installer ISO**, and click **START** to create the bootable installer.
3. Keep both the external M.2 SSD and the Kali Installer USB plugged in. Restart your PC, mash your motherboard boot menu key (`F12`, `F11`, `Del`, or `Esc`), and select the **Kali USB Installer** (ensure Secure Boot is Disabled).
4. Select **Graphical install** from the boot menu.
5. Step through language, location, keyboard layout, and user account creation prompts.
6. When you reach the **Partition disks** screen:
   * Select **Manual** partitioning mode.
   * **MANDATORY CHECK:** Verify that the target storage device listed is your **External M.2 SSD** (e.g., `/dev/sdb` or `/dev/sdc`), **NOT** your internal NVMe drive (`/dev/nvme0n1`).
   * Select the remaining unallocated free space on the external SSD (`~128 GB` total remaining).
   * Create a new primary partition of size `120000 MB` (`117 GB`), formatted as `ext4`, with mount point set to `/` (root).
   * Create a second primary partition using the remaining free space (`~8192 MB` / `8 GB`), formatted as `swap area`.
   * Select **Finish partitioning and write changes to disk** and confirm.
7. Complete software installation (leave default Xfce desktop and standard toolsets checked) and finish the installer.

---

## Phase 5: Configuring the Custom Unified GRUB2 Multi-Boot Menu

To unify Portable Windows 11, Kali Linux, and Hiren's BootCD PE under a single interactive boot menu, we will script a custom GRUB configuration in Kali Linux.

1. Boot into your newly installed **Kali Linux** from the external M.2 SSD.
2. Open your terminal and run the following command to check your partition UUIDs and drive layout:
   ```bash
   lsblk -f
   ```
   *Note down the UUID of your Windows NTFS partition and your Kali root (`ext4`) partition.*

3. Open the custom GRUB script template in nano editor:
   ```bash
   sudo nano /etc/grub.d/40_custom
   ```

4. Replace or append the contents with the following robust configuration script (replace `YOUR_WINDOWS_NTFS_UUID` and `YOUR_KALI_EXT4_UUID` with your actual captured UUID strings):
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
