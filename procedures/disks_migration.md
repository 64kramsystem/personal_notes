# Disks Migration Procedure

## To improve

- remove the Old WinRE partition (update partition indexes etc.)
- must set GUIDs (see section below)
  - ensure to change the disk GUID of the original disk
- update the fstab editing, since the old setup won't use mappers anymore
- modify ISO with newest kernel before creating the live key

### GUIDs

```sh
$ sudo sgdisk -p /dev/sda | grep GUID
# Disk identifier (GUID): AAAAAAAA-BBBB-CCCC-DDDD-EEEEEEEEEEE1

$ for i in {1..5}; do sudo sgdisk -i $i /dev/sda | grep 'Partition unique GUID'; done
# Partition unique GUID: AAAAAAAA-BBBB-CCCC-DDDD-EEEEEEEEEEE2
# Partition unique GUID: AAAAAAAA-BBBB-CCCC-DDDD-EEEEEEEEEEE3
# Partition unique GUID: AAAAAAAA-BBBB-CCCC-DDDD-EEEEEEEEEEE4
# Partition unique GUID: AAAAAAAA-BBBB-CCCC-DDDD-EEEEEEEEEEE5
# Partition unique GUID: AAAAAAAA-BBBB-CCCC-DDDD-EEEEEEEEEEE6

sudo sgdisk -U=AAAAAAAA-BBBB-CCCC-DDDD-EEEEEEEEEEE1 /dev/nvme0n1

sudo sgdisk -u 1:AAAAAAAA-BBBB-CCCC-DDDD-EEEEEEEEEEE2 /dev/nvme0n1
sudo sgdisk -u 2:AAAAAAAA-BBBB-CCCC-DDDD-EEEEEEEEEEE3 /dev/nvme0n1
sudo sgdisk -u 3:AAAAAAAA-BBBB-CCCC-DDDD-EEEEEEEEEEE4 /dev/nvme0n1
sudo sgdisk -u 4:AAAAAAAA-BBBB-CCCC-DDDD-EEEEEEEEEEE5 /dev/nvme0n1
sudo sgdisk -u 5:AAAAAAAA-BBBB-CCCC-DDDD-EEEEEEEEEEE6 /dev/nvme0n1
```

## Notes

- When changing disk sizes, use GParted to create partitions and compute start/ends
  - it supports adding partitions to the right, which simplifies the computations
  - it aligns partitions to 1 MiB by default, so one doesn't need to care about the alignment (for typical cases)
  - **do not allow LLMs to do compute size changes, as they will mess up**
- Disk sizes:
  - WD_BLACK SN8100 4 TB size: 4000787030016 bytes
  - WD_BLACK SN8100 2 TB size: 2000398934016 bytes
- This script assumes that the `nvme0` device is the 4 TB disk; in case the hardware detects the reverse, just perform a textual search - the end setup uses UUIDs, anyway

## Prerequisites

### Stage Linux boot partitions to usbdata

From old machine, copy the Linux boot partitions to the USB data partition:

```bash
# WATCH OUT! The order of the devices doesn’t necessarily match that of the new machine - check and update the command.

sudo dd if=/dev/nvmeXn1p1 of=/media/$(whoami)/usbdata/efi.img bs=1M status=progress
sudo dd if=/dev/nvmeXn1p2 of=/media/$(whoami)/usbdata/boot.img bs=1M status=progress
sync
```

After staging, power off old machine and put the old large disk in USB enclosure.

---

## Part 1: Setup and Partition Both New Disks

### 1.1 Initial Setup

Boot new machine from live USB with:

- New nvme0n1 (Windows) and nvme1n1 (Linux) installed internally
- Old large disk in USB adapter (appears as `/dev/sda`)
- USB data partition mounted at `/media/ubuntu-mate/usbdata`

Verify devices:

```bash
lsblk -f
# Old large disk: /dev/sda (via USB)
# USB data partition: /media/ubuntu-mate/usbdata
# New small disk: /dev/nvme1n1 (internal)
# New large disk: /dev/nvme0n1 (internal)
```

Install tools:

```bash
apt update
apt install -y partclone
```

### 1.2 Partition new small disk

```bash
parted --script /dev/nvme1n1 mklabel gpt

# p1: EFI (512M)
parted /dev/nvme1n1 mkpart primary fat32 1MiB 513MiB
parted /dev/nvme1n1 set 1 esp on

# p2: /boot (1709M)
parted /dev/nvme1n1 mkpart primary 513MiB 2222MiB

# p3: LUKS (1905505MiB = rest of the disk)
parted /dev/nvme1n1 mkpart primary 2222MiB 1907727MiB

# Check alignment
for n in {1..3}; do
  parted /dev/nvme1n1 align-check optimal "$n" # "$n aligned" = success
done
```

### 1.3 Partition new large disk

