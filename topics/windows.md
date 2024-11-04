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
  - [Security](#security)
    - [Enable PIN to boot](#enable-pin-to-boot)
  - [Taskbar pinned items programmatic manipulation](#taskbar-pinned-items-programmatic-manipulation)
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

# Mount a drive connected to Windows (not done automatically).
#
mount -o drvfs $letter: $path
```

### Symlinks/Junctions

WATCH OUT!! In order to create Windows junctions in the host from WSL, use `cmd.exe /c mklink` in an Admin WSL session (don't forget to quote!); any other approach fails in inconsistent and confusing ways.

In order to check if a symlink is valid, run in PowerShell (read comment carefully):

```sh
# Search the inconsistently invalid symlinks (without recursively following), and replace them with
# junctions.
#
# - The PS command prints nothing if symlink is invalid
#   - WATCH OUT! not fully effective - file symlinks to a dir are invalid, but they pass the test
#   - the test works as intended also if WSL is running as Admin.
# - Symlinks with level change (ie. "../foo") actually work fine on Windows, so won't be triggered
#   by `Get-Item.Target`.
#
# For simplicity, we don't use `find -L . -xtype l`, which is slower.
#
find . -type l -printf '%P\n' \
  | xargs -I {} bash -c '
    if [[ -z $(powershell.exe -Command "(Get-Item \"{}\").Target") ]]; then
      target=$(readlink "{}" | perl -pe "s|/home/myuser|C:\\\\Users\\\\Myuser|g" | perl -pe "s|/|\\\\|g")
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

## Security

Check if running as admin (No errors if running as Windows Admin): `net session`

### Enable PIN to boot

In order to make the option visible in the Bitlocker management:

- open `gpedit.msc`
- `Computer Configuration` -> `Administrative Templates` -> `Windows Components` -> `BitLocker Drive Encryption` -> `Operating System Drives`
- Set `Require additional authentication at startup` (and according options)

## Taskbar pinned items programmatic manipulation

The files location is `%APPDATA%\Microsoft\Internet Explorer\Quick Launch\User Pinned\TaskBar`, however, trying to populate it with pregenerated files, won't work.
In order to restart the taskbar, use: `PS> Stop-Process -Name explorer`

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