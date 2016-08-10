---
layout: post
title: "Building the Linux kernel on a Mac inside Docker: Attempt #2"
description: "Breaking xargs by trying to cross compile the linux kernel inside a Docker container on OSX."
category:
tags: ['docker', 'osx', 'linux', 'kernel', 'compile']
---
Todays failure is about `xargs`. For whatever reason inside a `qemu-user-static` environment inside docker it can no longer do its part to help build a kernel:

```
  CLEAN   arch/arm/boot
/usr/bin/xargs: rm: Argument list too long
Makefile:1502: recipe for target 'clean' failed
make[2]: *** [clean] Error 126
make[2]: Leaving directory '/src/debian/linux-source-4.7.0/usr/src/linux-source-4.7.0'
debian/ruleset/targets/source.mk:35: recipe for target 'debian/stamp/install/linux-source-4.7.0' failed
make[1]: *** [debian/stamp/install/linux-source-4.7.0] Error 2
make[1]: Leaving directory '/src'
debian/ruleset/local.mk:96: recipe for target 'kernel_source' failed
```

It looks like I need to [patch](https://groups.google.com/forum/#!msg/linux.kernel/HuHeqLe79mw/gb6gt0Z4x7kJ) the kernels `Makefile` to workaround some limit qemu is introducing.
