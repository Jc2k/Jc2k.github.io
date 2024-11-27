+++
title = "Random MAC Addresses That Aren't"
description = "2 network devices with random MAC addresses... getting the same MAC address?"
aliases = ["/kernel/2018/07/10/random-mac-addresses-that-arent/"]

[taxonomies]
categories = ["kernel"]
tags = ['kernel', 'systemd', 'usb', 'networking']
+++

What to do when both sides of a USB network data transfer cable have the same MAC address??

<!-- more -->

I've built a Raspberry Pi based GameBoy based on [Kite's SuperAIO](https://www.youtube.com/watch?v=jweRkxGF1mA). Where the original GameBoy had a GameLink socket I have a USB socket. I thought it would be a nice retro touch if i could plug a cable into 2 GameBoys in the same place and have working GameLink emulation (2 player tetris, pokemon trading, etc).

Data transfer cables like [this one](https://www.amazon.co.uk/d/Cables/Transfer-Cable/B01B6X8QP0/) (i tried the USB 2 version of it) used to be popular for migrating data from an old Windows computer to a new one. These days they come with their own synchronisation software. But really they are just 2 usb network adapters connected together. On a linux box (like my GameBoy) both ends will get a new `usb0` interface which you can configure any way you like.

In my previous post I talked about how i'd configured systemd to start some python code when the cable was plugged in. This does UDP multicast discovery (sort of like [this](https://gist.github.com/mcfloundinho/4f785d4546057b49b56c)) and writes the Gambatte core options file for both sides. And as it was a host-to-host link I just use link local addressing. But there was a hitch, obviously.

My first assumption was that the cable would have two proper MAC address. Like 2 real hardware network cards connected together. On testing with a raspberry pi it looked like both ends had the same mac address. That was weird and felt broken but I didn't think too much of it until I realised my link local addresses were the same on both systems. And it looked like duplicate address detection wasn't happening (presumably it assumes unique mac addresses?).

The first thing I tried was in systemd (its my hammer, everything is now a nail). In `/etc/systemd/system/00-gamelink.link`:

```
[Match]
Driver=plusb

[Link]
MACAddressPolicy=random
```

And various permutations of. But none of them did anything. Sigh.

At first i wasn't sure I had the right `[Match]` stanza. I did.

Then I wasn't sure about the name of the `.link` file. The sort order of `.link` files is important. Each device is tested against all criteria in the `[Match]` section and its the first `.link` file to match the criteria that is used. So if the sort order in use meant that `99-default.link` came before `gamelink.link` then it would never be used. It wasn't that.

Eventually I ended up in the systemd source code. From [here](https://github.com/systemd/systemd/blob/6b3d378331fe714c7bf2263eaa9a8b33fc878e7c/src/udev/net/link-config.c#L458):

```
case MACPOLICY_RANDOM:
        if (!mac_is_random(device)) {
                r = get_mac(device, true, &generated_mac);
                if (r == -ENOENT) {
                        log_warning_errno(r, "Could not generate random MAC address for %s: %m", old_name);
                        break;
                } else if (r < 0)
                        return r;
                mac = &generated_mac;
        }
        break;
```

So if the MAC address is already random the policy won't do anything. The only think other than systemd that might set the mac address in my setup is the kernel itself. From [here](https://github.com/torvalds/linux/blob/bfe9b9d2df669a57a95d641ed46eb018e204c6ce/drivers/net/usb/usbnet.c#L2254) I found:

```
static int __init usbnet_init(void)
{
        /* Compiler should optimize this out. */
        BUILD_BUG_ON(
                FIELD_SIZEOF(struct sk_buff, cb) < sizeof(struct skb_data));

        eth_random_addr(node_id);
        return 0;
}
```

A driver's init is called when the driver is loaded, not when a device is loaded. Surely it can't be this though - it seems unlikely that a module loaded at boot time would be loaded so deterministically that the mac would be the same on 2 separate systems, even if they were using the same image.

```
cat output/build/linux-*/.config
<snip>
CONFIG_USB_USBNET=y
```

Ah, it's compiled in.

Forcing `usbnet` to be a module hides the issue. I'm not sure what this is actually doing. If usbnet is compiled in:

 * Does usbnet_init get called before the PRNG is initialized at all?
 * Does usbnet_init get called before there is any entropy?
 * Is the entropy for the PRNG actually unreliable/missing this early (e.g. there is no RTC and no physical ethernet)

Either way, as a module there is enough of a delay for `eth_random_addr` to return some slightly random data for us, so link local addressing works, so my python code works. Hurrah.

Also given the driver allocates a single random mac address at boot (or module load time) does this mean you can't plug multiple usbnet cables into the same linux box?