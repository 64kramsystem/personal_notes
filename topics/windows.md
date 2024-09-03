# Windows

- [Windows](#windows)
  - [Licensing](#licensing)
    - [License operations](#license-operations)
  - [Installation (OOBE): Open Command Prompt](#installation-oobe-open-command-prompt)
  - [Screen Capture](#screen-capture)
  - [Exit on Windows 11 first boot installation (OOBE)](#exit-on-windows-11-first-boot-installation-oobe)
  - [Workaround account login requirement on first setup / Wifi requirement](#workaround-account-login-requirement-on-first-setup--wifi-requirement)

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

## Installation (OOBE): Open Command Prompt

Tap `Shift + F10`.

## Screen Capture

- Program: Snipping tool
- Shortcut: Win + Shift + S

## Exit on Windows 11 first boot installation (OOBE)

- `Shift+F10`
- `shutdown /s /t 0`

Don't forget to hold the `Fn` key, if they keyboard disables function keys by default!

## Workaround account login requirement on first setup / Wifi requirement

To skip the wifi step, run in the command prompt:

    OOBE\BypassNRO.cmd

If wifi was setup by accident, run the following in the command prompt, then perform the wifi skip procedure:

    netsh wlan show profiles
    netsh wlan delete profile name="ProfileName"