```bash
parted --script /dev/nvme0n1 mklabel gpt

# p1: Old WinRE (529M)
parted /dev/nvme0n1 mkpart primary ntfs 1MiB 530MiB
parted /dev/nvme0n1 set 1 hidden on
parted /dev/nvme0n1 set 1 diag on

# p2: Windows EFI (100M)
parted /dev/nvme0n1 mkpart primary fat32 530MiB 630MiB
parted /dev/nvme0n1 set 2 esp on

# p3: MSR (16M)
parted /dev/nvme0n1 mkpart primary 630MiB 646MiB
parted /dev/nvme0n1 set 3 msftres on

# p4: Windows System (957111MiB)
parted /dev/nvme0n1 mkpart primary ntfs 646MiB 957757MiB
parted /dev/nvme0n1 set 4 msftdata on

# (Free space here)

# p5: WinRE (710M)
parted /dev/nvme0n1 mkpart primary ntfs 1909232MiB 1909942MiB
parted /dev/nvme0n1 set 5 hidden on
parted /dev/nvme0n1 set 5 diag on

# p6: LUKS (1905505MiB = same as mirror)
parted /dev/nvme0n1 mkpart primary 1909942MiB 3815447MiB

# Check alignment
for n in {1..6}; do
  parted /dev/nvme0n1 align-check optimal "$n" # "$n aligned" = success
done
```

---

## Part 2: Clone Boot Partitions

The filesystem UUIDs are all preserved.

### 2.1 Restore Linux boot partitions from usbdata

```bash
# Restore EFI, /boot
dd if=/media/ubuntu-mate/usbdata/efi.img of=/dev/nvme1n1p1 bs=1M status=progress
dd if=/media/ubuntu-mate/usbdata/boot.img of=/dev/nvme1n1p2 bs=1M status=progress
```

### 2.2 Clone Windows partitions from USB

```bash
# Clone partitions
dd if=/dev/sda1 of=/dev/nvme0n1p1 bs=1M status=progress  # Old WinRE
dd if=/dev/sda2 of=/dev/nvme0n1p2 bs=1M status=progress  # Windows EFI
# Skip p3 (MSR)
partclone.ntfs -b -s /dev/sda4 -O /dev/nvme0n1p4          # Windows System
partclone.ntfs -b -s /dev/sda5 -O /dev/nvme0n1p5          # WinRE
```

---

## Part 3: Setup LUKS and LVM

```sh
# Open old large disk LUKS (contains Btrfs mirror)
cryptsetup open /dev/sda6 old_mirror

# Ensure that there is only one LV
if [[ $(lvs --noheadings | wc -l) -ne 1 ]]; then >&2 echo 'Unexpected number of LVs found!'; fi
```

For simplicity, we use the same UUID for LUKS and VG.

```bash
UUID_NEW_PRIMARY=$(uuidgen)
UUID_NEW_SECONDARY=$(uuidgen)

LUKS_PRIMARY=luks-"$UUID_NEW_PRIMARY"
LUKS_SECONDARY=luks-"$UUID_NEW_SECONDARY"

VG_PRIMARY=vg-$UUID_NEW_PRIMARY
VG_SECONDARY=vg-$UUID_NEW_SECONDARY

LV_PRIMARY=${VG_PRIMARY//-/--}-root
LV_SECONDARY=${VG_SECONDARY//-/--}-root
LV_SWAP=${VG_PRIMARY//-/--}-swap

OLD_MIRROR_LV=$(lvs --noheadings | awk '{print $2}' | sed 's/-/--/g')-$(lvs --noheadings | awk '{print $1}')
```

### 3.1 Create LUKS on both new disks

```bash
# Create LUKS on new small disk
cryptsetup luksFormat --batch-mode --type luks2 \
  --uuid="$UUID_NEW_PRIMARY" \
  /dev/nvme1n1p3

# Create LUKS on new large disk (mirror)
cryptsetup luksFormat --batch-mode --type luks2 \
  --uuid="$UUID_NEW_SECONDARY" \
  /dev/nvme0n1p6

# Open new LUKS devices
cryptsetup open /dev/nvme1n1p3 "$LUKS_PRIMARY"
cryptsetup open /dev/nvme0n1p6 "$LUKS_SECONDARY"
```

### 3.2 Create LVM on new disks

```bash
# Create PV and VG on new small disk
pvcreate /dev/mapper/"$LUKS_PRIMARY"
vgcreate "$VG_PRIMARY" /dev/mapper/"$LUKS_PRIMARY"

# Create LVs on new small disk
lvcreate -L 2G -n swap "$VG_PRIMARY"
lvcreate -l 100%FREE -n root "$VG_PRIMARY"

# Create PV and VG on new large disk (mirror)
pvcreate /dev/mapper/"$LUKS_SECONDARY"
vgcreate "$VG_SECONDARY" /dev/mapper/$LUKS_SECONDARY

# Create LV on new large disk (mirror)
lvcreate -l 100%FREE -n root "$VG_SECONDARY"
```

---

## Part 4: Create RAID1 and Copy Btrfs Data

### 4.1 Activate old VG and mount old mirror

```bash
# Verify all VG/LV names
lvs
vgs
```

### 4.2 Create Btrfs RAID1 on both new disks

