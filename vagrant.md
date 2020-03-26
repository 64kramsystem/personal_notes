## Table of contents

- [Boxes](#boxes)
- [VirtualBox Guest tools issues](#virtualbox-guest-tools-issues)

## Boxes

Create (and import) a box from a virtual machine.  
`--force`: overwrite the existing box, if any.  
Vagrant will automatically convert the disks to VMDK when packaging a box.

```sh
vagrant package <vagrant_machine_name>
vagrant box add --name <box_name> [--force] <box_file_name>
```

## VirtualBox Guest tools issues

Vagrant: fix mismatching guest tools version.  
Reload the machine after this; use  `VBoxManage list vms` for gathering UUIDs.

```sh
VBoxManage guestproperty set <machine_uuid> /VirtualBox/GuestAdd/Version
```

Note that his may not work; if for example, the host reports `5.0.16 r105871` and the guest reports `5.0.16`, then check that the guest has the right version, then manually set it:

```sh
VBoxManage guestproperty set <machine_uuid> /VirtualBox/GuestAdd/Version <short_version>
```
