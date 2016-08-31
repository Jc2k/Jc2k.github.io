---
layout: post
title: "Private Sphinx docs with heroku and GitHub"
description: "Using a heroku buildpack and flask app to have private Sphinx docs online"
category:
tags: ['heroku', 'sphinx', 'github', 'flask', 'oauth', 'private', 'free']
---

Wouldn't it be great if you could push your sphinx docs to GitHub and in moments Heroku would have deployed them. Not only that but they are private: You need to be a member of a GitHub org to access them.

Last year I made [heroku-buildpack-sphinx](https://github.com/Jc2k/heroku-buildpack-sphinx).

At its core is a [single file flask app](https://github.com/Jc2k/heroku-buildpack-sphinx/blob/master/vendor/protect_and_serve.py) that serves static files via [send_from_directory](http://flask.pocoo.org/docs/0.11/api/#flask.send_from_directory) - but only if a user is logged in via GitHub. The build pack deploys a virtualenv and installs sphinx, builds your sphinx package and then copies in the flask app. Your docs are then served via gunicorn.
