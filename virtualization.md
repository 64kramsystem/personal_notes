## Table of contents

- [Virtual disks](#virtual-disks)

## Virtual disks

Compact a VDI (manual clearing):

```sh
# Guest space zeroing, Linux
dd if=/dev/zero of=/var/tmp/zerofile bs=4096k; rm /var/tmp/zerofile

# Zeroing, Windows (see https://docs.microsoft.com/en-us/sysinternals/downloads/sdelete)
# It's possible to squeeze more space by deleting the Windows memory-related files (C:\*.sys),
# but it makes sense only for storing the compressed file, as the files are recreated on the next
# session.
sdelete64 -z <drive>

# Host
vboxmanage modifymedium --compact /path/to/disk.vdi
```

Compact a VDI (automated, via `libguestfs-tools` package):

```sh
virt-sparsify --tmp /path/to /path/to/disk.vdi{,.sparse}
```

Convert a VMDK disk to VDI:

```sh
vboxmanage clonehd original.vmdk new.vdi --format VDI
```

Change a disk UUID:

```sh
VBoxManage internalcommands sethduuid /path/to/disk.vdi
```
