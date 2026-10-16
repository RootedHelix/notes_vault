Related to [[Arch]], [[Linux]] 
#linux #arch_linux_dual_boot
   
   ## DEEP SEARCH
Thanks! I’ll now prepare a detailed, step-by-step guide for dual-booting Arch Linux with Windows 10 in UEFI mode, including manual partitioning and setting up Hyperland as the graphical environment. I’ll walk you through everything from booting into the USB to setting up the bootloader and configuring your Wayland desktop.

### Dual-Booting Windows 10 and Arch Linux with Hyprland

## Pre-Installation Steps (in Windows & BIOS/UEFI)

- **Verify UEFI Mode:** Ensure Windows 10 is installed in UEFI mode (e.g. run `msinfo32` in Windows and check _BIOS Mode_ shows UEFI). If not, convert Windows to UEFI before proceeding.
    
- **Disable Secure Boot (and Fast Boot):** In your UEFI firmware settings, turn off Secure Boot (this prevents boot issues with the Arch installer). Also disable any “Fast Boot” options in BIOS/UEFI to avoid bootloader issues.
    
- **Disable Windows Fast Startup:** In Windows, open _Control Panel → Power Options_. Click "Choose what the power buttons do" and uncheck **Turn on Fast Startup**, then save changes. This ensures Windows fully shuts down (preventing disk lock issues when accessing NTFS from Linux).
    
- **Shrink Windows Partition:** Free up unallocated space for Arch. In Windows Disk Management (`diskmgmt.msc`), shrink the Windows volume (C:) by ~200 GB (half of your 400 GB SSD). This will create ~200 GB of **unallocated**space for Arch Linux.
    
- **Backup Important Data:** Before modifying partitions or installing another OS, it’s wise to back up any critical data. This process is generally safe if instructions are followed, but backups are a precaution.
    

## Booting into the Arch Linux USB Installer (UEFI Mode)

1. **Boot from USB:** Insert your Arch Linux bootable USB and reboot. Use your BIOS’s boot menu (usually via **F12**, **F11**, or **Esc** key during boot) to select the USB drive. Make sure to boot the USB in **UEFI mode** (often the USB will be listed with a UEFI prefix).
    
2. **Check UEFI Mode:** Once the Arch ISO boots to a prompt, verify you’re in UEFI mode by running:
    
    ```bash
    ls /sys/firmware/efi/efivars
    ```
    
    If the directory exists (no error), you have booted in UEFI mode. (If not, reboot and select the UEFI USB option.)
    
3. **Connect to Internet:** If on a wired connection, network should be active by default. For Wi-Fi, you can connect by running `iwctl` (use `station wlan0 connect <SSID>` commands) or the older `wifi-menu` utility. Ensure you have internet by pinging a site (e.g. `ping -c 3 archlinux.org`).
    
4. **Sync Time:** Enable network time sync with:
    
    ```bash
    timedatectl set-ntp true
    ```
    
    This ensures the system clock is correct, which helps prevent certain issues during installation.
    

## Partitioning the Disk for Arch Linux

With Windows already occupying half the SSD, you’ll partition the remaining **unallocated ~200GB** for Arch. We will create three partitions in that space: **root (`/`)**, **swap**, and **home (`/home`)**.

1. **Identify Disk:** Determine your disk device name using `fdisk -l` or `lsblk`. It’s usually `/dev/sda` for a SATA drive or `/dev/nvme0n1` for an NVMe SSD. In the examples below, we’ll use **`/dev/sda`**.
    
2. **Launch Partition Tool:** Run `cfdisk /dev/sda` (or use `gdisk /dev/sda`) to edit the partition table. Ensure the disk is using GPT (required for UEFI – if Windows is UEFI, the disk is likely already GPT).
    
3. **Create a Swap Partition:** In the free space, create a new partition for swap. Size it about equal to your RAM (e.g. if 8 GB RAM, make ~8 GB; if you plan to use hibernation, make it at least as large as RAM). Set its type to **Linux swap**.
    
4. **Create the Root (`/`) Partition:** Create another new partition in the remaining space – or allocate, say, 30–50 GB for the root filesystem. Set its type to **Linux filesystem** (GUID type `8300`). This will be mounted as the root (`/`) of Arch.
    
5. **Create the Home Partition:** Finally, create one more partition with the **rest of the free space** (whatever is left after root/swap). This will be for `/home` (to store your user files separately). Also type **Linux filesystem**.
    
