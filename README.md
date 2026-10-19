# Triple-Boot Portable Multi-OS M.2 SSD (256GB Edition)

A complete, zero-skip, production-grade guide to building a high-speed **triple-boot portable NVMe M.2 SSD on a 256GB capacity drive**. When connected to any compatible host PC, this single external drive will present a clean, interactive boot menu allowing you to instantly launch:

1. **Ventoy & Hiren's BootCD PE:** Your advanced emergency recovery, disk cloning, and hardware diagnostics toolkit.
2. **Portable Windows 11 (Windows To Go):** Your lightweight daily productivity and application environment.
3. **Kali Linux:** Your specialized penetration testing and security auditing workspace.

Your host PC’s internal storage and motherboard bootloader remain 100% untouched and safe.

---

## 📋 Table of Contents

1. [Hardware & Software Prerequisites](#hardware--software-prerequisites)
2. [Partition Sizing & Layout Strategy (Exact 256GB MB/GB Allocation)](#partition-sizing--layout-strategy-exact-256gb-mbgb-allocation)
3. [Phase 1: Flashing Ventoy & Building the ISO Storage Partition](#phase-1-flashing-ventoy--building-the-iso-storage-partition)
4. [Phase 2: Adding Hiren's BootCD PE & Ventoy Customization Menu](#phase-2-adding-hirens-bootcd-pe--ventoy-customization-menu)
5. [Phase 3: Deploying Portable Windows 11 (Windows To Go)](#phase-3-deploying-portable-windows-11-windows-to-go)
6. [Phase 4: Installing Kali Linux in Native Layout](#phase-4-installing-kali-linux-in-native-layout)
7. [Phase 5: Unifying the Boot Menu & Universal UEFI Fallback](#phase-5-unifying-the-boot-menu--universal-uefi-fallback)
8. [Phase 6: Verification & Troubleshooting](#phase-6-verification--troubleshooting)

---

## 🛠️ Hardware & Software Prerequisites

### Hardware Requirements
* **External M.2 NVMe SSD (256GB):** Usable capacity is approximately `238,470 MB` (`238.4 GB`).
* **High-Speed Enclosure:** USB 3.2 Gen 2 (10Gbps) NVMe enclosure.
* **Installer USB Flash Drive:** 1 separate 8GB+ USB stick for the Kali Linux installer.
* **Host PC:** Running Windows 11 with **Secure Boot disabled** in BIOS/UEFI.

### Software Downloads
* **Ventoy:** Download the Windows ZIP package from [Ventoy Official Website](https://www.ventoy.net/).
* **Windows 11 ISO:** Download from [Microsoft Official Page](https://www.microsoft.com/software-download/windows11).
* **Kali Linux Installer ISO:** Download the 64-bit Installer version from [Kali Official Downloads](https://www.kali.org/get-kali/#kali-installer-images).
* **Hiren's BootCD PE ISO:** Download from [Hiren's BootCD Official Website](https://www.hirensbootcd.org/download/).
* **Rufus:** Portable utility from [rufus.ie](https://rufus.ie/).

---

## 📏 Partition Sizing & Layout Strategy (Exact 256GB MB/GB Allocation)

To fit three distinct operating systems comfortably onto a strict **256GB SSD** (`238,470 MB` usable space), we will use the following optimized GPT partition layout:

| Partition Name | File System | Size in MB | Size in GB | Purpose / Contents |
| :--- | :--- | :--- | :--- | :--- |
| **1. Ventoy EFI / Boot** | FAT32 | `500 MB` | `0.5 GB` | Primary UEFI boot sector and Ventoy core payload. |
| **2. Ventoy ISO Storage** | exFAT | `35,000 MB` | `34.1 GB` | Dedicated storage container for Hiren's BootCD PE ISO. |
| **3. Windows To Go (OS)** | NTFS | `110,000 MB` | `107.4 GB` | Main system partition for Portable Windows 11. |
| **4. Kali Root (`/`)** | EXT4 | `85,000 MB` | `83.0 GB` | Linux root filesystem for Kali Linux system files & tools. |
| **5. Kali Swap Partition** | Linux-swap | `4,096 MB` | `4.0 GB` | Virtual memory paging space for Kali Linux. |
| **Unallocated / Buffer** | None | `~3,874 MB` | `~3.8 GB` | Drive alignment buffer & wear-leveling headroom. |
| **Total Drive Space** | **GPT** | **238,470 MB** | **238.5 GB** | **Complete 256GB M.2 NVMe Allocation** |

---

## Phase 1: Flashing Ventoy & Building the ISO Storage Partition

Ventoy will serve as our primary multi-boot manager and container host for Hiren's BootCD PE.

1. Connect your external 256GB M.2 SSD to your host PC via the USB enclosure.
2. Extract the downloaded **Ventoy** ZIP package and run `Ventoy2Disk.exe` as Administrator.
3. Click **Option** > **Non-destructive installation** (or clear the disk completely if brand new by selecting the device directly).
4. Select your external M.2 SSD from the device dropdown. *(Verify the disk size matches ~238GB to ensure you aren't selecting your internal drive).*
5. Click **Install**. Ventoy will format the first portion of the drive into a dual-partition layout (EFI + exFAT ISO storage).
6. Open **Disk Management** (`Win + X` > Disk Management) to verify your external drive now has:
   * A small FAT32 EFI partition (`500 MB`).
   * A large exFAT partition (`~34 GB` labeled `Ventoy`).

---

## Phase 2: Adding Hiren's BootCD PE & Ventoy Customization Menu

1. Open File Explorer and navigate to your `Ventoy` (exFAT) partition.
2. Create a folder named `isos` on the root of the drive (`E:\isos\`, assuming `E:` is your Ventoy drive).
3. Copy your downloaded `HBCD_PE_x64.iso` file directly into `E:\isos\`.
4. Create a configuration file named `ventoy.json` inside a folder named `ventoy` on the root (`E:\ventoy\ventoy.json`) to customize the boot menu theme and behavior:

```json
{
  "control": [
    { "vtoa": 1 },
    { "theme": { "file": "/ventoy/themes/default/theme.txt", "screen_resolution": "1920x1080" } }
  ],
  "menu_alias": [
    { "image": "/isos/HBCD_PE_x64.iso", "alias": "Hiren's BootCD PE (Emergency Repair Toolkit)" }
  ]
}
```

---

## Phase 3: Deploying Portable Windows 11 (Windows To Go)

We will allocate `107 GB` of the remaining unallocated space to deploy Portable Windows 11.

1. Open **Disk Management**, right-click the remaining unallocated space on your external M.2 SSD, select **New Simple Volume**, and create a partition of size `110000 MB` (`107.4 GB`). Format it as **NTFS**, label it `WIN_PORTABLE`, and finish.
2. Open **Rufus** as Administrator.
3. Click the gear icon (**Show advanced drive properties**) and check **List USB Hard Drives**.
4. Select your external M.2 SSD (or specifically point to your newly created NTFS partition if Rufus allows, otherwise select the whole disk and rely on precise partitioning). *Alternative safe method:* Use **WinToUSB Free** to target the `WIN_PORTABLE` partition directly with your Windows 11 ISO.
5. If using Rufus: Select your **Windows 11 ISO**, choose **Windows To Go** as the image option, check **Prevent Windows To Go from accessing internal disks**, and click **START**.

---

## Phase 4: Installing Kali Linux in Native Layout

We will use the final remaining unallocated space (`87GB`) to install Kali Linux natively (`83GB` root + `4GB` swap).

1. Plug in your separate **8GB+ USB flash drive**.
2. Open **Rufus**, select your USB flash drive, choose the **Kali Linux Installer ISO**, and click **START**.
3. Keep both the external M.2 SSD and the Kali Installer USB plugged in. Restart your PC, press your boot menu key (`F12`, `F11`, `Del`, or `Esc`), and boot from the **Kali USB Installer** (Secure Boot Disabled).
4. Select **Graphical install** and step through initial configurations.
5. At the **Partition disks** screen:
   * Select **Manual** partitioning mode.
   * **CRITICAL CHECK:** Ensure target device is your **External M.2 SSD** (e.g., `/dev/sdb`), **NOT** your internal drive (`/dev/nvme0n1`).
   * Select the remaining unallocated free space on the external SSD (`~87 GB`).
   * Create a new primary partition of size `85000 MB` (`83 GB`), formatted as `ext4`, mount point `/`.
   * Create a second primary partition using remaining space (`~4096 MB` / `4 GB`), formatted as `swap area`.
   * Select **Finish partitioning and write changes to disk** and confirm.
6. Complete software installation (leave default Xfce desktop checked) and finish.

---

## Phase 5: Unifying the Boot Menu & Universal UEFI Fallback

Because Ventoy manages ISO booting, we will add custom GRUB entries to Kali's bootloader so you can jump between Ventoy, Windows, and Kali seamlessly from a single unified menu.

1. Boot into **Kali Linux** from your external M.2 SSD.
2. Open a terminal and find your partition UUIDs:
   ```bash
   lsblk -f
   ```
3. Open the custom GRUB script:
   ```bash
   sudo nano /etc/grub.d/40_custom
   ```
4. Paste the following configuration (replacing `YOUR_WINDOWS_NTFS_UUID` and `YOUR_KALI_UUID` with actual UUID values):
   ```sh
   #!/bin/sh
   exec tail -n +3 $0

   menuentry "Portable Windows 11 (Windows To Go)" --class windows {
       insmod part_gpt
       insmod ntfs
       insmod chain
       search --no-floppy --fs-uuid --set=root YOUR_WINDOWS_NTFS_UUID
       chainloader /EFI/Microsoft/Boot/bootmgfw.efi
   }

   menuentry "Kali Linux (Penetration Testing)" --class kali {
       insmod part_gpt
       insmod ext4
       search --no-floppy --fs-uuid --set=root YOUR_KALI_UUID
       linux /boot/vmlinuz-* root=UUID=YOUR_KALI_UUID ro quiet splash panic=10
       initrd /boot/initrd.img-*
   }
   ```
5. Save (`Ctrl + O`, `Enter`, `Ctrl + X`) and update GRUB:
   ```bash
   sudo chmod +x /etc/grub.d/40_custom
   sudo update-grub
   sudo grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=TRIPLE_BOOT_SSD --removable
   ```

---

## Phase 6: Verification & Troubleshooting

1. Shut down completely, remove the installer USB, and plug the M.2 SSD into any target PC.
2. Power on, press your motherboard boot key, and select the external SSD.
3. **Time Sync Fix:** Run this command in Kali to prevent Windows clock drift:
   ```bash
   timedatectl set-local-rtc 1 --adjust-system-clock
   ```
