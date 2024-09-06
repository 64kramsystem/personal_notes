# Linux system

- [Linux system](#linux-system)
  - [Processes](#processes)
    - [Useful operations](#useful-operations)
    - [Signals/exit codes/suspension](#signalsexit-codessuspension)
    - [Process tree display](#process-tree-display)
    - [Locking (via flock)](#locking-via-flock)
  - [Memory (measurement)](#memory-measurement)
  - [Security (Permissions)](#security-permissions)
    - [Sudo](#sudo)
    - [Commands logging (auditd)](#commands-logging-auditd)
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
  - [crypttab](#crypttab)
  - [sysctl](#sysctl)
  - [Modules](#modules)
  - [Grub](#grub)
    - [Install from live cd (or perform chrooted operations)](#install-from-live-cd-or-perform-chrooted-operations)
    - [Add a Windows entry](#add-a-windows-entry)
  - [Logging/syslog/tools](#loggingsyslogtools)
    - [Logrotate](#logrotate)
  - [Terminal (emulator)](#terminal-emulator)
  - [Desktop Environment](#desktop-environment)
    - [Windows](#windows)
    - [MIME (extensions) (file associations) handling](#mime-extensions-file-associations-handling)
      - [Adding and associating a new application](#adding-and-associating-a-new-application)
      - [Split MAFF association](#split-maff-association)
    - [Applications autostart](#applications-autostart)
  - [Distro info](#distro-info)
  - [Dconf/Gsettings](#dconfgsettings)
  - [LVM](#lvm)
  - [Encryption](#encryption)
    - [Unmount LUKS encrypted partitions](#unmount-luks-encrypted-partitions)
    - [Mount encrypted (.ecryptfs) home](#mount-encrypted-ecryptfs-home)
    - [Setup hibernation on Ubuntu-setup LUKS volume](#setup-hibernation-on-ubuntu-setup-luks-volume)
  - [Benchmarking system](#benchmarking-system)

## Processes

WATCH OUT!! When using the [debug log strategy](bash.md#log-a-script-outputenable-debugging-log), process handling becomes an unholy mess (eg. HUP causes ERR in the tree). In such cases, it's possible that alternative strategies are simpler, eg. old instances exit if they find they're old, rather than newer ones killing them.

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
pkill --parent $(< parent_pidfile)

# Invoke a command, and kill the process after a timeout, if it doesn't exit!
#
timeout --signal=KILL --verbose 5 mycommand

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

### Locking (via flock)

Exclusive locking:

```sh
# "200" (file descriptor) is arbitrary; since it's per-process, but the lock is still on the file, there's no risk of collision.
# `exec` creates the file (the FD can't be a variable), if it doesn't exist.
# Script-wise, the simplest thing is not to delete the lockfile on exit/error.
#
exec 200>/run/mylockfile.lock
flock -n 200 || {
  >&2 echo 'error!'
  exit 1
}
```

## Memory (measurement)

Measuring memory is a complex topic. A simple and accurate enough tool that can be used is [`ps_mem`](https://github.com/pixelb/ps_mem).

## Security (Permissions)

```sh
chown $name $file...                    # change owner
chgrp $name $file...                    # change group
chmod [ugoa][-+][rwx] $file...          # change permissions; [u]ser [g]group [o]thers [a]ll
chsh -s $shell $file                    # change shell; use `/usr/sbin/nologin` as shell disable it for a user

chage [-m $min] [-M $max] [-W $warn] [-I $inactive_days] [-E $expire_date] $login # change password properties
chage -l $login                         # show info

passwd -d                               # set a blank password
chmod -R go-rx .ssh                     # setup SSH permissions
chage -d $(date +%Y-%m-%d) $user        # set the last changed date to today, and the mandatory change date to (today + max days)
chage -I 0 $user                        # expire password
chage -E -1 $user                       # disable account expiry
```

Rename user/group/home:

```sh
usermod -l $new_user $old_user
groupmod -n $new_user $old_user
usermod -d $new_home -m $user

usermod -a -G $group $user              # add user to group
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
```

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
$SUDO_USER                                        # User who invoked sudo
[[ $EUID -eq 0 ]]                                 # Shell (all) check for user being root
sudo -n true && echo 'no pwd!'                    # Check if password is required to sudo (WATCH OUT SEMANTICS!)
/usr/bin/pkexec --disable-internal-agent $command # Graphical sudo; doesn't cache to following `sudo` (execute a shell if necessary)

# Passwordless sudo (run as regular user)
#
echo "$(whoami) ALL=(ALL:ALL) NOPASSWD: ALL" | sudo tee "/etc/sudoers.d/$(whoami)_no_sudo_pwd"
sudo chmod 440 !$

# Run a specific command (also scripts) as sudo, without asking the password.
#
# WATCH OUT! The command must be invoked *exactly* as "sudo <full_command>"; this implies:
#
# 1. in this case `sudo /path/to/command param1 paramN`.
# 2. if one wants to run the command without `sudo`, they need to use the `prepare_sudo` strategy, because
#    commands including sudo in the script will otherwise ask the password.
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

### Commands logging (auditd)

```sh
# The package `audispd-plugins` is not needed to monitor bash commands.

# Append; there are already some values.
#
# `-F euid=`/`-F auid=` filter by user id (username not supported); otherwise, all users are logged.
# `euid` is the effective user id; `auid` is the original. e.g.
#
#     ssh foo -> cmd1 -> sudo -u bar cmd2
#
# yields:
#
# - `-F euid $foo`: cmd1
# - `-F auid $foo`: cmd1, cmd2
# - `-F euid $bar`: cmd2
# - `-F auid $bar`: (nothing)
#
# Without uid filter, the log is very verbose, as many processes are logged.
#
# In order to filter multiple users, specify one rule per line.
#
# Generally, a `-F arch=b32` rule is added, but it has no purpose if the system doesn't support
# 32-bit binaries (can verify via `dpkg --print-foreign-architectures`; returns blank if i386 is
# not supported).
#
# Add `-k searchkey` to add a key that can be used to search.
#
# The log is verbose, and can't be reduced.
#
cat >> /etc/audit/rules.d/audit.rules << CONF
-a exit,always -F arch=b64 -S execve -F euid=$myuid
CONF

# Disable the built-in logging, and setup logratote, for consistency with other services.
#
perl -i -pe 's/^max_log_file_action = \K/IGNORE/'

# `systemctl reload auditd` is not supported (as of Ubuntu 22.04).
#
cat >> /etc/logrotate.d/auditd << 'CONF'
/var/log/audit/audit.log
{
  daily
  missingok
  rotate 7
  compress
  create 0640 root root
  postrotate
    service auditd reload > /dev/null
  endscript
}
CONF

service auditd restart

# Verify existing rules.
#
auditctl -l

# Delete old logs.
#
service auditd rotate; rm /var/log/audit/audit.log.*

# Search for a given user
#
ausearch -ua $myuser

# Tail the log (ausearch doesn't support tailing).
#
tail -f /var/log/audit/audit.log | ausearch -i
```

## Filesystems/partitions/mounts

Useful operations:

```sh
# Mount ISO without sudo; filenames are lower case.
#
fuseiso

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
if findmnt $path > /dev/null; then echo is mountpoint; fi
if mountpoint -q $path      ; then echo is mountpoint; fi
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

Copy partitions layout across devices:

```sh
# Destination layout is overwritten. WATCH OUT! Must randomize dest GUIDs.
#
# -R|--replicate; -G|--randomize-guids.
#
sgdisk /dev/source -R /dev/dest
sgdisk -G /dev/dest

# COOL!!! Duplicates the layout, without fully overwriting the destination.
#
# - requires an existing destination partition table
# - duplicates the names
# - assumes that names exist and don't include spaces
# - UUIDS are generated
#
# Sample input:
#
#   Number  Start      End        Size       File system  Name   Flags
#    1      2048s      20973567s  20971520s  ext2         PIZZA
#    2      20973568s  29566975s  8593408s   ext3         PIZZA  msftdata
#
echo 'unit s print' |
  parted /dev/source |
  perl -lne 'print "mkpart $4 $3 $1 $2" if /^ *\d+ +(\d+s) +(\d+s) +\d+s +(\S+) +(\S+)/' |
  parted /dev/dest
```

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

# Check if a package is installed, in the most standard way possible. Runs fast
# States found: `install`, `hold`, `deinstall`; the last seems only cfg installed.
# `dpkg` exits with success even if the package is not found.
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

# Display the repositories with a given package
#
apt-cache policy $package

# Find the repository of a package
#
apt-cache showpkg $package
```

Depedency-related:

```sh
# Basic package dependencies are included in the package info (see commands above)

# Show why a package was installed
#
aptitude why -v $package

# (Extra tool) Check dependencies of installed package.
#
apt-rdepends $package

# Produce a package's dependency tree.
#
debtree –max-depth=2 mypackage | dot -Tpng > mypackage.png
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

# Display package info, including dependencies.
#
dpkg -I $file
apt show $package
aptitude show $package

# Find which packages include a file with the given name (no need for path). This is not a preinstalled tool.
# Require its case to be refreshed `apt-file cache`.
#
apt-file search $filename
```

Useful generic snippets:

```sh
# Refresh keys (updates expired).
#
apt-key adv --refresh-keys --keyserver keyserver.ubuntu.com

# If a PPA label has been updated, it causes the error "Repository 'http:...' changed its 'Label' value from 'Foo' to 'Bar"; run one of the two
#
apt update                                # interactive
apt-get update --allow-releaseinfo-change # non-interactive

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
crontab -e   # edit; use also to delete entries
```

The variables defined on top are exported. `MAILTO=dest@email.com` sends email.

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

Tweaks:

- `noatime`: big discussion; nobody ever mentiones experience of failing programs besides mutt 🙄
  - `lazytime`: additional (doesn't replace others); possibly, still causes cascading writes on COW filesystems

Test the content of fstab (not reliable for actual mounting; for example, doesn't detect if the nfs tools are installed, and reports success):

```sh
# [f]ake, [a]ll, [v]erbose
mount -fav
```

Default mount options (mountpoint must exist):

```fstab
UUID=foo     /path   ntfs rw,auto,user,fmask=133,dmask=022,uid=1000,gid=1000 0 0

UUID=foo     /       btrfs defaults,subvol=@                                 0 1
UUID=bar     /home   btrfs defaults,subvol=@home                             0 2
```

## crypttab

Default:

```
nvme0n1p3_crypt UUID=foo none luks,discard
```

## sysctl

`/etc/sysctl.d` files are read in lexicographic order; in order to go past `99-`, use `99a-` and similar, NOT `100-`.

```sh
sysctl -a                                             # list all values
sysctl -w kernel.perf_event_paranoid=1                # set a value
echo "my.value = 1" > /etc/sysctl.d/50-mysetting.conf # make a value permanent
sysctl --system                                       # reload values from sys dirs; also lists the load order!
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

It seems that pseudo-grub hooks are possible; see [here](https://samwhelp.github.io/book-ubuntu-qna/read/case/linux-package/hook-update-grub).

### Install from live cd (or perform chrooted operations)

```sh
sudo su
mount -o subvol=@ /dev/sda3 /mnt # use '-o subvol=@' for btrfs
mount /dev/sda2 /mnt/boot        # assumes a boot partition
mount /dev/sda1 /mnt/boot/efi    # efi partition

# If /run is not mounted, must manually add the nameserver to /etc/resolv.conf, since it's
# linked to /run/systemd/resolve/stub-resolv.conf.
for vdev in dev proc run sys; do mount --bind /$vdev /mnt/$vdev; done
chroot /mnt
grub-install /dev/sda
exit
# This may not work; if that's the case, use (but makes the system unstable):
#
#   for vdev in run proc sys dev; do umount --recursive --force --lazy /mnt/$vdev; done
#
umount --recursive /mnt
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

## Terminal (emulator)


```sh
# Open a new tab, execute a command, and leave the tab open:
#
mate-terminal --tab --command 'sh -c "echo abc; zsh"'

# Terminal size:
#
stty size
```

## Desktop Environment

### Windows

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

# Actions
#
xdotool windowactivate $window_id        # bring to front
xdotool windowminimize $window_id        # minimize
xdotool windowactivate --sync $window_id # restore (e.g. after minimization)
```

### MIME (extensions) (file associations) handling

Associations can be done at system level or user level; commands can be run for the user or via sudo (depending on the files they operate on).

The involved parts are:

- optionally, a `.xml` file, to associate a glob pattern with a MIME type
- a `.desktop` file, which declares a binary
- `xdg-mime default` associates the `.desktop` file withe the MIME
- `update-mime-database` updates the MIME
  - the result is immediatley available for local users, but unclear for system

The global directory for applications is `/usr/share/applications`.

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

#### Adding and associating a new application

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

#### Split MAFF association

Associate an extension based on the file extension rather than the MIME type:

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

### Applications autostart

`mate-session-properties`: Configure MATE (gnome) autostart applications.

Store `.desktop` files under `~/.local/share/applications` in order to make them autostart; the legacy path is `~/.config/autostart`. Minimalist example:

```conf
# The file exec permission is not required.

[Desktop Entry]
Type=Application
Name=Maestral
Exec=/usr/bin/python3 -m maestral_qt -c maestral
X-GNOME-Autostart-enabled=true
# X-GNOME-Autostart-Delay=2
```

Applications can be launched (for testing) via `gtk-launch app_name.desktop`; WATCH OUT! The termina environment may be different from the DE's.

## Distro info

Get distro infos (multiple options):

- `lsb_release -a`
- `cat /etc/*release`
- `cat /etc/debian_version`: Complete version; specific for debian

## Dconf/Gsettings

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

## LVM

Create a whole LVM container:

```sh
# Create a physical container.
#
pvcreate /dev/mapper/"$CONTAINER2_NAME"

# List physical containers; sample output:
#
#    PV                     VG            Fmt  Attr PSize  PFree
#    /dev/mapper/sda3_crypt vgubuntu-mate lvm2 a--  61.81g     0
#    /dev/mapper/sdb1_crypt               lvm2 ---  63.98g 63.98g
pvs

# Create a volume group.
#
vgcreate vgubuntu-mate-mirror /dev/mapper/"$CONTAINER2_NAME"

# Display the volume groups; sample output:
#
#  VG                   #PV #LV #SN Attr   VSize  VFree
#  vgubuntu-mate          1   2   0 wz--n- 61.81g     0
#  vgubuntu-mate-mirror   1   0   0 wz--n- 63.98g 63.98g
#
vgs

# Create a logical volume (in the volume group).
# [n]ame; [l] size in extents
#
lvcreate -l 100%FREE -n root vgubuntu-mate-mirror

# List the logical volumes; sample output:
#
#    LV     VG                   Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
#    root   vgubuntu-mate        -wi-a----- 58.16g
#    swap_1 vgubuntu-mate        -wi-ao---- <3.65g
#    root   vgubuntu-mate-mirror -wi-a----- 63.98g
#
lvs
```

Now, one can create an FS in `/dev/mapper/vgubuntu--mate-root`; setup crypttab/fstab:

```sh
echo "/dev/mapper/vgubuntu--mate-root /     btrfs defaults,subvol=@,$BTRFS_OPTS     0 1" >> /target/etc/fstab

# `keyscript` caches passwords across VGs, so that they're not asked twice.
#
LUKS_DISK2_PART_UUID=$(blkid -s UUID -o value ${DISK2_DEV}1)
echo "$CONTAINER2_NAME UUID=$LUKS_DISK2_PART_UUID none luks,discard,keyscript=decrypt_keyctl" >> /target/etc/crypttab
```

## Encryption

LUKS general stuff:

```sh
# When `-` is accepted, the password can be piped (MUST USE `echo -n`)

cryptsetup luksDump /dev/sda3                    # print LUKS info; use the physical dev partition
cryptsetup luksFormat --batch-mode /dev/sda3 [-] # interactive; batch-mode=no warning
cryptsetup luksOpen /dev/sda3 mapper-name [-]    # interactive
cryptsetup luksClose mapper-name                 # mapper device
```

To check the lock status of a LUKS device, see https://unix.stackexchange.com/a/695898/12814.

### Unmount LUKS encrypted partitions

See [ejectdisk](https://github.com/64kramsystem/openscripts/blob/416f4a456412210df86a574a4d19a649481133b2/ejectdisk#L76); this involves:

```sh
lsblk -n -o NAME,TYPE,MOUNTPOINT,UUID $device  # find the given device tree properties; WATCH OUT! If the dev is encrypted, it may return two entries
blkid -s UUID -o value $device                 # when requiring the UUID of one device, this is a better choice (always returns only one entry)
udisksctl unmount -b /dev/mapper/$luks_device  # unmount the encrypted partition(s)
udisksctl lock -b /dev/$luks_device            # lock the LUKS device
```

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

## Benchmarking system

For real world apps and comparisons, use the Phoronix test suite:

- download from https://www.phoronix-test-suite.com/?k=downloads
- run `phoronix-test-suite benchmark build-linux-kernel-1.15.0` (version is optional)
- compare on https://openbenchmarking.org/test/pts/build-linux-kernel-1.15.0
