---
layout: post
title: "Adding and removing GPG uid's"
description: "How to edit your GPG keypair to remove an e-mail address and add another"
category: GPG
tags: ['gpg', 'key', 'encryption', 'opsec']
---

A GPG master key can be associated with multiple subkeys and multiple identities. If your key is well signed there is some value in keeping it as your identity evolves. For example, if you became an Apache Foundation [committer](http://people.apache.org/committer-index.html) you might get an `@apache.org` e-mail address. If you change jobs you might lose one e-mail address but gain another. You don't need to throw away the signatures for the e-mail address you are keeping. Obviously adding a new e-mail address to an existing key does not mean that the new identity is as immediately well trusted.

On an air-gapped machine, import your latest public key and your secret key:

    gpg --import /media/gpg-master/public.asc
    gpg --import /media/gpg-master/secret.asc

Drop in to edit mode for your key:

    $ gpg --edit-key 4FB676F4
    pub  2048R/4FB676F4  created: 2014-05-27  expires: 2017-05-26  usage: SCA
                         trust: ultimate      validity: ultimate
    sub  2048R/4CA0D49E  created: 2014-05-27  expires: 2017-05-26  usage: E
    [ultimate] (1). John Carr <john.carr@example.tld>
    [ultimate] (2)  John Carr <john.carr@corp.tld>

Select the uid you want to revoke:

    gpg> uid 2

    pub  2048R/4FB676F4  created: 2014-05-27  expires: 2017-05-26  usage: SCA
                         trust: ultimate      validity: ultimate
    sub  2048R/4CA0D49E  created: 2014-05-27  expires: 2017-05-26  usage: E
    [ultimate] (1). John Carr <john.carr@example.tld>
    [ultimate] (2)* John Carr <john.carr@corp.tld>

Revoke it:

    gpg> revuid
    Really revoke this user ID? (y/N) y
    Please select the reason for the revocation:
      0 = No reason specified
      4 = User ID is no longer valid
      Q = Cancel
    (Probably you want to select 4 here)
    Your decision? 4
    Enter an optional description; end it with an empty line:
    >
    Reason for revocation: User ID is no longer valid
    (No description given)
    Is this okay? (y/N) y

    You need a passphrase to unlock the secret key for
    user: "John Carr <john.carr@example.tld>"
    2048-bit RSA key, ID 4FB676F4, created 2014-05-27


    pub  2048R/4FB676F4  created: 2014-05-27  expires: 2017-05-26  usage: SCA
                         trust: ultimate      validity: ultimate
    sub  2048R/4CA0D49E  created: 2014-05-27  expires: 2017-05-26  usage: E
    [ultimate] (1). John Carr <john.carr@example.tld>
    [ revoked] (2)  John Carr <john.carr@corp.tld>

Save the changes (this will drop you back to the shell):

    gpg> save

Now to add a new uid:

    $ gpg --edit-key 4FB676F4            
    pub  2048R/4FB676F4  created: 2014-05-27  expires: 2017-05-26  usage: SCA
                         trust: ultimate      validity: ultimate
    sub  2048R/4CA0D49E  created: 2014-05-27  expires: 2017-05-26  usage: E
    [ultimate] (1). John Carr <john.carr@example.tld>
    [ revoked] (2)  John Carr <john.carr@corp.tld>

Use `adduid` to add a new identity:

    gpg> adduid
    Real name: John Carr
    Email address: john.carr@evilcorp.tld
    Comment:
    You selected this USER-ID:
        "John Carr <john.carr@evilcorp.ltd>"

    Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o

    You need a passphrase to unlock the secret key for
    user: "John Carr <john.carr@evilcorp.tld>"
    4096-bit RSA key, ID 4FB676F4, created 2014-05-27


    pub  2048R/4FB676F4  created: 2014-05-27  expires: 2017-05-26  usage: SCA
                         trust: ultimate      validity: ultimate
    sub  2048R/4CA0D49E  created: 2014-05-27  expires: 2017-05-26  usage: E
    [ultimate] (1). John Carr <john.carr@example.tld>
    [ revoked] (2)  John Carr <john.carr@corp.tld>
    [ultimate] (3)  John Carr <john.carr@evilcorp.tld>

Save the changes (this will drop you back to the shell):

    gpg> save

Update your master copy of your public key:

    gpg -a --export 4FB676F4 > /media/gpg-master/public.asc

Then:

 * Copy it onto another removal storage (usb key, sd card, etc) and take it back to a machine with gpg and internets.
 * Import it with `gpg --import /Volume/Untitled/public.asc`
 * Update the public key servers with `gpg --send-key 4FB676F4`
 * Update your keybase [profile](https://keybase.io/jc2k).
 * Update your blog's [copy](http://unrouted.io/gpg/key.txt) of it

FIN.
