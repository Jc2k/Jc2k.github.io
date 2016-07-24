---
layout: post
title: "Blogging with the aid of docker-compose"
description: "A super simple docker-compose example: How I test changed to my jekyll blog before publishing"
category:
tags: ['docker', 'docker-compose', 'blog', 'jekyll']
---
Much has been said about [Docker](https://www.docker.com), but for me the most transformational aspect of it has been for my dev boxes. I've been using [GitHub Pages](https://pages.github.com/) for a while but i've always resisted testing with jekyll locally - I don't want to mess around with `gem` and have it make a mess of a fairly pristine install just so I can blog something. With Docker i've finally moved past this: today I added a `docker-compose.yml` to this repo:

```yaml
version: '2'
services:
  jekyll:
    image: jekyll/jekyll:pages
    command: jekyll serve --drafts --watch -H 0.0.0.0
    volumes:
      - .:/srv/jekyll
    ports:
      - "4000:4000"
```

When I run `docker-compose up` my checkout is mounted in a docker container and port 4000 is available on `127.0.0.1` to view a preview of my blog. As I edit files jekyll automatically updates itself. I just `Ctrl+C` when i'm done, and if i want to really clean up then I can finish off with `docker-compose down`.

And this is with [Docker for Mac](https://www.docker.com/products/docker#/mac), running a linux container transparently on an OS X machine. And yes, folder watching is working just as well as on Linux! And port forwarding works just as well too.
