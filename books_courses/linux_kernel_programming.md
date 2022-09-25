# Linux Kernel Programming

- [Linux Kernel Programming](#linux-kernel-programming)
  - [Useful infos/tools](#useful-infostools)
  - [Configuration/Building](#configurationbuilding)
    - [Built objects](#built-objects)
  - [Initramfs](#initramfs)

## Useful infos/tools

Useful targets:

- `tags`     : generates `ctags` metadata, useful to navigate the codebase
- `cscope`   : similar to `tags`, for the `cscope` program

## Configuration/Building

Configuration entry values:

- `y`: integrate in the kernel
- `m`: build as module (`.ko`)
- `n`: don't build

Related targets:

- `oldconfig`   : use the running kernel options (plus any new one)
- `defconfig`   : use defaults of the supplied ARCH
- `olddefconfig`: like `defconfig`, but sets new symbols to their defaults

- `mrproper` : cleans more stuff, including config
- `distclean`: most thorough cleanup

- `all`    : build all targets
- `vmlinux`: bare kernel
- `modules`: modules
- `bzImage`: compressed kernel image (arch specific target)

- `install`        : install the kernel on the system
  - the `vmlinuz` file is the compressed (gzip) image
- `modules_install`: install the modules in the current kernel modules dir (`lib/modules/$(uname -r)`) or under `$INSTALL_MOD_PATH`
  - runs `depmod`, which generates and stores the modules dependency tree

### Built objects

- `vmlinux`   : uncompressed image (for debug; includes debug info)
- `System.map`: symbols addresses
- `bzImage`   : compressed image (output dir: `arch/$ARCH/boot`)

## Initramfs

Boot sequence (legacy):

- stage 1 bootloader (BIOS)
- stage 2 bootloader (first sector)
- stage 3 bootloader (GRUB)
- Linux inits the hw/sw
- initramfs in-memory loading
- initramfs mount
- `initrd` startup scripts now run (e.g. loading modules)
- initramfs unmount
- `/sbin/init` loading

Tools:

- `mkinitramfs`:       generates and installes the initramfs file
- `lsinitramfs $file`: list the content of an initramfs file