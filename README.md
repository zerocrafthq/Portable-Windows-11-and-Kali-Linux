# Triple-Boot Portable Multi-OS M.2 SSD (256GB Edition)

A complete, zero-skip, production-grade guide to building a high-speed **triple-boot portable NVMe M.2 SSD on a 256GB capacity drive**. When connected to any compatible host PC, this single external drive will present a clean, interactive boot menu allowing you to instantly launch:

1. **Ventoy & Hiren's BootCD PE:** Your advanced emergency recovery, disk cloning, and hardware diagnostics toolkit.
2. **Portable Windows 11 (Windows To Go):** Your lightweight daily productivity and application environment.
3. **Kali Linux:** Your specialized penetration testing and security auditing workspace.

Your host PC’s internal storage and motherboard bootloader remain 100% untouched and safe.

> **⚠️ CRITICAL WORKFLOW ORDER NOTICE:**
> **Ventoy MUST be flashed FIRST.** Ventoy completely formats and wipes the target drive to build its own EFI and ISO storage partitions. If you create the Windows or Kali partitions before flashing Ventoy, Ventoy will wipe them out. Follow the phases below in strict chronological order.

---

## 📋 Table of Contents

1. [Hardware & Software Prerequisites](#hardware--software-prerequisites)
2. [Exact 256GB Partition Sizing & Layout Strategy](#exact-256gb-partition-sizing--layout-strategy)
3. [Phase 1: Flashing Ventoy (Exact Settings & UI Walkthrough)](#phase-1-flashing-ventoy-exact-settings--ui-walkthrough)
4. [Phase 2: Adding Hiren's BootCD PE & Ventoy Menu Customization](#phase-2-adding-hirens-bootcd-pe--ventoy-menu-customization)
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
* **Ventoy:** Download the Windows ZIP package from the [Ventoy Official Website](https://www.ventoy.net/).
* **Windows 11 ISO:** Download from the [Microsoft Official Page](https://www.microsoft.com/software-download/windows11).
* **Kali Linux Installer ISO:** Download the 64-bit Installer version from [Kali Official Downloads](https://www.kali.org/get-kali/#kali-installer-images).
* **Hiren's BootCD PE ISO:** Download from the [Hiren's BootCD Official Website](https://www.hirensbootcd.org/download/).
* **Rufus:** Portable utility from [rufus.ie](https://rufus.ie/).

---

## 📏 Exact 256GB Partition Sizing & Layout Strategy

To fit three distinct operating systems comfortably onto a strict **256GB SSD** (`238,470 MB` usable space), we will structure our disk partitions in this exact sequence:

| Partition # | Partition Name | File System | Size in MB | Size in GB | Purpose / Contents |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | **Ventoy EFI** | FAT32 | `500 MB` | `0.5 GB` | Primary UEFI boot sector and Ventoy core payload. |
| **2** | **Ventoy ISO Storage** | exFAT | `35,000 MB` | `34.1 GB` | Dedicated storage container for Hiren's BootCD PE ISO. |
| **3** | **Windows To Go (OS)** | NTFS | `110,000 MB` | `107.4 GB` | Main system partition for Portable Windows 11. |
| **4** | **Kali Root (`/`)** | EXT4 | `85,000 MB` | `83.0 GB` | Linux root filesystem for Kali Linux system files & tools. |
| **5** | **Kali Swap Partition** | Linux-swap | `4,096 MB` | `4.0 GB` | Virtual memory paging space for Kali Linux. |
| **-** | **Unallocated Buffer** | None | `~3,874 MB` | `~3.8 GB` | Drive alignment buffer & wear-leveling headroom. |
| **Total** | **Total Drive Space** | **GPT** | **238,470 MB** | **238.5 GB** | **Complete 256GB M.2 NVMe Allocation** |

---

## 🚀 Phase 1: Flashing Ventoy (Exact Settings & UI Walkthrough)

We flash Ventoy first because its installer automatically formats the beginning of the drive into a clean layout containing the Ventoy EFI partition and the ISO storage partition.

1. Connect your external 256GB M.2 SSD to your host PC via the USB enclosure.
2. Extract the downloaded **Ventoy** ZIP package and run `Ventoy2Disk.exe` as **Administrator**.
3. **Configure Options Before Installing (Referencing Ventoy UI):**
   * Click on the top menu bar item **Option** > **Show All Devices**. *(This is vital because Ventoy sometimes hides external NVMe enclosures by default).*
   * Click **Option** > **Partition Style** > Select **GPT**. *(Required for modern UEFI systems and Windows 11 compatibility).*
   * Leave **Secure Boot Support** unchecked in Ventoy for now (since we will disable Secure Boot in the PC's BIOS during boot).
4. Select your external M.2 SSD from the device dropdown list at the top.
   > ⚠️ **Double-check the drive size (~238GB) to ensure you are selecting your external enclosure and NOT your internal PC drive.**
5. Click **Install**. Ventoy will warn you that all data will be lost. Confirm by clicking **Yes** twice.
6. Once complete, open **Disk Management** (`Win + X` > Disk Management) to verify your external drive now displays:
   * A small FAT32 EFI partition (`500 MB`).
   * A large exFAT partition (`~34.1 GB` labeled `Ventoy`).
   * A remaining block of **Unallocated** space (`~203 GB`).

---

## 📦 Phase 2: Adding Hiren's BootCD PE & Ventoy Menu Customization

1. Open File Explorer and click into your newly created `Ventoy` (exFAT) partition.
2. Create a folder named `isos` on the root of that drive (`E:\isos\`, assuming `E:` is your Ventoy drive letter).
3. Copy your downloaded `HBCD_PE_x64.iso` file directly inside `E:\isos\`.
4. Create a folder named `ventoy` on the root (`E:\ventoy\`), and inside it create a text file named `ventoy.json` with the following configuration code to enable clean menu labelling:

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

## 🪟 Phase 3: Deploying Portable Windows 11 (Windows To Go)

Next, we carve out our Windows partition from the unallocated space and deploy Windows To Go.

1. Open **Disk Management**, right-click the remaining **Unallocated** space on your external M.2 SSD, and select **New Simple Volume**.
2. Click Next, and set the partition size to exactly `110000 MB` (`107.4 GB`). Click Next.
3. Assign a drive letter, format the file system as **NTFS**, set the Volume label to `WIN_PORTABLE`, and click **Finish**.
4. Open **Rufus** as Administrator.
5. **Exact Rufus Settings:**
   * **Device:** Select your external M.2 SSD (or `WIN_PORTABLE` partition).
   * **Boot selection:** Click **SELECT** and choose your downloaded Windows 11 ISO file.
   * **Image option:** Click the dropdown and change it from "Standard Windows installation" to **Windows To Go**.
   * **Partition scheme:** Set to **GPT**.
   * **Target system:** Set to **UEFI (non CSM)**.
   * Click **Show advanced drive properties** at the bottom and make sure **List USB Hard Drives** is checked if your enclosure doesn't show up.
6. Click **START**. When the Windows User Experience window appears, check **Prevent Windows To Go from accessing internal disks** (this protects your host PC's internal drive) and click **OK**. Wait for Rufus to complete.

---

## 🐧 Phase 4: Installing Kali Linux in Native Layout

We will now use the final remaining unallocated block of space on the external SSD (`~87 GB`) to install Kali Linux natively.

1. Plug in your separate **8GB+ USB flash drive**.
2. Open **Rufus**, select your USB flash drive, choose your **Kali Linux Installer ISO**, and click **START** to flash it.
3. Leave both your external M.2 SSD and the Kali Installer USB plugged into your PC.
4. Restart your PC, press your motherboard boot menu key (`F12`, `F11`, `Del`, or `Esc`), and select the **Kali USB Installer** (ensure Secure Boot is disabled in BIOS).
5. Select **Graphical install** from the Kali boot menu and progress through language and user settings.
6. When you arrive at the **Partition disks** screen:
   * Select **Manual** partitioning mode.
   * **CRITICAL CHECK:** Verify that the target drive selected is your **External M.2 SSD** (e.g., `/dev/sdb`), **NOT** your internal PC drive (`/dev/nvme0n1`).
   * Select the remaining **Free space** (approx `87 GB`) on the external SSD.
   * Create a new primary partition of size `85000 MB` (`83 GB`), choose `ext4` filesystem, and set the mount point to `/` (root).
   * Create a second primary partition using the remaining free space (`~4096 MB` / `4 GB`), formatted as `swap area`.
   * Select **Finish partitioning and write changes to disk** and confirm.
7. Leave software selection at defaults (Xfce desktop + default tools) and let the installation finish. Unplug the Kali USB installer when prompted to reboot.

---

## 🔗 Phase 5: Unifying the Boot Menu & Universal UEFI Fallback

Because Ventoy manages ISO booting, we will add custom GRUB configuration entries inside Kali so you can jump seamlessly between Ventoy, Windows, and Kali from a single boot menu.

1. Boot into **Kali Linux** from your external M.2 SSD.
2. Open a terminal and run the following command to find the file system UUIDs of your partitions:
   ```bash
   lsblk -f
   ```
   *(Note down the UUID of your `WIN_PORTABLE` NTFS partition and your Kali `ext4` root partition).*
3. Open the custom GRUB script editor with root privileges:
   ```bash
   sudo nano /etc/grub.d/40_custom
   ```
4. Paste the following configuration block into the file (replacing `YOUR_WINDOWS_NTFS_UUID` and `YOUR_KALI_UUID` with your actual copied UUID strings):
   ```bash
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
5. Save the file (`Ctrl + O`, then `Enter`, then `Ctrl + X`).
6. Apply executable permissions and update GRUB across your external drive:
   ```bash
   sudo chmod +x /etc/grub.d/40_custom
   sudo update-grub
   sudo grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=TRIPLE_BOOT_SSD --removable
   ```

---

## 🧪 Phase 6: Verification & Troubleshooting

1. **Test Universal Boot:** Shut down your host PC completely, disconnect all other drives/USBs, and plug your external M.2 SSD into any computer. Power on and press your boot menu key. Select your external SSD to verify your boot options load correctly.
2. **Fix Windows Clock Drift:** When booting back and forth between Linux and Windows, your Windows system clock may appear off by a few hours. Open your Kali terminal and run this command once to align UTC/local time interpretation:
   ```bash
   timedatectl set-local-rtc 1 --adjust-system-clock