6. **Preserve Windows Partitions:** **Do not touch or modify** any existing Windows partitions. You should already have a small **EFI System Partition** (FAT32, ~100–500 MB) and the Windows NTFS partition on the disk – leave those intact.
    
7. **Write Changes:** Save the partition table (in `cfdisk`, choose **Write**). Your disk now has the new partitions for Arch.
    

> 🔹 _Using the existing Windows EFI partition:_ Since you’re dual-booting in UEFI, **do not create a new EFI partition** for Arch – instead, you will reuse the **existing EFI System Partition (ESP)** that Windows created. Avoid reformatting that ESP, as it contains Windows boot files. We will simply mount it later and add Arch’s bootloader to it.

## Formatting and Mounting Partitions

Now format the new Linux partitions and set up mount points:

1. **Format Linux Partitions:** Format the root and home partitions as your chosen filesystem (ext4 is a good default). For example:
    
    ```bash
    mkfs.ext4 /dev/sdaX   # replace sdaX with your root partition number
    mkfs.ext4 /dev/sdaY   # replace sdaY with your home partition number
    ```
    
    (If you prefer Btrfs or another FS, you can use the appropriate mkfs command, but ext4 is straightforward.)
    
2. **Enable Swap:** Set up the swap partition:
    
    ```bash
    mkswap /dev/sdaZ   # replace sdaZ with your swap partition number
    swapon /dev/sdaZ
    ```
    
    This formats the swap area and activates it. You can verify with `free -m` that swap shows up.
    
3. **Mount the Root Partition:** Create a mount point and mount root:
    
    ```bash
    mount /dev/sdaX /mnt
    ```
    
    (The Arch installer uses `/mnt` as the target mount for the new system.)
    
4. **Mount the Home Partition:** Create a directory for home and mount it:
    
    ```bash
    mkdir /mnt/home  
    mount /dev/sdaY /mnt/home
    ```
    
5. **Mount the EFI System Partition:** Identify the existing EFI partition. It’s usually a small FAT32 partition (you can find it via `lsblk -f` – look for a partition with **vfat** filesystem and label like “ESP” or “SYSTEM”). Mount this at the future `/boot` location. For example, if the EFI partition is `/dev/sda1`:
    
    ```bash
    mkdir /mnt/boot   # (if /mnt/boot doesn't already exist)  
    mount /dev/sda1 /mnt/boot
    ```
    
    After this, `/mnt/boot` corresponds to the contents of the EFI partition. **Do not format it!** We’re preserving Windows’ boot files. You should be able to `ls /mnt/boot/EFI` and see a `Microsoft` directory (that’s Windows Boot Manager).
    

At this point, the root, home, and EFI partitions are mounted at `/mnt`, `/mnt/home`, and `/mnt/boot` respectively (and swap is active). Double-check with `lsblk` that these are correct.

## Installing the Arch Linux Base System

With partitions mounted, you will install the minimal base system to the new Arch root (`/mnt`):

1. **Select Mirrors (optional):** (You can skip this if the default mirrors are fine, but optimizing mirror speed can help.) Optionally, run `pacman -Sy reflector` and use `reflector` to fetch the fastest mirrors for your region, then update `/etc/pacman.d/mirrorlist`.
    
2. **Install Base Packages:** Use the `pacstrap` script to install essential packages into `/mnt`. For example:
    
    ```bash
    pacstrap /mnt base linux linux-firmware networkmanager vim nano os-prober ntfs-3g
    ```
    
    - **`base`**: core Arch system packages.
        
    - **`linux`**: the latest Linux kernel.
        
    - **`linux-firmware`**: firmware for hardware support.
        
    - **`networkmanager`**: a convenient networking service (so you have network on first boot).
        
    - **Text editor** (vim/nano) for convenience editing config files.
        
    - **`os-prober`** and **`ntfs-3g`**: recommended for dual-boot. `os-prober` helps detect Windows in the bootloader, and `ntfs-3g` allows Arch to read/write NTFS partitions.
        
    - _(Optional)_ **`base-devel`** if you plan to compile software or use AUR in the future.
        
    
    This command will download and install the packages to the mounted system (it may take a few minutes).
    
3. **Generate Fstab:** After pacstrap finishes without errors, generate the filesystem table so your mounts persist on boot:
    
    ```bash
    genfstab -U /mnt >> /mnt/etc/fstab
    ```
    
    Double-check the resulting fstab with `nano /mnt/etc/fstab` – it should list your `/`, `/home`, and the EFI (FAT32) partition (usually mounted at `/boot` in the file) with correct UUIDs.
    
