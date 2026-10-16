
- EFI Partition is for storing bootloaders


- I need a Swap file

Target Path /dev/sda
Target DRIVE
DeviceId FriendlyName                               SerialNumber               Size
0             Samsung SSD 870 EVO 500GB     S6PXNL0Y405315R      500107862016



powershell : 
Get-PhysicalDisk | Select-Object DeviceId, FriendlyName, SerialNumber, Size

1. Connected to the internet by default through ethernet
2. sync time
3. partitioned drives with fdisk (Didn't have arrow keys)
4. Format Partitions
NAME   FSTYPE LABEL UUID                                 MOUNTPOINTS
sda
├─sda1 vfat         XXXX-YYYY
└─sda2 ext4         a1b2c3d4-e5f6-....

5. Mount Partitions
[ ROOT ]
mount /dev/sdX2 /mnt

mkdir /mnt/boot

[ EFI] 
mount /dev/sdX1 /mnt/boot


# ==== TIMEZONE ====
ln -sf /usr/share/zoneinfo/America/Phoenix /etc/localtime
hwclock --systohc

# ==== LOCALE ====
sed -i '/^#en_US.UTF-8 UTF-8/s/^#//' /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf

# ==== HOSTNAME ====
echo "myhostname" > /etc/hostname   # <-- change "myhostname" to whatever you want

# ==== HOSTS FILE ====
cat > /etc/hosts <<EOF
127.0.0.1   localhost
::1         localhost
127.0.1.1   myhostname.localdomain myhostname
EOF
# (replace "myhostname" with the same name you used above)

# ==== ROOT PASSWORD ====
passwd

# ==== BOOTLOADER (GRUB for EFI systems) ====
pacman -S grub efibootmgr

# Create EFI mount point
mkdir -p /boot/EFI
mount /dev/sdX1 /boot/EFI   # <-- replace /dev/sdX1 with your EFI partition (512M FAT32)

# ==== ROOT PASSWORD ====
passwd

# ==== BOOTLOADER (GRUB for EFI systems) ====
pacman -S grub efibootmgr

# Create EFI mount point
mkdir -p /boot/EFI
mount /dev/sdX1 /boot/EFI   # <-- replace /dev/sdX1 with your EFI partition (512M FAT32)

# Install GRUB
grub-install --target=x86_64-efi --efi-directory=/boot/EFI --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg


