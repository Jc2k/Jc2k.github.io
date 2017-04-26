---
layout: post
title: "Multi-core twisted with systemd socket activation"
description: "Run your twisted app on multiple cores without haproxy or iptables magic"
category: Ops
tags: ['systemd', 'twisted', 'python', 'socket']
---

With a stateless Twisted app you can scale by adding more instances. Unless you are explicitly offloading to subprocesses you will often have spare cores on the same box as your existing instance. But to exploit them you end up running haproxy or faking a [load balancer with iptables](https://www.webair.com/community/simple-stateful-load-balancer-with-iptables-and-nat/).

With the `SO_REUSEPORT` socket flag multiple [processes can listen on the same port](https://lwn.net/Articles/542629/), but this isn't available from twisted ([yet](https://github.com/twisted/twisted/pull/759)). But with systemd and socket activation we can use it today.

As a proof of concept we'll make a 4 core HTTP static web service. In a fresh Ubuntu 16.04 VM install `python-twisted-web`.

In `/etc/systemd/system/web@.socket`:

```
[Unit]
Description=Socket for worker %i

[Socket]
ListenStream = 8080
ReusePort = yes
Service = www@%i.service

[Install]
WantedBy = sockets.target
```

And in `/etc/systemd/system/web@.service`

```
[Unit]
Description=Worker %i
Requires=www@%i.socket

[Service]
Type=simple
ExecStart=/usr/bin/twistd --nodaemon --logfile=- --pidfile= web --port systemd:domain=INET:index:0 --path /tmp
NonBlocking=yes
User=nobody
Group=nobody
Restart=always
```

Then to get 4 cores:

```
$ systemctl enable --now www@0.socket
$ systemctl enable --now www@1.socket
$ systemctl enable --now www@2.socket
$ systemctl enable --now www@3.socket
```

Lets test it. In a python shell:

```
import urllib
import time

while True:
    urllib.urlopen('http://172.16.140.136:8080/').read()
    time.sleep(1)
```

And in another terminal you can tail the logs with `journalctl`:

```
$ sudo journalctl -f -u www@*.service
Apr 26 02:43:51 ubuntu twistd[10441]: 2017-04-26 02:43:51-0700 [-] - - - [26/Apr/2017:09:43:51 +0000] "GET / HTTP/1.0" 200 2081 "-" "Python-urllib/1.17"
Apr 26 02:43:52 ubuntu twistd[10441]: 2017-04-26 02:43:52-0700 [-] - - - [26/Apr/2017:09:43:52 +0000] "GET / HTTP/1.0" 200 2081 "-" "Python-urllib/1.17"
Apr 26 02:43:53 ubuntu twistd[10444]: 2017-04-26 02:43:53-0700 [-] - - - [26/Apr/2017:09:43:53 +0000] "GET / HTTP/1.0" 200 2081 "-" "Python-urllib/1.17"
Apr 26 02:43:54 ubuntu twistd[10452]: 2017-04-26 02:43:54-0700 [-] - - - [26/Apr/2017:09:43:54 +0000] "GET / HTTP/1.0" 200 2081 "-" "Python-urllib/1.17"
Apr 26 02:43:55 ubuntu twistd[10452]: 2017-04-26 02:43:55-0700 [-] - - - [26/Apr/2017:09:43:55 +0000] "GET / HTTP/1.0" 200 2081 "-" "Python-urllib/1.17"
Apr 26 02:43:56 ubuntu twistd[10447]: 2017-04-26 02:43:56-0700 [-] - - - [26/Apr/2017:09:43:56 +0000] "GET / HTTP/1.0" 200 2081 "-" "Python-urllib/1.17"
Apr 26 02:43:57 ubuntu twistd[10450]: 2017-04-26 02:43:57-0700 [-] - - - [26/Apr/2017:09:43:57 +0000] "GET / HTTP/1.0" 200 2081 "-" "Python-urllib/1.17"
```

As you can see the `twisted[pid]` changes as different cores handle requests.

If you deploy new code you can `systemctl restart www@*.service` to restart all cores.

`systemctl enable` will mean the 4 cores are available on next boot, too.
