---
layout: post
title: "Making a node virtualenv"
description: "I'm spoiled by python virtualenvs and want Node virtualenvs"
category: 
tags: ['node', 'virtualenv']
---
{% include JB/setup %}

These days npm ships with nodejs, which you can install on ubuntu with::

    sudo add-apt-repository ppa:chris-lea/node.js -y
    sudo apt-get update
    sudo apt-get install nodejs

By default I have 2 ways of using npm::

    sudo npm -g install bower

This (depending on how and where npm was installed) installs itself into /usr/local. The thing you installs goes on your PATH. But I don't really let anything near /usr unless it's in a Debian package. So I can also do::

    npm install bower

This will create a ``node_modules`` directory in the current working directory. If i want to run ``bower`` I can just run::

    node_modules/.bin/bower

Which is a bit yuck. Neither of these really cut it for me - I have been spoiled by virtualenv. Can I ``npm`` into my virtualenv - and have ``. bin/activate`` work for the node stuff too?

First of all I want a virtualenv::

    virtualenv /home/john/myvirtualenv

``npm -g`` tries to write into ``/usr/local`` because that is the default ``prefix`` config option. But I need to use ``-g`` so that I can get the ``bin/bower`` to appear. Helpfully you can override the prefix from the command line::

    npm -g --prefix /home/john/myvirtualenv install bower

Now when i ``. bin/activate`` the JS binaries are available too.

Taking this one step further, you can install npm into a virtualenv and then set the default prefix::

    npm -g --prefix /home/john/myvirtualenv install npm

    cat > /home/john/myvirtualenv/lib/node_modules/npm/npmrc << EOF
    prefix /home/john/myvirtualenv
    global true
    EOF

Now I can do this::

    . /home/john/myvirtualenv/bin/activate
    npm install bower lessc requirejs

And the dependencies are installed into that virtualenv - so as long as i have sourced my ``activate`` file I can just::

    bower

I've written ``virtualenv-js`` which does the 2 steps for you. 

You can use it with ``mkvirtualenv`` of virtualenv wrapper by putting it in ``~/.virtualenvs/`` and adding this to ``~/.virtualenvs/postmkvirtualenv``::

    #!/bin/bash
    # This hook is run after a new virtualenv is activated.
    ~/.virtualenvs/virtualenv-js $VIRTUAL_ENV

(Make sure you chmod +x virtualenv-js). (You could also just save virtualenv-js as postmkvirtualenv if you want).

