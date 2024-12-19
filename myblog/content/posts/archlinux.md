+++
date = '2024-12-17T15:02:12+11:00'
toc = true
title = 'Archlinux Setup and common Linux Errors'
+++

# How to Install Arch Linux
Make sure that the machine is not in Secure Boot mode to prevent further authentication or you will end up with a black and yellow window and can’t do anything.

Refer to the link below for how to set up a VM in virtualbox for any OS: https://docs.google.com/document/d/1-uePhx7kDsEjF_vA6O1jVVHpX85VpOUiTH6j-3TIvz4/edit?usp=sharing 

Refer to the https://wiki.archlinux.org/title/Installation_guide.

## Keyboard and Font (Optional)
Set up the keyboard layout and console font:

```bash
loadkeys us           # Set keyboard to US layout
setfont iso09.16      # Set console font (example)
# ls /usr/share/kbd/consolefonts/   # List available console fonts
```
- `loadkeys us` ensures the keyboard input follows the US layout, which is helpful if you are used to it.
- `setfont` changes the console font. This step is optional but improves readability in some cases.

---

## Verify Boot Mode (Optional)
Check if the system is booted in UEFI mode:

```bash
cat /sys/firmware/efi/fw_platform_size
# If the result is "64", the system is using UEFI boot.
```
- UEFI boot mode is required for systems that use modern boot managers like GRUB in EFI mode.

---

## Internet Connection

### On a Virtual Machine
1. Check for network connectivity:
   ```bash
   ip link
   ```
   - `ip link` shows the status of all network devices. Look for the "UP" symbol.

2. If all network interfaces show `DOWN`, follow these steps:
   - Reboot to the boot page.
   - Enter **Firmware Settings**.
   - Go to the network tab and select the displayed **MAC address** (the network adapter).
   - Reboot into the Arch Linux installation medium.
   - Verify connectivity:
     ```bash
     ip link   # One interface should now show "UP"
     ```

### On a Bare Metal System
1. Use `iwctl` to connect to Wi-Fi:
   ```bash
   iwctl
   device list
   station wlan0 get-networks
   station wlan0 connect <wifi_name>
   ```
   - `device list` displays available network adapters.
   - Replace `wlan0` with the name of your wireless interface.
   - Use `get-networks` to see nearby Wi-Fi networks and `connect` to join one.

2. For Ethernet (wired) connections:
   ```bash
   ip link   # A network adapter should show "UP"
   ```

3. Verify the connection:
   ```bash
   ping youtube.com
   ```
   - A successful `ping` means your internet connection is active.

---

## Partitioning
1. Start the partition tool and point it to the desired drive, not the bootable drive:
   ```bash
   cfdisk /dev/sda
   ```
   Choose "GPT", "DOS" is typically for usb sticks!
   - GPT (GUID Partition Table) is the modern standard for partitioning.

2. Create the following partitions:
   - **EFI System**: ≥ 500MB (for booting)
   - **Swap**: ≥ 1GB (for memory overflow)
   - **Linux Root**: Remaining storage

3. Verify the partitions:
   ```bash
   lsblk
   ```
   - `lsblk` lists all storage devices and partitions. Note their names (e.g., `/dev/sda1`, `/dev/sda2`).

---

## Formatting the Partitions
Format the partitions as follows:

```bash
# EFI System
mkfs.fat -F 32 /dev/<efi_partition>

# Swap
mkswap /dev/<swap_partition>

# Linux Root
mkfs.ext4 /dev/<root_partition>
```
- **EFI System**: Use `mkfs.fat -F 32` because EFI requires the FAT32 file system.
- **Swap**: `mkswap` initializes a swap partition.
- **Linux Root**: `mkfs.ext4` sets up the main filesystem.

---

## Mounting the Partitions
Mount the partitions in this order:

```bash
# Linux Root
mount /dev/<root_partition> /mnt

# EFI System
mkdir -p /mnt/boot/efi
mount /dev/<efi_partition> /mnt/boot/efi

# Swap
swapon /dev/<swap_partition>
```
Verify with:
```bash
lsblk
```
- The mounted partitions should now appear under `/mnt`.

---

## Installing the Base System
Install essential packages:
```bash
pacstrap -K /mnt base linux linux-firmware sudo vim grub efibootmgr
# Optional: Add git and networkmanager
```
- **base**: Core Arch Linux system packages.
- **linux**: The kernel.
- **linux-firmware**: Firmware for hardware compatibility.
- **sudo**: Required for managing administrative tasks.
- **vim**: A text editor for editing configuration files.
- **grub** and **efibootmgr**: Tools for bootloader setup.

