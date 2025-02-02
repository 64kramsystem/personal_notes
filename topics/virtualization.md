# Virtualization

- [Virtualization](#virtualization)
  - [Virtual disks](#virtual-disks)
    - [Creation/copy/conversion:](#creationcopyconversion)
    - [Zeroing/compacting/compressing an image](#zeroingcompactingcompressing-an-image)
    - [Working with an image content](#working-with-an-image-content)
      - [Filesystem operations](#filesystem-operations)
      - [Mount an image](#mount-an-image)
  - [Software-specific](#software-specific)
    - [VMWare](#vmware)
      - [Performance](#performance)
      - [Compatibility](#compatibility)
      - [Modules compilation/install](#modules-compilationinstall)
      - [General issues/tweaks](#general-issuestweaks)
      - [Network fix](#network-fix)
      - [Commandline Management](#commandline-management)
    - [VirtualBox](#virtualbox)
    - [QEMU](#qemu)
      - [Usermode](#usermode)
      - [RISC-V](#risc-v)
  - [3D Acceleration/VFIO](#3d-accelerationvfio)
    - [Cards consumption](#cards-consumption)
    - [nvidia-smi](#nvidia-smi)
  - [Vagrant](#vagrant)
  - [WINE](#wine)
  - [DOSBox(-X)](#dosbox-x)
    - [Key bindings](#key-bindings)
    - [Floppy](#floppy)
    - [Tools](#tools)
    - [Basic configuration](#basic-configuration)

## Virtual disks

### Creation/copy/conversion:

Create:

```sh
# The $format defaults to raw.
# The $image is overwritten, if it exists!
#
# Backing:
# - if the backing image path is relative, it's relative to the image.
# - it's possible to create a diff image of a diff image
# - WATCH OUT!: It's possible to specify a size, but in at least one instance, it caused subtle issues.
#
qemu-img create [-f $image_format] $image [-F $backing_image_format] [-b $backing_image] [20G]

# Create a qcow image (with NTFS filesystem) from a directory:
#
virt-make-fs --verbose --format=qcow2 --type=ntfs $input_dir $dest.qcow2

# Create a raw file, with a partition, with standard linux tools
# 8300: Partition type: native Linux FS
# Guestmount is easier (see [Mount an image](#mount-an-image)).
#
dd if=/dev/zero bs=1M count=10240 of=disk.img
sgdisk -n1:0:0 -t1:8300 disk.img
loop_device=$(sudo losetup --show --find --partscan disk.img)
sudo mkfs.ext4 ${loop_device}p1
sudo mount ${loop_device}p1 /mnt
# ...copy stuff etc...
sudo umount /mnt
sudo losetup -d $loop_device
# now, use losetup (see below)
```

Copy:

```sh
virt-copy-in -a $image.qcow2 $source_dir $image_dest_dir
```

Convert:

```sh
vboxmanage clonehd --format VDI $source.vmdk $dest.vdi

# Options: [p]rogress, [O]utput format (not autodetected)
#
qemu-img convert -p -O qcow2 $source.qcow2 $dest.qcow2
```

### Zeroing/compacting/compressing an image

Zero an image from a guest; a strategy for zeroing via (Linux) host is to mount the image.

```sh
# From a Linux guest
#
dd if=/dev/zero of=/var/tmp/zerofile bs=4096k; rm /var/tmp/zerofile

# From a Windows guest (see https://docs.microsoft.com/en-us/sysinternals/downloads/sdelete).
#
# It's possible to squeeze more space by deleting the Windows memory-related files (C:\*.sys),
# but it makes sense only for storing the compressed file, as the files are recreated on the next
# session.
#
sdelete64 -z drive:
```

(If compacting Windows, don't forget to delete the page/hibernate files)

Compact a VDI image (must be pre-zeroed):

```sh
vboxmanage modifymedium --compact $image.vdi
```

Zero/compact/compress an image with dedicated tooling:

```sh
# Zero+compact
#
sudo virt-sparsify --tmp $tmp_path $image.vdi{,.sparse}

# Compress
# Options: [p]rogress, [O]utput format, [c]ompress
#
qemu-img convert -p -c -O qcow2 $source.qcow2 $dest.qcow2

# Zero, compact, and compress!!
# If `--convert` is not specified, the same format as the source is used.
#
sudo virt-sparsify --convert qcow2 --compress $source.raw $dest.qcow2
```

### Working with an image content

#### Filesystem operations

Print partition informations:

```sh
# FS/partition infos
#
# `--all`               : both FS and partitions
# `-a|--add $image`
# `-h|--human-readable`
#
sudo virt-filesystems --long --human-readable --all --add $image

# Partitions space usage
#
sudo virt-df -h -a $image
```

Resize a disk/partition:

```sh
# Watch out! If the underlying FS has no partitions (e.g. Busybox), this won't automatically make the
# space available.
#
qemu-img resize $source.qcow2 +20G

# Resize a disk/partition to an exact destination size.
#
# There is a "resize" action (e.g. add 40G), but it's dumb, as it requires the output disk to be at least
# the required size, which can't be exactly calculated (even for a qcow destination).
#
# - `-v`: verbose
# - `-x`: trace underlying calls, for better error messages
#
# The output is still sparse, even if raw.
#
truncate -s 40G $dest.raw
sudo virt-resize -v -x --expand /dev/sda4 $source $dest.raw
```

#### Mount an image

Easy way, via libguestfs-tools:

```sh
# $block_device is /dev/sdaN; typically a partition, but can be the entire disk (e.g. busybear image).
#
# - `-r|--ro`: readonly
#
sudo guestmount -a $image -m $block_device [-r|--ro] $mountpoint

# WATCH OUT!
#
# The libguestfs stack is functionally poor.
# Unmounting (also via umount), causes an odd `fuse: mountpoint is not empty` error; the guestunmount
# help seems to acknowledge this (ie. retries option), so we don't display errors.
# Additionally, on unmount, the sync is tentative, so need to manually check that the file is closed.
#
if sudo mountpoint -q "$c_local_mount_dir"; then
  sudo guestunmount -q "$c_local_mount_dir"

  # Just ignore the gvfs warning.
  while [[ -n $(sudo lsof -e "$XDG_RUNTIME_DIR/gvfs" "$1" 2> /dev/null) ]]; do
    sleep 0.5
  done
fi
```

It's possible to use qemu-nbd, but the events management (for automation) is insane.

Mount:

```sh
modprobe nbd max_part=8

if [[ $image == *.vhd ]]; then
  format_option=(-f vpc)
fi

qemu-nbd --connect=/dev/nbd0 "${format_option[@]}" "$1"

# Complete clusterduck. Even waiting via:
#
#     while [[ ! -b /dev/nbd0p4 ]]; do sleep 0.1; done
#
# will cause:
#
#     ls -l /dev/nbd0p4
#     ls: cannot access '/dev/nbd0p4': No such file or directory
#
# if there is no sleep.
#
# Therefore, we use `ls` instead.
#
# Fdisk doesn't need to wait (in case one wants to present a list via `fdisk /dev/nbd0 -l`).

partprobe /dev/nbd0

partition_device=/dev/nbd0p$partition_number

while ! ls -l "$partition_device" > /dev/null 2>&1; do
  sleep 0.1
done

mount "$partition_device" /mnt
```

Unmount:

```sh
umount /mnt
qemu-nbd --disconnect /dev/nbd0
rmmod nbd
```

## Software-specific

### VMWare

#### Performance

WATCH OUT!! Set only one virtual CPU when emulating 3d; on v16/17, multiple CPUs caused slowdowns.

#### Compatibility

It seems that VMWare exposes an iGPU. Rarely, games may not be compatible with it; for example, Fallout 3 requires a [patch](https://www.mediafire.com/file/idum0obp1zkfwu9/Win_7_and_8_patch-20371-1-0.iso/file).

(check content: https://docs.vmware.com/en/VMware-Workstation-Pro/17/com.vmware.ws.using.doc/GUID-EA588485-718A-4FD8-81F5-B6E1F04C5788.html)

#### Modules compilation/install

```sh
make clean
make build
make tarballs
sudo mv -f *.tar /usr/lib/vmware/modules/source
# Also restarts the services
sudo vmware-modconfig --console --install-all
```

#### General issues/tweaks

General configuration file is `~/.vmware/preferences`; where general is not specified, it doesn't apply.

- AMD GPU virtualization requires [AMDVLK](https://github.com/GPUOpen-Drivers/AMDVLK/releases) to be installed and configured (set `AMD_VULKAN_ICD=AMDVLK` if RADV drivers are installed)
  - don't use `mks.gl.allowUnsupportedDrivers = "TRUE"`!
- In order to autoattach USB devices, add `usb.autoConnect.device0 = "0xcafe:0xbabe"` to VM cfg
- Symlinks are not followed by default in shared folders; add one `sharedFolder0.followSymlinks = "TRUE"` to the VM cfg for each shared folder
- In order to force a GPU:
  - set `mks.forceDiscreteGPU = "TRUE"`
  - by vendor: set `mks.dx11.driverType = "unknown"` and `mks.dx11.vendorID = 0x10de` in the VM cfg (see https://tinyurl.com/2qmzgr5s)

#### Network fix

The network connection doesn't go up (internally, it actually goes up and down); see https://fluentreports.com/blog/?p=717.

```diff
commit bb84e4f278d24d6ffce5edde1f06dd692f17a16e (HEAD -> workstation-16.2.4)
Author: Saverio Miroddi <saverio.pub2@gmail.com>
Date:   Wed Oct 12 19:29:27 2022 +0200

    Networking fix

diff --git a/vmnet-only/userif.c b/vmnet-only/userif.c
index 2c5a24a..c98178b 100644
--- a/vmnet-only/userif.c
+++ b/vmnet-only/userif.c
@@ -1012,6 +1012,8 @@ VNetUserIfSetUplinkState(VNetPort *port, uint8 linkUp)
    VNet_LinkStateEvent event;
    int retval;
 
+   if (!linkUp) return 0;
+
    userIf = (VNetUserIF *)port->jack.private;
    hubJack = port->jack.peer;
```

#### Commandline Management

```sh
vmrun stop /path/to/vmname.vmx  # poweroff
```

### VirtualBox

BIOS key: EFI=Esc, Legacy=F12 (must tap very quickly)

Rebuild modules:

```sh
/sbin/vboxconfig
```

Change an image UUID:

```sh
vboxmanage internalcommands sethduuid $image.vdi $uuid
```

Clone a disk:

```sh
vboxmanage clonehd $source.vdi $dest.vdi
```

Create VMDK disk pointing to a physical disk!:

```sh
vboxmanage internalcommands createrawvmdk -filename $image.vdi -rawdisk /dev/$device

# Then give the disk access privileges to the current user (!! remove them after usage !!):
usermod -a -G disk $(whoami)
```

Allow the host user accessing an external DVD reader:

```sh
usermod -a -G vboxusers $(whoami)
```

### QEMU

Run a machine in the background, and shut it down:

```sh
QEMU_MONITOR_FILE=$(mktemp)

qemu-system-riscv64 \
   -daemonize -monitor unix:"$QEMU_MONITOR_FILE",server,nowait \
   -display none \
   # ...

echo system_powerdown | socat - UNIX-CONNECT:"$QEMU_MONITOR_FILE"
```

Other options:

- `-nographic`: runs in the foreground, in the terminal (without window)

#### Usermode

```sh
# `/lib` suffix is implied, and must not be specified!
#
qemu-riscv64 -L /usr/riscv64-linux-gnu pigz
qemu-riscv64 -L /path/to/riscv-gnu-toolchain/build/sysroot pigz
```

#### RISC-V

See https://risc-v-getting-started-guide.readthedocs.io/en/latest/linux-qemu.html.

## 3D Acceleration/VFIO

Lots of interesting stuff on the [Arch Wiki](https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF).

IOMMU groups, with reset info:

```sh
for iommu_group in $(ls -dv /sys/kernel/iommu_groups/*/); do
  echo "IOMMU group $(basename "$iommu_group")"
  for device in $(ls -1 "$iommu_group"/devices/); do
    echo -n $'\t'
    [[ -e $iommu_group/devices/$device/reset ]] && echo -n "[RESET] "
    lspci -nns "$device"
  done
done
```

AMD reset problem discussion: https://www.reddit.com/r/VFIO/comments/gq0emj/amd_reset_bug_what_are_we_waiting_for.

With `AutoAddGPU = false`, there is no process running on the card 1 (there is, without it), but still, unbinding hangs.

VirtIO GPU Vulkan Support: https://gist.github.com/peppergrayxyz/fdc9042760273d137dddd3e97034385f

### Cards consumption

Unknown kernel version, from ca. Sep/2022.

- 6600 XT
  - 1'  : 58/59W
  - ~30': 47/48 (!!)
- GT 1030 (active) + 6600 XT (vfio)
  - 1'  : 62/65W
- 6600 XT (active) + GT 1030 (vfio)
  - 1'  : 68/69W
  - 5'  : decreased

See: https://wiki.archlinux.org/title/AMDGPU#Power_profiles.

### nvidia-smi

See [`nv-switch` program](https://github.com/64kramsystem/openscripts/blob/master/nv-switch).
`persistenced` reference: https://docs.nvidia.com/deploy/driver-persistence/index.html#persistence-daemon

```sh
nvidia-smi --help-query-supported-clocks
nvidia-smi --help-query-gpu

nvidia-smi -i 1 --format=csv --query-supported-clocks=mem
nvidia-smi -i 1 --format=csv --query-supported-clocks=gr

nvidia-smi -i 1 --format=csv,noheader -l 1 --query-gpu=pstate,power.draw,clocks.mem,clocks.gr # P8, 13.54 W, 405 MHz, 300 MHz

# Power state can't be set.
# This is the clocks, but it seems it can't be set.
#
nvidia-smi -i 1 -ac 405,300 # mem/gr; not supported!

# Power limits can be set, but the minimum may be too high.
# Setting an invalid
#
nvidia-smi -i 1 --format=csv --query-gpu=power.power.min_limit,power.max_limit
sudo nvidia-smi -i 1 -pl 16.66
```

## Vagrant

Find SSH key:

```sh
vagrant ssh-config $host | awk '/IdentityFile/ {print $NF}'
```

## WINE

If fonts are not rendering correctly, install the core fonts: `winetricks corefonts`.

## DOSBox(-X)

### Key bindings

- `F12 + C`: for the configuration GUI
- `F12 + O`: swap floppy (unclear if there is a GUI option)

### Floppy

WATCH OUT! When installing a program from multiple floppies, try to copy all of their content to a single dir, and install from there; some installers support this.

IMGs can be mounted via `IMGMOUNT` command; multiple images can be mounted (only via command), and swapped via key binding:

```sh
# See bindings for swapping.
#
IMGMOUNT A ms_c-cpp_compiler_19920818/*.IMG
```

### Tools

There is a bundled DPMI host (`CWSDPMI`), but it's not very robust. A better one is [HX](https://github.com/Baron-von-Riedesel/HX); run `C:\HX_PATH\BIN\HDPMI32` (`/r` make it resident).

Under `Z:\TEXTUTILS` there are tools to change the text mode (e.g. to `80x50`), but beware that they may interfere with programs.

### Basic configuration

Full config reference: https://github.com/joncampbell123/dosbox-x/blob/master/dosbox-x.reference.full.conf.

```conf
[sdl]
maximize = true

[dosbox]
fastbioslogo = true
startbanner = false
quit warning = false

[render]
aspect = true

[cpu]
# 486DX at 33MHz (https://dosbox-x.com/wiki/Guide%3ACPU-settings-in-DOSBox%E2%80%90X#_cycles)
# PII/300 = 200000
cycles = 12019
# Enable for fastest CPU speed. WATCH OUT! This is disabled automatically after a keypress; see
# https://dosbox-x.com/wiki/Guide%3AInstalling-Windows-3.x.
turbo = false

# [fdc, primary]
# Enable for fastest floppy speed; it's not clear how faster it is, and if `enabled = true` is necessary.
# instant mode = false

[autoexec]
# By default, the free size is whole disk; it's not clear if GBs of free size may cause programs to
# crash, so better restrict it. Size is in MB.
MOUNT C . -freesize 128
C:
```
