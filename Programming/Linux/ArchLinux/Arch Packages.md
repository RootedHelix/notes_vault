Related to [[Arch]] , [[Linux]],
#arch_packages

sudo pacman -S hyprland waybar rofi mako swaybg xdg-desktop-portal-hyprland xorg-xwayland



## General
### Base Devel

### pacman-contrib

### Bear

### Clang

### reflector

### Yay 

- User AUR package manager
- Direct Page, [[Yay]]

### Go

### Git

### 1 Password

### Firefox

### wireless-regdb

### tree
- Used to recursively list the children of a directory

### Pulsewire 
 - Audio package

### Reflector 
- Used for managing arch mirrors
- Direct Page, [[Reflector]]
### Discord

sudo pacman -Syu wireless-regdb

Then reboot (the kernel loads this very early).
ArchWiki

    Set your country code (persistent)
    Edit /etc/conf.d/wireless-regdom and set e.g. WIRELESS_REGDOM="US", then reboot.
    (Temporary test without reboot: sudo iw reg set US; check with iw reg get.)
    ArchWiki

    Verify it’s loaded
    You should now see no regulatory.db failed messages, and iw reg get should show your country instead of 00 (the “world” domain).
    ArchWiki

### Cooler Control
- [[CoolerControl]]
- Application for monitoring temperature and fan speeds of hardware 
### liquidctl

### lm_sensors

### nvidia-open
- Driver for newer nvidia cards 

### nvidia-utils

### mesa-utils
- Utility to check for current graphics usage

### Zip


### Ollama
- Model engine for 5080 and hermes
##### ollama-cuda

## Docker

###### docker-rootless-extras

Used to run applications in isolation

## Xclip


## dmidecode

- Linux administrative utility used for observing hardware

### Keyring

### gnome-keyring

### libsecret

### seahorse

---
# Security

##### `sudo bash -c 'echo "nameserver 1.1.1.1" > /etcresolv.conf'`


### UFW (Uncomplicated Firewall)



### fail2ban ( IP banning)

is a **security tool** that protects your system from **brute-force attacks** and other suspicious behavior — especially over **services like SSH, FTP, or web apps**.

### Lynis ( Machine Audit)


### RkHunter ( Scans Machine for rootkits and backdoors )

#### Exact Commands

Install:
no need for cat EOF if from terminal
cat << 'EOF' > ~/arch_hardening.sh
#!/bin/bash
set -e

echo "[+] Updating system..."
sudo pacman -Syu --noconfirm

echo "[+] Installing firewall..."
sudo pacman -S ufw --noconfirm
sudo systemctl enable --now ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw enable

echo "[+] Installing fail2ban..."
sudo pacman -S fail2ban --noconfirm
sudo systemctl enable --now fail2ban
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo systemctl restart fail2ban

echo "[+] Installing Lynis and rkhunter..."
sudo pacman -S lynis rkhunter --noconfirm

echo "[+] Running rkhunter update and check..."
sudo rkhunter --update
sudo rkhunter --check

echo "[+] System hardened. Run 'sudo lynis audit system' manually for suggestions."
EOF

chmod +x ~/arch_hardening.sh



### Pipewire 

- Used for audio compatibility

## Linux NTP Time sync

bash  
sudo timedatectl set-ntp true  
systemctl status systemd-timesyncd.service  
### If it's inactive/disabled:  
sudo systemctl enable --now systemd-timesyncd.service