> **Note**: If you encounter errors such as "Failed to install packages," refer to the [Arch Wiki](https://wiki.archlinux.org/title/Pacman/Package_signing#Cannot_import_keys) for troubleshooting.

---

## Generating fstab
Generate the filesystem table and verify:
```bash
genfstab -U /mnt >> /mnt/etc/fstab
vim /mnt/etc/fstab   # Check partition entries
```
- `fstab` stores partition mounting information. Ensure it includes all three partitions.

---

## Chroot into the New System
Change root into the new installation:
```bash
arch-chroot /mnt
```
- This changes the environment to the newly installed Arch Linux system.

---

## Set Time and Location
1. Set the system time:
   ```bash
   ln -sf /usr/share/zoneinfo/<Region>/<City> /etc/localtime
   hwclock --systohc
   ```
2. Verify:
   ```bash
   hwclock
   ```
- `hwclock --systohc` synchronises the hardware clock with the system time.

---

## Localisation
1. Edit locale configuration:
   ```bash
   vim /etc/locale.gen
   ```
   - Uncomment:
     ```
     en_US.UTF-8 UTF-8
     ```
2. Generate the locale:
   ```bash
   locale-gen
   ```

---

## Setting the Hostname
Create a hostname for your machine:
```bash
vim /etc/hostname
# Enter your desired machine name
```
- The hostname identifies your computer on a network.

---

## Set Root Password
Set the root password:
```bash
passwd
```
- This password is required for the root user to log in.

---

## Adding a User
1. Add a new user and set a password:
   ```bash
   useradd -m -G wheel -s /bin/bash <username>
   passwd <username>
   ```
2. Enable sudo for the user:
   - Edit `visudo`:
     ```bash
     EDITOR=vim visudo
     ```
   - Uncomment the line for the "wheel" group:
     ```
     %wheel ALL=(ALL) ALL
     ```

---

## Installing the Bootloader
Install and configure GRUB:
```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```
- `grub-install` sets up the bootloader in EFI mode.
- `grub-mkconfig` generates the GRUB configuration file.

---

## Reboot
1. Exit the chroot environment:
   ```bash
   exit
   ```
2. Optionally, unmount all partitions:
   ```bash
   umount -R /mnt
   ```
3. Reboot the system:
   ```bash
   reboot
   ```

Remove the installation medium and log in as the root user.

---

## Post-Installation
Congratulations! Your Arch Linux system is now installed. Proceed to install additional software and configure the system as needed.

## Arch Linux Post-Installation Guide

This section covers essential tasks after installing Arch Linux, including resolving common errors, configuring Wi-Fi, and setting up tools for daily usage.

---

### Resolving Common Errors

#### **Invalid or Corrupted Package (PGP Signature) when Updating or Installing Packages (ARM64)**

**Example Error:**
```bash
$ pacman -S vim
error: vim-runtime: signature from "Arch Linux ARM Build System <builder@archlinuxarm.org>" is unknown trust
:: File /var/cache/pacman/pkg/vim-runtime-8.1.0022-1-aarch64.pkg.tar.xz is corrupted (invalid or corrupted package (PGP signature)).
Do you want to delete it? [Y/n]
...
error: failed to commit transaction (invalid or corrupted package (PGP signature))
Errors occurred, no packages were upgraded.
```

**Solution (x86_64 Systems):**
```bash
pacman-key --init && pacman-key --populate archlinux
```

**Solution (ARM64 Systems, e.g., MacBook Virtual Machine):**
```bash
pacman -S archlinuxarm-keyring
pacman-key --init && pacman-key --populate archlinuxarm
```

**If `archlinuxarm.gpg` Keyring File Does Not Exist:**
```bash
$ pacman-key --init && pacman-key --populate archlinuxarm
==> ERROR: The keyring file /usr/share/pacman/keyrings/archlinuxarm.gpg does not exist.
```

**Fix:**
1. Temporarily disable signature checking in `/etc/pacman.conf` by setting:
   ```
   SigLevel = Never
   ```
2. Reinitialize the keyring:
   ```bash
   pacman-key --init
   pacman -S archlinuxarm-keyring
   pacman-key --populate archlinuxarm
   ```
3. Revert the `SigLevel` setting back to:
   ```
   SigLevel = Required DatabaseOptional
   ```
4. Perform a system upgrade:
   ```bash
   pacman -Syu
   ```

---

### Connecting to Wi-Fi

#### **1. Using `nmtui` (Recommended)**
`nmtui` is a simple text-based GUI tool for managing networks:
```bash
nmtui
```
Follow the on-screen instructions to connect to Wi-Fi.

#### **2. Using `NetworkManager` (Manual Method)**
1. Install and enable NetworkManager:
   ```bash
   sudo pacman -S networkmanager
   sudo systemctl enable NetworkManager
   sudo systemctl start NetworkManager
   ```
2. List available networks:
   ```bash
   nmcli device wifi list
   ```
3. Connect to a Wi-Fi network:
   ```bash
   nmcli device wifi connect SSID password PASSWORD
   ```
4. Connect to hidden networks:
   ```bash
   nmcli device wifi connect SSID password PASSWORD hidden yes
   ```

---

### Common Tools for Daily Usage

#### **1. Fix Touchpad Issues (Click/Scroll Not Working)**
Use `xinput` to identify and configure touchpad properties:
```bash
xinput list
xinput list-props DEVICE_NAME
xinput set-prop DEVICE_NAME PROPERTY VALUE
```
For persistent settings, add the commands to `~/.xprofile` or `~/.xinitrc`.

#### **2. Adjust Screen Brightness with `brightnessctl`**
Increase or decrease brightness:
```bash
brightnessctl set +10%  # Increase by 10%
brightnessctl set 10%-  # Decrease by 10%
```
For keybindings, use:
- `XF86MonBrightnessUp` (e.g., Fn+Brightness Up)
- `XF86MonBrightnessDown`.

#### **3. Adjust Audio Volume with `pactl` (PulseAudio)**
```bash
pactl set-sink-volume @DEFAULT_SINK@ +10%  # Increase by 10%
pactl set-sink-volume @DEFAULT_SINK@ -10%  # Decrease by 10%
```

#### **4. Connect Bluetooth Devices with `bluetoothctl`**
1. Install required packages:
   ```bash
   sudo pacman -S bluez bluez-utils
   sudo systemctl enable --now bluetooth.service
   ```
2. Use `bluetoothctl` CLI:
   ```bash
   bluetoothctl
   [bluetooth] power on
   [bluetooth] agent on
   [bluetooth] default-agent
   [bluetooth] scan on
   [bluetooth] pair MAC_ADDRESS
   [bluetooth] connect MAC_ADDRESS
   ```
3. Enable autostart for Bluetooth:
   Edit `/etc/bluetooth/main.conf`:
   ```
   AutoEnable=true
   ```

---

### Resolving ProtonVPN Kill Switch Issues

**Problem:** If ProtonVPN's kill switch is active, internet connection will be blocked when the VPN disconnects unexpectedly.

**Fix:**
1. Check for the kill switch interface:
   ```bash
   ip link
   ```
   If you see `ipv6leakintrf0`, delete it:
   ```bash
   sudo ip link delete ipv6leakintrf0
   ```
2. Alternatively, disable autoconnect:
   ```bash
   nmcli con down pvpn-ipv6leak-protection
   nmcli con mod pvpn-ipv6leak-protection connection.autoconnect no
   ```

---

### Configure Touchpad Gestures
Install `libinput-gestures` for touchpad gesture support:
```bash
sudo gpasswd -a $USER input
sudo pacman -S libinput-gestures wmctrl xdotool
libinput-gestures-setup stop desktop autostart start
```

---

### Configure Nvidia GPU Driver
Install the Nvidia driver:
```bash
sudo pacman -S nvidia
```
Edit `/etc/mkinitcpio.conf` and add:
```bash
MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)
```

---

### Correcting System Time
Set time synchronization with NTP:
```bash
timedatectl set-ntp true
```

---

### Screen Colours Not Rendering Correctly
If using Optimus Manager, configure AMD GPU:
1. Install AMD driver:
   ```bash
   sudo pacman -S xf86-video-amdgpu
   ```
2. Update `/etc/optimus-manager/optimus-manager.conf`:
   ```
   driver=amdgpu
   ```
3. Reboot.

---

### Final Notes
For detailed information, consult the Arch Wiki links provided in each section. Regular system updates (`pacman -Syu`) ensure your system remains stable and secure.

---

This guide aims to streamline the most common post-installation tasks for Arch Linux users, offering solutions to common errors and tools for improved functionality.

