Related to [[Linux]] , [[WSL]] , [[SSH]]
#arch_linux #backup #borg #borg_commands 

# Personal Notes 
Arch Linux is on PC A and the backup moving into PC B on Windows through wsl using borg. 

- Install WSL on PC B
- Install borg via apt
- Mount the QNAP DAS into WSL (e.g. `/mnt/e`) in my case it was `/mnt/q`
- create dir for qnap for the borg-repos
- then we need to check `sudo service ssh status` make sure that it is installed and ready
	- then we look to ssh into PC B from PC A using `ssh user@<PC-B-IP>` where user is the linux username on PC B and `<PC-B-IP` is the ip adddress that PC A can reach for PC B


- confirm ssh is running and listening on WSL - `ss -tlnp | grep ssh`

- `cntrl + d ` closes the ssh session

> PC A: The ArchLinux PC,  PC B: The wsl and Backup Destination

### 1. Start the SSH Session within WSL on PC B

First Start the service
```bash
sudo service ssh start
```

Confirm
```bash
ss -tlnp | grep :22
```

### 2. Get WSL's internal IP (PC B)


```bash
hostname -I
```

keep the ip

### 3. Set Windows Port Forwarding (PowerShell as Admin)

**On PC B Windows Powershell Admin
```Powershell
netsh interface portproxy add v4tov4 listenport=22 listenaddress=0.0.0.0 connectport=22 connectaddress=WSL_IP
```
Replace WSL_IP with the one you got from `hostname -I`

### Step 4 — Allow Windows Firewall

Still on PC B:
```Powershell
New-NetFirewallRule -DisplayName "WSL SSH" -Direction Inbound -Protocol TCP -LocalPort 22 -Action Allow
```
### Step 5 — Get Windows LAN IP (PC B)

On PC B PowerShell:
```PowerShell
ipconfig
```


Find your LAN IPv4, OR the Ethernet adapter Ethernet IPv4 example:

`192.168.1.50` -> Goes into next command

---

#  Step 6 — From PC A (Linux)

Now connect:
```bash
ssh your_wsl_username@192.168.1.50
```


You should land inside WSL on PC B.


#### ⚠ Important WSL Reality

WSL IP changes after reboot.

So you must either:

- Re-run portproxy after reboot  
    OR
    
- Create a startup script to auto-update it
    

If you want, I can give you the permanent auto-fix method.


## Creating the repo ( NOT needed everytime )
- init the borg on PC B from PC A 
 `borg init \`
  `--encryption=repokey-blake2 \`
  `--rsh "ssh -p 2222" \`
  `--remote-path /usr/bin/borg \`
  `helixback@x.x.x.x:/mnt/q/borg-repos/linux-pc-newgen`



# Quick Check to ensure PC A can reach the borg Repo on PC B

```bash
borg info \
--rsh "ssh -p 2222" \
--remote-path /usr/bin/borg \
helixback@x.x.x.x:/mnt/q/borg-repos/linux-pc-newgen
```

 > This is going to ask for a BORG passphrase when you ssh in : it is under "borg passphrase" in 1password

### Backup command from PC A


```bash
sudo borg create \
  --stats \
  --progress \
  --compression zstd,6 \
  --one-file-system \
  --numeric-ids \
  --exclude-caches \
  --rsh "ssh -p 2222" \
  --remote-path /usr/bin/borg \
  --exclude /dev \
  --exclude /proc \
  --exclude /sys \
  --exclude /run \
  --exclude /tmp \
  --exclude /mnt \
  --exclude /media \
  --exclude /var/tmp \
  --exclude /var/cache \
  --exclude /swapfile \
  --exclude /lost+found \
  helixback@x.x.x.x:/mnt/q/borg-repos/linux-pc-newgen::{hostname}-{now} \
  / \
  /boot
```

- You Don't have to run this when ssh connection has already been established - this will establish its own ssh connection
- Make sure to check the port that is exposed on the `WSL` machine with: 
```bash
ss -tlnp | grep :22
```

the x.x.x.x is the ip address of PC B from powershell ipconfig

> This will back up root and boot directories ensuring a **Full System Backup**



### Check the data on the borg/qnap from PC A

```bash

borg info \
  --rsh "ssh -p 2222" \
  --remote-path /usr/bin/borg \
  helixback@x.x.x.x:/mnt/q/borg-repos/linux-pc-newgen