4. **Enter the New System (chroot):** Change root into the new system to configure it:
    
    ```bash
    arch-chroot /mnt
    ```
    
    Your shell prompt will change, indicating you are now operating inside the installed Arch system (with `/` as the root of `/mnt`). All following configuration steps happen inside the chroot.
    

## Configuring the Arch System

Inside the chroot, perform these configuration steps to set up Arch:

- **Timezone:** Set your time zone:
    
    ```bash
    ln -sf /usr/share/zoneinfo/<Region>/<City> /etc/localtime
    hwclock --systohc
    ```
    
    Replace `<Region>/<City>` (e.g. `US/Pacific` or `Europe/London`) with your locale. The `hwclock` command syncs hardware clock to system time.
    
- **Locale:** Edit **`/etc/locale.gen`** and uncomment your desired locale(s), for example `en_US.UTF-8 UTF-8`. Then run:
    
    ```bash
    locale-gen
    ```
    
    Create **`/etc/locale.conf`** and set the LANG, e.g.:
    
    ```bash
    echo "LANG=en_US.UTF-8" > /etc/locale.conf
    ```
    
    (Add other locale environment variables as needed, e.g. `LC_TIME`, or leave defaults.)
    
- **Keyboard Layout (optional):** If you need a non-US keyboard layout, you can set `KEYMAP` in **`/etc/vconsole.conf`**(for console tty) – e.g., `echo "KEYMAP=de-latin1" > /etc/vconsole.conf` for German.
    
- **Hostname:** Choose a hostname for your machine (one word). For example:
    
    ```bash
    echo "archlinux" > /etc/hostname
    ```
    
    Then edit **`/etc/hosts`** and add:
    
    ```plaintext
    127.0.0.1   localhost  
    ::1         localhost  
    127.0.1.1   archlinux.localdomain   archlinux  
    ```
    
    (Replace `archlinux` with your hostname in both lines.)
    
- **Networking:** Enable NetworkManager so you have network on boot:
    
    ```bash
    systemctl enable NetworkManager
    ```
    
    (This was installed earlier. It will start on first boot and handle wired/wireless networks.)
    
- **Root Password:** Set a root password by typing `passwd` and entering a secure password. This lets you log in as root if needed.
    

### Bootloader Setup (GRUB for Dual-Boot)

Next, install and configure GRUB as the bootloader to handle both Arch and Windows:

1. **Install GRUB and EFI Tools:** Still in the chroot, install GRUB and EFI boot manager:
    
    ```bash
    pacman -S grub efibootmgr
    ```
    
    (We already installed `os-prober` earlier for detecting other OS, but if not, install it now with `pacman -S os-prober`.)
    
2. **Install GRUB to EFI:** Run the `grub-install` command to install the GRUB UEFI files to the EFI System Partition. Since we mounted the ESP at `/boot`, use:
    
    ```bash
    grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=ArchLinux
    ```
    
    - `--target=x86_64-efi` specifies UEFI mode on 64-bit.
        
    - `--efi-directory=/boot` tells it where the ESP is mounted (our `/boot`).
        
    - `--bootloader-id=ArchLinux` is the name that will appear in your UEFI firmware’s boot menu. You can use a different name if desired (e.g. "Arch").  
        This should output “Installation finished. No error reported.” and place GRUB’s EFI file in the ESP (e.g. `/boot/EFI/ArchLinux/grubx64.efi`). You can verify by checking `/boot/EFI/ArchLinux/` exists.
        
3. **Enable OS Probing:** By default, newer GRUB disables the os-prober (for detecting other OSes) for security reasons. To have GRUB detect Windows, enable os-prober: edit **`/etc/default/grub`** and ensure this line is present:
    
    ```plaintext
    GRUB_DISABLE_OS_PROBER=false
    ```
    
    (If the line is commented or set to `true`, change it to false.) Save the file.
    
