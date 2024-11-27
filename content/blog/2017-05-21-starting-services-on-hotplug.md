+++
title = "Starting services on hotplug"
description = "Using systemd and udev to automatically start and stop a systemd service"

[taxonomies]
categories = ["systemd"]
tags = ['systemd', 'udev', 'hotplug']
+++

I want systemd to start a service when a USB device is plugged in and stop it when i remove it.

<!-- more -->

Use systemctl to get a list of units:

```
$ systemctl
UNIT                                       LOAD   ACTIVE SUB       DESCRIPTION
sys-subsystem-net-devices-gamelink0.device loaded active plugged   PL25A1 Host-Host Bridge
<snip>
```

There's no configuration required here - the `.device` unit just appears in response to udev events without any configuration. I've previously set up udev rules so that my USB host-to-host cable is consistently named `gamelink0`, but even without that it would show up under its default name.

The simplest way is just to take advantage of `WantedBy`. In `gamelink.service`:

```
$ cat /etc/systemd/systemd/gamelink.service
[Unit]
Description = Gamelink cable autoconf
After=sys-subsystem-net-devices-gamelink0.device
BindsTo=sys-subsystem-net-devices-gamelink0.device

[Service]
Type=simple
Environment=PYTHONUNBUFFERED=1
ExecStart=/usr/bin/python3 /home/john/gamelink.py

[Install]
WantedBy = sys-subsystem-net-devices-gamelink0.device
```

And then install it:

```
$ systemctl enable gamelink
Created symlink from /etc/systemd/system/sys-subsystem-net-devices-gamelink0.device.wants/gamelink.service to /etc/systemd/system/gamelink.service.
```

The [`WantedBy` directive](https://www.freedesktop.org/software/systemd/man/systemd.unit.html#WantedBy=) tells `systemctl enable` to drop the symlink in a `.wants` directory for the device. Whenever a new unit becomes active systemd will look in `.wants` to see what other related services needs to be started, and that applies to `.device` units just as much as `.service` units or `.target` units. That behaviour is all we need to start our daemon on hotplug.

The [`BindsTo` directive](https://www.freedesktop.org/software/systemd/man/systemd.unit.html#BindsTo=) lets us stop the service when the `.device` unit goes away (i.e. the device is unplugged). Used in conjunction with `After` we ensure that the service may never be in active state without a specific device unit also in active state.
