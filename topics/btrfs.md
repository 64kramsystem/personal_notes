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
  - [Swapfile on Btrfs](#swapfile-on-btrfs)

## General information

WATCH OUT!:

- **When operating on subvolumes, must mount the parent first** (either another SV or the FS).
  For example, if a root subvolume `.snapshots` has a child `.snapshots/root`, must mount `.snapshot` first, in order to delete the child. In turn, in order to delete `.snapshots`, must mount the filesystem.
  If accidentally operating on paths of subvolumes, when the parent FS is not mounted, one will get the confusing message `ERROR: Could not statfs: No such file or directory`.
  This is not a problem, though: even if a filesystem's subvolumes are mounted, the filesystem itself can still be mounted somewhere else.
- **Some error messages are not reported in the console**, but in the kernel ring buffer
  Use `dmesg | grep -i btrfs` to see the messages

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

**WATCH OUT!!!!!!!!!**: If the underlying partitions are not of the exact same size, the filesystem usage will be confusing, as the non-mirrored data size will be higher than the difference between the partition sizes (in this case, ~1.5GB):

    Data, single: total=5.00GiB, used=4.65GiB
    Data, RAID1: total=97.00GiB, used=95.49GiB
    [...]
    WARNING: Multiple block group profiles detected, see 'man btrfs(5)'
    WARNING:    Data: single, raid1

```sh
# Create a mirror
#
# `-d raid1` specified both data and metadata to be redundant.
#
mkfs.btrfs -d raid1 $device1 $device2

# => Now mount (see [Creation/subvolumes](#creationsubvolumes)).

# (mount; CHECK IF JUST MOUNT DEVICE1!)

# Add a device to a filesystem, but does not initiate any filesystem change.
#
btrfs device add [-f] $device3 $mount

# Converts a single profile to RAID-1.
# WATCH OUT! If the conversions are not specified, the default is striping (for data).
#
# - `-v`                               : verbose (global option)
# - `-f|--force`                       : overwrite the destination FS, if there is any
# - `-dconvert=raid1 -mconvert=raid1`  : convert data and metadata to RAID1
# - `--bg|--background`
# - `--full-balance`                   : skip pause and warning
#
btrfs -v balance start --force -dconvert=raid1 -mconvert=raid1 --background $mount

# Monitor rebalance status (no built-in monitoring)
#
watch -n 1 btrfs balance status $mount

# => In order to observe that a FS is in RAID-1, see [Filesystem info](#filesystem-info).

# Convert RAID1 to single profiles (no RAID, uses the total devices space); no progress is displayed
# (see monitoring above).
#
# - `--force`                          : without, btrfs complains that metadata redundancy is reduced
#
# WATCH OUT!!:
#
# - SLOOOOOOOOOOOOOOOOOOOW!!!!
# - the two devices are still part of the array; need to remove one of the if want to convert to a
#   single drive - in such case, the removed drive won't contain data
# - if the objective is to convert to a single profile, it's simpler to unmount, overwrite one of
#   the devices, mount in degraded mode, and run the conversion
#   - if ultimately moving the data to a new mirror, just keep the filesystem degraded
#
btrfs -v balance start --force -mconvert=single -dconvert=single --background $mount

# Remove a device [from a mirror] (does rebalancing if required).
# Use `missing` as $device, if the device is not functional.
#
btrfs device remove $device $mount

# If a drive fails and the system is rebooted and the degraded option is not in the default mount,
# will need to mount with such option.
#
mount -o degraded /dev/sdb $mount
```

## Filesystem info

```sh
# Show FS info; can be executed globally on any specific device.
# 'filesystem' can be shortened to 'fi'.
#
btrfs filesystem show [$device|$mount]

# RAID-1 (mirror) info can be inferred from `RAID1` descriptions:
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

# Detailed data stats (including mirroring info)
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

### Swapfile on Btrfs

By default, swap files can't be used on a Btrfs filesystem. Need to disable compression:

```sh
mkdir /swapdir
chattr +C /swapdir
# ... now move the swap file, and update fstab
```
