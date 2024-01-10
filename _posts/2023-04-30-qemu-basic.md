---
title: Basic of QEMU
date: 2023-04-30 09:37:50 08:00
image: assets/img/covers/qemu.png
categories: [Linux]
tags: [linux, qemu]
---

> QEMU is a generic and open source machine emulator and virtualizer

QEMU has two main function

- Run OSes and programs made for one machine on a different machine
  (eg. run ARM programs on x86 PC).
- Use other hypervisors like `Xen` or `KVM` to use CPU extensions for virtualization.
  It provides near native performances.

All parameters to run a virtual machine must be specified on the command line
at every launch.

## QEMU varients

### Full-system emulation (qemu-system-\*)

- In this mode, QEMU emulates a full system, including processors and peripherals.
- More accurate but slower.
- Does not require thr emulated OS to be linux.

### Usermode emulation

- In this mode, QEMU invoke a **Linux** executable that may be complied
  for a different architecture.

- Leverage the host system resources.

- There may be compatibility issues

## Creating new virualized system

### Creating a hard disk image

A hard disk image is a file which stores the contents of the emulated hard disk.

- _raw_: byte-by-byte the same as what the guest sees and the least I/O overhead
- _qcow2_:
  - only allocates space to the image file when the guest operating
    system actually writes
  - supports QEMU snapshotting
  - will like affect performance

```sh
qemu-img create -f raw image_file 4G
qemu-img create -f qcow2 image_file 4G
```

#### Overlay storage images

Create a backing image and have QEMU keep mutations to this image in an overlay image.

```sh
qemu-img create -o backing_file=img1.raw,backing_fmt=raw -f qcow2 img1.cow
```

run QEMU VM

```sh
qemu-system-x86_64 img1.cow
```

When the path to the backing image changes, repair is required.

Attention: Make sure the original image's path still leads to this image
(use symbolic link).

```sh
qemu-img rebase -b /new/img1.raw /new/img1.cow
```

unsafe mode where the old path to the backing image is not checked

```sh
qemu-img rebase -u -b /new/img1.raw /new/img1.cow
```

#### Resizing an image

It works for both raw and qcow2.

```sh
qemu-img rebase -b /new/img1.raw /new/img1.cow
```

Attention: **reduce the allocated file systems and partition sizes** before
shrink the image size, and increase when enlarging the image size.

#### Converting an image

```sh
qemu-img convert -f raw -O qcow2 input.img output.qcow2
```

### Preparing the installation media

The installation medium should not be mounted because QEMU accesses the media directly.

### Installing the operating system

```sh
qemu-system-x86_64 -cdrom iso_image -boot order=d -drive file=disk_image,format=raw
```

- By default 129 MiB of memory is assigned. Use `-m` to adjust, like `-m 512M`
- When running in headless mode, it starts a local VNC server on port 5900

## Running virtualized system

```sh
qemu-system-x86_64 options disk_image
```

> when the mouse pointer is grabbed, us `Ctrl-Alt-g` to release it

### Enabling KVM

append `-accel kvm` to options. (`-enable-kvm` `-machine accel=kvm`)

### Enabling IOMMU (Intel VT-d/AMD-Vi) support

`-device intel-iommu`

### Booting in UEFI mode

use `OVMF`

- first way: copy `OVMF.fd`

  ```sh
  -drive if=pflash,format=raw,file=/copy/of/OVMF.fd
  ```

- second way: split OVMF into two files

  ```bash
  -drive \
  if=pflash,format=raw,readonly=on,file=/usr/share/edk2-ovmf/x64/OVMF_CODE.fd \
  -drive \
  if=pflash,format=raw,file=/copy/of/OVMF_VARS.fd
  ```

### Trusted Platform Module emulation

see [archlinux wiki](https://wiki.archlinux.org/title/QEMU#Installing_the_operating_system)

## Share data between host and guest

### Network

The default user-mode networking allows the guest to access the host OS
at the IP address 10.0.2.2.

### QEMU's port forwarding(IPv4-only)

to bind port 60022 on the host with port 22 on the guest, and `hostfwd` can be repeated

```sh
qemu-system-x86_64 disk_image -nic user,hostfwd=tcp::60022-:22
```

### QEMU's built-in SMB server

### Host file sharing with virtiofsd

see [archlinux wiki](https://wiki.archlinux.org/title/QEMU#Installing_the_operating_system)

### Using filesystem passthrough and VirtFS (9pfs)

VirtFS : Plan 9 folder sharing over Virtio - I/O virtualization framework)

```sh
-virtfs FSDRIVER,path=PATH_TO_SHARE,mount_tag=MOUNT_TAG,security_model=mapped|mapped-xattr|mapped-file|passthrough|none[,id=ID][,writeout=immediate][,readonly][,fmode=FMODE][,dmode=DMODE][,multidevs=remap|forbid|warn][,socket=SOCKET|sock_fd=SOCK_FD]
```

- FSDRIVER: `local`, `proxy`, `synth`. In short, use `local`
- MOUNT_TAG: Specifies the tag name to be used by the guest
  to mount this export point.
- security_mode recommended is `mapped-xattr`

mount the shared folder using

```sh
mount -t 9p -o trans=virtio [mount tag] [mount point] -oversion=9p2000.L
```

## Networking

### Link-level address caveat

`-net nic`

to specify mac

```sh
qemu-system-x86_64 -net nic,macaddr=52:54:XX:XX:XX:XX -net vde disk_image
```

### User-mode networking

- By default
- The virtual machines will not be directly visible on the external network,
  nor will virtual machines be able to talk to each other
  if you start up more than one concurrently.
