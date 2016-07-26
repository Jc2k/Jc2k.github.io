---
layout: post
title: "Finding when a file was added to Git and when it was last changed"
description: "I wanted to annotate a visualisation with when a db migration was added and if it was changed. This is what i did."
category:
tags: ['python', 'git', 'tag', 'history', 'walk']
---

I recently built a visualisation of all Django migrations in a project and the dependencies between them. I was most interested in recent migrations, and in particular if a migration had been changed after it had been deployed. So adding the tag a migration was introduced in (and the tag it was last modified in) seemed like a good idea.

My first attempt was to query each migration with `git log` with a diff filter to find out when it was added. Then I could use `git tag` to see which tags it was in:

```
$ git log --format="format:%H" --follow --diff-filter=A touchdown/core/adapters.py
184d8e88017726e695ee9cb22e428b667f6d22de
$ git tag --contains 184d8e88017726e695ee9cb22e428b667f6d22de
0.0.1
0.0.10
0.0.11
0.0.12
0.0.13
0.0.14
<snip>
```

This was slow. I was traversing the same log again and again 100's of times. So for version 2 I traverse the log in tag order just once.. What changed between 0.0.1 and 0.0.2? What changed between 0.0.2 and 0.0.3?. If it's changed and i've never seen it before then it must be a new file. The new version is much faster.

```python
import subprocess
from distutils.version import StrictVersion


# Finds the root commit of the repository
tags = [
    subprocess.check_output([
        "git", "rev-list", "--max-parents=0", "HEAD"
    ]).strip()
]
# Every tag in order
tags.extend(
    sorted(
        subprocess.check_output(["git", "tag", "-l"]).split(),
        key=StrictVersion,
    ),
)

added = {}
changed = {}

for left, right in zip(tags, tags[1:]):
    files = set(subprocess.check_output([
        "git",
        "show",
        "{}...{}".format(left, right),
        "--no-commit-id",
        "-r",
        "--name-only",
        "--format=format:",
    ]).strip().split())

    for file in files:
        if file not in added:
            added[file] = right
        changed[file] = right


for file, version in added.items():
    print file, version, changed[file]
```

One additional changed I could make is to remove files from `added` and `changed` if they aren't in `git ls-files {tag}`.
