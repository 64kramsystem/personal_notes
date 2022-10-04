# Linux Kernel Programming

- [Linux Kernel Programming](#linux-kernel-programming)
  - [Useful infos/tools](#useful-infostools)
  - [Configuration/Building](#configurationbuilding)
    - [Built objects](#built-objects)
    - [Cross compilation](#cross-compilation)
    - [Symbols for debugging](#symbols-for-debugging)
  - [Initramfs](#initramfs)
  - [Kernel architecture overview](#kernel-architecture-overview)
  - [LKM Framework/Modules](#lkm-frameworkmodules)
    - [Module tools/general info](#module-toolsgeneral-info)
    - [Sample module](#sample-module)
    - [Module Makefile](#module-makefile)
    - [Modules interop](#modules-interop)
  - [Logging](#logging)
    - [Console levels](#console-levels)
    - [Rate limiting](#rate-limiting)
    - [Macros, data types, printk specifiers](#macros-data-types-printk-specifiers)
  - [PAG b239](#pag-b239)

## Useful infos/tools

Useful targets:

- `tags`     : generates `ctags` metadata, useful to navigate the codebase
- `cscope`   : similar to `tags`, for the `cscope` program

## Configuration/Building

The running configuration is in `/boot/config-$(uname -r)`.

Configuration entry values:

- `y`: integrate in the kernel ("core")
- `m`: build as module (`.ko`)
- `n`: don't build

Related targets:

- `oldconfig`   : use the existing configuration, and ask new symbols
- `olddefconfig`: like `defconfig`, but sets new symbols to their defaults
- `defconfig`   : use defaults of the supplied ARCH
- `localmodconfig`: update config: disable currently not loaded modules
- `localyesconfig`: update config: convert local modules to core

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

### Cross compilation

Basic cross compilation; for more advanced use-cases, see [here](https://www.raspberrypi.org/documentation/linux/kernel/building.md).

```sh
apt install -y ​crossbuild-essential-armhf # 32bit, or `​crossbuild-essential-arm64` (64bit)

# - `kernel7`/`arm`       : 32-bit kernel/arch
# - `arm-linux-gnueabihf-`: tools prefix
# - `bcm2709_defconfig`   : board (see [here](https://raspberrypi.stackexchange.com/q/840))
#
KERNEL=kernel7 make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bcm2709_defconfig
```

See [module makefile](#module-makefile) for module-specific details.

### Symbols for debugging

Useful symbols for debugging (`Kernel Hacking` submenu)

- `CONFIG_DEBUG_INFO`
- `CONFIG_DEBUG_FS` (the debugfs pseudo filesystem)
- `CONFIG_MAGIC_SYSRQ` (the Magic SysRq hotkeys feature)
- `CONFIG_DEBUG_KERNEL`
- `CONFIG_DEBUG_MISC`
- Memory debugging:
  - `CONFIG_SLUB_DEBUG`
  - `CONFIG_DEBUG_MEMORY_INIT`
  - `CONFIG_KASAN` (some archs only)
- `CONFIG_DEBUG_SHIRQ`
- `CONFIG_SCHED_STACK_END_CHECK`
- Lock debugging:
  - `CONFIG_PROVE_LOCKING` (selects others)
  - `CONFIG_LOCK_STAT`
- `CONFIG_DEBUG_ATOMIC_SLEEP`
- `CONFIG_STACKTRACE`
- `CONFIG_DEBUG_BUGVERBOSE`
- `CONFIG_FTRACE` (within its sub-menu, select at least a couple of "tracers")
- `CONFIG_BUG_ON_DATA_CORRUPTION`
- `CONFIG_KGDB` (kernel GDB; optional)
- `CONFIG_UBSAN`
- `CONFIG_EARLY_PRINTK`
- `CONFIG_DEBUG_BOOT_PARAMS`
- `CONFIG_UNWINDER_FRAME_POINTER` (selects `FRAME_POINTER` and `CONFIG_STACK_VALIDATION`)

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

## Kernel architecture overview

There are two privilege levels/modes - kernel- and user- space. CPUs have more levels (x64: 4 "ring levels", ARM: <= 7 "execution modes").

All usermode apps are linked to libc (but kernel mode doesn't have access to it).

Address spaces are virtual; they're mapped to physical ones in pages, using the "master kernel paging table".

## LKM Framework/Modules

LKM is used to develop modules, although not all the kernel functionalities are provided (e.g scheduling).

Kernel modules live in kernel space, and run with kernel privileges.

### Module tools/general info

- `modinfo /path/to/module.ko`: generic info
- `lsmod`: list modules; 4th column is depending modules; 3rd col includes processes using a module, so it can be higher than the depending ones
- `rmmod [-r] $mod1 ... $modN`: modules are removed from 1st to last
- `/sys/module/$module/parameters/$param`: live module param value

All the module programs are symlink to `/sbin/kmod`!

Modules are built against `/lib/modules/$(uname -r)/build`, which links to the `/usr/src/linux-headers-$(uname -r)`.

### Sample module

`helloworld_lkm.c`:

```c
// Kernel headers
#include <linux/init.h>
#include <linux/kernel.h>
#include <linux/module.h>

// modinfo(8) includes (also) this info
MODULE_AUTHOR("<name here>");
MODULE_DESCRIPTION("helloworld_lkm: hello, world, our first LKM");
MODULE_LICENSE("Dual MIT/GPL");
MODULE_VERSION("0.1");

// Naming convention: `<module_name>_(init|exit)`
// Functions are private to the module.
//
// `__init` and `__exit` are memory optimizations; they store the function in the binary init.data
// section, since they're invoked only during initialization.
//
static int __init helloworld_lkm_init(void)
{
	printk(KERN_INFO "Hello, world\n");
  // Modules must return either: 0 for success, or -ERRNO for failure (e.g. (`-ENOMEM`)).
  // In case of failure, the kernel prints a warning, but doesn't panic.
  //
  // There are macros (in `err.h`) to convert between pointer and integer (so that errvals can be
  // returned in functions that return a pointer):
  //
  // - ERR_PTR(<ERRNO>) -> *void
  // - PTR_ERR(<ptr>)   -> ERRNO
  // - IS_ERR(<ptr>)
  //
	return 0;
}

static void __exit helloworld_lkm_exit(void)
{
	printk(KERN_INFO "Goodbye, world\n");
}

module_init(helloworld_lkm_init);
module_exit(helloworld_lkm_exit);
```

### Module Makefile

Extract; includes cross-compiling concepts.

When cross compiling, at least a partial kernel build should be performed (page b210), but for simplicity, just fully compile it.

```Makefile
PWD := $(shell pwd)

# object files to build as `m`odules
# use `obj-y` for build as core (`y`es)
#
obj-m += printk_loglvl.o

# 'KDIR' is the Linux 'kernel headers' package on the host system
#
# In case of another architecture, we need to point to the specific source tree
ifeq ($(ARCH),arm)
  KDIR ?= ~/rpi_work/kernel_rpi/linux
else
  KDIR ?= /lib/modules/$(shell uname -r)/build
endif

all:
  # - `C`hange directory, in order to build against the current source tree
  # - then, change to the `M`odule directory, and build that
  #
  make -C $(KDIR) M=$(PWD) modules
```

### Modules interop

Compositional approach to using modules as libraries:

```Makefile
obj-m                 += lowlevel_mem_lib.o
lowlevel_mem_lib-objs := lowlevel_mem.o ../../klib_llkd.o # 2nd is the dependency
```

Export a symbol to the global namespace; this apporach is "module stacking".

```c
int my_glob = 5;
EXPORT_SYMBOL(my_glob);

// Export the symbol only to modules with GPL in their MODULE_LICENSE().
EXPORT_SYMBOL_GPL(my_glob);
```

Compositional approach is preferrable.

## Logging

Logging goes:

- (at least) kernel log buffer in RAM, which can be read via `dmesg`; it's small, 256 KiB by default
- log files (syslog - use `journalctl` to view it)
- console device

```c
// Default is KERN_WARNING
printk(KERN_INFO "Hello, world\n");
```

Can also pipe to `/dev/kmsg` to print messages.

Error levels, with convenience functions (and descriptions):

```c
#define KERN_EMERG   KERN_SOH "0" // `pr_emerg`            // system is unusable
#define KERN_ALERT   KERN_SOH "1" // `pr_alert`            // action must be taken immediately
#define KERN_CRIT    KERN_SOH "2" // `pr_crit`             // critical conditions
#define KERN_ERR     KERN_SOH "3" // `pr_err`              // error conditions
#define KERN_WARNING KERN_SOH "4" // `pr_warn[ing]`        // warning conditions
#define KERN_NOTICE  KERN_SOH "5" // `pr_notice`           // normal but significant condition
#define KERN_INFO    KERN_SOH "6" // `pr_info`             // informational
#define KERN_DEBUG   KERN_SOH "7" // `pr_devel`<>`dev_dbg` // debug-level messages (kernel<>module)
```

Debug level messages don't show up unless the module has been compiled with the `DEBUG` symbol defined.

`dmesg` options:

- `--decode`      : add source/level
- `--ctime`       : human date
- `--color=always`: force color

If the `pr_fmt` macro is defined, e.g.:

```c
// Includes module name and running function; can also use __LINE__ for line number.
//
#define pr_fmt(fmt) "%s:%s(): " fmt, KBUILD_MODNAME, __func__
```

every message will be prefixed with its content.

### Console levels

Delivering to console is regulated by certain parameters; cat `/proc/sys/kernel/printk` for 4 levels:

- current (console)
- default
- minimum
- boot-time default

Default: `4	4	1	7` -> anything lower than 4 is logged in the console; level 0 messages are printed *regardless*.

By setting `8` as console level, any level is printed there.

Can use `echo Y > /sys/module/printk/parameters/ignore_loglevel` to ignore the messages level.

### Rate limiting

- `/proc/sys/kernel/printk_ratelimit_burst` (10): # messages before rate limiting kicks in
- `/proc/sys/kernel/printk_ratelimit`       (5) : interval in jiffies for a burst

Default = 10 messages allowed every 5 jiffies.

### Macros, data types, printk specifiers

- `BITS_PER_LONG`: 32/64
- `CONFIG_ARM`   : defined on ARM
- `__BIG_ENDIAN` : undefined on little endian

- `[s]size_t` : `zd`/`zu`
- pointers
  - `pK`: hashed
  - `px`: generic
  - `pa`: physical address
  - `*ph`: raw buffer as string of chars (up to 64 chars)
    - more than 64 chars: use `print_hex_dump_bytes()`
- ip addresses (v4/v6): `pI4`, `pI6`

## PAG b239
