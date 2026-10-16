

## Linux SSD: SATA6G_3 



# 1. Connect to Wi-Fi (skip if Ethernet)
iwctl
# inside iwctl:
######  #`device list`
#####    #`station <device> scan`
`#   station <device> get-networks`
`#   station <device> connect <SSID>`
`#   exit`

# 2. Update clock
timedatectl set-ntp true

# 3. Partition disk (EFI + root)
parted /dev/sda mklabel gpt
parted /dev/sda mkpart ESP fat32 1MiB 513MiB
parted /dev/sda set 1 esp on
parted /dev/sda mkpart primary ext4 513MiB 100%

# 4. Format partitions
mkfs.fat -F32 /dev/sda1
mkfs.ext4 /dev/sda2

# 5. Mount partitions
mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot

# 6. Install base system + firmware
pacstrap /mnt base linux linux-firmware vim networkmanager sudo

# 7. Generate fstab
genfstab -U /mnt >> /mnt/etc/fstab

# 8. Chroot into new system
arch-chroot /mnt

# ===== Inside chroot =====

# 9. Timezone, locale, hostname
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
hwclock --systohc
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
echo "archlinux" > /etc/hostname

# 10. Enable network
systemctl enable NetworkManager

# 11. Create user and set passwords
passwd
useradd -m -G wheel -s /bin/bash sam
passwd sam
EDITOR=vim visudo   # Uncomment "%wheel ALL=(ALL) ALL"

# 12. Install and configure bootloader (GRUB, UEFI)
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg

# 13. Install Hyprland dependencies + packages
pacman -S \
    hyprland \
    waybar \
    rofi \
    kitty \
    wlogout \
    thunar \
    polkit \
    polkit-gnome \
    xdg-desktop-portal-hyprland \
    grim slurp \
    wl-clipboard \
    brightnessctl \
    bluez bluez-utils \
    pavucontrol pipewire pipewire-audio wireplumber pipewire-pulse \
    ttf-jetbrains-mono-nerd \
    firefox \
    unzip git

# 14. Enable services
systemctl enable bluetooth

# 15. Exit and reboot
exit
umount -R /mnt
reboot
