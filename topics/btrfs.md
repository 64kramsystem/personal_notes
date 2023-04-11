## Table of contents

- [Table of contents](#table-of-contents)
- [Basic usage](#basic-usage)
- [Corruption](#corruption)
- [Mount options](#mount-options)
- [Other stuff](#other-stuff)

## Basic usage

Create the filesystem;
`-d raid1` specified both data and metadata to be redundant.

```sh
mkfs.btrfs -d raid1 <device1> <device2>
```

Show FS info; can be executed globally on any specific device.
'filesystem' can be shortened to 'fi'.

```sh
btrfs filesystem show [<device>|<mount>]

# Mirror can be observed from `RAID1` descriptions:
#
#   Data,RAID1: Size:8.00GiB, Used:7.16GiB (89.56%)
#   Metadata,RAID1: Size:1.00GiB, Used:230.91MiB (22.55%)
#   System,RAID1: Size:32.00MiB, Used:16.00KiB (0.05%)
#
btrfs filesystem usage $mount | grep -P '^\w+,'
```

Mount the FS.

```sh
mount <device> <mount>
```

Detailed data stats.

```sh
btrfs fi df [-h] <mount>
```

Add device (mirror); WATCH OUT! The balancing default is striping (for data).
Also converts a single device to mirror.

- `-f`                               : overwrite the destination FS, if there is any
- `--bg`|`--background`
- `-dconvert=raid1 -mconvert=raid1`  : convert data and metadata to RAID1
- `--full-balance`                   : skip pause and warning; not meaningful with `--balance` set
- `-v`                               : verbose; not meaningful with `--balance` set

```sh
btrfs device add [-f] /dev/sde <mount>
btrfs balance start -dconvert=raid1 -mconvert=raid1 -v --background <mount>
```

Monitor rebalance status (no built-in monitoring).

```sh
watch -n 1 btrfs balance status <mount>
```

Convert RAID1 to single devices; doesn't show the status.

```sh
btrfs balance start -v --force -mconvert=single -dconvert=single <mount>
```

Remove a device \[from a mirror\] (does rebalancing if required).

```sh
btrfs device device delete <device> <mount>
```

Replace a failed drive. At the end, add a device as specified above.

```sh
umount <mount>
mount -o degraded /dev/sdb <mount>
```

Find mounted volumes.

- [n]o header

```sh
findmnt -nt btrfs | awk '{print $1}'
```

Scrub.

- [-B] run in foreground, but best to watch status
- per [-d]evice stats

```sh
btrfs scrub start [-B] <mount>
btrfs scrub status -d <mount>
```

Defragment.

- use `-o autodefrag` option for automatic defragmentation (on light workloads)
- use `-o nodatacow` to disable CoW (so that data is modified in-place)

```sh
btrfs filesystem defrag <mount>
```

## Corruption

If data corruption is detected, then btrfs will log errors to syslog. If RAID 1 is enabled, btrfs will also automatically fix the corrupted data.

Print error stats.

```sh
btrfs device stats <mount>
```

## Mount options

- `defaults` is good enough (it includes `rw` and `relatime`)
  - set `0 0` as `dump`/`pass`
- `space_cache=v2` is commonly agreed to be useful
- `degraded` is useful, otherwise a broken device will prevent boot
- `discard=async`: don't use; daily (or even less frequent) trim is sufficient, so there's no need for optimization

## Other stuff

Recovery.

```sh
mount -o recovery /dev/sdb <mount>
```

Resize.

```sh
btrfs fi resize -2g <mount>
btrfs fi resize max <mount>
```
