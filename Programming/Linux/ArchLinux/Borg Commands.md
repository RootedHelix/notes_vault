Related to [[Borg]], [[Linux]]
#borg #linux #archlinux #borg_commands


**List Borg Archives**: 

```bash
borg list \
  --rsh "ssh -p 2222" \
  --remote-path /usr/bin/borg \
  helixback@x.x.X.xxx:/mnt/q/borg-repos/linux-pc-newgen

```

**Prune Borg**

```bash
borg prune --rsh "ssh -p 2222" --keep-daily 14 --keep-weekly 8 --keep-monthly 24 helixback@192.168.20.109:/mnt/q/borg-repos/linux-pc-newgen

```


**Validate if the Back Up Worked with a Test Restore**

```bash
mkdir ~/borg-test
cd ~/borg-test
```

```bash
borg extract \
--rsh "ssh -p 2222" \
helixback@x.x.x.x:/mnt/q/borg-repos/linux-pc-newgen::ARCHIVE_NAME \
etc/fstab
```