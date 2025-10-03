# Windows

- [Windows](#windows)
  - [Licensing](#licensing)
    - [License operations](#license-operations)
  - [Installation](#installation)
    - [OOBE: Open Command Prompt](#oobe-open-command-prompt)
    - [OOBE: Exit on Windows 11 first boot installation](#oobe-exit-on-windows-11-first-boot-installation)
    - [Workaround account login requirement on first setup / Wifi requirement](#workaround-account-login-requirement-on-first-setup--wifi-requirement)
    - [Mass-install drivers from multiple subdirectories](#mass-install-drivers-from-multiple-subdirectories)
  - [Setting locations](#setting-locations)
  - [WSL](#wsl)
    - [Symlinks/Junctions](#symlinksjunctions)
    - [Mount ext4 flash keys](#mount-ext4-flash-keys)
  - [Clipboard management](#clipboard-management)
  - [GUI/Notifications](#guinotifications)
  - [Variables](#variables)
  - [Processes](#processes)
  - [Security](#security)
    - [Run an elevated command from non-elevated WSL](#run-an-elevated-command-from-non-elevated-wsl)
    - [WSL: Cache SSH keys password](#wsl-cache-ssh-keys-password)
    - [Enable PIN to boot (encrypted disk)](#enable-pin-to-boot-encrypted-disk)
  - [Taskbar pinned items programmatic manipulation](#taskbar-pinned-items-programmatic-manipulation)
  - [Services](#services)
  - [Drives](#drives)
  - [Screen capture](#screen-capture)
  - [Packages management](#packages-management)
  - [Power management](#power-management)
  - [Other fixes](#other-fixes)
    - [Steam icons not loading the game](#steam-icons-not-loading-the-game)

## Licensing

WATCH OUT!!! Can't (officially) use a license key to upgrade Windows (11) from Home to Pro

Windows 11 Home to Pro upgrade:

- Remove the license (`slmgr /upk` and `slmgr /cpky`)
- Disconnect from internet
- Set the Pro generic key: `VK7JG-NPHTM-C97JM-9MPGT-3V66T`
- Wait for upgrade to complete, and reconnect to internet
- Set the key

### License operations

Transfer key:

```sh
slmgr /dlv # display (partial) license info

slmgr /upk  # uninstall key (need to do this and the below)
slmgr /cpky # remove key from registry

slmgr /ipk  # install key; better done via GUI, which handles more cases
```

Use a program like ProduKey (not a malware!) to display the key.

## Installation

### OOBE: Open Command Prompt

Tap `Shift + F10`.

### OOBE: Exit on Windows 11 first boot installation

Open a command prompt, and execute `shutdown /s /t 0`.

Don't forget to hold the `Fn` key, if they keyboard disables function keys by default!

### Workaround account login requirement on first setup / Wifi requirement

To skip the wifi step, run in the command prompt:

    OOBE\BypassNRO.cmd

If wifi was setup by accident, run the following in the command prompt, then perform the wifi skip procedure:

    netsh wlan show profiles
    netsh wlan delete profile name="ProfileName"

### Mass-install drivers from multiple subdirectories

Go to the parent directory, and run `pnputil /add-driver *.inf /subdirs /install`.

## Setting locations

- `Change what closing the lid does`: From `Control Panel` -> `Hardware …` -> `Power options`

## WSL

```sh
# Execute a Windows program from WSL (!!!).
# The path is the current.
#
cmd.exe /d $mycommand
powershell.exe -Command $mycommand

# Open file with Windows related program (wslu package)
#
wslview $file

# Convert path to/from Windows.
#
# WATCH OUT! As of Nov/2024, it resolves symlinks, so if one wants to keep the them, it must convert manually:
#
#     echo "$path" | perl -pe 's|/mnt/c/Users/([^/]+)|C:/Users/\1|' | perl -pe 's|/|\\|g'
#
wslpath $from_path $to_path

# Mount a drive connected to Windows (not done automatically).
#
mount -o drvfs $letter: $path
```

### Symlinks/Junctions

- Directory symlinks can be of two types: dirs (`/D`) and junctions (`/J`); the latter are more compatible, but the have minor restrictions.
- In order to create Windows junctions in the host from WSL, use `cmd.exe /c mklink` (don't forget to quote).
- symlink targets are always absolute, so if a relative one is passed to `mklink`, it's converted to absolute
- `mlink` interprets relative targets as relative to the current directory, not the target one
  - for this reason, in scripts, it's much easier to `cd` into the target directory
- WATCH OUT!!
  - Must run the terminal/prompt in an Admin WSL session (otherwise, in the past, it'd fail in inconsistent and confusing ways)
    - See [Run an elevated command from non-elevated WSL](#run-an-elevated-command-from-non-elevated-wsl)
  - `mklink.exe` doesn't exist; `mklink` is a built-in command!!

In order rebuild symlinks, use this script (takes link as `$1`):

```sh
#!/bin/bash

link_with_path=$1
# Need to change the Unix link path, since we've changed the current directory.
# As advantage, we don't need to keep separate copies for Windows and Unix, since there is no path.
link=$(basename "$link_with_path")

cd "$(dirname "$link_with_path")"

# Don't use `wslpath`, which resolves the symlink; additionally, we want to maintain symlinks in
# intermediate paths.
new_target=$(
  readlink "$link" \
  | perl -pe 's|^/mnt/([a-z]+)|\U\1:|' \
  | perl -pe 's|/|\\\\|g'
)

[[ -d "$link" ]] && mklink_opt=(/J) || mklink_opt=()
echo "Rebuilding${mklink_opt[*]} ($link_with_path) -> ($new_target)"

rm "$link"
# Ignore the double backslashes in the output, it's just a `cmd.exe` quirk.
cmd.exe /c mklink "${mklink_opt[@]}" "$link" "$new_target"
```

Formerly, a smarter attempt, detecting invalid symlinks, has been made, however there were some problems (see history).

### Mount ext4 flash keys

(can also just use specific programs)

WATCH OUT!:

- as `v5.15.153.1-microsoft-standard-WSL2`, it requires recompilation with `USB Storage` enabled
- requires the `usbipd` program

Under Windows:

```
>usbipd list

Connected:
BUSID  VID:PID    DEVICE                                                        STATE
2-5    0781:5588  USB Mass Storage Device                                       Not shared
[...]

>usbipd bind --busid=2-5
>usbipd attach --wsl --busid=2-5
```

Under WSL:

```
$ lsblk # will show the disk, which can mount
```

## Clipboard management

```sh
clip.exe                                # Pastes stdin into the clipboard
powershell.exe -command `Get-Clipboard` # Prints clipboard content to stdout
```

## GUI/Notifications

Display a dialog:

```
# From WSL
message=${message//\'/''}
title=${title//\'/''}
powershell.exe -Command "
  [void][System.Reflection.Assembly]::LoadWithPartialName('System.Windows.Forms');
  [System.Windows.Forms.MessageBox]::Show(\"$message\", \"$title\")
"
```

## Variables

- Home:            `%USERPROFILE%`
- AppData\Roaming: `%APPDATA%`

## Processes

Kill processes programmatically:

```sh
# Returns 0 if the process was found; 1 otherwise.
#
tasklist | findstr thunderbird.exe && taskkill /IM thunderbird.exe /F
```

## Security

Check if running as admin (No errors if running as Windows Admin): `net session`

### Run an elevated command from non-elevated WSL

```sh
# Don't forget the link full path, otherwise, the current dir is System32.
#
powershell.exe -Command "Start-Process cmd.exe -ArgumentList '/c mklink $(wslpath -w .)\\link.txt X:\\path\\to\\foo.txt' -Verb RunAs"
```

### WSL: Cache SSH keys password

```sh
apt install keychain

cat >> $shell_init_script << SH
/usr/bin/keychain --nogui $keyfile_full_path
source $HOME/.keychain/$(hostname)-sh
SH
```

### Enable PIN to boot (encrypted disk)

In order to make the option visible in the Bitlocker management:

- `gpedit.msc`
  - `Computer Configuration` -> `Administrative Templates` -> `Windows Components` -> `BitLocker Drive Encryption` -> `Operating System Drives`
  - open `Require additional authentication at startup` (and according options)
    - Enable
    - `Configure TPM startup PIN:` -> `Require startup PIN with TPM`
    - Disallow the other options

## Taskbar pinned items programmatic manipulation

The files location is `%APPDATA%\Microsoft\Internet Explorer\Quick Launch\User Pinned\TaskBar`, however, trying to populate it with pregenerated files, won't work.
In order to restart the taskbar, use: `PS> Stop-Process -Name explorer`

## Services

```powershell
# List running services:

Get-Service |
  Where-Object { $_.Status -eq 'Running' } |
  Format-Table Name,DisplayName -AutoSize -Wrap

# Disable list of services:

$disable = @('Browser', '...')

foreach ($svc in $disable) {
  Stop-Service $svc -ErrorAction SilentlyContinue -Force
  Set-Service  $svc -StartupType Disabled
  Write-Host "$svc disabled"
}
```

## Drives

Unmount a drive (**must run as admin**):

```sh
cmd.exe /c "mountvol D: /D"
```

## Screen capture

- Program: "Snipping tool"
- Shortcut: `Win + Shift + s`

## Packages management

Windows has a centralized repository, which can be used via the cmdline program `winget`.
It doesn't use the Windows Store, but it handles also programs downloaded via it.

```sh
winget.exe search $pattern$ # case insensitive subpattern

winget.exe install $package_name
winget.exe install --exact --id $package_id # [exact] match; [id] match

winget.exe source update
winget.exe upgrade --all

# Winget listing truncates the output based on the console size (🤦).
# In order to workaround, decrease the font size, and run via PS, **without grepping**.

powershell.exe -command "winget list"
```

## Power management

Display remaining battery charge:

```
WMIC PATH Win32_Battery Get EstimatedChargeRemaining
```

## Other fixes

### Steam icons not loading the game

Setting: `Choose defaults by file type` → Set `.URL` to `Internet Browser`
