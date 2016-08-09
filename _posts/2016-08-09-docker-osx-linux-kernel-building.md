---
layout: post
title: "Building the Linux kernel on a Mac inside Docker: Attempt #1"
description: "First stab at building a debian package of the Linux kernel inside Docker running on a Mac ends in failure."
category:
tags: ['docker', 'osx', 'linux', 'kernel', 'compile']
---
I've recently been using my Docker for Mac install to try my hand at packaging the latest upstream kernel for my Raspberry Pi. I have a Debian Jessie ARM rootfs with `kernel-package` installed and the ideas was to run `make-kpkg` on an OSX-local checkout. It was able to build the main kernel but then:

```
make[3]: *** Documentation/Kbuild: Is a directory.  Stop.
Makefile:1260: recipe for target '_clean_Documentation' failed
make[2]: *** [_clean_Documentation] Error 2
make[2]: Leaving directory '/src/debian/linux-source-4.7.0/usr/src/linux-source-4.7.0'
debian/ruleset/targets/source.mk:35: recipe for target 'debian/stamp/install/linux-source-4.7.0' failed
make[1]: *** [debian/stamp/install/linux-source-4.7.0] Error 2
make[1]: Leaving directory '/src'
debian/ruleset/local.mk:96: recipe for target 'kernel_source' failed
make: *** [kernel_source] Error 2
```

The moral of this story is to not try and build kernels on **case insensitive** filesystems.
