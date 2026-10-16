3
here’s a compact linux command cheat sheet you can keep open in a terminal. it’s tailored a bit toward arch users (pacman/yay), but works broadly.

# basics & navigation

```bash
pwd                     # print working directory
whoami                  # current user
id                      # uid/gid & groups
hostnamectl             # host info
date; cal               # time/date; calendar
clear                   # clear screen

ls -lah                 # list (long, all, human sizes)
cd /path/to/dir         # change directory
pushd /dir; popd        # dir stack (go/return)
tree -L 2               # directory tree (install `tree`)
```

# files & directories

```bash
touch file              # create empty file (or update mtime)
mkdir -p dir/sub        # make dirs (parents)
cp -r src/ dst/         # copy (recursive)
mv old new              # move/rename
rm -i file              # remove (prompt)
rm -rf dir              # remove recursively (force)  # DANGEROUS
ln file link            # hard link
ln -s target link       # symlink
stat file               # metadata
file bin                # detect file type
du -sh ./*              # per-item sizes
df -h                   # disk free
```

# view & edit

```bash
cat file                # print
tac file                # reverse
nl file                 # number lines
less file               # pager (q to quit, /search)
head -n 20 file         # first 20 lines
tail -f logfile         # follow
wc -lwm file            # counts (lines/words/chars)
nano file               # simple editor
```

# search & find

```bash
grep -RIn "pat" .       # recursive, show line nums
rg "pat"                # ripgrep (faster, if installed)
find . -name "*.c"      # find by name
find . -type f -size +10M -mtime -7   # filters
```

# permissions & ownership

```bash
chmod 644 file          # rw-r--r--
chmod +x script.sh      # add execute
chown user:group file   # change owner
umask                   # default perms mask
```

# processes & jobs

```bash
ps aux | less           # all processes
top / htop              # live view
pgrep -fl name          # find pids by name
kill -TERM <pid>        # graceful stop
kill -9 <pid>           # force kill
jobs; bg %1; fg %1      # shell jobs
nohup cmd &             # run immune to hangups
```

# networking

```bash
ip a                    # interfaces & IPs
ip r                    # routes
ss -tulpen              # listening sockets
curl -I https://…       # HTTP head
wget URL                # download
ping -c 4 host          # reachability
traceroute host         # route (install `traceroute`)
dig +short A example.com  # DNS (package: bind / dnsutils)
```

# archives & compression

```bash
tar -czf out.tgz dir/   # create tar.gz
tar -xzf file.tgz       # extract
zip -r out.zip dir/     # zip dir
unzip file.zip          # unzip
xz -z file ; xz -d file.xz  # xz compress/decompress
```

# text processing (pipes)

```bash
sort file | uniq -c | sort -nr     # frequency count
awk '{print $1,$3}' file           # column select
cut -d: -f1 /etc/passwd            # cut fields
sed -n '5,10p' file                # print lines 5–10
sed -E 's/foo/bar/g' file          # substitute
paste f1 f2                        # merge columns
```

# users & groups

```bash
sudo -v                  # refresh sudo credentials
sudo useradd -m sam      # add user (home dir)
sudo passwd sam          # set password
groups                   # your groups
```

# systemd (services & logs)

```bash
systemctl status nginx             # service status
sudo systemctl restart nginx       # restart
sudo systemctl enable --now svc    # enable + start
journalctl -u nginx --since "1 hour ago"   # recent logs
journalctl -f                     # follow system log
```

# packages (arch & generic)

```bash
# Arch (pacman / yay)
sudo pacman -Syu             # sync & upgrade
sudo pacman -S pkg           # install
sudo pacman -Rns pkg         # remove with deps
pacman -Qo /path/bin         # which pkg owns file
pacman -Qs regex             # search local db
yay -S aurpkg             wirele   # install from AUR
yay -Syu                     # upgrade inc. AUR

# Debian/Ubuntu
sudo apt update && sudo apt upgrade
sudo apt install pkg
```

# disks, mounts, hardware

```bash
lsblk -f                 # block devices & filesystems
mount | column -t        # current mounts
sudo mount /dev/sdb1 /mnt
sudo umount /mnt
lspci; lsusb             # PCI/USB devices
free -h                  # memory
uname -a                 # kernel info
```

# ssh & scp

```bash
ssh user@host                    # login
scp file user@host:/path         # copy to host
rsync -avh --progress src/ dest/ # efficient sync
ssh-keygen -t ed25519            # create key
ssh-copy-id user@host            # install key
```

# git (quick)

```bash
git clone URL
git status; git add -A; git commit -m "msg"
git pull --rebase; git push
git log --oneline --graph --decorate --all
```

# shell tips

```bash
history | tail                 # last commands
CTRL+R                         # reverse search history
alias ll='ls -lah'             # quick alias (put in ~/.bashrc or ~/.zshrc)
export PATH="$HOME/bin:$PATH"  # prepend to PATH
```

## Launch Processes Detached from the Terminal

```shell
nohup my_program > output.log 2>&1 &
```

# safety notes

- be cautious with `sudo`, `rm -rf`, and wildcard `*`.
    
- double-check paths before destructive commands: `echo` first (e.g., `echo rm -rf "$dir"`).
    
- use `--dry-run` when available (e.g., `rsync --dry-run`).
    

want this as a printable one-pager or a PDF tailored to your everyday commands (arch + hyprland dev workflow)? I can generate it.