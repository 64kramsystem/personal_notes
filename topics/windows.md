# Windows

- [Windows](#windows)
  - [Screen Capture](#screen-capture)
  - [License operations](#license-operations)
  - [Exit on Windows 11 first boot installation (OOBE)](#exit-on-windows-11-first-boot-installation-oobe)

## Screen Capture

- Program: Snipping tool
- Shortcut: Win + Shift + S

## License operations

Transfer key:

```sh
slmgr /dlv # display (partial) license info

slmgr /upk  # uninstall key (need to do this and the below)
slmgr /cpky # remove key from registry

slmgr /ipk  # install key; better done via GUI, which handles more cases
```

Use a program like ProduKey (not a malware!) to display the key.

## Exit on Windows 11 first boot installation (OOBE)

- `Shift+F10`
- `shutdown /s /t 0`

Don't forget to hold the `Fn` key, if they keyboard disables function keys by default!