4. **Generate GRUB Config:** Now create the GRUB config file which includes menu entries:
    
    ```bash
    grub-mkconfig -o /boot/grub/grub.cfg
    ```
    
    GRUB will output detection of the Arch Linux kernel and it should also detect “Windows Boot Manager” on the ESP if os-prober is enabled. For example, you’ll see a line about Windows in the output. This writes a `/boot/grub/grub.cfg` file with a menu for Arch and Windows.
    
    - _If you don’t see Windows detected here:_ make sure `os-prober` and `ntfs-3g` are installed, and that `GRUB_DISABLE_OS_PROBER=false` is set, then run the `grub-mkconfig` again. (On Arch, you do **not** run `update-grub` like Ubuntu; use the `grub-mkconfig` command as shown.)
        

GRUB is now installed. It should have created a UEFI boot entry named "ArchLinux" (or the ID you gave) that points to the GRUB .efi. You can check with `efibootmgr -v` (optional) to see entries. The Windows Boot Manager is still intact on the ESP and will be chain-loaded by GRUB.

## Finalizing Installation and Reboot

1. **Exit Chroot:** Type `exit` to leave the chroot environment and return to the live USB prompt.
    
2. **Unmount Partitions:** Unmount all mounted partitions with:
    
    ```bash
    umount -R /mnt
    ```
    
    (Double-check no errors and that `/mnt` is unmounted.)
    
3. **Reboot:** Moment of truth – reboot into your new system:
    
    ```bash
    reboot
    ```
    
    Make sure to remove the USB drive so the PC boots from the disk.
    

On reboot, you should see the **GRUB menu**. It will have an entry for Arch Linux and one for Windows (usually labeled “Windows Boot Manager” or similar). By default it might boot Arch after a timeout, but you can use the arrow keys to pick Windows if desired. If you don’t see the GRUB menu and it boots straight to Windows, you may need to go into your UEFI settings and change the boot order to prioritize the “ArchLinux” (GRUB) boot entry, or manually select it from the one-time boot menu. Once GRUB is loading, you have a functioning dual-boot setup!

Log in to Arch (the prompt will ask for login). Use the **root** account (and password you set). Next, you’ll want to create a normal user account and set up your desktop environment (Hyprland).

## Post-Install: Creating User and Setting Up Hyprland Desktop

Now that Arch is installed and dual-boot configured, you can set up your user and the Hyprland graphical environment.

**1. Create a Regular User:** It’s not recommended to run a desktop as root. Create a new user (e.g. username “alice” – choose your own):

```bash
useradd -m -G wheel,audio,video -s /bin/bash alice
passwd alice
```

This creates user “alice” with a home directory and adds them to the **wheel** group (for sudo), and common groups like **audio** and **video** (for sound and GPU access). The shell is set to Bash. Then `passwd` sets the user’s password.

**2. Enable Sudo for Wheel:** Install sudo if not already: `pacman -S sudo`. Then edit `/etc/sudoers` (with visudo or nano) to uncomment the line:

```plaintext
%wheel ALL=(ALL) ALL
```

This allows users in the wheel group to use `sudo` for admin commands. Save the file.

_(At this point, you can logout as root and login as your new user for the remaining steps: e.g. press `Ctrl+D` to logout, then login with your username and password.)_

**3. Install Hyprland and Dependencies:** Update package lists (`sudo pacman -Syu`), then install Hyprland and related packages:

```bash
sudo pacman -S wayland wlroots hyprland xdg-desktop-portal-hyprland \
               waybar rofi firefox alacritty
```

- **Hyprland:** The Wayland tiling compositor you want to use (available in Arch’s official repo).
    
- **wayland** and **wlroots:** Base Wayland libraries (often installed as dependencies).
    
- **xdg-desktop-portal-hyprland:** Allows Hyprland to integrate with screenshot and screencast portals (important for screensharing, etc.).
    
- **waybar:** A lightweight status bar (so you have a panel for workspaces, time, etc., since Hyprland itself doesn’t provide one by default).
    
- **rofi:** An application launcher (use `rofi-wayland` if available, or plain rofi will run under XWayland). Alternatively, you can use **wofi** or others for launching apps.
    
- **firefox, alacritty** (example user apps): Firefox as a web browser, Alacritty as a terminal emulator. (Feel free to choose your favorites – e.g., `kitty` or `gnome-terminal` for terminal, etc.)
    
- **swaybg, swaylock** (optional): You might also install `swaybg` (to set a wallpaper) and `swaylock` (screen locker), since Hyprland can use those for background and locking. These were originally made for Sway but work on Wayland in general.
    
- **network-manager-applet** (optional): If you want a GUI network icon for Wi-Fi, etc., you can install `network-manager-applet` (and `blueman` for Bluetooth, etc.).
    

