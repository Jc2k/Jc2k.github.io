+++
title = "Session management with systemd"
description = "Managing user service lifecycle in response to external stimulus"

[taxonomies]
categories = ['systemd']
tags = ['gemini', 'systemd', 'linux', 'session']
+++
How can I use systemd user services to handle policies (like what to start and stop when my internet connection comes and goes) on my tiny Linux palmtop?

<!-- more -->

I finally got Debian installed on my Gemini PDA (or should I say [Gemian](https://github.com/gemian)). It's a tiny form factor and there is no mouse so i've been looking into tiling and keyboard friendly window managers. Of those ratpoison is undoubtedly the king.

My typical desktop has services running. "Auto login" scripts and programs that invariably end up in the tray. But my new window manager doesn't have a tray. Arguably there isn't a 'session' in the same way as with GNOME. You can `exec` programs when ratpoison starts but after years of working with modern and sane init systems this felt janky.

As well as wanting to keep my session services running I want to be able to react to events. The 2 main interesting ones right now are when my internet comes and goes and when my lid opens and closes.

Enter systemd. It's a hammer, and i've got a lot of nails to hit.

So I have a script that monitors my internet connection and pops up a ratpoison alert when I connect/disconnect from wifi. It's fairly boring python and I can create a user .service for it:

```
$ cat ~/.config/systemd/user/watch-networking.service
[Unit]
Description=Notify when network online /offline

[Service]
Type=simple
ExecStart=/home/gemini/bin/routetest.py

[Install]
WantedBy=default.target
```

This just says start when the user session starts. I can enable it on next login with:

```
$ systemctl --user enable watch-networking.service
```

And start it immediately for my current session with:

```
$ systemctl --user start watch-networking.service
```

And check on it like a normal systemd task:

```
$ systemctl --user status watch-networking.service
● watch-networking.service - Notify when network online /offline
   Loaded: loaded (/home/gemini/.config/systemd/user/watch-networking.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2018-07-19 22:46:39 BST; 1s ago
 Main PID: 28685 (python)
   CGroup: /user.slice/user-100000.slice/user@100000.service/watch-networking.service
           └─28685 python /home/gemini/bin/routetest.py

Jul 19 22:46:39 kanonji systemd[16589]: Started Notify systemd when network online /offline.
```

You don't need to break out root to set this up, its entirely user level stuff. You can use systemd unit settings like `Restart` to recover from failure. So far so boring.

Next I want to start a service when my internet connection becomes available, and kill it when it goes away.

I create 2 targets: `internet-online.target` and `internet-offline.target`. They conflict so only one can be active at once:

```
$ cat ~/.config/systemd/user/internet-online.target
[Unit]
Description=There is a default route so it looks like we are on the internet
Conflicts=internet-offline.target

$ cat ~/.config/systemd/user/internet-offline.target
[Unit]
Description=There is no default route so it looks like we are not on the internet
Conflicts=internet-online.target
```

My routetest.py script actually `systemctl --user start` the relevant target when the default gateway goes away or comes back. So now I can make user services that are part of the `internet-online.target` target. Here is a service for a script that handles IMAP IDLE for my mail server:

```
$ cat ~/.config/systemd/user/watch-mailbox.service
[Unit]
Description=Watch mailbox for new email
BindsTo=internet-online.target

[Service]
Type=simple
ExecStart=/home/gemini/bin/emailsync.py

[Install]
WantedBy=internet-online.target
```

Which I activate with:

```
$ systemctl --user enable watch-mailbox.services
```

The next time my network watch service transitions to the `internet-online.target` state my IMAP script will start. The next time it enters `internet-offline.target` it will stop, because `BindsTo` means it will stop when `internet-online.target` stops, which it will because it conflicts with `internet-offline.target`.

You can use the same thing with `oneshot` scipts. I do basically the same thing to turn off my screen and wifi/cellular when the lid is closed.
