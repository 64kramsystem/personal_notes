# Linux Systemd

- [Linux Systemd](#linux-systemd)
  - [Generic commands and notes](#generic-commands-and-notes)
  - [Systemctl](#systemctl)
    - [Uninstall (remove) a unit](#uninstall-remove-a-unit)
    - [Disabling and dependencies](#disabling-and-dependencies)
    - [System commands and misc options](#system-commands-and-misc-options)
    - [User units](#user-units)
  - [journalctl](#journalctl)
  - [Configuring a service unit](#configuring-a-service-unit)
    - [Notify unit type](#notify-unit-type)
    - [Timeouts/killing](#timeoutskilling)
    - [Overrides](#overrides)
    - [Hooks](#hooks)
  - [Timers (scheduled events)](#timers-scheduled-events)
    - [Systemd time format](#systemd-time-format)
    - [Once-off (transient)](#once-off-transient)
  - [Event triggers (execute program on resume, etc.)](#event-triggers-execute-program-on-resume-etc)

## Generic commands and notes

List all the services, and their running status: `service --status-all`. Great for disabling unnecessary services!!

Units are stored under `/etc/systemd/system`, however, it's best practice not to edit them directly, instead, use the `edit` command.

If an underlying service is *not* started via Systemd, from the perspective of the latter, the former is not started; one of the consequences is that stop won't be performed. Therefore, services managed by Systemd should consistently managed through it.

## Systemctl

```sh
# Status-related commands support more services

systemctl enable|disable [--now] $service # enable/disable start on boot; --now will also start/stop (but not restart!)
systemctl restart|start|stop $service # (re)start/stop immediately; asynchronous
systemctl reload $service             # reload; not always functional
systemctl reload-or-restart $service  # if reload is not defined (or has no effect), restart; WATCH OUT! don't define `ExecReload`
systemctl status $service
systemctl cat $service                # print unit file
systemctl edit --full $service        # edit unit file (--full: edit the service, instead of creating an override)
                                      # in order to create, add `--force`
                                      # in order to programmatically edit, workaround by setting `SYSTEMD_EDITOR=tee [-a]` and piping the content.
systemctl mask $service               # "mask": disable a service, by symlinking it to /dev/null; WATCH OUT! doesn't fail if the service doesn't exist
# it seems that deletion must be performed manually

systemctl daemon-reload               # invoke this after updating a unit
systemctl daemon-reexec               # required to reload Sytemd's own configuration (e.g. changes to `/etc/systemd/system.conf`)

systemctl list-units                  # active loaded units
systemctl list-unit-files [$pattern]  # all units, with their states; glob pattern
systemctl [--user] list-timers        # list timers
systemctl --failed                    # show units that failed to start

# Query the status of a unit; prints the status.
# The exit status is not usable; force it to true and use the output.
# `is-active` returns: `active`, `failed`, ...
# WATCH OUT!! (Wrong service name) or (not active status) will return a false state.
#
systemctl is-(active|enabled|failed) $service

# Check if a unit exists (.service is required):
#
systemctl list-unit-files --user pipewire.service | grep -q '1 unit files listed'
# Alternative:
systemctl status myunit > /dev/null 2>&1
[[ $? -eq 4 ]] && echo "unit doesn't exist"  # must run this in a separate command (due to $?)
```

In order to check if a program is running from a Systemd context, check if `INVOCATION_ID` is set.

If a unit is stored with ill-formed content, it won't be found by commands, which print instead a confusing `No files found` message.

The `--user` is for units belonging to the user (eg. transient timers).

### Uninstall (remove) a unit

Systemd doesn't provide any mean to uninstall units; it must be done manually:

```sh
systemctl disable --now $service
rm /etc/systemd/system/$service.service
systemctl daemon-reload
```

### Disabling and dependencies

A static service can't be manually enabled or disabled; use mask for that.

WATCH OUT! If a service is disabled, but other ones depend on it, it will be started; in this case, find and disable the dependending units:

```sh
systemctl list-dependencies --reverse snapd.socket
# ... now disable the dependencies ...
```

### System commands and misc options

```sh
systemctl poweroff                # shutdown the system
systemctl suspend                 # suspend the system

systemctl --nopager               # disable pager (keypress)
```

### User units

It's best to configure a unit as user unit, rather than setting `User`/`Group`, since this caused odd behaviors in some cases.

Systemd can be configured on a per-user basis, using the `--user` option (see for example, the above `--list-timers`). User units are stored under `$HOME/.config/systemd/user`

IMPORTANT!

- `WantedBy` must be `default.target`, not `multi-user.target`
- user units require "lingering" enabled in order to start at boot
  - also, without lingering, the services are terminated on user logoff
  - enable via `loginctl enable-linger`, or manually create `/var/lib/systemd/linger/$USER`
  - lingering can be checked (also) via `loginctl showloginctl show-user $user --property=Linger`

In order to manage a user unit via root user, add the option `--machine=$user@`, e.g. `sudo systemctl restart --user --machine=myuser@ myservice`, otherwise, it won't work on non-login shells; alternatively, set the env vars:

```sh
export XDG_RUNTIME_DIR=/run/user/$UID
export DBUS_SESSION_BUS_ADDRESS=$XDG_RUNTIME_DIR/bus
```

## journalctl

```sh
journalctl -b -xp 3 [-k]            # view log for current [b]oot; with e[x]tra information, only errors (`-p 3` = level);
                                    # only [k]ernel messages; can also add `-u` for filtering a certain service
journalctl -b -1 ...                # `-1` = previous boot (!)
journalctl -f -u $service           # [-f|--follow]; [-u|--unit] log; use `page-end` to go to end
journalctl --vacuum-time=1d         # clean systemd journal (/var/log/journal)

# Purge the logs:
#
sudo journalctl --rotate
sudo journalctl --vacuum-time=1s
```

Display the log of the last run only:

```sh
# WATCH OUT! In scripts, it's better to check first if the result of internal `show` is blank (e.g.
# when a unit didn't detect a program restart, etc.), because it won't cause the external command
# to fail.
#
journalctl _SYSTEMD_INVOCATION_ID="$(systemctl show --value -p InvocationID $unit)"
```

Limit the max use:

```sh
mkdir -p /etc/systemd/journald.conf.d
cat > /etc/systemd/journald.conf.d/00-journal-size.conf <<'CFG'
[Journal]
SystemMaxUse=128M
CFG

# Restart the journal service (won't purge the existing journal)
systemctl kill --kill-who=main --signal=SIGUSR2 systemd-journald

# Purge the existing journal
journalctl --vacuum-size=128M
```

## Configuring a service unit

See:

- https://www.man7.org/linux/man-pages/man5/systemd.exec.5.html
- https://www.freedesktop.org/software/systemd/man/systemd.service.html
- https://www.freedesktop.org/software/systemd/man/systemd.kill.html

General location of systemd units (not the only one): `/lib/systemd/system/$name.service`.

Almost all the entries are optional. Escaping is not standard; in order to escape, use `systemd-escape`.

```sh
# If entries need escaping, use double quotes, but not slashes. In at least the `ExecStart` entry, if,
# for example, a space is escaped with a slash, both the slash and the space will be part of the string
# (!).
#
SYSTEMD_EDITOR="tee" systemctl edit --force --full myservice << 'UNIT'
[Unit]
Description=My Service

# `Requires` states only dependency; `After` mandates ordering.
#
Requires=network.target
# Multiple services are separated by a space.
#
After=network.target

# Conditional unit execution; use `!` to negate
#
ConditionPathExists=!/path/to/file

# Example of waiting for a network device to be online.
#
# After=sys-subsystem-net-devices-wlan0.device
# BindsTo=sys-subsystem-net-devices-wlan0.device

[Service]
# If ExecStart is specified (and `BusName` not specified), `simple` is implicit.
#
# Use `forking` when the process forks (e.g. nmon). See the `Notify` section below for automatic forking.
# WATCH OUT! If a `forking` unit invokes a script that invokes another program, don't forget to background
# the program!
#
# Generally prefer `oneshot` to `simple` (see https://stackoverflow.com/a/39050387 and https://trstringer.com/simple-vs-oneshot-systemd-service/#summary).
# WATCH OUT!! These types are identical; for example, Mailcatcher worked only in `simple` mode, with the `-f` executable parameter.
#
Type=oneshot

# Outputs are overwritten, unless the `append` file mode is used (e.g. `append:/path/to/my-service.log`);
# this is available from v240, though (Bionic provides v237).
#
StandardOutput=syslog        # default: journal
StandardError=syslog         # ^^
SyslogIdentifier=my-service  # defaults to the name of the executed process
SyslogFacility=daemon        # default; use `syslog` to send there (and then filter via log service)

# Owner defaults to root, but if user-specific vars like $HOME needs to be set, then set `User=root`.
# See [User Units](#user-units) section for guidelines on this.
#
User=app
Group=app

WorkingDirectory=/path/to/base_dir

Environment="BUNDLE_GEMFILE=/path/to/Gemfile" "RAILS_ENV=my_rail_env"
# This is needed if one wants to interact with the desktop, e.g. use `notify-send`.
# WATCH OUT!! Desktop notifications are sent asynchronously; if the script exits immediately after executing
# `notify-send` (or so), the notification won't be displayed!
Environment="DISPLAY=:0" "XAUTHORITY=/home/myuser/.Xauthority"

# In other to extend $PATH, use something like:
#
#     ExecStart=/bin/bash -c 'PATH=/new/path:$PATH exec /bin/mycmd'
#
# WATCH OUT! The PATH needs to be absolute; `WorkingDirectory` does not make relative paths work.
#
ExecStart=/usr/bin/bundle exec my_command
ExecReload=/bin/kill -HUP $MAINPID

# Execute a command before the killing procedure.
# See https://www.freedesktop.org/software/systemd/man/systemd.service.html and https://www.freedesktop.org/software/systemd/man/systemd.kill.html.
#
ExecStop=/usr/bin/bundle exec my_stop_command

# Killing procedure; by default follows `KillSignal=`, and then sends SIGKILL if necessary.
# Better not to specify, unless required.
#
KillMode=control-group     # default; other options: `mixed`, `process`, `none`.

# By default, SIGTERM; always followed by SIGCONT.
#
KillSignal=SIGTERM

# Other options: `no` (default), `on-failure`, ...
#
Restart=always

[Install]
# General-purpose setting.
#
WantedBy=multi-user.target
UNIT

# now enable and start
```

### Notify unit type

If one wants Systemd to fork the process, then the best is `notify` (see https://askubuntu.com/q/1120023); this requires the program to report success via `systemd-notify --ready`, otherwise, Systemd will kill
the program after timeout.

`NotifyAccess=` can be left to the default, if the program process itself sends the notification.

References: [Type=](https://www.freedesktop.org/software/systemd/man/systemd.service.html#Type=) and [NotifyAccess=](https://www.freedesktop.org/software/systemd/man/systemd.service.html#NotifyAccess=).

### Timeouts/killing

By default, `TERM` is sent, then `KILL` on timeout.

Use `TimeoutStartSec=` and `TimeoutStopSec=` in order to set the timeouts (default: 90"); can also use `TimeoutSec=` as shorthand for setting both.

`KillSignal=SIGQUIT`: specify what to send on the (first) stop

In order to specify what to send after the timeout:

```
SendSIGKILL=no
ExecStopPost=/bin/kill -HUP $MAINPID
```

### Overrides

Overrides are created via `systemctl edit` (without `--full`); they create the file+dir `/etc/systemd/system/myservice.$type.d/override.conf`.

At least certain entries are additive, e.g. `OnCalendar`; in order to clear the existing entries, set an empty one (eg. `OnCalendar=`) and the new one.

### Hooks

If the `Type=oneshot`, `[Service] ExecStartPost` can be used to specify a command to execute on successful exit; the entry is additive.

## Timers (scheduled events)

Reference: https://www.freedesktop.org/software/systemd/man/systemd.timer.html.

Timers are represented by `.timer` files accompanying the main service (a service is required).

See [overrides](#overrides) for overriding notes.

Example:

```sh
SYSTEMD_EDITOR=tee systemctl edit --full myservice.timer << UNIT
[Unit]
Description=myservice timer

[Timer]
OnCalendar=*-*-* 0/5:*:*

# Other options/configurations:
#
# AccuracySec=1s          # defaults to 1m
# OnCalendar=*-*-* 6:00
# RandomizedDelaySec=30m  # defaults o 0
# Persistent=true         # stores the last run on disk, so jobs are not missed; requires OnCalendar

[Install]
WantedBy=timers.target
UNIT

systemctl enable myservice.timer
systemctl start myservice.timer
```

### Systemd time format

Reference: https://www.freedesktop.org/software/systemd/man/systemd.time.html.

Examples:

```
  Sat,Thu,Mon..Wed,Sat..Sun → Mon..Thu,Sat,Sun *-*-* 00:00:00
      Mon,Sun 12-*-* 2,1:23 → Mon,Sun 2012-*-* 01,02:23:00
                    Wed *-1 → Wed *-*-01 00:00:00
           Wed..Wed,Wed *-1 → Wed *-*-01 00:00:00
                 Wed, 17:48 → Wed *-*-* 17:48:00
Wed..Sat,Tue 12-10-15 1:2:3 → Tue..Sat 2012-10-15 01:02:03
                *-*-7 0:0:0 → *-*-07 00:00:00
                      10-15 → *-10-15 00:00:00
        monday *-12-* 17:00 → Mon *-12-* 17:00:00
  Mon,Fri *-*-3,1,2 *:30:45 → Mon,Fri *-*-01,02,03 *:30:45
       12,14,13,12:20,10,30 → *-*-* 12,13,14:10,20,30:00
            12..14:10,20,30 → *-*-* 12..14:10,20,30:00
  mon,fri *-1/2-1,3 *:30:45 → Mon,Fri *-01/2-01,03 *:30:45
             03-05 08:05:40 → *-03-05 08:05:40
                   08:05:40 → *-*-* 08:05:40
                      05:40 → *-*-* 05:40:00
     Sat,Sun 12-05 08:05:40 → Sat,Sun *-12-05 08:05:40
           Sat,Sun 08:05:40 → Sat,Sun *-*-* 08:05:40
           2003-03-05 05:40 → 2003-03-05 05:40:00
 05:40:23.4200004/3.1700005 → *-*-* 05:40:23.420000/3.170001
             2003-02..04-05 → 2003-02..04-05 00:00:00
       2003-03-05 05:40 UTC → 2003-03-05 05:40:00 UTC
                 2003-03-05 → 2003-03-05 00:00:00
                      03-05 → *-03-05 00:00:00
                     hourly → *-*-* *:00:00
                      daily → *-*-* 00:00:00
                  daily UTC → *-*-* 00:00:00 UTC
                    monthly → *-*-01 00:00:00
                     weekly → Mon *-*-* 00:00:00
    weekly Pacific/Auckland → Mon *-*-* 00:00:00 Pacific/Auckland
                     yearly → *-01-01 00:00:00
                   annually → *-01-01 00:00:00
                      *:2/3 → *-*-* *:02/3:00
```

### Once-off (transient)

Once-off events can be scheduled; mind that they're not second-accurate.

```sh
# Execute after certain interval, with output.
#
# `--unit`: optional name; without, a long random name is set
#
# WATCH OUT! On some systems, `--on-active` caused, for very mysterious reasons, the timer to fire
# with a long delay, seemingly because it was resetting. The alternative, in that case, has been to
# run via fixed time: `--on-calendar=$(date +'%R' -d '+$delay minutes')`
#
$ systemd-run --user --on-active=10min --unit=test-example /bin/systemctl suspend
Running timer as unit: test-example.timer
Will run service as unit: test-example.service

# Execute at given time, as root.
#
$ systemd-run --on-calendar=09:12 /bin/systemctl suspend

# Cancel. Don't forget the `.timer`!!!!
#
$ systemctl --user stop test-example.timer
```

## Event triggers (execute program on resume, etc.)

Run a script on suspend/hibernate/thaw/resume (the change applies immediately):

```sh
# The standard `bin` paths are in the path during execution, so there's no need to use full paths.
#
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
