Related to [[SSH]]
#ssh #ssh_remove_known_hosts

## Personal Notes 

I just nvim into the #ssh_file_path and then manually delete the 192.x.x.x ips because all of the other entries like github or what not are not from the **local host** like that ip. 

## AI Response 

Those IP entries are stored in your SSH **known hosts database**. On Linux this is typically:

> SSH File Path #ssh_file_path
```
~/.ssh/known_hosts
```

Every time you connect to a new address (like `192.168.20.106`, `192.168.20.78`, etc.), SSH records the server’s host key there.

You have three clean ways to remove them.

---

## Method 1 — Remove a specific IP (recommended)

Use `ssh-keygen` to delete an entry safely.

Example:

```bash
ssh-keygen -R 192.168.20.106
```

If you connected on a custom port (like `2222`), use:

```bash
ssh-keygen -R "[192.168.20.106]:2222"
```

This edits `known_hosts` correctly and removes only that entry.

---

## Method 2 — Remove all entries for your backup machine

If several entries accumulated, you can remove them all:

```bash
ssh-keygen -R 192.168.20.78
ssh-keygen -R 192.168.20.106
```

---

## Method 3 — Reset the entire known hosts file

If you want to wipe everything:

```bash
rm ~/.ssh/known_hosts
```

SSH will recreate it automatically when you connect again.

Downside: you lose verification history for every host.

---

## Where your current entries likely are

Since your shell prompt earlier showed:

```
/mnt/c/Users/samo1
```

you might actually be using the **Windows SSH known_hosts file** instead:

```
C:\Users\samo1\.ssh\known_hosts
```

Inside WSL that path is:

```
/mnt/c/Users/samo1/.ssh/known_hosts
```

You can inspect it with:

```bash
cat /mnt/c/Users/samo1/.ssh/known_hosts
```

or remove entries with:

```bash
ssh-keygen -f "/mnt/c/Users/samo1/.ssh/known_hosts" -R "[192.168.20.106]:2222"
```

---

## Best long-term fix

Instead of constantly deleting entries, use a **stable SSH target**:

Either:

```
helixback@DESKTOP-QOCTMJJ.local
```

or give PC B a **reserved DHCP IP** in your router.

Then SSH will only store **one host identity**, and the warnings disappear.

---

If you want, I can also show a **clean SSH config file setup for Borg** that removes the need to type ports, IPs, and options every time. It makes the backup command much simpler and more reliable.
