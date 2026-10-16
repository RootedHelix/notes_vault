Related to [[ArchLinux]], [[Linux]], [[Arch Linux Backup System]]
#linux #archlinux #borg

# Functionality

[[Borg Pruning]]
[[Borg Commands]]


# Summary

**Borg** (more precisely **BorgBackup**) is an open-source **deduplicating backup system** designed for efficient, secure backups. It is widely used on Linux and Unix-like systems for backing up entire machines or large datasets to local or remote repositories.

---

## Core Concept

Borg performs **content-defined chunking and deduplication**. Instead of storing full copies of files every time you run a backup, it:

1. Splits files into chunks.
    
2. Hashes each chunk.
    
3. Stores only chunks that have **not been stored before**.
    

If a chunk already exists in the repository, Borg simply references it.

Result:

- Repeated backups consume **very little additional storage**.
    
- Incremental backups are extremely efficient.
    

---

## Key Features

### Deduplication

Borg deduplicates at the **chunk level**, not just file level.

Example:

| Backup                      | Data stored |
| --------------------------- | ----------- |
| First backup                | 100 GB      |
| Second backup (few changes) | ~100 MB     |

Most data is reused.

---

### Encryption

All data can be encrypted before it leaves the machine.

Supported modes include:

- `repokey`
    
- `keyfile`
    
- `authenticated`
    

Encryption uses **AES-256** with authentication.

---

### Compression

Borg supports several compression algorithms:

- `lz4` (very fast)
    
- `zstd`
    
- `zlib`
    
- `lzma`
    

Your command used:

```
--compression zstd,6
```

That means:

- Zstandard compression
    
- level 6 (balanced speed vs compression)
    

---

### Remote Backups

Borg can send backups to remote systems over SSH.

Example architecture:

```
Local machine
     |
     | SSH
     v
Remote server / NAS
     |
     Borg repository
```

This is exactly what you are doing with your **QNAP backup target**.

---

### Snapshot Archives

Each backup run creates an **archive** inside a repository.

Example repository:

```
repo/
   archive1
   archive2
   archive3
```

Each archive looks like a full backup but internally only stores new chunks.

---

## Important Borg Terminology

|Term|Meaning|
|---|---|
|Repository|The storage location for backups|
|Archive|A snapshot created during a backup run|
|Chunk|Small piece of file data used for deduplication|
|Prune|Deletes old archives based on retention policy|

---

## Example Workflow

Typical Borg lifecycle:

1. **Initialize repository**
    

```
borg init --encryption=repokey repo
```

2. **Create backups**
    

```
borg create repo::backup-2026-03-11 /
```

3. **List backups**
    

```
borg list repo
```

4. **Prune old backups**
    

```
borg prune repo --keep-daily=7 --keep-weekly=4
```

---

## Why Borg Is Popular

Compared with traditional backup tools:

|Feature|Borg|rsync backups|
|---|---|---|
|Deduplication|Yes|No|
|Encryption|Built-in|Usually external|
|Compression|Built-in|Limited|
|Snapshot archives|Yes|Manual|
|Remote backup|Native SSH|Possible but inefficient|

---

## Practical Use Case (Your Situation)

Given your setup:

- Linux system
    
- Remote **QNAP NAS**
    
- Borg over **SSH**
    
- ~10 TB available storage
    

Borg works well because:

- Incremental backups remain small
    
- Deduplication prevents uncontrolled growth
    
- Encryption protects remote storage
    
- `borg prune` manages retention
    

---

## One Important Clarification

Borg **does not automatically delete old backups**.

It will keep **every archive forever** until you run:

```
borg prune
```

So pruning is what prevents repositories from growing indefinitely.

---

If you'd like, I can also show you:

- a **proper Borg retention policy** (daily/weekly/monthly)
    
- how **Borg repository size actually grows over time**
    
- how **deduplication affects system backups**
    

These three concepts are crucial for understanding why Borg works so well for full-system backups.
