# Linux hardware

- [Linux hardware](#linux-hardware)
  - [Hardware info](#hardware-info)
  - [Storage](#storage)
    - [Securely erase an SSD](#securely-erase-an-ssd)
    - [SMART monitoring (values)](#smart-monitoring-values)
  - [CPU/Threads](#cputhreads)
    - [Disable SMT (hyperthreading)](#disable-smt-hyperthreading)
    - [Topology](#topology)
    - [Isolating processors](#isolating-processors)
    - [Setting the CPU frequency](#setting-the-cpu-frequency)
  - [Removable devices](#removable-devices)
    - [Disable mouse/keyboard](#disable-mousekeyboard)
    - [Keyboard layout](#keyboard-layout)
    - [Disconnect and power off a device](#disconnect-and-power-off-a-device)
    - [Reset a USB device](#reset-a-usb-device)
  - [Power Management](#power-management)
  - [Graphic cards/drivers](#graphic-cardsdrivers)
    - [Vulkan](#vulkan)
  - [Screen](#screen)
    - [HiDPI possible solution (fractional scaling)](#hidpi-possible-solution-fractional-scaling)
    - [HiDPI legacy solution (add new resolution)](#hidpi-legacy-solution-add-new-resolution)
    - [iGPU memory size](#igpu-memory-size)
  - [Audio](#audio)

## Hardware info

Get hardware information:

```sh
# Nice page of info about the hardware
#
lshw -html

# Without options, all the system information is reported.
#
dmidecode -s \
  system-product-name   # (laptop) model
  bios-version
  chassis-serial-number # (dell) service tag
```

Disk infos:

```sh
hdparm -I $device
```

## Storage

### Securely erase an SSD

```sh
# Run both!
#
hdparm --user-master u --security-set-pass $phonypwd $device
hdparm --user-master u --security-erase $phonypwd $device
```

### SMART monitoring (values)

Check:

```sh
for dev in /dev/sd?; do
  echo "# $dev"
  sudo smartctl -a /dev/sda | perl -ne 'print if /^ID#/ .. /^$/' | grep -P '^ *(ID|5|18[12378]|19[5789])\b'
  echo
done
```

Useful values (https://www.backblaze.com/blog/hard-drive-smart-stats):

- `SMART 5`   : Reallocated_Sector_Count
- `SMART 187` : Reported_Uncorrectable_Errors (*important*)
- `SMART 188` : Command_Timeout
- `SMART 197` : Current_Pending_Sector_Count
- `SMART 198` : Offline_Uncorrectable

Other (likely) useful found locally:

- 181: `Program_Fail_Cnt_Total`
- 182: `Erase_Fail_Count_Total`
- 183: `Runtime_Bad_Block`
- 195: `ECC_Error_Rate`
- 199: `CRC_Error_Count`

## CPU/Threads

### Disable SMT (hyperthreading)

```sh
sudo tee /sys/devices/system/cpu/smt/control <<< "off"
```

### Topology

```sh
lstopo --of console --no-io --no-caches # ubuntu
hwloc-ls -v --no-io                     # fedora
```

### Isolating processors

In order to isolate processors from the kernel scheduling, add `isolcpus=isolcpus=1-4,5,666` to the kernel commandline.

Isolated processors can be inspected via `/sys/devices/system/cpu/isolated`.

Then, one can use `taskset -c 1-4,5,666 -p $vcpu_task_pid` . Not all `taskset` support `-c`; example to exclude processor 0: `taskset 0xFFFFFFFE` (bitmask; bits for non-existing processors are ignored).

WATCH OUT! Processes associated via `taskset` do not load balance, therefore, one will, for example, observe that the associated process (tree) runs on a single processor:

```sh
$ ps -aLo comm,psr | grep qemu
qemu-system-x86 1
qemu-system-x86 1
```

For a softer isolation (that is, with the kernel still being scheduled on the desired processors), use `cpuset` (see https://www.codeblueprint.co.uk/2019/10/08/isolcpus-is-deprecated-kinda.html).

### Setting the CPU frequency

EDIT: Read the [kernel guide](https://www.kernel.org/doc/html/v5.8/admin-guide/pm/cpufreq.html) to see if something applies.

It seems that there isn't a stable software way to do that; on a Ryzen 3950x, the strategies:

- setting `userspace` governor
- using `cpufreq-set`
- using `cpupower`
- setting `/sys/**/cpufreq/scaling_max_freq`

didn't have any effect (verified via `grep MHz /proc/cpuinfo`).

## Removable devices

### Disable mouse/keyboard

```sh
xinput --list
# ⎡ Virtual core pointer                      id=2  [master pointer  (3)]
# ⎜ [...]
# ⎣ Virtual core keyboard                     id=3  [master keyboard (2)]
#     ↳ [...]

xinput --list-props 3
# Device 'Virtual core keyboard':
#   Device Enabled (156):  1
#   [...]

# Enable
xinput set-int-prop 3 "Device Enabled" 8 0

# Disable
xinput set-int-prop 3 "Device Enabled" 8 1
```

### Keyboard layout

```sh
# Reconfigure keyboard
#
dpkg-reconfigure console-data

# Set keyboard layout
#
setxkbmap us
```

### Disconnect and power off a device

```sh
udisksctl dump                  # check disks with "\bRemovable: +true"
udisksctl unmount -b /dev/sdb1  # unmount parts from dump which have "MountPoints: +$"
udisksctl power-off -b /dev/sdb # yay!
```

### Reset a USB device

See https://askubuntu.com/q/645.

```sh
# Option 1 (find the device in dmesg)
# Use the parent device (eg. 1-3 instead of 1-3:1.1)
for auth in 0 1; do echo "$auth" | sudo tee /sys/bus/usb/devices/1-3/authorized; done

# Option 2
#
reset_usb.py list | grep -B 1 Sams; ./reset_usb.py path /dev/bus/usb/001/008
```

## Power Management

Disable wakeup events:

```sh
for dev in $(perl -lne 'print /S3\s+\*enabled\s+pci:(\S+)/' /proc/acpi/wakeup); do
  echo disabled | sudo tee /sys/bus/pci/devices/"$dev"/power/wakeup
done
```

Useful references:

- https://wiki.archlinux.org/title/Power_management/Suspend_and_hibernate
- https://community.frame.work/t/waking-immediately-from-suspend-or-hibernate-how-i-fixed-it-for-me/19763/6

## Graphic cards/drivers

### Vulkan

AMD card can use two Vulkan backends: RADV and AMDVLK; RADV is typically better. Can choose them via:

```sh
AMD_VULKAN_ICD=RADV cargo run
```

## Screen

```sh
# For xrandr
#
screen_name=$(xrandr | awk '/ connected/ {print $1}')

# Set resolution
#
xrandr --output $screen_name --mode 1920x1080

# Set scaling factor (=new resolution)
#
xrandr --output $screen_name --scale 0.75x0.75

# Check if the monitor is not sleeping
#
xset -q | grep -q "Monitor is On"

# Disable blanking (may not work anymore)
#
setterm -blank 0 -powersave off -powerdown 0 => xset s off

# Send the screen to sleep. WATCH OUT! Doesn't work; conflicts with the GNOME screensaver.
#
xset dpms force off
```

### HiDPI possible solution (fractional scaling)

See:

- https://wilfredwee.github.io/entry/how-to-xrandr
- https://askubuntu.com/questions/1231818/fractional-scaling-via-xrandr-setting-at-boot-time

### HiDPI legacy solution (add new resolution)

Create resolution mode:

    cvt 1500 1000 60

    # 1504x1000 59.85 Hz (CVT) hsync: 62.12 kHz; pclk: 124.25 MHz
    Modeline "1504x1000_60.00"  124.25  1504 1600 1752 2000  1000 1003 1013 1038 -hsync +vsync

Declare it (using the `Modeline` data):

    xrandr --newmode "1500x1000_60.00" 124.25  1504 1600 1752 2000  1000 1003 1013 1038 -hsync +vsync

Find the screen name:

    xrandr

    eDP1 connected 1504x1000+0+0 (normal left inverted right x axis y axis) 290mm x 190mm
       3000x2000     59.99 +
       2560x1600     59.99
    [...]

Add the resolution to the device:

    sudo xrandr --addmode eDP1 1500x1000_60.00

Now it can be chosen in the `Displays` desktop environment settings.

For setting it at startup, create a shell script $HOME/.xprofile, and assign executable permissions.

### iGPU memory size

To find the amount used:

```sh
lspci -k | grep -EA3 'VGA|3D|Display'                      # find the the PCI bus id
sudo cat /sys/kernel/debug/dri/0000:0f:00.0/amdgpu_vram_mm # check 'usage'
```

On the Nov/2024 system, watching a YT video uses ~500 MB (local = less).

The amount set by the `Auto` setting of the current mobo is static, not dynamically allocated during O/S usage.

## Audio

Pulseaudio commands:

```sh
# See http://askubuntu.com/questions/14077/how-can-i-change-the-default-audio-device-from-command-line
#
pacmd list-sinks                        # list name or index number of possible sinks
pacmd set-default-sink $ink             # set the default output sink
pacmd set-default-source $source        # set the default input
pacmd set-sink-volume $index $volume
pacmd set-source-volume $index $volume  # volume control (0 = Mute, 65536 = 100%)
```

Create remapped mono sink (downmix stereo to mono):

```sh
pacmd list-sinks | grep name:
pacmd load-module module-remap-sink sink_name=mono master=THE_NAME_FROM_THE_PREVIOUS_COMMAND channels=2 channel_map=mono,mono
```
