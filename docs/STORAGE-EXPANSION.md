# Storage Expansion Runbook

This document records the storage expansion performed on the Nextcloud Ubuntu VM.

## Starting State

The Proxmox VM disk was initially configured at approximately 50 GiB.

Inside Ubuntu, the disk initially showed a smaller partition/LVM allocation even after the virtual disk was resized.

## Step 1 — Resize the Proxmox Disk

The VM disk was expanded from 50 GiB to 500 GiB in Proxmox.

## Step 2 — Expand the Guest Partition

The third partition was expanded with:

```bash
sudo growpart /dev/sda 3
```

The command reported that partition 3 changed from its original size to consume the available disk space.

## Step 3 — Resize the LVM Physical Volume

```bash
sudo pvresize /dev/sda3
```

Verification:

```bash
sudo pvs
```

The physical volume then reported approximately:

```text
PSize    <498.00g
PFree     474.00g
```

## Step 4 — Expand the Logical Volume

The Ubuntu root logical volume was expanded to consume the available space.

## Step 5 — Verify the Filesystem

```bash
df -h /
```

Final verification reported approximately:

```text
Size   490G
Used   8.5G
Avail  462G
Use%   2%
```

## Important Lesson

A VM disk resize occurs at the virtualization layer. Ubuntu does not automatically expand every storage layer underneath it.

The layers are:

```text
Proxmox virtual disk
        ↓
Partition
        ↓
LVM physical volume
        ↓
LVM logical volume
        ↓
Filesystem
```

Each layer must be verified and expanded as required.

## Thin-Pool Note

During the Proxmox resize, a warning indicated that the configured thin volume size exceeded the currently available physical thin-pool capacity. This is an important capacity-planning warning and should not be ignored on a production system.
