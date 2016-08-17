---
layout: post
title: "Dealing with EBS volume hotplug"
description: "EBS volumes are hotplugged. They may need formatting. And your service can't start until the volume is ready, so the default init scripts just won't cut it"
category: Amazon
tags: ['amazon', 'aws', 'ebs', 'ec2', 'volume', 'storage', 'upstart', 'init', 'trusty', 'ubuntu']
---

EBS volumes are attached to EC2 instances that are running. So if you have a solr instance that starts on boot and needs to use that EBS volume you have a problem. How do you make solr wait for that EBS volume to be ready? How do you format it? How do you make all this idempotent.

On Linux, `udev` is how we get notified of a block device getting plugged in. One option that has popped up is [ebsmount](https://www.turnkeylinux.org/blog/ebsmount) which uses [custom udev rules](https://github.com/turnkeylinux/ebsmount/blob/master/85-ebsmount.rules.in) to run your code when a new EBS volume is attached. However on Ubuntu Trusty we have upstart. Upstart has a udev bridge and so we can start a service or run a task in response to the event:

```
start on block-device-added DEVNAME='/dev/xvdf'
task
script
    if [ ! "`blkid /dev/xvdf -o value -s TYPE -c /dev/null`" = "ext4" ]
    then
        mkfs -t ext4 /dev/xvdf
    fi
    if [ ! -d /data ]; then
        echo "/data does not exist. creating..."
        mkdir /data
    fi
    mount /dev/xvdf /data
    initctl emit path-mounted device=/dev/xvdf path=/data
end script
```

We use `blkid` to check if the EBS volume is formatted as `ext4`. If it's not then it soon will be. And then when it's formatted we can mount it. The internet aludes to an upstart event type called `path-mounted` that we would use to start solr when `/data` was mounted. This doesn't seem to work - you can work around it and generate it manually with `initctl emit`.

Then the solr init script is replaced with:

```
start on path-mounted path=/data
env JAVA_HOME="/usr/lib/jvm/java-7-openjdk-amd64"

pre-start script
    if [ ! -d /data/solr ]; then
        mkdir /data/solr
        chown jetty: /data/solr
    fi
end-script

exec /usr/bin/jsvc \
    -nodetach \
    -user jetty \
    <...> \
    -Djetty.host=0.0.0.0 \
    -Djetty.port=8983
```

I totally replace `/etc/init.d/jetty` with upstart configuration - having `/etc/init.d/jetty` and `/etc/init/jetty.conf` seems to cause `sudo start jetty` and `sudo stop jetty` to hang. Just start solr with `/etc/init.d/jetty start` and grap the right command line with `ps aux`. Stick in a `-nodetach` to stop in forking so it plays nice with upstarts process supervision. This also gives us a nice place to create `/data/solr` and ensure its writeable by the `jetty` user.

Don't forget to update `solr.xml` to point at this new location! :wink:
