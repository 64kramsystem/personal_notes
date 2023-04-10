# ZFS

- [ZFS](#zfs)
  - [Pools](#pools)
    - [Basic operations](#basic-operations)
    - [Features](#features)
      - [Find current GRUB supported features](#find-current-grub-supported-features)
    - [Import/export](#importexport)
    - [Admin](#admin)
    - [Encryption](#encryption)
  - [Mirror/devices](#mirrordevices)
  - [Datasets](#datasets)
  - [Mountpoints](#mountpoints)
  - [Settings](#settings)
    - [Compression](#compression)
  - [Snapshotting](#snapshotting)
    - [Lightweight per-file rollback workflow](#lightweight-per-file-rollback-workflow)
    - [Cool diffing functions](#cool-diffing-functions)
  - [Packaging/installation](#packaginginstallation)

## Pools

### Basic operations

For more advanced concepts, see http://j.mp/18tSgvx.

```sh
# Create a pool from a file/device (if file, it must be present).
# - `-d` + `-o $features`: enable only the specified features.
#
zpool create [-d {-o feature@$name>=enabled, ...}] $pool $device

# List the pools.
#
zpool list

# More details about the pools.
# [-v] displays files with error, if there are any.
#
zpool status [-v]

# Destroy a pool.
#
zpool destroy $pool
```

### Features

WATCH OUT! Once a feature is enabled, it can't be disabled.

```sh
# Get the pool features/values.
#
zpool get all $pool

# Hackish way of getting the supported features.
#
man zpool-features | grep -E '^       \w+$'

# Without $pool list the upgrade status of all the pools; with it, upgrades it.
#
zpool upgrade [$pool]
```

#### Find current GRUB supported features

```sh
# First, enable the src repository

apt source grub-pc
cd grub2-*/
perl -ne 'print if /\*spa_feature_names\[\]/ .. /^\};$/' grub-core/fs/zfs/zfs.c
```

As of 2.04:

- `com.delphix:embedded_data`
- `com.delphix:extensible_dataset`
- `com.delphix:hole_birth`
- `org.illumos:lz4_compress`
- `org.open-zfs:large_blocks`

### Import/export

```sh
# Import in the given path.
#
zpool import -d $path $pool

# Import pool using $alt_root as temporary root.
#
zpool import -R $alt_root $pool

# Rename a pool (permanently).
#
zpool import $pool $new_name

# Export must not be used to unmount, otherwise the pool is not automatically imported on boot.
#
zpool export $device_or_mountpoint
```

### Admin

```sh
# Expand a pool to the entire partition. If performed on a mirror, the operation must be performed on each device.
#
parted -s $disk_device resizepart $part_number_1_based 100%
zpool online -e $pool $part_device

# Scrub a pool. Results are displayed both in :list and :status.
#
zpool scrub $pool
```

### Encryption

Import an encrypted pool. IMPORTANT!:

- if `-l` is not specified, and the pool is encrypted, it will be imported, with successful exit status (!), and with empty content (!!)
- if `-l` is specified, and the pool is not encrypted, it will be imported, without prompt

```sh
zpool import -l $pool
```

Importing an Ubuntu-configured pool is more complex; attempting a vanilla import will result in the `Failed to open key material file` error. Procedure from [Stack Overflow](https://askubuntu.com/a/1351546):

```sh
zpool import -R /mnt $pool
cryptsetup open /dev/zvol/$pool/keystore zfskey
# The mountpoint is typically /dev/dm-0
mount /dev/dm-0 /mnt
cat /mnt/system.key | zfs load-key -L prompt $pool
umount /mnt
zfs mount -a
```

## Mirror/devices

```sh
# Create a mirrored pool.
#
zpool create $pool mirror $device1 $device2

# Replace a device.
# If the new device is in the same place as the old one, don't specify the new one.

zpool replace $pool $old_device [$new_device]

# Add a new device to a pool (doesn't mirror!)
#
zpool add $pool $device

# Add a new top level mirror
#
zpool add $pool mirror $device1 $device2

# Attach a new device to a mirror (convert the pool to a mirror, if it isn't already so).
# Resilvering will start immediately.
# If a device is detached and reattached, it will still take a long time to resilver (use offline/online
# for that use case).
# Best to use the /dev/disk/by-id device for both.
#
zpool attach $pool $device_in_mirror $new_device

# Detach a device from a pool (/mirror):
#
zpool detach $pool $device

# Temporarily disconnect a device (from a mirror); resumes quickly.
#
zpool offline $pool $device
zpool online $pool $device

# Delete a device
#
zfs destroy $pool/$device

# ZFS mirroring with different sector sizes (9=512, 12=4096). Watch out! It has a significant impact on performance.
#
fdisk -l                                                         # show the SS for the disks
zdb | grep ashift                                                # show the pool current ashift
zpool attach -o ashift=9 $pool $device_in_mirror $new_device     # force the optimal ashift for the attaching device
```

## Datasets

```sh
# Rename a dataset. Can be performed on mounted datasets.
# Since (presumably) child datasets refer to parents, renaming a parent will propagate to children.
#
#
zfs rename $from $to
```

## Mountpoints

WATCH OUT!: The dataset mountpoints are inherited from children, unless `mountpoint` is specified.

```sh
# Don't allow pool mounting.
# Useful on creating a pool that is purely a container
#
zpool create -O canmount=off

# Create with a permanent mountpoint (-O mountpoint), but a different temporary one (-R).
# The mountpoints are created on pool creation, but not destroyed on pool destruction.
#
zpool create -O mountpoint=/ -R /media/disk_a

# Store the mountpoint and mount a dataset/pool.
# The dataset is mounted automatically when importing the pool!
# The mountpoint presence is optional, but it won't be deleted when unmounting.
#
zfs set mountpoint=$mountpoint $dataset
zfs set readonly=on $dataset

# Get mountpoint/mount status -> mount.
#
zfs get mountpoint $filesystem

# Watch out! Don't confuse $filesystem with the path - using the second will print the mount status
# of the pool the path belongs to.
#
zfs get mounted [$filesystem]

zfs mount $filesystem

# (Re)mount in readonly mode; if a FS is mounted, use the 'remount' option to change a property.
#
zfs mount -o remount,ro $dataset
```

## Settings

### Compression

```sh
$ zfs get all [$pool] | grep -P '\bcompress'
rpool  compressratio         1.21x                 -
rpool  compression           lz4                   local

$ zfs set compression=zstd $pool
```

## Snapshotting

(`$volume` can also be a pool)

Create a subvolume, then a snapshot.

```sh
zfs create -o mountpoint=$mountpoint $pool/$volume
zfs snapshot $volume@$snaphot
```

List snapshots.

```sh
zfs list -t snapshot
```

Rollback after modifications!

```sh
zfs rollback $volume@$snaphot
```

Destroy snapshot (changes are kept), then subvolume.

```sh
# `-r`: destroy subsequent snapshots, if present
#
zfs destroy -r $volume@$snaphot
zfs destroy $volume
```

Check SIMD performance (see https://github.com/zfsonlinux/zfs/issues/9215).

```sh
cat /proc/spl/kstat/zfs/fletcher_4_bench
cat /proc/spl/kstat/zfs/vdev_raidz_bench
```

### Lightweight per-file rollback workflow

```sh
zfs create -o mountpoint=/busy rpool/busy

# Move the file, symlink it, and create the snapshot
mv components/busybear.raw /busy/
ln -s /busy/busybear.raw components/
zfs snapshot rpool/busy@unchanged

# do all the work, and when required, rollback:
zfs rollback rpool/busy@unchanged

# Restore the file and destroy the snapshot/volume
zfs rollback rpool/busy@unchanged
mv -f /busy/busybear.raw components/
zfs destroy rpool/busy@unchanged
zfs destroy rpool/busy
```

### Cool diffing functions

The function don't work with ecryptfs, due to the files at a higher level than the storage layer.

`zdiff` and `zversions` could be improved (simplified) by searching `.zfs` from the bottom up.

```sh
# Recreate a snapshot.
#
# $1: pool, $2: snapshot
#
function zreset {
  zfs destroy "$1@$2"
  zfs snapshot "$1@$2"
}

# Diffs against a snapshot.
#
# $1: pool, $2: snapshot
#
function zdiffs {
  zfs diff "$1@$2"
}

# Diff a file (with `meld`).
#
# $1: filesystem mount (eg. /home), $2: snapshot, $3: file path (relative to mount)
#
function zdifff {
  meld "$1/.zfs/snapshot/$2/$3" "$1/$3"
}

# Creates a `.before` (=at snapshot time) and `.after` versions of a given file.
# Won't create the .before if the file doesn't exist before.
#
# $1: filesystem mount (eg. /home), $2: snapshot, $3: file path (relative to mount, $4: output directory
#
function zversions {
  mkdir -p "$4"

  if [[ -f "$1/.zfs/snapshot/$2/$3" ]]; then
    cp "$1/.zfs/snapshot/$2/$3" "$4/$(basename "$3").before"
  fi

  cp "$1/$3" "$4/$(basename "$3").after"
}
```

## Packaging/installation

In order to setup the project for packaging or other tasks:

```sh
./autogen.sh
./configure
```

Packaging the debs can be done via `make deb`, however, it causes dependency issues on installation.

The easiest way is probably to install and hold `zfs-dkms` package, replace the `/usr/src/zfs-x.y` directory content with the configured project, and copy the old `dkms.conf` to the project root (optionally editing it); initramfs updates will now compile the new version.
