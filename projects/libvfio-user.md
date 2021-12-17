# libvfio-user

- [libvfio-user](#libvfio-user)
  - [Setup/Build/Jobs](#setupbuildjobs)
  - [General references](#general-references)
  - [Concepts](#concepts)
  - [Connection](#connection)

## Setup/Build/Jobs

```sh
apt install libjson-c-dev libcmocka-dev libssl-dev python3-pytest

# Produces libvfio-user.so
make BUILD_TYPE=rel
make install

make coverity   # upload a coverity build
make pytest
```

## General references

- API info: `include/libvfio-user.h`
- Example of simple servers: see `README.md`
- vfio-user specification: https://patchew.org/QEMU/20210614104608.212276-1-thanos.makatos@nutanix.com

## Concepts

vfio/-user:

- vfio-user reuses the core VFIO concepts defined in its API, but implements them as messages to be sent over a socket
- many vfio operations use the ioctl() system call

Other:

- Unix Domain Socket (`AF_UNIX`, "IPC socket"): Communication endpoint to efficiently communicate inter-process
  - Manpage: https://man7.org/linux/man-pages/man7/unix.7.html

Skipped:

- vfio-user specification
  - region details

## Connection

(`VFIO_*` are messages)

- Handshake: client sends `VFIO_USER_VERSION` and capabilities; server sends compatible versions/capabilities (or closes)
- Device general information is asked via `VFIO_USER_DEVICE_GET_INFO`
- Device regions information is asked via `VFIO_USER_DEVICE_GET_REGION_INFO`
  - Returned: R/W permissions, extra capabilities, mappable, etc.
  - If a region is mappable, the served sends an FD, that the client can mmap()
    - The server polls the client for updates!
- Device interrupt types are asked via `VFIO_USER_DEVICE_GET_IRQ_INFO`
  - ITs are specific to the bus
  - interrupts are signalled (to the client) via event FD
    - the client configures signalling via `VFIO_USER_SET_IRQS`
- Unmapped R/W are performed via `VFIO_USER_REGION_READ`/`VFIO_USER_REGION_WRITE`
- The client informs the server of regions that can be mapped via `VFIO_USER_DMA_MAP`/`VFIO_USER_DMA_UNMAP`
  - the servers accesses them via `VFIO_USER_DMA_READ`/`VFIO_USER_DMA_WRITE` (so this is not really DMA)
    - real DMA can be performed via mmap on FDs provided by the client
