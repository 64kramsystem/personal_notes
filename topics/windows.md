# Windows

- [Windows](#windows)
  - [Licensing](#licensing)
    - [License operations](#license-operations)
  - [Installation](#installation)
    - [OOBE: Open Command Prompt](#oobe-open-command-prompt)
    - [OOBE: Exit on Windows 11 first boot installation](#oobe-exit-on-windows-11-first-boot-installation)
    - [Workaround account login requirement on first setup / Wifi requirement](#workaround-account-login-requirement-on-first-setup--wifi-requirement)
    - [Mass-install drivers from multiple subdirectories](#mass-install-drivers-from-multiple-subdirectories)
  - [WSL](#wsl)
    - [Symlinks/Junctions](#symlinksjunctions)
    - [Mount ext4 flash keys](#mount-ext4-flash-keys)
  - [Clipboard management](#clipboard-management)
  - [Variables](#variables)
  - [Processes](#processes)
  - [Security](#security)
    - [WSL: Cache SSH keys password](#wsl-cache-ssh-keys-password)
    - [Enable PIN to boot (encrypted disk)](#enable-pin-to-boot-encrypted-disk)
  - [Taskbar pinned items programmatic manipulation](#taskbar-pinned-items-programmatic-manipulation)
  - [Drives](#drives)
  - [Screen capture](#screen-capture)
  - [Packages management](#packages-management)

## Licensing

WATCH OUT!!! Can't (officially) use an OEM license key to upgrade Windows (11) from Home to Pro

Windows 11 Home to Pro upgrade:

- Remove the license
- Disconnect from internet
- Set the Pro generic key: `VK7JG-NPHTM-C97JM-9MPGT-3V66T`
- Wait for upgrade to complete, and reconnect to internet
- Set the OEM key

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

WATCH OUT!! In order to create Windows junctions in the host from WSL, use `cmd.exe /c mklink` in an Admin WSL session (don't forget to quote!); any other approach fails in inconsistent and confusing ways.

In order rebuild symlinks, run:

```sh
# Should also work with links including relative paths (`../`), but it hasn't been concretely tested.
#
# For simplicity, we don't use `find -L . -xtype l`, which is slower.
#
find "${config_paths[@]}" -type l \
  | xargs -I {} bash -c '
    target=$(
      readlink "{}" \
      | perl -pe "s|/mnt/c/Users/([^/]+)|C:/Users/\\1|" \
      | perl -pe "s|/|\\\\|g"
    )
    link=$(echo {} | perl -pe "s|/|\\\\|g")

    [[ -d "{}" ]] && dir_option=(/d) || dir_option=()
    echo "Rebuilding${dir_option[@]} ($link) -> ($target)"
    rm "{}"
    cmd.exe /c mklink "${dir_option[@]}" "$link" "$target"
  '
```

Formerly, a smarter attempt, detecting invalid symlink, has been made, however there were some problems:

```sh
# - The PS command prints nothing if symlink is invalid
#   - WATCH OUT! not fully effective - file symlinks to a dir are invalid, but they pass the test
#   - the test works as intended also if WSL is running as Admin.
# - Symlinks with level change (ie. "../foo") actually work fine on Windows, so won't be triggered
#   by `Get-Item.Target`.
#
find . -type l -printf '%P\n' \
  | xargs -I {} bash -c '
      if [[ -z $(powershell.exe -Command "(Get-Item \"{}\").Target") ]]; then
        target=$(
          readlink "{}" \
          | perl -pe "s|/mnt/c/Users/([^/]+)|C:/Users/\\1|" \
          | perl -pe "s|/|\\\\|g"
        )
        link=$(echo {} | perl -pe "s|/|\\\\|g")

        [[ -d "{}" ]] && dir_option=(/d) || dir_option=()
        rm "{}"
        cmd.exe /c mklink "${dir_option[@]}" "$link" "$target"

        [[ -z $(powershell.exe -Command "(Get-Item \"{}\").Target") ]] && echo "Link $link is still invalid!"
      fi
    '
```

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

## Drives

Unmount a drive:

```sh
powershell.exe -Command 'Dismount-Volume -DriveLetter D -Confirm:$false'
```

## Screen capture

- Program: "Snipping tool"
- Shortcut: `Win + Shift + s`

## Packages management

Windows has a centralized repository, which can be used via the cmdline program `winget`.
It doesn't use the Windows Store, but it handles also programs downloaded via it.

```sh
winget.exe install $package_name
winget.exe install --exact --id $package_id # [exact] match; [id] match

winget.exe source update
winget.exe upgrade --all

# Winget listing truncates the output based on the console size (🤦).
# In order to workaround, decrease the font size, and run via PS, **without grepping**.

powershell.exe -command "winget list"
```