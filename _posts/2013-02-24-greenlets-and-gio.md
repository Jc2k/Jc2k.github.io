---
layout: post
title: "Greenlets and Gio"
description: "I'm spoiled by python virtualenvs and want Node virtualenvs"
category:
tags: ['python', 'greenlets', 'gio']
---
I made a [thing](https://github.com/Jc2k/cringe)! It's like gevent but on top of the GLib stack instead of libev.

Most GLib async code has a sync and async variant, and the same pattern is used. So we can hook into ``GObjectMeta`` and automatically wrap them in something like this:

```python
def some_api(*args, **kwargs):
    current = greenlet.getcurrent()

    # Call the function and pass it current.switch as the callback - this
    # is what allows the current coroutine to be resumed
    args = args + (current.switch, None)
    some_api_async(*args, **kwargs)

    # Pause the current coroutine. It will be resumed here when the
    # callback calls current.switch()
    obj, result, _ = current.parent.switch()

    # Actually return the expected value
    return some_api_finish(result)
```

So all use of the synchronous API's that have an asynchronous version would be replaced by a asynchronous greenlet version.