All the above can be installed in one pacman command. Arch will pull in any additional needed libraries.

**4. Configure Hyprland:** Create a configuration directory in your user’s home and copy the default config:

```bash
mkdir -p ~/.config/hypr
cp /usr/share/hyprland/examples/hyprland.conf ~/.config/hypr/
```

This gives you a starting `hyprland.conf` which you can later edit to customize keybindings, wallpaper, themes, etc. By default, Hyprland’s config enables basic tiling window management and a few keybinds. _(For example, try `Super+Q` to close a window, `Super+Enter` to open a terminal – if using Alacritty, ensure the config points to alacritty as the terminal.)_

**5. (Optional) Setup a Display Manager:** A display manager provides a graphical login screen. You can install one like **SDDM** or **LightDM** if you want to graphically log in and start Hyprland. For example:

```bash
sudo pacman -S sddm
sudo systemctl enable sddm
```

Hyprland’s package includes a session file, so Hyprland should appear as a session option in SDDM’s greeter. If you go this route, edit `/etc/sddm.conf` and set `Session=hyprland` under the `[LastSession]` section or choose Hyprland from the GUI. Then upon reboot, you’ll get a GUI login where you can select your user and launch Hyprland.

**6. **Running Hyprland (without a display manager):** You can also start Hyprland manually. From a TTY login as your user, just run:

```bash
Hyprland
```

This will launch the Hyprland Wayland session directly on your display. (Ensure you are **not** running under root – Hyprland should be run as your normal user.) Using no display manager is perfectly fine – in fact, it avoids some potential issues. _“If you just want to run Hyprland, just run Hyprland”_. You’ll be dropped into your Hyprland desktop environment.

**7. Post-Login:** Once Hyprland starts, you should see a blank desktop (likely with a default colored background or none if not set). You can launch a terminal with the default keybinding (often `Super + Enter`). From there, you can launch other apps or use rofi (bind it to a key in the config, e.g. `Super + D` for app menu). Use Waybar to view status (if it’s not auto-started, you can launch `waybar &` from your Hyprland config or terminal). At this point, you have a minimal Arch + Hyprland running!

## Final Tips

- **Verifying Dual-Boot:** Reboot and test booting into Windows from the GRUB menu as well. GRUB should chainload the Windows Boot Manager. If Windows doesn’t boot, you might need to turn off Fast Startup (as we did) or in rare cases run `bootrec /fixboot` from a Windows recovery – but if we reused the ESP without formatting, Windows should boot fine.
    
- **EFI Partition Space:** The Windows-created ESP (~100 MB) can hold the GRUB bootloader and is usually sufficient. (If it’s extremely small and you run into space issues with many kernels, consider enlarging it to ~300–500 MB, but this is rarely necessary).
    
- **Troubleshooting:** If Arch doesn’t show in the boot menu, ensure the `grub-install` was done correctly to the ESP. You can also use your firmware boot menu to select “ArchLinux”. If GRUB shows but no Windows entry, make sure `os-prober` was enabled as described.
    
