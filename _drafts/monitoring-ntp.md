---
layout: post
title: "Monitoring NTP"
description: "When a clock is out of sync things randomly start to break (especially crypto), so detecting drift early is super important"
category: Monitoring
tags: ['cloud', 'ops', 'ntp', 'monitoring']
---

I've seen celery drop tasks on the floor because the queuers clock is to far ahead of the workers. I've seen boto fail to update Route53 because a clock was wrong. Time is super important. Here is a little script to look at the output of `ntpq -pn` and try and explain whats going on. It's really meant to be fed into a metrics system like [Influx](http://www.influxdb.org) or [CloudWatch](https://aws.amazon.com/cloudwatch/), but thats left as an exercise for the reader.

The output of `ntpq -pn` is something like this:

         remote           refid      st t when poll reach   delay   offset  jitter
    ==============================================================================
    +128.4.2.6    132.249.16.1       2  u 131  256  373     9.89   16.28   23.25
    *128.4.1.20   .WWVB.             1  u 137  256  377   280.62   21.74   20.23
    -128.8.2.88   128.8.10.1         2  u  49  128  376   294.14    5.94   17.47
    +128.4.2.17   .WWVB.             1  u  173 256  377   279.95   20.56   16.40

The things to note are:

 * Remotes marked with `*` are currently in use - there should only be one.
 * Remotes marked with `+` are of high enough quality that they could replace the primary
 * `delay` is the round trip delay to the remote in ms.
 * `reach` is a left-shift register represented in octal. This would be 377 if no data was lost.
 * We only really care about `delay`, `jitter` and `reach` for the primary peer.

A nice summary of this can be produced with:
    
    from __future__ import division
    import re
    import subprocess
    
    lines = subprocess.check_output(["ntpq", "-pn"]).splitlines()
    if lines[0] == "No association ID's returned":
        raise SystemExit("No timeservers available")
    
    peers = lines[2:]
    overall_health = 0
    candidates = 0
    for line in peers:
        tmp = re.split(" +", line.decode("utf-8").strip())
    
        reachability = (bin(int(tmp[6], 8))[2:].count("1") / 8.0) * 100
        overall_health += reachability * (1 / len(peers))
    
        if tmp[0].startswith("*"):
            print("NTP/Delay={}".format(tmp[7]))
            print("NTP/Offset={}".format(tmp[8]))
            print("NTP/Jitter={}".format(tmp[9]))
            print("NTP/PrimaryHealth={}".format(reachability))
    
        if tmp[0][0] in ("+", "*"):
            candidates += 1
    
    print("NTP/OverallHealth={}".format(overall_health))
    print("NTP/CandidateCount={}".format(candidates))

We probably want to alarm if:

 * There are no candidate peers (probably with a long poll interval - it takes a while to warm up after boot).
 * The reachibility of the primary peer is too low (again, long poll interval).
