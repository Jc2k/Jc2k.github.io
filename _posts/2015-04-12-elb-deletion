---
layout: post
title: "Deleting an Amazon ELB properly"
description: "With no 'deleting' state how do you know when it's really gone?"
category: 
tags: ['aws', 'elb', 'devops']
---
Recently I automated deletion of an Elastic Load Balancer and the Subnet it was in. Fairly straightforward stuff. One slight problem is that an ELB doesn't have states. When you delete one it disappears immediately. But in the background it is still there. When you try to delete the subnet it was in you get a dependency error. After a couple of minutes it does work.

The same is true if you try to delete a security group that the ELB was using.

What's going on?

If you look at the Network interfaces view in the EC2 control panel after you've deleted an ELB you will see that its network interfaces still exist and are cleaned up in the background. Internally it's in a 'deleting' state, but we just can't see that in the ELB API.

We can simulate it to some extent by polling the network interfaces API. It turns out (thanks Andrew @ AWS support) that ELB sets the description of its ENI's to 'ELB yournamehere'. So with botocore we can do something like:

    description = "ELB " + your_balancers_name
    for i in range(120):
        interfaces = client.describe_network_interfaces(
            Filters=[
                {"Name": "description", "Values": [description]},
            ]
        ).get('NetworkInterfaces', [])
        if len(interfaces) == 0:
            return
        time.sleep(1)
    raise Exception("ELB not deleted after 2 minutes")

This is now done automatically when deleting an ELB with [Touchdown](http://docs.yaybu.com/projects/touchdown/en/latest/). It won't consider an ELB deleted until all of its network interfaces have been cleaned up. So you shouldn't get any errors deleting subnets or security groups!

Support told me this behaviour is unlikely to change (and has been that way for at least 2 years), and they have fed back internally that the possible values for the ENI description field should be documented (they aren't right now).