- **Using Hyprland:** Hyprland is a modern Wayland tiling compositor – consult the [Hyprland wiki](https://wiki.archlinux.org/title/Hyprland) for documentation on configuring the `hyprland.conf`, keybindings, and adding things like wallpaper (e.g. via `hyprpaper` or `swaybg`) and notifications, etc. There’s a thriving community, and many user configuration dotfiles are available for inspiration.
    
- **Enjoy your Dual-Boot system!** You now have Windows 10 and Arch Linux (with Hyprland) side by side. You can choose either at boot, and have the best of both worlds on one machine. Good luck, and welcome to Arch!

Absolutely! Here’s a **step-by-step overview of a “manual” Arch Linux installation (without the automatic installer/script)**, aimed at a first-timer. This is the **classic, hands-on Arch install**—the one that’s famous for teaching you how Linux works under the hood.

---

## **Arch Linux Manual Install – Step-by-Step Overview**

---

### **0. Preparation**

- **Download the Arch ISO:** [archlinux.org/download](https://archlinux.org/download/)
    
- **Create a bootable USB** (Rufus for Windows, `dd` or `balenaEtcher` for Linux/macOS)
    
- **Back up your data!**
    
- **Check your internet connection**—wired makes life easier, but wireless is fine too.
    

---

### **1. Boot from USB**

- Boot into the USB via your BIOS/UEFI boot menu.
    
- At the prompt, you’ll see a live environment (command-line only).
    

---

### **2. Set Keyboard Layout (if needed)**

- Default is US; to change:  
    `loadkeys uk` (for UK layout, for example)
    

---

### **3. Verify Internet Connection**

- **Wired:** Usually works out of the box.
    
- **WiFi:** Use `iwctl` to connect:
    
    ```
    iwctl
    station device scan
    station device get-networks
    station device connect SSID
    ```
    
- Test with: `ping archlinux.org`
    

---

### **4. Update System Clock**

- `timedatectl set-ntp true`
    

---

### **5. Partition the Disks**

- **View existing partitions:** `lsblk`, `fdisk -l`, or `parted -l`
    
- **Partition:** Use `cgdisk`, `fdisk`, or `parted` to create:
    
    - **EFI partition** (if UEFI): ~512MB, type “EFI System”
        
    - **Root partition** (`/`): rest of the space
        
    - _(Optional: swap partition)_
        
- **Be careful not to overwrite Windows or data partitions if dual booting!**
    

---

### **6. Format the Partitions**

- EFI:  
    `mkfs.fat -F32 /dev/sda1`
    
- Root:  
    `mkfs.ext4 /dev/sda2`
    
- _(Swap: `mkswap /dev/sda3` and `swapon /dev/sda3` if used)_
    

---

### **7. Mount the Filesystems**

- Root:  
    `mount /dev/sda2 /mnt`
    
- EFI:  
    `mkdir /mnt/boot`  
    `mount /dev/sda1 /mnt/boot`
    

---

### **8. Install the Base System**

- `pacstrap /mnt base linux linux-firmware nano sudo networkmanager`
    

---

### **9. Generate fstab**

- `genfstab -U /mnt >> /mnt/etc/fstab`
    

---

### **10. Chroot into the New System**

- `arch-chroot /mnt`
    

---

### **11. Set Timezone, Locale, Hostname**

- Set timezone:  
    `ln -sf /usr/share/zoneinfo/Region/City /etc/localtime`  
    `hwclock --systohc`
    
- Set locale:  
    Edit `/etc/locale.gen`, uncomment `en_US.UTF-8 UTF-8`, then run `locale-gen`  
    `echo "LANG=en_US.UTF-8" > /etc/locale.conf`
    
- Set hostname:  
    `echo "myhostname" > /etc/hostname`
    
- Set hosts file:
    
    ```
    127.0.0.1   localhost
    ::1         localhost
    127.0.1.1   myhostname.localdomain myhostname
    ```
    

---

### **12. Set Root Password**

- `passwd`
    

---

### **13. Install a Bootloader**

- For UEFI (systemd-boot, recommended):
    
    ```
    bootctl --path=/boot install
    ```
    
    - Edit `/boot/loader/loader.conf` and create `/boot/loader/entries/arch.conf`
        
    - Or, use GRUB if you prefer (especially for dual boot).
        

---

### **14. Enable NetworkManager**

- `systemctl enable NetworkManager`
    

---

### **15. Create a User**

- `useradd -m -G wheel -s /bin/bash myuser`
    
- `passwd myuser`
    
- `EDITOR=nano visudo`  
    _(Uncomment `%wheel ALL=(ALL) ALL` for sudo access)_
    

---

### **16. Exit, Unmount, and Reboot**

- `exit`
    
- `umount -R /mnt`
    
- `reboot`
    
- **Remove USB stick!**
    

---

### **17. First Boot:**

- Log in as your user or root.
    
- From here, install a desktop environment (KDE, GNOME, etc.), graphics drivers, extra software, etc.
    

---

## **Summary Table**

|Step|What You Do|
|---|---|
|Download & prepare|ISO, USB, backups|
|Boot & connect|Start installer, set keyboard, get online|
|Partition & format|Use tools to set up disks/partitions|
|Mount & install base|Mount partitions, install base Arch system|
|Configure system|Locale, timezone, user, root password|
|Bootloader|Install systemd-boot or GRUB|
|Network & user setup|Enable networking, create user, give sudo access|
|Finish|Unmount, reboot, log in, install extra software|

---

### **Note:**

Manual install means **you control every step**—no GUI, no handholding.  
It’s very educational, but pay close attention at the partitioning and formatting stage, especially for dual boot.

---

**If you want a detailed walkthrough, help with any command, or how to do dual boot, just ask!**