```bash
# Create Btrfs as RAID1 from the start (both devices)
mkfs.btrfs -d raid1 -m raid1 /dev/"$VG_PRIMARY"/root /dev/"$VG_SECONDARY"/root

# Mount and create subvolumes
mkdir /mnt/new
mount /dev/"$VG_PRIMARY"/root /mnt/new
btrfs subvolume create /mnt/new/@
btrfs subvolume create /mnt/new/@home
umount /mnt/new
```

### 4.3 Copy data from old large disk mirror to new RAID1

```bash
# Mount old subvolumes - read-only and degraded, since there's only one device of the mirror
# Mount the LV (not the LUKS device), since structure is LUKS -> LVM -> Btrfs
mkdir -p /mnt/old_mirror /mnt/old_mirror_home
mount -o subvol=@,ro,degraded /dev/mapper/"$OLD_MIRROR_LV" /mnt/old_mirror
mount -o subvol=@home,ro /dev/mapper/"$OLD_MIRROR_LV" /mnt/old_mirror_home

# Mount new subvolumes (data automatically written as RAID1)
mount -o subvol=@ /dev/"$VG_PRIMARY"/root /mnt/new
mkdir /mnt/new/home
mount -o subvol=@home /dev/"$VG_PRIMARY"/root /mnt/new/home

# Run this in a separate terminal, in order to verify read corruptions
sudo dmesg -w | egrep -i 'btrfs|csum|checksum'

# Copy data (this may take a long time)
rsync -aHAXv --numeric-ids --info=progress2 --no-inc-recursive /mnt/old_mirror/ /mnt/new/
rsync -aHAXv --numeric-ids --info=progress2 --no-inc-recursive /mnt/old_mirror_home/ /mnt/new/home/
```

---

## Part 5: Verify RAID1 and Create Swap

### 5.1 Verify RAID1 status

```bash
# Verify RAID1 is active (should show both devices)
btrfs filesystem show /mnt/new
btrfs filesystem df /mnt/new
# Should show: Data, RAID1; Metadata, RAID1
```

### 5.2 Create swap

```bash
# Format swap
mkswap /dev/"$VG_PRIMARY"/swap
```

---

### 5.3 Sync

```sh
sync
```

## Part 6: Chroot and Configure System

### 6.1 Mount full system

```bash
# Mount boot partitions
mount /dev/nvme1n1p2 /mnt/new/boot
mount /dev/nvme1n1p1 /mnt/new/boot/efi

# Bind mounts for chroot
for sysdir in dev proc sys run; do
  mount --rbind /$sysdir /mnt/new/$sysdir
  mount --make-rslave /mnt/new/$sysdir
done
```

### 6.2 Chroot and regenerate boot files

```bash
# Chroot
env \
  UUID_NEW_PRIMARY=$UUID_NEW_PRIMARY \
  UUID_NEW_SECONDARY=$UUID_NEW_SECONDARY \
  LV_PRIMARY=$LV_PRIMARY \
  LV_SWAP=$LV_SWAP \
  LUKS_PRIMARY=$LUKS_PRIMARY \
  LUKS_SECONDARY=$LUKS_SECONDARY \
chroot /mnt/new

# Edit fstab
cp /etc/fstab{,.bak}
sed -i -e '/\/dev\/mapper/ d' /etc/fstab
cat >> /etc/fstab << CONF
UUID=$(blkid -s UUID -o value /dev/mapper/"$LV_PRIMARY") /     btrfs subvol=@,compress=zstd,nodiscard     0 1
UUID=$(blkid -s UUID -o value /dev/mapper/"$LV_PRIMARY") /home btrfs subvol=@home,compress=zstd,nodiscard 0 2
UUID=$(blkid -s UUID -o value /dev/mapper/"$LV_SWAP")    none  swap  sw                                   0 0
CONF

# Verify fstab
cat /etc/fstab

# Rewrite crypttab
cat > /etc/crypttab << CONF
$LUKS_PRIMARY UUID=$UUID_NEW_PRIMARY none luks,discard,keyscript=decrypt_keyctl
$LUKS_SECONDARY UUID=$UUID_NEW_SECONDARY none luks,discard,keyscript=decrypt_keyctl
CONF

# Regenerate initramfs and grub
update-initramfs -u -k all
update-grub
grub-install /dev/nvme1n1

exit
```

---

## Part 7: Cleanup and Reboot

### 7.1 Unmount and close

```bash
# Unmount everything from chroot
umount --recursive /mnt/new

# Unmount old mirror mounts
umount /mnt/old_mirror_home /mnt/old_mirror

# Deactivate VGs
vgchange -an

# Close LUKS
cryptsetup close old_mirror
cryptsetup close $LUKS_PRIMARY
cryptsetup close $LUKS_SECONDARY
```

### 7.2 Reboot

```bash
reboot
```

---

## Post-Migration Verification

After booting into the new system:

```bash
# Check RAID1 status
btrfs filesystem show /
btrfs filesystem df /

# Check disk usage (should be much larger now)
df -h

# Run scrub to verify data integrity
btrfs scrub start -Bd /
btrfs device stats /

# Check WINDOG partition
ls -lh /media/saverio/WINDOG

# Verify both LUKS devices unlock and LVM layout
lsblk -f
lvs
```
