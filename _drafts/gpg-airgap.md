---
layout: post
title: "Managing your GPG keys with an airgapped machine"
description: "Using an airgapped ancient laptop with a Debian live environment to manage your GPG keys"
category: GPG
tags: ['gpg', 'key', 'encryption', 'opsec']
---

I have an offline environment for managing changes to my GPG keys. It's an old laptop with no WIFI card and no permanent internal storage. This is **not** a complete writeup - it's meant to be a TL;DR summary of what you could do.

I use a stock Debian LiveCD [(here)](http://cdimage.debian.org/debian-cd/current-live/i386/iso-hybrid/debian-live-8.6.0-i386-standard.iso). For this CD the username/password is user/live. You can use `sudo` without a password. It has GPG already installed.

You will need a USB flash drive for copying files between the live environment and a machine with an internet connection. You will need a 2nd USB flash drive for long term offline storage of your master key.


## Initial environment set up

Ideally you should generate your key in this environment and never expose it to an internet connected system. [Attilla Lendvai](https://github.com/attila-lendvai/gpg-keygen) has a great script and a great explanation of some of the concepts. It's worth a read, but the TL;DR is as follows.

We need a good `~/.gnupg/gpg.conf`:

    fixed-list-mode
    keyid-format 0xlong
    personal-digest-preferences SHA512
    default-preference-list SHA512 SHA384 SHA256 SHA224 AES256 AES192 AES CAST5 BZIP2 ZLIB ZIP Uncompressed
    verify-options show-uid-validity
    list-options show-uid-validity
    cert-digest-algo SHA512

Now we can generate the main key. This key will only be used to sign subkeys. It is important to keep this keep protected - if a subkey is lost it can be revoked and replaced without falling out of the web of trust. If your main key is compromised you will need to start over. Go for a 4096 RSA only key:

    $ gpg --gen-key
    gpg (GnuPG) 1.4.20; Copyright (C) 2015 Free Software Foundation, Inc.
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.

    gpg: directory '/home/john/.gnupg' created
    gpg: new configuration file '/home/john/.gnupg/gpg.conf' created
    gpg: WARNING: options in '/home/john/.gnupg/gpg.conf' are not yet active during this run
    gpg: keyring '/home/john/.gnupg/secring.gpg' created
    gpg: keyring '/home/john/.gnupg/pubring.gpg' created
    Please select what kind of key you want:
       (1) RSA and RSA (default)
       (2) DSA and Elgamal
       (3) DSA (sign only)
       (4) RSA (sign only)
    Your selection? 4
    RSA keys may be between 1024 and 4096 bits long.
    What keysize do you want? (2048) 4096
    Requested keysize is 4096 bits
    Please specify how long the key should be valid.
             0 = key does not expire
          <n>  = key expires in n days
          <n>w = key expires in n weeks
          <n>m = key expires in n months
          <n>y = key expires in n years
    Key is valid for? (0) 0
    Key does not expire at all
    Is this correct? (y/N) y

    You need a user ID to identify your key; the software constructs the user ID
    from the Real Name, Comment and Email Address in this form:
        "Heinrich Heine (Der Dichter) <heinrichh@duesseldorf.de>"

    Real name: John Smith
    Email address: john.smith@example.com
    Comment:
    You selected this USER-ID:
        "John Smith <john.smith@example.com>"

    Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
    You need a Passphrase to protect your secret key.

    We need to generate a lot of random bytes. It is a good idea to perform
    some other action (type on the keyboard, move the mouse, utilize the
    disks) during the prime generation; this gives the random number
    generator a better chance to gain enough entropy.
    ..+++++
    ..................................................+++++
    gpg: /home/john/.gnupg/trustdb.gpg: trustdb created
    gpg: key B05E3F37 marked as ultimately trusted
    public and secret key created and signed.

    gpg: checking the trustdb
    gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
    gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
    pub   4096R/B05E3F37 2016-10-26
          Key fingerprint = 4C0A CCA7 8170 E95A 22FE  C7B2 EA57 D591 B05E 3F37
    uid                  John Smith <john.smith@example.com>

    Note that this key cannot be used for encryption.  You may want to use
    the command "--edit-key" to generate a subkey for this purpose.

Generate the subkeys:

    $ gpg --edit-key EA57D591B05E3F37
    gpg (GnuPG) 1.4.20; Copyright (C) 2015 Free Software Foundation, Inc.
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.

    Secret key is available.

    pub  4096R/B05E3F37  created: 2016-10-26  expires: never       usage: SC  
                         trust: ultimate      validity: ultimate
    [ultimate] (1). John Smith <john.smith@example.com>

    gpg> addkey
    Key is protected.

    You need a passphrase to unlock the secret key for
    user: "John Smith <john.smith@example.com>"
    4096-bit RSA key, ID B05E3F37, created 2016-10-26

    gpg: gpg-agent is not available in this session
    Please select what kind of key you want:
       (3) DSA (sign only)
       (4) RSA (sign only)
       (5) Elgamal (encrypt only)
       (6) RSA (encrypt only)
    Your selection? 6
    RSA keys may be between 1024 and 4096 bits long.
    What keysize do you want? (2048) 4096
    Requested keysize is 4096 bits
    Please specify how long the key should be valid.
             0 = key does not expire
          <n>  = key expires in n days
          <n>w = key expires in n weeks
          <n>m = key expires in n months
          <n>y = key expires in n years
    Key is valid for? (0) 3y
    Key expires at Sat 26 Oct 2019 08:50:37 AM PDT
    Is this correct? (y/N) y
    Really create? (y/N) y
    We need to generate a lot of random bytes. It is a good idea to perform
    some other action (type on the keyboard, move the mouse, utilize the
    disks) during the prime generation; this gives the random number
    generator a better chance to gain enough entropy.
    ..................+++++
    ..............................+++++

    pub  4096R/B05E3F37  created: 2016-10-26  expires: never       usage: SC  
                         trust: ultimate      validity: ultimate
    sub  4096R/BF4CEBBA  created: 2016-10-26  expires: 2019-10-26  usage: E   
    [ultimate] (1). John Smith <john.smith@example.com>

    gpg> addkey
    Key is protected.

    You need a passphrase to unlock the secret key for
    user: "John Smith <john.smith@example.com>"
    4096-bit RSA key, ID B05E3F37, created 2016-10-26

    Please select what kind of key you want:
       (3) DSA (sign only)
       (4) RSA (sign only)
       (5) Elgamal (encrypt only)
       (6) RSA (encrypt only)
    Your selection? 4
    RSA keys may be between 1024 and 4096 bits long.
    What keysize do you want? (2048)
    Requested keysize is 2048 bits
    Please specify how long the key should be valid.
             0 = key does not expire
          <n>  = key expires in n days
          <n>w = key expires in n weeks
          <n>m = key expires in n months
          <n>y = key expires in n years
    Key is valid for? (0) 3y
    Key expires at Sat 26 Oct 2019 08:52:00 AM PDT
    Is this correct? (y/N) y
    Really create? (y/N) y
    We need to generate a lot of random bytes. It is a good idea to perform
    some other action (type on the keyboard, move the mouse, utilize the
    disks) during the prime generation; this gives the random number
    generator a better chance to gain enough entropy.
    .+++++
    +++++

    pub  4096R/B05E3F37  created: 2016-10-26  expires: never       usage: SC  
                         trust: ultimate      validity: ultimate
    sub  4096R/BF4CEBBA  created: 2016-10-26  expires: 2019-10-26  usage: E   
    sub  2048R/45F1A70C  created: 2016-10-26  expires: 2019-10-26  usage: S   
    [ultimate] (1). John Smith <john.smith@example.com>

    gpg> save

You can also generate authentication keys in `--expert` mode. I use a GPG authentication key for SSH.

    $ gpg --expert --edit-key EA57D591B05E3F37
    gpg (GnuPG) 1.4.20; Copyright (C) 2015 Free Software Foundation, Inc.
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.

    Secret key is available.

    pub  4096R/B05E3F37  created: 2016-10-26  expires: never       usage: SC  
                         trust: ultimate      validity: ultimate
    sub  4096R/BF4CEBBA  created: 2016-10-26  expires: 2019-10-26  usage: E   
    sub  2048R/45F1A70C  created: 2016-10-26  expires: 2019-10-26  usage: S   
    [ultimate] (1). John Smith <john.smith@example.com>

    gpg> addkey
    Key is protected.

    You need a passphrase to unlock the secret key for
    user: "John Smith <john.smith@example.com>"
    4096-bit RSA key, ID B05E3F37, created 2016-10-26

    gpg: gpg-agent is not available in this session
    Please select what kind of key you want:
       (3) DSA (sign only)
       (4) RSA (sign only)
       (5) Elgamal (encrypt only)
       (6) RSA (encrypt only)
       (7) DSA (set your own capabilities)
       (8) RSA (set your own capabilities)
    Your selection? 8

    Possible actions for a RSA key: Sign Encrypt Authenticate
    Current allowed actions: Sign Encrypt

       (S) Toggle the sign capability
       (E) Toggle the encrypt capability
       (A) Toggle the authenticate capability
       (Q) Finished

    Your selection? A

    Possible actions for a RSA key: Sign Encrypt Authenticate
    Current allowed actions: Sign Encrypt Authenticate

       (S) Toggle the sign capability
       (E) Toggle the encrypt capability
       (A) Toggle the authenticate capability
       (Q) Finished

    Your selection? S

    Possible actions for a RSA key: Sign Encrypt Authenticate
    Current allowed actions: Encrypt Authenticate

       (S) Toggle the sign capability
       (E) Toggle the encrypt capability
       (A) Toggle the authenticate capability
       (Q) Finished

    Your selection? E

    Possible actions for a RSA key: Sign Encrypt Authenticate
    Current allowed actions: Authenticate

       (S) Toggle the sign capability
       (E) Toggle the encrypt capability
       (A) Toggle the authenticate capability
       (Q) Finished

    Your selection? Q
    RSA keys may be between 1024 and 4096 bits long.
    What keysize do you want? (2048) 4096
    Requested keysize is 4096 bits
    Please specify how long the key should be valid.
             0 = key does not expire
          <n>  = key expires in n days
          <n>w = key expires in n weeks
          <n>m = key expires in n months
          <n>y = key expires in n years
    Key is valid for? (0) 3y
    Key expires at Sat 26 Oct 2019 08:56:53 AM PDT
    Is this correct? (y/N) y
    Really create? (y/N) y
    We need to generate a lot of random bytes. It is a good idea to perform
    some other action (type on the keyboard, move the mouse, utilize the
    disks) during the prime generation; this gives the random number
    generator a better chance to gain enough entropy.
    ....+++++
    .......+++++

    pub  4096R/B05E3F37  created: 2016-10-26  expires: never       usage: SC  
                         trust: ultimate      validity: ultimate
    sub  4096R/BF4CEBBA  created: 2016-10-26  expires: 2019-10-26  usage: E   
    sub  2048R/45F1A70C  created: 2016-10-26  expires: 2019-10-26  usage: S   
    sub  4096R/AF3A5B1C  created: 2016-10-26  expires: 2019-10-26  usage: A   
    [ultimate] (1). John Smith <john.smith@example.com>

    gpg> save

It's a good idea to generate a revocation cert:

    $ gpg -a  --output /media/revocation.asc --gen-revoke EA57D591B05E3F37

    sec  4096R/B05E3F37 2016-10-26 John Smith <john.smith@example.com>

    Create a revocation certificate for this key? (y/N) y
    Please select the reason for the revocation:
      0 = No reason specified
      1 = Key has been compromised
      2 = Key is superseded
      3 = Key is no longer used
      Q = Cancel
    (Probably you want to select 1 here)
    Your decision? 1
    Enter an optional description; end it with an empty line:
    > Revocation certificate generated when key itself was; if this cert is used then i no longer have access to the key and it should be treated as compromised
    >
    Reason for revocation: Key has been compromised
    Revocation certificate generated when key itself was; if this cert is used then i no longer have access to the key and it should be treated as compromised
    Is this okay? (y/N) y

    You need a passphrase to unlock the secret key for
    user: "John Smith <john.smith@example.com>"
    4096-bit RSA key, ID B05E3F37, created 2016-10-26

    gpg: gpg-agent is not available in this session
    Revocation certificate created.

    Please move it to a medium which you can hide away; if Mallory gets
    access to this certificate he can use it to make your key unusable.
    It is smart to print this certificate and store it away, just in case
    your media become unreadable.  But have some caution:  The print system of
    your machine might store the data and make it available to others!

Finally you need to export an offline copy of your keys:

    gpg -a --export EA57D591B05E3F37 > /media/public-key.asc
    gpg -a --export-secret-key EA57D591B05E3F37 > /media/master-key.asc
    gpg -a --export-secret-subkeys EA57D591B05E3F37 > /media/secret-sub-keys.asc


## Subsequent usage of live environment

When I boot the machine I copy over my gpg.conf from an SD card and import my keys:

    mount /dev/mmcblk0 /media
    mkdir ~/.gnupg
    cp /media/gpg.conf ~/.gnupg/gpg.conf
    gpg --import /media/public-key.asc
    gpg --import /media/master-key.asc

(I actually have that scripted, so i mount `/media` and run `/media/init`).

At that point you can perform whatever changes needed, for example `gpg --edit-key`.

When i've done making changes i export the keys back to the SD card. Again, there is a bash script at `/media/save`:

    cp ~/.gnupg/gpg.conf /media/gpg.conf
    gpg -a --export > /media/public-key.asc
    gpg -a --export-secret-key > /media/master-key.asc
    gpg -a --export-secret-subkeys > /media/secret-sub-keys.asc

(You'll want to give those commands your key id).
