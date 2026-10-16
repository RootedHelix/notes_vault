Related to [[Linux]], [[ArchLinux]], [[Arch Linux Backup System]], [[Borg]]
#borg #linux #borg_pruning

# My Borg Pruning Policy

```bash
borg prune --rsh "ssh -p 2222" --keep-daily 14 --keep-weekly 8 --keep-monthly 24 helixback@192.168.20.109:/mnt/q/borg-repos/linux-pc-newgen
```


Borg prunes **archives**, not files.

In Borg terminology:

- **Repository** → the storage location (your QNAP directory)
    
- **Archive** → a snapshot created by `borg create`
    
- **Chunks** → deduplicated pieces of file data stored in the repo
    

Your command:

```bash
borg create ... REPO::{hostname}-{now}
```

creates an **archive** each time it runs.

Example repository state after several runs:

```
linux-pc-newgen
 ├── archboxx-2026-03-11T10:00
 ├── archboxx-2026-03-12T10:00
 ├── archboxx-2026-03-13T10:00
 ├── archboxx-2026-03-14T10:00
 └── archboxx-2026-03-15T10:00
```

Each of those is a **snapshot of the filesystem at that time**.

## What `borg prune` does

`borg prune` removes **old archives** according to a retention policy.

Example:

```bash
borg prune \
  --keep-daily 7 \
  --keep-weekly 4 \
  --keep-monthly 12 \
  REPO
```

Meaning:

- keep the **last 7 daily snapshots**
    
- keep **1 snapshot per week for 4 weeks**
    
- keep **1 snapshot per month for 12 months**
    

Older archives are deleted.

Example result:

```
archboxx-2026-03-15  ← kept (daily)
archboxx-2026-03-14  ← kept
archboxx-2026-03-13  ← kept
archboxx-2026-03-12  ← kept
archboxx-2026-03-11  ← kept
archboxx-2026-03-10  ← kept
archboxx-2026-03-09  ← kept

archboxx-2026-02-28  ← kept (weekly)
archboxx-2026-02-21  ← kept
archboxx-2026-02-14  ← kept
archboxx-2026-02-07  ← kept

archboxx-2025-12-01  ← kept (monthly)
archboxx-2025-11-01  ← kept
```

Everything older gets removed.

## Important: pruning does NOT delete data immediately

Because Borg deduplicates chunks.

Example:

```
Archive A → references chunks {1,2,3}
Archive B → references chunks {1,2,4}
Archive C → references chunks {1,2,5}
```

If Archive A is pruned:

```
chunks {1,2,3}
```

- chunks 1 and 2 stay (still used by B and C)
    
- chunk 3 becomes unused and can be reclaimed
    

This is why Borg repositories remain efficient.

## After pruning

To reclaim unused storage you run:

```
borg compact REPO
```

This cleans up orphaned chunks.

## In simple terms

Your repository becomes a **timeline of system states**:

```
backup1 → March 1
backup2 → March 2
backup3 → March 3
...
```

Pruning removes **older timeline points**, not files inside the backups.

## Example restore capability

If something breaks on March 20:

```
borg extract REPO::archboxx-2026-03-18
```

You restore the machine to how it looked on March 18.

That is the purpose of the archive snapshots.

## What your 10 TB QNAP will store

It will store:

```
repo
 ├── archive snapshots
 └── deduplicated chunk storage
```

Even if you have 100 archives, most unchanged files are stored only once.

## Summary

Borg prune removes:

- **old snapshots (archives)**
    

It does **not delete individual files inside a backup**.

The goal is to keep useful restore points while preventing the archive list from growing indefinitely.