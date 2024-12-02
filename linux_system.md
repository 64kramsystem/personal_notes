# Linux system

- [Linux system](#linux-system)
  - [Security](#security)
  - [Filesystems/partitions](#filesystemspartitions)
  - [Packages](#packages)
    - [Apt/dpkg installation hooks (`etc/apt/apt.conf.d`)](#aptdpkg-installation-hooks-etcaptaptconfd)
  - [Repositories](#repositories)
    - [Keys handling](#keys-handling)
  - [Debconf](#debconf)
  - [Debian alternatives](#debian-alternatives)
  - [Environment](#environment)
    - [Available variables](#available-variables)
    - [Shell (initscripts)](#shell-initscripts)
      - [Example cases](#example-cases)
      - [sudo -i, login shell test, and bash](#sudo--i-login-shell-test-and-bash)
  - [Update system time](#update-system-time)
  - [Job scheduling](#job-scheduling)
    - [Cron](#cron)
    - [At](#at)
  - [Systemd/SystemV](#systemdsystemv)
    - [Systemctl](#systemctl)
    - [journalctl](#journalctl)
    - [Configuring a unit](#configuring-a-unit)
    - [Event triggers](#event-triggers)
    - [Timers (scheduled events)](#timers-scheduled-events)
  - [Apparmor](#apparmor)
  - [Networking](#networking)
    - [NetworkManager](#networkmanager)
      - [Changing Network manager DNS (18.04+)](#changing-network-manager-dns-1804)
  - [fstab](#fstab)
  - [Modules](#modules)
  - [Install grub from live cd (or perform chrooted operations)](#install-grub-from-live-cd-or-perform-chrooted-operations)
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
  - [Mount encrypted (.ecryptfs) home](#mount-encrypted-ecryptfs-home)
  - [Hardware](#hardware)
    - [Disable SMT (hyperthreading)](#disable-smt-hyperthreading)
    - [Disable mouse/keyboard](#disable-mousekeyboard)
    - [Screen stuff](#screen-stuff)
      - [Add new resolution (HiDPI problem)](#add-new-resolution-hidpi-problem)
    - [Audio](#audio)
    - [Disconnect and power off a device](#disconnect-and-power-off-a-device)
    - [Keyboard](#keyboard)

## Security

```sh
chown $name $file...                        # change owner
chgrp $name $file...                        # change group
chmod [ugoa][-+][rwx] $file...              # change permissions; [u]ser [g]group [o]thers [a]ll

chage [-m $min] [-M $max] [-W $warn] [-I $inactive_days] $login # change password properties
chage -l $login                             # show info
```

Snippets:

```sh
chmod -R go-rx .ssh                         # setup SSH permissions
chage -d $(date +%Y-%m-%d) myuser           # set the change date to today!
chage -I 0 myuser                           # expire password
```

## Filesystems/partitions

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

## Packages

Informations:

```sh
# Show package info; for this operation, `apt` is stable [enough].
#
apt show $package

# Check if a package is installed, in the most standard way possible.
#
if dpkg --get-selections | grep -qP '^trash-cli\s$'; then echo installed; fi

# Advanced filtering with aptitude.
#
# Aptitude uses regexes by default, but it doesn't support all the patterns.
# In order to filter out i386 packages, use the suffix `~rnative`.
# The option `-F %p` prints only the package name.
#
aptitude search -F %p 'suld-driver2-[0-9.]+$~rnative'

# Better use this than `apt search`; the latter is undocumented and has an atrocious output.
#
apt-cache search '^linux-image-unsigned-5.4.[[:digit:]-]+-generic'

# Show dependencies of a package
#
aptitude why -v $package

# Display the sources of given package
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

# Mark a package as manually installed (apt[itude])
#
aptitude install "${package}&m"

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

# Unpack deb file
#
ar -vx $package.deb
tar xvf data.tar.gz

# Find which package a file belongs to
#
dpkg -S $file
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
debconf-get-selections | grep packagename/    # more detailed, but requires `debconf-utils package`
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
update-alternatives --query <program>
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
```

Clear the alternatives:

```sh
update-alternatives --remove-all ruby
```

Example: set `/etc/alternatives/editor` to vim (requires the alternative to be installed):

```sh
update-alternatives --set editor /usr/bin/vim
```

## Environment

### Available variables

Current user run dir:

```sh
$XDG_RUNTIME_DIR
```

### Shell (initscripts)

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

## Update system time

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
```

## Job scheduling

### Cron

Cron doesn't need to be restarted when files are changed.

Note that besides the `/etc/cron*` files/dirs, there are also files in the locations `/var/spool/cron/crontabs/$user`.

User operation:

```sh
crontab -l   # list
crontab -e   # edit
crontab -d   # delete
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

## Systemd/SystemV

List all the services, and their running status: `service --status-all`. Great for disabling unnecessary services!!

### Systemctl

```sh
systemctl enable|disable $service     # enable/disable start on boot
systemctl start|stop $service         # immediately start/stop
systemctl reload $service             # reload; not always functional
systemctl reload-or-restart $service  # if reload is not defined (or has no effect), restart; WATCH OUT! don't define `ExecReload`
systemctl status $service
systemctl cat $service                # print unit file
systemctl edit --full $service        # edit unit file

systemctl daemon-reload               # invoke this after updating a unit
systemctl daemon-reexec               # required to reload Sytemd's own configuration (e.g. changes to `/etc/systemd/system.conf`)

systemctl is-(active|enabled|failed) $service  # query status
systemctl list-units                  # active loaded units
systemctl --failed                    # show units that failed to start
systemctl --all                       # list everything (with their states), including timers
```

### journalctl

```sh
journalctl -b [-k]                                 # view log for current Boot; only [k]ernel messages
journalctl --pager-end --unit=$service.service     # show unit log; `page-end`: go to end
journalctl --vacuum-time=1d                        # clean systemd journal (/var/log/journal)
```

### Configuring a unit

See:

- https://www.freedesktop.org/software/systemd/man/systemd.service.html
- https://www.freedesktop.org/software/systemd/man/systemd.kill.html

General location of systemd units (not the only one): `/lib/systemd/system/$name.service`.

Almost all the entries are optional. Escaping is not standard; in order to escape, use `systemd-escape`.

```sh
# If entries need escaping, use double quotes, but not slashes. In at least the `ExecStart` entry, if,
# for example, a space is escaped with a slash, both the slash and the space will be part of the string
# (!).
#
cat > /etc/systemd/system/myservice.service <<UNIT
[Unit]
Description=My Service

# `Requires` states only dependency; `After` mandates ordering.
#
Requires=network.target
After=network.target

# Example of waiting for a network device to be online.
#
# After=sys-subsystem-net-devices-wlan0.device
# BindsTo=sys-subsystem-net-devices-wlan0.device

[Service]
# If ExecStart is specified (and `BusName` not specified), `simple` is implicit.
# Use `forking` when the process forks (e.g. nmon).
#
Type=simple

# Convenient.
#
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=my-service

User=app
Group=app

WorkingDirectory=/path/to/base_dir

Environment=BUNDLE_GEMFILE=/path/to/Gemfile
Environment=RAILS_ENV=my_rail_env

# In other to extend $PATH, use something like:
#
#     ExecStart=/bin/bash -c 'PATH=/new/path:$PATH exec /bin/mycmd'
#
ExecStart=/usr/bin/bundle exec my_command
ExecReload=/bin/kill -HUP $MAINPID

# Other options: `mixed`, `process`, `none`
# Better not to specify, unless required.
#
KillMode=control-group

# Other options: `no` (default), `on-failure`, ...
#
Restart=always

[Install]
# Generally, this is the setting.
#
WantedBy=multi-user.target
UNIT

# now enable and start
```

### Event triggers

Run a script on suspend/hibernate/thaw/resume (the change applies immediately):

```sh
cat > /lib/systemd/system-sleep/50_myscript << 'SH'
#!/bin/bash
set -o errexit

case "$1" in
  pre)
    # ACTION BEFORE SUSPEND/HIBERNATE
    ;;
  post)
    # ACTION AFTER THAW/RESUME
    ;;
esac
SH

chmod +x /lib/systemd/system-sleep/50_myscript
```

Run a script on screen lock/unlock:

```sh
dbus-monitor --session "type='signal',interface='org.gnome.ScreenSaver'" |
  while true; do
    read X;
    if echo $X | grep "boolean true" &> /dev/null;
      then echo locked >> /tmp/pizza.txt;
    elif echo $X | grep "boolean false" &> /dev/null;
      then echo unlocked >> /tmp/pizza.txt;
    fi
  done
```

### Timers (scheduled events)

Once-off events can be scheduled; mind that they're not second-accurate.

```sh
# Execute after certain interval, with output.
#
$ systemd-run --user --on-active=10min /bin/systemctl suspend
Running timer as unit: run-r4eeea4bc49524bb8b40f609cb183db48.timer
Will run service as unit: run-r4eeea4bc49524bb8b40f609cb183db48.service

# Execute at given time
#
$ systemd-run --user --on-calendar=09:12 /bin/systemctl suspend
```

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

## Networking

Show all the interfaces, even if unconfigured:

```sh
ip link show
```

Sample configuration for static IP (`/etc/network/interfaces`):

```
auto eth0
iface eth0 inet (dhcp|static
        address   192.168.56.103
        netmask   255.255.255.0
        network   192.168.56.0
        broadcast 192.168.56.255
        gateway   192.168.1.1)
```

Remove when changing the mac/card for an interface, otherwise the new one won't be recognized (SIOCSIFADDR error): `/etc/udev/rules.d/70-persistent-net.rules`

### NetworkManager

When adding hooks, the scripts must be executable!

#### Changing Network manager DNS (18.04+)

On 18.04 Desktop, one needs to modify the Network manager connection; see https://askubuntu.com/a/1104305/46091.

## fstab

Columns:

- `options` (-3)
  - `X-mount.mkdir[=mode]`: create the mountpoint (as `mkdir -p`) if not exists
  - `nofail`: don't report error if the device is not present; best used with timeout (`x-systemd.device-timeout`)
  - `x-systemd.device-timeout=<time>`: time can be specified with suffixes, eg. `1ms`; `0` is infinite; defaults to 90"
- `dump` (-2): `1` if the partition is included when using dump tool
- `pass` (-1): order when checking with fsck

Test the content of fstab (not reliable for actual mounting; for example, doesn't detect if the nfs tools are installed, and reports success):

```sh
# [f]ake, [a]ll, [v]erbose
mount -fav
```

## Modules

```sh
# Find if a module is loaded.
#
if lsmod | grep -qP '^zfs\s'; then echo loaded; fi

# Find if a module is _installed_ on the rootfs (different from loaded!).
#
if modinfo zfs > /dev/null 2>&1; then echo installed; fi
```

## Install grub from live cd (or perform chrooted operations)

```sh
sudo su
mount -o subvol=@ /dev/sda1 /mnt	# use '-o subvol=@' for btrfs
mount --bind /dev /mnt/dev		    # mirror dev etc. (see https://unix.stackexchange.com/questions/198590/what-is-a-bind-mount)
mount --bind /sys /mnt/sys
mount --bind /proc /mnt/proc
mount /dev/sda1 /mnt/boot		      # only if there is a separate boot partition
chroot /mnt
grub-install /dev/sda
exit
umount -R /mnt				            # `R`ecursively
```

## Logging/syslog/tools

```sh
logger [-t $tag] $message           # write to system logger; --[t]ag
```

### Logrotate

```sh
logrotate -d -f $config_file        # test configuration; --[d]ebug: dry run; --[f]orce: force file rotation
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

## Mount encrypted (.ecryptfs) home

Obsolete, but kept for reference. Requires `ecryptfs-utils` package.

```sh
ecryptfs-unwrap-passphrase /mnt/home/.ecryptfs/saverio/.ecryptfs/wrapped-passphrase # mount passphrase
ecryptfs-add-passphrase --fnek                                                      # use mount PP; store the second signature
mount -t ecryptfs /mnt/home/.ecryptfs/saverio/.Private /mnt                         # use mount PP; enable filename encryption; use signature
```

## Hardware

Get hardware information:

```sh
# Without options, all the system information is reported.
#
dmidecode -s \
  system-product-name   # (laptop) model
  bios-version
  chassis-serial-number # (dell) service tag
```

### Disable SMT (hyperthreading)

```sh
sudo tee /sys/devices/system/cpu/smt/control <<< "off"
```

### Disable mouse/keyboard

```sh
xinput --list
# ⎡ Virtual core pointer                    	id=2	[master pointer  (3)]
# ⎜ [...]
# ⎣ Virtual core keyboard                   	id=3	[master keyboard (2)]
#     ↳ [...]

xinput --list-props 3
# Device 'Virtual core keyboard':
# 	Device Enabled (156):	1
# 	[...]

# Enable
xinput set-int-prop 3 "Device Enabled" 8 0

# Disable
xinput set-int-prop 3 "Device Enabled" 8 1
```

### Screen stuff

```sh
# disable screen blanking
setterm -blank 0 -powersave off -powerdown 0 => xset s off
```

#### Add new resolution (HiDPI problem)

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

### Audio

```sh
# Create remapped mono sink (downmix stereo to mono)
#
pacmd list-sinks | grep name:
pacmd load-module module-remap-sink sink_name=mono master=THE_NAME_FROM_THE_PREVIOUS_COMMAND channels=2 channel_map=mono,mono
```

### Disconnect and power off a device

```sh
udisksctl dump                  # check disks with "\bRemovable: +true"
udisksctl unmount -b /dev/sdb1  # unmount parts from dump which have "MountPoints: +$"
udisksctl power-off -b /dev/sdb # yay!
```

### Keyboard

Reconfigure keyboard:

```sh
dpkg-reconfigure console-data
```
