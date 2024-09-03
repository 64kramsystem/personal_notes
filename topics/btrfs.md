## Table of contents

- [Table of contents](#table-of-contents)
- [General information](#general-information)
- [Creation/subvolumes](#creationsubvolumes)
  - [Mount options (flags)](#mount-options-flags)
- [Mirroring](#mirroring)
- [Filesystem info](#filesystem-info)
- [Snapshots](#snapshots)
- [Sanity](#sanity)
- [Other operations/concepts](#other-operationsconcepts)

## General information

WATCH OUT!!! When operating on subvolumes, must mount the parent first (either another SV or the FS).

For example, if a root subvolume `.snapshots` has a child `.snapshots/root`, must mount `.snapshot` first, in order to delete the child. In turn, in order to delete `.snapshots`, must mount the filesystem.

If accidentally operating on paths of subvolumes, when the parent FS is not mounted, one will get the confusing message `ERROR: Could not statfs: No such file or directory`.

This is not a problem, though: even if a filesystem's subvolumes are mounted, the filesystem itself can still be mounted somewhere else.

## Creation/subvolumes

Create a FS, with two subvolumes, mount them under `/mnt`, and set fstab:

```sh
# Create the filesystem
#
mkfs.btrfs /dev/ubuntu-vg/btest

# Mount the filesystem ("top-level volume").
#
mount /dev/ubuntu-vg/btest /mnt

# Create subvolumes `@` and `@home`.
#
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@home
#
ls -1 /mnt
# @
# @home

# Unmount, remount, and prepare the directories structure.
#
umount /mnt
mount -o subvol=@ /dev/ubuntu-vg/btest /mnt
mkdir /mnt/home
mount -o subvol=@home /dev/ubuntu-vg/btest /mnt/home

# List; notice how the SVs' top level is the FS.
#
btrfs subvolume list /mnt
#
# ID 256 gen 12 top level 5 path @
# ID 257 gen 7 top level 5 path @home

# Update fstab
#
UUID=$btrfs_uuid /     btrfs defaults,subvol=@     0 1
UUID=$btrfs_uuid /home btrfs defaults,subvol=@home 0 2
```

### Mount options (flags)

Suggested:

- `defaults`        : good enough (it includes `rw` and `relatime`); set `0 0` as `dump`/`pass`
- `compress=zstd`   : levels 1-3 are realtime (format: `zstd:3`)
- `degraded`        : useful, otherwise a broken device will prevent boot
- `nodiscard`       : Ubuntu's daily job is sufficient

Use-case dependent:

- `noatime`         : supported, but think before using it

Unnecessary

- `space_cache=v2`  : default
- `discard=async`   : default; see `nodiscard`

## Mirroring

```sh
# Create a mirror
#
# `-d raid1` specified both data and metadata to be redundant.
#
mkfs.btrfs -d raid1 $device1 $device2

# (mount; CHECK IF JUST MOUNT DEVICE1!)

# Add device (mirror); WATCH OUT! The balancing default is striping (for data).
# Also converts a single device to mirror.
#
# - `-f`                               : overwrite the destination FS, if there is any
# - `--bg`|`--background`
# - `-dconvert=raid1 -mconvert=raid1`  : convert data and metadata to RAID1
# - `--full-balance`                   : skip pause and warning; not meaningful with `--balance` set
# - `-v`                               : verbose; not meaningful with `--balance` set
#
btrfs device add [-f] $device3 $mount
btrfs balance start -dconvert=raid1 -mconvert=raid1 -v --background $mount

# Monitor rebalance status (no built-in monitoring)
#
watch -n 1 btrfs balance status $mount

# Convert RAID1 to single devices; doesn't show the status
#
btrfs balance start -v --force -mconvert=single -dconvert=single $mount

# Remove a device [from a mirror] (does rebalancing if required)
#
btrfs device device delete $device $mount

# Replace a failed drive. At the end, add a device as specified above
#
umount $mount
mount -o degraded /dev/sdb $mount
```

## Filesystem info

```sh
# Show FS info; can be executed globally on any specific device.
# 'filesystem' can be shortened to 'fi'.
#
btrfs filesystem show [$device|$mount]

# Mirror can be observed from `RAID1` descriptions:
#
#   Data,RAID1: Size:8.00GiB, Used:7.16GiB (89.56%)
#   Metadata,RAID1: Size:1.00GiB, Used:230.91MiB (22.55%)
#   System,RAID1: Size:32.00MiB, Used:16.00KiB (0.05%)
#
btrfs filesystem usage $mount | grep -P '^\w+,'
```

## Snapshots

WATCH OUT!: Mount the FS, not the root SV!

```sh
# We use `.` (hidden) instead of `@` just for differentiation.
#
btrfs subvolume create /mnt/.snapshots
#
btrfs subvolume list /mnt
#
# ID 256 gen 12 top level 5 path @
# ID 257 gen 7 top level 5 path @home
# ID 259 gen 16 top level 5 path .snapshots

# Create.
# WATCH OUT!!!! Don't forget to preceed the mount source with `@`!!
#
# - `-r`: readonly; good for backups
#
btrfs subvolume snapshot -r /mnt/@     /mnt/.snapshots/root
btrfs subvolume snapshot -r /mnt/@home /mnt/.snapshots/home

# Restore
#
btrfs subvolume delete /mnt/@home
btrfs subvolume snapshot /mnt/.snapshots/home /mnt/@home
```

## Sanity

If data corruption is detected, then btrfs will log errors to syslog. If RAID 1 is enabled, btrfs will also automatically fix the corrupted data.

```sh
# Print error stats
#
btrfs device stats $mount

# Scrub
#
# - [-B] run in foreground, but best to watch status
# - per [-d]evice stats
#
btrfs scrub start [-B] $mount
btrfs scrub status -d $mount

# Recovery
#
mount -o recovery /dev/sdb $mount
```

## Other operations/concepts

```sh
# Find mounted volumes
#
# - [n]o header
#
findmnt -nt btrfs | awk '{print $1}'

# Detailed data stats
#
btrfs fi df [-h] $mount

# Defragment
#
# - use `-o autodefrag` option for automatic defragmentation (on light workloads)
# - use `-o nodatacow` to disable CoW (so that data is modified in-place)
#
btrfs filesystem defrag $mount

# Resize
#
btrfs fi resize -2g $mount
btrfs fi resize max $mount
```