```



### Since there is no stable ip or address currently known hosts needs to be cleared

Those IP entries are stored in your SSH **known hosts database**. On Linux this is typically:

```bash
~/.ssh/known_hosts
```


Every time you connect to a new address (like `192.168.20.106`, `192.168.20.78`, etc.), SSH records the server’s host key there.




## Validate the backup by testing Restoring a Directory




```bash
mkdir ~/borg-test
cd ~/borg-test
```

###### Find the ARCHIVE NAME by listing the archives on borg

```bash
borg list \
  --rsh "ssh -p 2222" \
  helixback@x.x.x.x:/mnt/q/borg-repos/linux-pc-newgen
```



```bash
borg extract \
--rsh "ssh -p 2222" \
helixback@x.x.x.x:/mnt/q/borg-repos/linux-pc-newgen::ARCHIVE_NAME \
etc/fstab
```


## Borg Check
```bash 
borg check \
  --rsh "ssh -p 2222" \
  --remote-path /usr/bin/borg \
  helixback@YOUR_SERVER_IP:/mnt/q/borg-repos/linux-pc-newgen
```

# GPT


Let’s turn that into a **clean checklist** you can follow without second-guessing anything.

Below is the **exact order**, with **what “good” looks like** at each step. If one step fails, you stop there — no guessing.

---

## Borg / WSL / SSH CHECKLIST

### On **PC B (Windows + WSL)**

#### 1️⃣ Confirm WSL is running & sshd is listening

Open **Ubuntu (WSL)**:

```bash
ss -tlnp | grep ssh
```

✅ Good output looks like:

```
LISTEN 0 128 0.0.0.0:22
```

If you see nothing:

```bash
sudo service ssh start
```

Re-check until you see it listening.

---

#### 2️⃣ Get the **current WSL IP**

Still inside WSL:

```bash
ip -4 addr show eth0
```

You want:

```
inet 172.xx.yy.zz/20
```

📌 Copy that IP exactly.

---

#### 3️⃣ Verify / recreate the Windows port-forward

Open **PowerShell as Administrator** on PC B.

##### Reset old rules (safe):
- Optional

```powershell
netsh interface portproxy reset
```

##### Add the forward (replace with your WSL IP):

```powershell
netsh interface portproxy add v4tov4 
  listenaddress=0.0.0.0 listenport=2222 
  connectaddress=172.xx.yy.zz connectport=22
```
The WSL IP will go where that 172.xx.yy.zz is.

---

#### 4️⃣ Confirm Windows is actually listening on 2222

```powershell
netstat -ano | findstr :2222
```

✅ You must see:

```
LISTENING
```

If not — stop here and tell me.

---

#### 5️⃣ Allow port 2222 through Windows Firewall (once)

```powershell
netsh advfirewall firewall add rule name="Allow SSH WSL 2222" `
  protocol=TCP dir=in localport=2222 action=allow
```

---

#### 6️⃣ **Local test on PC B (critical)**

Still on **PC B**, in **PowerShell**:

```powershell
ssh -p 2222 sam@localhost 
```

or in my case

```powershell
ssh -p 2222 helixback@localhost 
```



✅ Success = you land in WSL  
❌ Failure = forwarding is broken (don’t test PC A yet)

> This will prompt you for a password this is the linux password

### Check the ssh session from inside WSL PC B

```bash
sudo service ssh status
```

---

### On **PC A (Linux)**

#### 7️⃣ Confirm PC B’s Windows IP

On **PC B**:

```powershell
ipconfig
```

Use the **Ethernet IPv4 address**, e.g.:

```
192.168.1.42
```

---

#### 8️⃣ Test from PC A → PC B

On **PC A**:

```bash
ssh -p 2222 wslusername@192.168.1.42
```

192.168.20.94

✅ Success = networking solved  
❌ Failure = paste the exact error

---

## 🔒 Once SSH works (next steps)

After this checklist passes, we immediately:

- Switch to SSH keys
    
- Disable password auth
    
- Create a Borg-only user
    
- Run:
    
    ```bash
    borg init
    ```
    

---

### 🧠 Mental anchor (remember this)

- **22** → sshd inside WSL
    
- **2222** → Windows “front door”
    
- **portproxy** → hallway between them
    

---

You’re doing this _exactly right_ — you stopped, summarized, and verified before moving on. That’s how you avoid brittle setups.

When you’re ready, run through the checklist top-to-bottom and tell me **which step fails** (or if they all pass).
