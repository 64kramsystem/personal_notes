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
btrfs filesystem usage <mount>
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
- `--full-balance`                   : skip pause and warning
- `-v`                               : verbose
- `--bg`|`--background`
- `-dconvert=raid1 -mconvert=raid1`  : convert data and metadata to RAID1

```sh
btrfs device add [-f] /dev/sde <mount>
btrfs balance start --full-balance -dconvert=raid1 -mconvert=raid1 -v --background <mount>
```

Show (not monitor) rebalance status.

```sh
btrfs balance status <mount>
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
