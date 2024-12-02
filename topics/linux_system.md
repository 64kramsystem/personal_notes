# Linux system

- [Linux system](#linux-system)
  - [Processes](#processes)
    - [Useful operations](#useful-operations)
    - [Signals/exit codes/suspension](#signalsexit-codessuspension)
    - [Process tree display](#process-tree-display)
  - [Memory (measurement)](#memory-measurement)
  - [Security (Permissions)](#security-permissions)
    - [Sudo](#sudo)
  - [Filesystems/partitions/mounts](#filesystemspartitionsmounts)
    - [Partitions](#partitions)
    - [In-memory filesystems](#in-memory-filesystems)
    - [Sparse files](#sparse-files)
    - [ulimit](#ulimit)
      - [Solve "Too many open files"](#solve-too-many-open-files)
  - [Udev](#udev)
  - [Add swap file](#add-swap-file)
  - [Packages](#packages)
    - [Apt/dpkg installation hooks (`etc/apt/apt.conf.d`)](#aptdpkg-installation-hooks-etcaptaptconfd)
  - [Repositories](#repositories)
    - [Keys handling](#keys-handling)
  - [Debconf](#debconf)
  - [Debian alternatives](#debian-alternatives)
  - [Environment](#environment)
    - [Available variables](#available-variables)
    - [Shell (initscripts: bashrc, ...)](#shell-initscripts-bashrc-)
      - [Example cases](#example-cases)
      - [sudo -i, login shell test, and bash](#sudo--i-login-shell-test-and-bash)
  - [Handle system date/time](#handle-system-datetime)
  - [Job scheduling](#job-scheduling)
    - [Cron](#cron)
      - [Schedule format](#schedule-format)
      - [Subminute granularity](#subminute-granularity)
    - [At](#at)
  - [Apparmor](#apparmor)
  - [fstab](#fstab)
  - [sysctl](#sysctl)
  - [Modules](#modules)
  - [Grub](#grub)
    - [Install from live cd (or perform chrooted operations)](#install-from-live-cd-or-perform-chrooted-operations)
    - [Add a Windows entry](#add-a-windows-entry)
  - [Logging/syslog/tools](#loggingsyslogtools)
    - [Logrotate](#logrotate)
  - [Terminal emulator](#terminal-emulator)
  - [Desktop Environment: windows](#desktop-environment-windows)
  - [Gnome stuff](#gnome-stuff)
    - [Distro info](#distro-info)
    - [Dconf/Gsettings](#dconfgsettings)
  - [MIME (extensions) (file associations) handling](#mime-extensions-file-associations-handling)
    - [Adding and associating a new application](#adding-and-associating-a-new-application)
    - [Split MAFF association](#split-maff-association)
  - [Encryption](#encryption)
    - [Mount encrypted (.ecryptfs) home](#mount-encrypted-ecryptfs-home)
    - [Setup hibernation on Ubuntu-setup LUKS volume](#setup-hibernation-on-ubuntu-setup-luks-volume)

## Processes

### Useful operations

```sh
# check if pid exists
ps -p $pid > dev/null

# x        : list all processes owned by the user
# -o       : format specifiers; `ni`: niceness, `args`: command with args
# --forest : tree format
#
ps x -o pid,ppid,ni,args --forest

# Pgrep/pkill use regexes.
# Character classes are a trick not to match its ancestor on sudo; this functionality will be supported
# via `-A` on procps 4.0.1.
#
sudo pkill -f '[p]rocessname'

# Kill the children.
# Be *very* careful when choosing which pid to kill. If the process has been invoked via sudo, the child
# process will need to be killed.
#
pkill --parent --pidfile $(< parent_pidfile)

# Set a process priority (two ways); 19 is the lowest.
# Without sudo permissions, the priority can only be lowered.
#
nice -n 19 $command
renice -n 19 $pid
```

### Signals/exit codes/suspension

Suspension:

```sh
kill -STOP $pid  # suspend
ps               # STAT=`T`: suspended
kill $pid        # when running on a suspended process, it won't kill it, but ->
kill -CONT $pid  # resume (in background) -> if previously sent TERM, it will now have effect
```

Exit codes: when a signal is sent, the exit code is (128 + signal value), e.g INT = (128 + 2 = 130).

### Process tree display

```sh
# pstree's modes are confusing; some imply each other, but not entirely.
# the below is the minimum to get a pretty view.
#
# a: disable processes compaction (and show cmdline args)
# u: uid transitions
# t: display full thread names
# p: show pids (disable compaction, but not entirely)
#
pstree -autp $(pgrep -f qemu-sys)

# H: Threads, b: batch, n1: iterations, -p $pid: one process only
#
top -Hb -n1 -p $(pgrep -f qemu-sys)

# Thread forest for a given process (HTML) - HTML output (required `aha`)
#
echo q | htop -p $(pgrep -f qemu-sys) | aha --black --line-fix > /tmp/htop.html
```

## Memory (measurement)

Measuring memory is a complex topic. A simple and accurate enough tool that can be used is [`ps_mem`](https://github.com/pixelb/ps_mem).

## Security (Permissions)

```sh
chown $name $file...                        # change owner
chgrp $name $file...                        # change group
chmod [ugoa][-+][rwx] $file...              # change permissions; [u]ser [g]group [o]thers [a]ll

chage [-m $min] [-M $max] [-W $warn] [-I $inactive_days] [-E $expire_date] $login # change password properties
chage -l $login                             # show info

passwd -d                                   # set a blank password
```

Rename user/group/home:

```sh
usermod -l $new_user $old_user
groupmod -n $new_user $old_user
usermod -d $new_home -m $user
```

Snippets:

```sh
usermod -a -G $group $user              # add user to group

chmod -R go-rx .ssh                     # setup SSH permissions
chage -d $(date +%Y-%m-%d) $user        # set the last changed date to today, and the mandatory change date to (today + max days)
chage -I 0 $user                        # expire password
chage -E -1 $user                       # disable account expiry
```

User name/home/sudo:

```sh
# Find current user (if sudo, find the logged on user).
# No method is foolproof; however, logname also works on an SSH sessions without terminal attached.
#
logname
who am i | awk '{print $1}'

# Find the user home.
#
getent passwd $username | cut -f6 -d:

Passwordless ssh:

```sh
perl -i -pe 's/^#?(PasswordAuthentication) \w+/$1 yes/' /etc/ssh/sshd_config
perl -i -pe 's/^#?(PermitEmptyPasswords) \w+/$1 yes/'   /etc/ssh/sshd_config
systemctl restart sshd
passwd -d $user

# On some system, like Debian, also run the following,
# and ensure that ChallengeResponseAuthentication is set to `no`.
perl -i -pe 's/^UsePAM \K\w+/no/'                       /etc/ssh/sshd_config
```

On Ubuntu, root can't modify `/tmp` files belonging to another user; fix using (see https://askubuntu.com/a/1340909/46091):

```sh
sysctl fs.protected_regular=0
```

Limit number of concurrent user logins (but not sudo); useful in rare circumstances, but typically discouraged:

```sh
echo "@user            -       maxlogins       1" >> /etc/security/limits.conf
```

### Sudo

```sh
$SUDO_USER                     # User who invoked sudo
[[ $EUID -eq 0 ]]              # Shell (all) check for user being root
sudo -n true && echo 'no pwd!' # Check if password is required to sudo (WATCH OUT SEMANTICS!)

# Passwordless sudo (run as regular user)
#
echo "$(whoami) ALL=(ALL:ALL) NOPASSWD: ALL" | sudo tee "/etc/sudoers.d/$(whoami)_no_sudo_pwd"
sudo chmod 440 !$


# Run a specific command (also scripts) without asking sudo password.
# WATCH OUT! The command must be invoked *exactly* as "sudo <full_command>", e.g. in this case
# `sudo /path/to/command param1 paramN`.
#
# The typical alternative to `root` is `ALL`, which allow running as any target user.
#
echo "$(whoami) ALL=(root) NOPASSWD: /path/to/command param1 paramN" | sudo tee "/etc/sudoers.d/mysetting"

# Pass the env vars to the root env.
#
sudo -E $command

# WATCH OUT! $PATH is special, and it's not passed if `secure_path` is set in `/etc/sudoers`. The
# below is a solution; if it's not usable, the `sudoers` file can be hacked, but in complex scripts,
# it's not reliable. An alternative solution is to add temporary symlinks to /usr/bin, or similar.
#
sudo -E env "PATH=$PATH" $command`
```

## Filesystems/partitions/mounts

Useful operations:

```sh
# Get the device to whom a partition belongs (parent)
#
# WATCH OUT!! Exit with error if nothing is found!!
#
lsblk -n -o PKNAME $partition

# Get the device(s) tree, and mountpoints (including unmounted partitions)
#
lsblk [/dev[partition]]

# Find which filesystem (/device) a path is stored on (also symlinks)
df $file

# Find if a path is a mountpoint (if it's mounted)
if mountpoint -q $path; then echo is mountpoint; fi
```

### Partitions

```sh
fdisk -l                        # list the partitions

cfdisk -z $disk_device          # GUI: create an empty partition

mkfs.ext4 $part_device          # create the FS

# Via sgdisk (more modern approach).
#
# - `-t<partnum>:type` is the partition type (see https://askubuntu.com/q/703443 and https://en.wikipedia.org/wiki/Partition_type).
# - `n<partnum>:start_pos:end_pos`
#   - `partnum`: if 0, the first available partition is chosen
#   - `+start_pos` is relative to the previous partition
#   - `+end_pos` is relative to `start_pos`
#
sgdisk -n1:1M:+512M -t1:EF00 "$selected_disk" # start at absolute 1M, ends at absolute (1+512)M
sgdisk -n2:0:-2G    -t2:BF01 "$selected_disk" # start after last partion, end 2G before end of available space
sgdisk -n3:0:0      -t3:BF01 "$selected_disk" # start after last partion, ends at end of available space

# Resize to given size/expand to fill empty space.
# Watch out the `--` required when passing a negative reference (e.g. `-18G`).
#
parted -s $disk_device unit s resizepart $part_number_1_based -- -18G
parted -s $disk_device resizepart $part_number_1_based 100%

# Remove a partition

parted -s $disk_device rm $part_number_1_based
```

In order to unmount encrypted partitions, see [ejectdisk](https://github.com/64kramsystem/openscripts/blob/416f4a456412210df86a574a4d19a649481133b2/ejectdisk#L76); this involves:

```sh
lsblk -n -o NAME,TYPE,MOUNTPOINT,UUID $device  # find the given device tree properties
udisksctl unmount -b /dev/mapper/$luks_device  # unmount the encrypted partition(s)
udisksctl lock -b /dev/$luks_device            # lock the LUKS device
```

To check the lock status of a LUKS device, see https://unix.stackexchange.com/a/695898/12814.

### In-memory filesystems

```sh
# Both tmpfs/ramfs use only the memory needed
# /dev/shm can be used as well; it's created by default (it's tmpfs), but if it's filled, other apps using it may crash.
#
mount -t tmpfs -o size=256m tmpfs /mnt/myfs  # tmpfs: limited size; can use swap
mount -t ramfs -o size=256m ramfs /mnt/myfs  # ramfs: unlimited size; all in RAM; it's an older FS - using tmpfs is encouraged
mount -o remount,size=8g /mnt/myfs           # !!! resize !!!
```

### Sparse files

```sh
# Create a sparse file (different ways)
#
truncate -s $size $file
dd of=$file bs=$size seek=1 count=0
dd if=/dev/zero of=$file conv=sparse bs=$block_size count=$count

# Copy a file in sparse mode; `bs=512` is crucial, because a block needs to be entirely zero in order
# to be discarded. Two ways; first is more convenient.
# See https://en.wikipedia.org/wiki/Sparse_file.
#
dd if=$source of=$dest conv=sparse bs=512

# Convert a file to sparse
#
fallocate -d $file

# Shows the sparse size (first column)
#
ls -ls

# COW!! Supported only on XFS/BTRFS, not (currently) ZFS (https://git.io/JqWqU)
#
cp --reflink=always $source $dest
```

### ulimit

```sh
# the command is a shell built-in; the first won't run, the second
# will:
#
sudo -u elasticsearch ulimit -l
sudo -u elasticsearch bash -c 'ulimit -l'
#
# there is generally a misunderstanding about the limits; setting
# /etc/security/limits.conf alone won't work.
#
# The general instructions are:
#
# - interactive login:
#   - set /etc/pam.d/common-session
#   - only the logged user's limit is applied
# - non-interactive login execution (sudo -u...)
#   - the parent user's limit is applied (not the child), so it needs
#     to be set
#   - some parameters may be enforced at the system level (eg. open
#     files, in sysctl)
#
# Examples (reboot required after settings):
#
# - interactive, memlock:
#   - set limits.conf for root: 'root - memlock unlimited'
#   - set common-session: 'session required pam_limits.so'
#   - sudo su
#   - sudo -u elasticsearch bash -c 'ulimit -l'
# - non-interactive/upstart, memlock:
#   - simply set 'limit memlock unlimited unlimited' in the config file (!!)
# - both interactive and non/init, open files:
#   - set sysctl.conf: 'fs.file-max = 65536'
#   - set limits.conf for mysql: 'mysql soft nofile 24000' 'mysql hard nofile 32000'
#   - set common-session: 'session required pam_limits.so'

# ulimits for a user (even if without shell login)
#
cat /proc/<uid>/limits
```

#### Solve "Too many open files"

```sh
# when mysql reports too many open files, look for open files limit, and extend it (as root):

# check the limit for a user
sudo -u mysql bash -c "ulimit -n"

# extend the limit at global level; may still be overridden by /etc/sysctl.conf
echo fs.file-max = 65536 >> /etc/sysctl.d/99-file-max.conf

# check the result
sysctl -p

# check the limits at user level; add to /etc/security/limits.conf
mysql             soft    nofile           24000
mysql             hard    nofile           32000

# needed; add to /etc/pam.d/common-session
session required pam_limits.so

# now check again
sudo -u mysql bash -c "ulimit -n"
```

## Udev

Write a rule that changes a device permissions:

```sh
# Matches /dev/sda1. No need for executable permissions on the rules file.
#
cat > /etc/udev/rules.d/20-myrule.rules <<UDEV
KERNEL=="sda1", MODE="0600", GROUP="mygroup"
UDEV
```

## Add swap file

WATCH OUT! Doesn't work on ZFS (see below)

```sh
dd if=/dev/zero of=/swapfile bs=1M count=512 status=progress # don't use fallocate, as it may fail
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile swap swap defaults 0 0' >> /etc/fstab # activate on restart
```

ZFS:

```sh
zfs create -V 8G -b $(getconf PAGESIZE) -o logbias=throughput -o sync=always -o primarycache=metadata -o com.sun:auto-snapshot=false mypool/myswap
mkswap -f /dev/zvol/mypool/myswap
swapon /dev/zvol/mypool/myswap

# after usage:

swapoff /dev/zvol/mypool/myswap
zfs destroy mypool/myswap
```

## Packages

Informations:

```sh
# Show package info; for this operation, `apt` is stable [enough].
#
apt show $package
dpkg --list $package

# Check if a package is installed, in the most standard way possible.
# States found: `install`, `hold`, `deinstall`; the last seems only cfg installed.
#
if dpkg --get-selections trash-cli | grep -qP '\s+(install|hold)$'; then echo installed; fi

# Advanced filtering with aptitude.
#
# In order to filter out i386 packages, use the suffix `~rnative`.
# The option `-F %p` prints only the package name.
#
# Aptitude uses regexes by default, but it doesn't support all the patterns; tested metachars:
#
# - ^,$ : supported
# - \b  : supported
# - ?   : supported, but not for groups
# - |   : supported
#
# Prefixes:
#
# - ~n : package name
# - ~r : arch
#
# Search terms reference: https://www.debian.org/doc/manuals/aptitude/ch02s04s05.en.html
#
aptitude search -F %p '~nxserver-xorg-video-nvidia-[[:digit:]]+$~ramd64'

# Better use this than `apt search`; the latter is undocumented and has an atrocious output.
#
apt-cache search '^linux-image-unsigned-5.4.[[:digit:]-]+-generic'

# Show dependencies of a package
#
aptitude why -v $package
dpkg -I $package.deb

# Display the repositories with a given package
#
apt-cache policy $package

# Find the repository of a package
#
apt-cache showpkg $package

# (Extra tool) Check dependencies of installed package.
#
apt-rdepends $package
```

Package/file operations:

```sh
# Downgrade to previous version of a package
#
apt-get install $package=$version

# Mark a package as manually installed (apt[itude]) - prevents automatic deletion.
#
aptitude install "${package}&m"
apt-mark manual linux-firmware amd64-microcode iucode-tool

# Install/remove package ignoring dependencies, via dpkg
#
dpkg (-i|-r) --ignore-depends=libdbd-mysql-perl libmysqlclient18

# Download the packages without installing (dependencies are not downloaded).
#
aptitude download ($packages)

# Download source code for package
#
apt-get source $package

# List the contents of a deb file
#
dpkg-deb -c $package.deb

# Unpack deb file.
#
dpkg-deb --fsys-tarfile $package.deb | tar xv # extract the data file; can pass /dev/stdin as input
ar -vx $package.deb                           # this extract the deb files (debian-binary, control, data)

# Find which package a file in the filesystem belongs to
#
dpkg -S $file

# Find which packages include a file with the given name (no need for path). This is not a preinstalled tool.
# Require its case to be refreshed `apt-file cache`.
#
apt-file search $filename
```

Useful generic snippets:

```sh
# Remove linux kernels other than the current one
# Note that if there is a version newer than the current (uname -r), it will be deleted.
#
dpkg -l 'linux-*' | sed '/^ii/!d;/'"$(uname -r | sed "s/\(.*\)-\([^0-9]\+\)/\1/")"'/d;s/^[^ ]* [^ ]* \([^ ]*\).*/\1/;/[0-9]/!d' | xargs sudo apt-get -y purge

# Purge packages matching a pattern, only if they're installed.
# apt can't search for a pattern and an installation status in one pass
#
aptitude purge '~i ruby1.9'
apt list --installed | perl -ne 'print "$1\n" if /^(<pattern>)\//' | xargs apt purge

# Find packages installed via PPA
#
apt-cache policy $(dpkg --get-selections | grep -v deinstall$ | awk '{ print $1 }') | perl -e '@a = <>; $a=join("", @a); $a =~ s/\n(\S)/\n\n$1/g;  @packages = split("\n\n", $a); foreach $p (@packages) {print "$1: $2\n" if $p =~ /^(.*?):.*?500 http:\/\/ppa\.launchpad\.net\/(.*?)\s/s}'

# Remove PPA versions of packages: use `ppa-purge` utility.
# Comments out the PPA file content after purging.
#
ppa-purge ppa:oibaf/graphics-drivers
```

### Apt/dpkg installation hooks (`etc/apt/apt.conf.d`)

Add the following to `/etc/apt/apt.conf.d/99-post-upgrade`:

- `Dpkg::Post-Invoke {"command";};`: Runs for every dpkg execution (package essentially) invoked by apt
- `APT::Update::Post-Invoke-Success {"command";};`: Run after successful apt update (**update only**)
- `APT::Update::Post-Invoke {"command";};`: Run after apt update (independently of exit status)

See [apt repository](https://salsa.debian.org/apt-team/apt.git) (search `RunScripts`).

Comments can be added with `//` and `/* */`.

## Repositories

```sh
# Add a repository; exit=0 if the repository is already enabled.
#
add-apt-repository universe
```

### Keys handling

List keys:

```sh
apt-key list

# /etc/apt/trusted.gpg
# --------------------
# pub   rsa4096 2014-11-17 [SC]
#       7053 72CB 1DF3 976C 44B7  B8A6 BBE4 75EB A386 83E4
# uid           [ unknown] Scout Packages (archive.scoutapp.com) <support@scoutapp.com>
# uid           [ unknown] Scout (scoutapp.com) <support@scoutapp.com>
# sub   dsa2048 2014-11-17 [S]
# sub   rsa4096 2014-11-17 [E]
#
# pub   rsa4096 2019-07-05 [SC]
#       394F 883E 0C43 5694 50FD  FB92 A9AF 1C7C 2ED6 5CC0
# uid           [ unknown] Fullstaq Ruby <info@fullstaq.com>
# sub   rsa4096 2019-07-05 [E]
```

Remove key:

```sh
# WATCH OUT: Returns OK!
#
apt-key del NOTPRESENT

# Deletes the Fullstaq Ruby key; use the last 8 digits of the displayed key.
#
apt-key del 2ED65CC0
```

## Debconf

```sh
debconf-show packagename                      # show properties of an installed package
debconf-get-selections | grep packagename/    # more detailed, but requires the `debconf-utils` package
```

Example of seeding:

```sh
echo "zfs-dkms zfs-dkms/note-incompatible-licenses note true" | debconf-set-selections
```

## Debian alternatives

Display the current info (for machine parsing).

```sh
# Sample:
#
#   Name: ruby
#   Link: /usr/bin/ruby
#   Status: auto
#   Best: /usr/bin/ruby2.6
#   Value: /usr/bin/ruby2.6
#
#   Alternative: /usr/bin/ruby2.5
#   Priority: 9999
#
#   Alternative: /usr/bin/ruby2.6
#   Priority: 9999
#
update-alternatives --query $program
```

Install an alternative:

```sh
# - format: <file path> <alternative name> <alternative path> <priority>
# - 9999 is high priority, and will (likely) set the alternative as default.
# - If the specified alternative exists, it will be updated.
# - "Slaves" are alternatives that depend on the "master" (e.g. irb -> ruby)
#
update-alternatives --install /etc/mysql/my.cnf my.cnf "$(pwd)/config/my.travis.cnf" 9999 \
                    --slave   /path/to/slave    slave  /path/to/slave_file

# Simple:
#
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-10 50
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9 100
```

Clear the alternatives:

```sh
update-alternatives --remove-all ruby
```

Example: set `/etc/alternatives/editor` to vim (requires the alternative to be installed):

```sh
sudo update-alternatives --set editor /usr/bin/vim
```

## Environment

### Available variables

Current user run dir:

```sh
$XDG_RUNTIME_DIR
```

### Shell (initscripts: bashrc, ...)

There are two aspects of a shell:

- interactive <> non interactive
- login <> non login

Best explanation: https://askubuntu.com/a/376386/46091.

Additionally, there are per-O/S specific compiled configurations. On Debian-based systems, there is `/etc/bash.bashrc` which is run on *non login* shells.

Files involved:

- `/etc/profile`: for login shells; conditionally sources `/etc/bash.bashrc`; always sources `/etc/profile.d/*.sh`
- `$HOME/.profile`: for login shells; sources `$HOME/.bashrc` (!)
- `/etc/bash.bashrc`: for interactive non-login shells (see [here](https://askubuntu.com/a/815083)), however, it's not always loaded when `$HOME/.bashrc` is (!!)
- `$HOME/.bashrc`: not always sourced; part of the body is optionally executed.

All in all, it'a bloody mess; see the next subsection after the examples.

#### Example cases

Example cases (the tracker has been set before the interactive test in every file):

```sh
# Interactive SSH session: /etc/profile /etc/bash.bashrc /etc/profile.d/track.sh /home/user1/.bashrc
ssh myserver
env | grep ^ENV_TRACKER

# Non-interactive SSH session: /etc/bash.bashrc /home/user1/.bashrc
ssh myserver 'env | grep ^ENV_TRACKER'

# `sudo -u`: none, [explanation](https://unix.stackexchange.com/a/472226): $HOME is not changed!
ssh -t myserver 'sudo -u user2 bash -c "env | grep ^ENV_TRACKER"'

# `sudo -iu` + non interactive shell (`bash -c`): /etc/profile /etc/profile.d/track.sh /home/user2/.bashrc
ssh -t myserver 'sudo -iu user2 bash -c "env | grep ^ENV_TRACKER"'

# `sudo -iu` -> interactive shell: /etc/profile /etc/bash.bashrc /etc/profile.d/myinit.sh /home/app/.bashrc
ssh -t myserver sudo -iu user2
env | grep ^ENV_TRACKER

# `sudo -Eu` + non interactive shell (`bash -c`): /etc/bash.bashrc /home/user1/.bashrc
ssh -t myserver 'sudo -Eu user2 bash -c "env | grep ^ENV_TRACKER"'
```

The above are **not** all the possible cases (!!); in a tested Capistrano workflow, `$HOME/.bashrc` was not sourced (wondering if it was using another shell).

If one wants to cover every case (except the bare `sudo -u`), the best is to use `$HOME/.bashrc` (modified at the top) **and** `/etc/profile.d/` scripts.

On a configuration managements system, a clean solution is likely to create a `$HOME/.bashrc.d`-style solution, for drop-in scripts.

On a server system, each login loads the new init scripts, so there's no need to reboot in order to test.

Don't forget that system services don't run any script. In the case of Systemd, use `Environment=...` for this purpose.

#### sudo -i, login shell test, and bash

The `-i` does **not** stand for `i`nteractive; it stands for `i`nitial login` (=login shell).

Joint usage of sudo and bash is confusing, because bash has its own login/interactive options.

Behavior:

```sh
sudo -iu app bash -c 'shopt -q login_shell && echo login shell'
# none

sudo -iu app bash
shopt -q login_shell && echo login shell
# none

sudo -iu app
shopt -q login_shell && echo login shell
# login shell
```

Bash options:

```
-i          : If the -i option is present, the shell is interactive.
-l, --login : Make bash act as if it had been invoked as a login shell (see INVOCATION below).
```

## Handle system date/time

```sh
# Modern approach.
#
systemctl restart systemd-timesyncd

# Old approach
#
apt-get install ntpdate
ntp stop
ntpdate ntp.ubuntu.com
ntp start

# In order to manually set time, disable NTP service (no sudo required)
#
timedatectl set-ntp no
```

## Job scheduling

### Cron

Cron doesn't need to be restarted when files are changed.

Note that besides the `/etc/cron*` files/dirs, there are also files in the locations `/var/spool/cron/crontabs/$user`.

Be **very** careful of the executed shell type (at least non-interactive).

User operations:

```sh
crontab -l   # list
crontab -e   # edit
crontab -d   # delete
```

`MAILTO=dest@email.com` (put on top) sends email.

#### Schedule format

In order to run something on boot, use `@reboot`, and *double check that it works* (see https://unix.stackexchange.com/a/109805):

```sh
@reboot date >> /var/log/reboots
```

#### Subminute granularity

(the following approaches handle the repetition inside cron)

If the job exits almost immediately, can run:

```cron
# The `&` is correct (the shell is dash)
* * * * * for i in 0 1 2; do some_job & sleep 15; done; some_job
```

If the job doesn't exit immediately, but takes less than the interval:

```cron
* * * * * sleep 00; some_job
* * * * * sleep 15; some_job
* * * * * sleep 30; some_job
* * * * * sleep 45; some_job
```

If the job can potentially take more than the interval, and must be killed if when this happens, use the GNU `timeout` program:

```cron
* * * * * sleep 00; timeout 15s some_job
# ...
```

### At

WATCH OUT! On Ubuntu, it requires at least `sendmail` (otherwise the syslog reports a confusing `mail missing` error).

```sh
# General format add job (uses `/bin/sh`; no path!).
#
at 22:20 <<< 'echo abc'

at now + 1 hour                       # minute, week; plural also accepted
at 10:00 Sun
at 10:00 July 25
at 10:00 6/22/2015                    # WATCH OUT! mm/dd/YYYY; doesn't accept month literal
at next month                         # same date on next month
at (tomorrow|midnight)

atq                                   # list job
at -c 5                               # show job content

atrm $jobnum                          # remove job (use number from `atq` output)
```

Some programs/systems won't work out of the box, due to the `at` context. For systemd suspend, see [Systemd timers](#timers-scheduled-events).

## Apparmor

Source: https://www.digitalocean.com/community/tutorials/how-to-create-an-apparmor-profile-for-nginx-on-ubuntu-14-04.

```sh
# apparmor log: `grep audit /var/log/kern.log`

aptitude install apparmor-utils

# create empty profile and set complain mode
# restart from here, with --force, if it's needed to update the profile with aa-logprof
#
aa-autodep --force /opt/nginx_app_server/sbin/nginx
aa-complain /opt/nginx_app_server/sbin/nginx

# restart nginx
#
/opt/nginx_app_server/sbin/nginx               # start (or restart if necessary)

# go through all the entries and authorize them, then save.
# if the profile file is not specified, all profiles are examined.
#
aa-logprof -f /etc/apparmor.d/opt.nginx_app_server.sbin.nginx
```

## fstab

Columns:

- `options` (-3)
  - `X-mount.mkdir[=mode]`                 : create the mountpoint (as `mkdir -p`) if not exists
  - `nofail`                               : don't report error if the device is not present; best used with timeout (`x-systemd.device-timeout`)
  - `x-systemd.device-timeout=<time>`      : time can be specified with suffixes, eg. `1ms`; `0` is infinite; defaults to 90"
  - `x-systemd.requires-mounts-for=/path`  : require paths to exist (constrain ordering)
  - `x-systemd.requires=zfs-mount.service` : require service to run
- `dump` (-2): `1` if the partition is included when using dump tool
- `pass` (-1): order when checking with fsck

Test the content of fstab (not reliable for actual mounting; for example, doesn't detect if the nfs tools are installed, and reports success):

```sh
# [f]ake, [a]ll, [v]erbose
mount -fav
```

Example NTFS mount (mountpoint must exist):

```fstab
/path/to/dev /path/to/mount ntfs rw,auto,user,fmask=133,dmask=022,uid=1000,gid=1000 0 0
```

## sysctl

```sh
sudo sysctl -a                                                                   # list all values
sysctl -w kernel.perf_event_paranoid=1                                           # set a value
echo "kernel.perf_event_paranoid = 1" > /etc/sysctl.d/50-allow-perf-nonroot.conf # make a value permanent
```

## Modules

```sh
modprobe $name             # load module
modprobe -r $name          # unload the module, with all the dependencies
lsmod                      # list loaded modules
insmod /path/to/$module.ko # load a module

# Find if a module is loaded.
#
if lsmod | grep -qP '^zfs\s'; then echo loaded; fi

# Find if a module is _installed_ on the rootfs (different from loaded!).
#
if modinfo zfs > /dev/null 2>&1; then echo installed; fi
```

## Grub

### Install from live cd (or perform chrooted operations)

```sh
sudo su
mount -o subvol=@ /dev/sda1 /mnt  # use '-o subvol=@' for btrfs
# mirror dev etc. (see https://unix.stackexchange.com/questions/198590/what-is-a-bind-mount)
for vdev in dev sys proc; do mount --bind /$vdev /mnt/$vdev; done
mount /dev/sda1 /mnt/boot          # only if there is a separate boot partition
chroot /mnt
grub-install /dev/sda
exit
for vdev in proc sys dev; do umount --recursive --force --lazy /mnt/$vdev; done
```

### Add a Windows entry

In some cases, the standard GRUB procedures (`os-probe`, etc.) may not work; in this case, manually add an entry:

```sh
# Windows EFI partition, NOT Windows partition!
#
win_efi_uuid=$(lsblk -n -o UUID /dev/sda2)

cat >> /etc/grub.d/40_custom <<GRUB
menuentry 'Windows 10' {
    search --fs-uuid --no-floppy --set=root $win_efi_uuid
    chainloader (\${root})/EFI/Microsoft/Boot/bootmgfw.efi
}
GRUB

update-grub
```

## Logging/syslog/tools

```sh
logger [-t $tag] $message           # write to system logger; --[t]ag
```

### Logrotate

Command:

```sh
logrotate -d -f $config_file        # test configuration; --[d]ebug: dry run; --[f]orce: force file rotation
```

Configuration:

```conf
notifempty                    # Don't rotate empty files (`ifempty` is the default)
```

Logrotate automatically sends the `HUP` signal to the processes, so that the files are reopened after move. If the service doesn't support it, either:

- use the `copyrotate` option;
- or manually perform the reload/restart:
  ```
  sharedscripts
  postrotate
      systemctl reload-or-restart nmon
  endscript
  ```

## Terminal emulator

Open a new tab, execute a command, and leave the tab open:

```sh
mate-terminal --tab --command 'sh -c "echo abc; zsh"'
```

## Desktop Environment: windows

Bringing a window to the front:

```sh
# Sample window names
#
xdotool getwindowname "$window_id"
# Google - Mozilla Firefox (Private Browsing) # private with website
# Mozilla Firefox                             # standard, with blank page

# Search only in the current desktop.
#
desktop_id=$(xdotool get_desktop)

# For a brief moment, xdotool changes the terminal name to the command, so if we search for a
# string, the terminal will be found as well! So one needs to make sure that the name regex doesn't
# match itself (any metacharacter will do).
#
# `--all` is important, otherwise, each token of the `--name` parameter is an independent match!
#
# The windows are listed is reverse access order (last accessed is at the bottom)
#
# Don't forget the `|| true`!
#
window_id=$(xdotool search --all --desktop "$desktop_id" --name 'Mozilla Firefox \(Private Browsing\)$' | tail -n 1 || true)

# Bring to front.
#
xdotool windowactivate "$window_id"
```

## Gnome stuff

```sh
mate-session-properties                        configure mate (gnome) autostart applications
```

### Distro info

Get distro infos (multiple options):

- `lsb_release -a`
- `cat /etc/*release`
- `cat /etc/debian_version`: Complete version; specific for debian

### Dconf/Gsettings

```sh
# Dump the whole tree.

dconf dump /
gsettings list-recursively

# Watch the whole tree.
#
# gsettings can't watch the whole tree; only one schema.
# but the second works fine 🙂

dconf watch /

gsettings monitor org.mate.caja

gsettings list-recursively > /tmp/before.gsettings
watch -n 1 -x bash -c 'diff /tmp/before.gsettings <(gsettings list-recursively)'
```

## MIME (extensions) (file associations) handling

```sh
# Find the MIME type.
# More accurate than `file --mime-type`;  on Markdown docs for example, the type found was `text/plain`.
#
mimetype "file"

# Open a file using MIME.
# On types which are not set yet, default will be asked.
#
xdg-open "file_or_uri" # supports URI (see https://wiki.archlinux.org/index.php/default_applications)
mimeopen "file"

# Find/set application default.
#
xdg-mime query default "x-scheme-handler/http"
xdg-mime default "sublime_text.desktop" "text/markdown"

# Reset custom associations.
#
rm "~/.local/share/applications/wine*.desktop"
rm "~/.local/share/applications/mimeinfo.cache"
```

### Adding and associating a new application

```sh
# If there is ambiguity with another mime type (or if it's not registered), associate the extension (see following section).

# Partial; see `/usr/share/applications/firefox.desktop` for a full example.
#
cat > ~/.local/share/applications/ << 'CONF'
[Desktop Entry]
Version=1.0
Name=Sav's Browser
...
CONF

# Optional: add to MATE menu.
#
perl -i -lpe '$. == 1 && print "location:/home/saverio/.local/share/applications/browser.desktop"' ~/.config/mate-menu/applications.list

# Register!
# WATCH OUT!! Don't use the full path to the .desktop file!!
#
xdg-mime default "browser.desktop" "x-scheme-handler/http"
xdg-mime default "browser.desktop" "x-scheme-handler/https"
```

### Split MAFF association

Associate an extension based on the filename rather than the MIME type:

```sh
cat > /usr/share/mime/packages/maff.xml << 'XML'
<?xml version="1.0" encoding="UTF-8"?>
<mime-info xmlns="http://www.freedesktop.org/standards/shared-mime-info">
  <mime-type type="application/x-maff">
    <comment>Maff File</comment>
    <glob pattern="*.maff"/>
  </mime-type>
</mime-info>
XML

update-mime-database /usr/share/mime
```

## Encryption

### Mount encrypted (.ecryptfs) home

Obsolete, but kept for reference. Requires `ecryptfs-utils` package.

```sh
ecryptfs-unwrap-passphrase /mnt/home/.ecryptfs/saverio/.ecryptfs/wrapped-passphrase # mount passphrase
ecryptfs-add-passphrase --fnek                                                      # use mount PP; store the second signature
mount -t ecryptfs /mnt/home/.ecryptfs/saverio/.Private /mnt                         # use mount PP; enable filename encryption; use signature
```

### Setup hibernation on Ubuntu-setup LUKS volume

Ubuntu creates a too small swap partition, and doesn't set the swap volume kernel option.

Run from live CD, after the installation is completed:

```sh
apt install -y partitionmanager
partitionmanager

# ... resize the partitions ...

mkswap "$(ls -1 /dev/mapper/*swap*)"

# now reboot
```
