+++
title = "Unrevoking your GPG key"
description = "How to sign something after your key has expired"

[taxonomies]
tags = ['gpg', 'key', 'encryption', 'opsec', 'revocation']
+++

I recently had a bit of a whoopsie - I revoked my key before I had signed my transition statement. With all copies of my keyrings now aflicted how do I get a working version of my key back so I can retrospectively sign the transition statement?

<!-- more -->

Once a key has been revoked and uploaded to the key servers there is no way to unrevoke it on the key servers. It's game over. But you can hand edit the key locally so that it works again. The process I followed is from the [GPG mailing list](https://lists.gnupg.org/pipermail/gnupg-users/2007-April/030726.html):

First pipe your public key into `gpgsplit` and split the key into the packets that it is composed of:

    gpg --export $YOURKEYID | gpgsplit

Ideally do this in a temporary and empty directory. You should see a bunch of files in your cwd with names like `000001-006.public_key`.

Hopefully you have a file called `000002-002.sig` - this should be the revocation. Check with:

    gpg --list-packets 000002-002.sig

If the `sigclass` is set to `0x20`, its the revocation signature. Delete it. Otherwise check the other files to find a `sigclass` of `0x20`.

Once you've delete the revocation signature you can delete the old version of your public key (in expert mode - gpg normally forbides deleting a public key which has a private key) and reimport the edited key:

    gpg --expert --delete-key $YOURKEYID
    cat 0000* | gpg --import
