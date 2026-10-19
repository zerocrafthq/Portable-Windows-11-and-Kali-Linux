# Dual-Boot Portable Windows 11 & Kali Linux on an External M.2 SSD

A complete, risk-free, step-by-step guide to running a fully functional **Portable Windows 11 (Windows To Go)** and a dedicated **Kali Linux** installation on a **single external M.2 NVMe SSD** via USB enclosure. 

Your host PC's internal hard drive is never modified, and the external SSD can be plugged into almost any PC to boot into either operating system.

---

## 📋 Table of Contents
1. [Prerequisites & Hardware Requirements](#prerequisites--hardware-requirements)
2. [Step 1: Flash Portable Windows 11 to the External M.2 SSD](#step-1-flash-portable-windows-11-to-the-external-m2-ssd)
3. [Step 2: Shrink the External Drive to Create Space for Linux](#step-2-shrink-the-external-drive-to-create-space-for-linux)
4. [Step 3: Prepare the Kali Linux Installer USB](#step-3-prepare-the-kali-linux-installer-usb)
5. [Step 4: Install Kali Linux Into the External SSD Free Space](#step-4-install-kali-linux-into-the-external-ssd-free-space)
6. [Step 5: Configure Universal EFI Bootloader (GRUB Removable Mode)](#step-5-configure-universal-efi-bootloader-grub-removable-mode)
7. [Troubleshooting & Tips](#troubleshooting--tips)

---

## 🛠️ Prerequisites & Hardware Requirements

* **External M.2 NVMe SSD** (256 GB minimum recommended; e.g., 150 GB for Windows + 100 GB for Kali).
* **High-Speed USB Enclosure:** A USB 3.2 Gen 2 (10Gbps) or Type-C NVMe enclosure to avoid performance bottlenecks.
* **1x Separate USB Flash Drive** (8 GB or larger) to act as the Kali Linux installation media.
* **Host PC** running Windows 11 with **Secure Boot disabled** in the BIOS.
* **Software Downloads:**
  * [Windows 11 ISO](https://www.microsoft.com/software-download/windows11)
  * [Kali Linux Installer ISO](https://www.kali.org/get-kali/#kali-installer-images)
  * [Rufus](https://rufus.ie/) (Portable version recommended)

---

## Step 1: Flash Portable Windows 11 to the External M.2 SSD

*⚠️ **WARNING:** This step will completely erase everything currently on your external M.2 SSD.*

1. Connect your external M.2 SSD to your PC via USB.
2. Open **Rufus**.
3. Click **Show advanced drive properties** (the small gear/checkbox icon) and check **List USB Hard Drives** so your M.2 SSD appears in the dropdown.
4. Select your external M.2 SSD under **Device**.
5. Click **SELECT** and choose your downloaded **Windows 11 ISO**.
6. Change the **Image option** dropdown from "Standard Windows installation" to **Windows To Go**.
7. Ensure **Partition scheme** is set to `GPT` and **Target system** is `UEFI (non CSM)`.
8. Click **START**.
9. When the Windows To Go edition selection pops up, choose your preferred edition (e.g., Windows 11 Pro).
10. In the final warning popup, check **Prevent Windows To Go from accessing internal disks** (this protects your host computer's internal storage) and click **OK**.
11. Wait for Rufus to finish (status bar turns green and says **READY**).

---

## Step 2: Shrink the External Drive to Create Space for Linux

1. Right-click the Windows Start button and select **Disk Management**.
2. Locate your external M.2 SSD (verify by checking the total disk size, usually listed as Disk 1 or Disk 2).
3. Right-click the large NTFS partition on the external M.2 SSD and choose **Shrink Volume**.
4. Enter the amount of space you want to allocate for Kali Linux in megabytes (e.g., enter `102400` for roughly 100 GB).
5. Click **Shrink**. You will now see an **Unallocated** black block right next to your portable Windows partition on the external SSD. Close Disk Management.

---

## Step 3: Prepare the Kali Linux Installer USB

1. Plug in your separate **8 GB+ USB flash drive**.
2. Open **Rufus** (ensure your M.2 SSD is **NOT** selected this time; select your USB flash drive).
3. Click **SELECT** and choose your downloaded **Kali Linux Installer ISO**.
4. Click **START**. If prompted to download missing syslinux files, click **Yes**, and choose **Write in ISO Image mode** if asked.
5. Wait for Rufus to finish. Once done, keep both the external M.2 SSD and the Kali USB flash drive plugged into your PC.
6. Restart your PC, mash your BIOS key (`F2`, `F12`, `Del`, or `Esc`), ensure **Secure Boot** is **Disabled**, and boot into your Kali USB flash drive. Select **Graphical install**.

---

## Step 4: Install Kali Linux Into the External SSD Free Space

1. Step through the initial Kali installation prompts (Language, Location, Keyboard layout, User account creation, Timezone).
2. When you reach the **Partition disks** screen:
   * Select **Guided - use the largest continuous free space**.
   * **CRITICAL CHECK:** Ensure the installer selects your **External M.2 SSD** (verify the drive size or model name matches your external enclosure) and **NOT** your internal PC drive (`/dev/nvme0n1`).
   * Choose **All files in one partition (recommended for new users)**.
   * Select **Finish partitioning and write changes to disk** and choose **Yes** to confirm.
3. On the **Software selection** screen, leave the defaults checked (`Xfce` desktop and the standard tools metapackage) and click **Continue**.
4. When installation finishes, leave the USB plugged in, complete the reboot, and use your PC's Boot Menu key (`F12`, `F11`, or `Esc`) to boot directly into **Kali Linux** on your external SSD.

---

## Step 5: Configure Universal EFI Bootloader (GRUB Removable Mode)

To ensure your external M.2 SSD can boot on *any* computer without relying on host NVRAM entries, and to bundle your Portable Windows and Kali into a single GRUB boot menu on the external drive, open your Kali terminal and run the following commands:

```bash
# 1. Update package lists and install necessary utilities
sudo apt update && sudo apt install -y grub-efi-amd64-signed shim-signed

# 2. Identify your external SSD's EFI partition (e.g., /dev/sdb1 or /dev/sdc1)
sudo lsblk

# 3. Mount your external SSD's EFI partition and root partition (Replace sdxY with your actual partitions)
# Example: sudo mount /dev/sdb2 /mnt
# Let's use the standard update-grub approach with removable fallback:
sudo grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=kali --removable

# 4. Update GRUB to detect Portable Windows 11 and generate entries
sudo update-grub
```

> **Tip:** If Windows doesn't automatically show up in the GRUB menu, make sure your Portable Windows partition is mounted (`sudo mkdir -p /mnt/win && sudo mount /dev/sdx1 /mnt/win`) before running `sudo update-grub`.

---

## 💡 Troubleshooting & Tips

* **Host Internal Drive Protection:** Because you selected "Prevent Windows To Go from accessing internal disks" during Step 1, your host PC's internal files remain completely isolated and secure.
* **Time Desync Between Windows & Linux:** If your clock changes hours when switching between Portable Windows and Kali, open the Kali terminal and run:
  ```bash
  timedatectl set-local-rtc 1 --adjust-system-clock
  ```
* **Wi-Fi Not Working in Kali?** Many internal laptop Wi-Fi cards (like MediaTek or Realtek) require proprietary firmware or a compatible external USB Wi-Fi adapter (such as an Alfa AWUS036ACS supporting monitor mode/packet injection) for advanced penetration testing tasks.