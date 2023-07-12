# Limine Bare Bones

This repository will show you how to set up a simple 64-bit kernel using Limine.

This project can be built using the host GCC toolchain on most Linux distros on x86_64 when targetting x86_64 (`ARCH=x86_64`), but it is recommended to set up an `*-elf` [cross compiler](https://wiki.osdev.org/GCC_Cross-Compiler) for the chosen target, or use a native cross toolchain such as Clang/LLVM.

It is also recommended to cross reference the contents of this repository with [the Limine Bare Bones](https://wiki.osdev.org/Limine_Bare_Bones) OSDev wiki page.

## Building

It is recommended to build this project using a standard Linux distro, using a Clang/LLVM toolchain capable of cross compilation.

The `ARCH` make variable determines the target architecture to build the kernel and image for. The `all` target (the default one) builds the kernel for the given architecture, and a bootable `.iso` image containing it. The `all-hdd` target builds a flat hard disk/USB image instead.

For example, to build for `aarch64` run:
```sh
make ARCH=aarch64 CC="clang" LD="ld.lld"
```

The image can then be tested with the one of the `run` or `run-hdd` targets.
```sh
make ARCH=aarch64 CC="clang" LD="ld.lld" run
```
