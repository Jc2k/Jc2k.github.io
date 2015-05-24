---
layout: post
title: "Raciness in Amazon RDS backups"
description: "Want to know what the earliest point in time restore you can do is? It's not straightforward."
category:
tags: ['aws', 'rds', 'postgres', 'backup']
---
I recently wrote a restore script for an AWS RDS instance. With RDS you spin up a new instance from a backup (rather than restoring into the existing instance). So you can:

 1. Rename the original out of the way (`foo` -> `foo-old`) then restore the backup in it's place. The final server will have the same hostname.
 2. Restore the backup with a new name (`foo-2`). This means it will have a different hostname and you'll need to deploy a settings change to your apps.

I went with option (1) so that I always knew which database was 'active'. That meant doing as much validation up front as possible. You don't want to move the existing database and then a few minutes in to the script have it fail, finding out that you asked it to restore to a point in time last year!

So how do you validate the target restore date? You can't restore to 30s ago - there is a lag of a few minutes. And obviously you can't restore before the database existed. But also you can't restore before the oldest backup.

With botocore the first part of this is easy:

```python
result = client.describe_db_instances(DBInstanceIdentifier=dbname)
db = result['DBInstances'][0]
if target > db['LatestRestorableTime']:
    raise ValueError("The target time is too recent")
if target < db['InstanceCreateTime']:
    raise ValueError('Cannot restore to before the db was created')
```

Unfortunately there isn't an `EarliestRestorableTime`. As far as I can tell you can use the `SnapshotCreateTime` time of earliest backup:

```python
result = client.describe_db_snapshots(DBInstanceIdentifier=dbname)
snapshots = result.get('DBSnapshots', [])
snapshots.sort(key=lambda snapshot: snapshot['SnapshotCreateTime'])
if not snapshots or target < snapshots[0]['SnapshotCreateTime']:
    raise ValueError("Can't restore before the first backup")
```

But that's still not enough. When you are testing your backup restore script you run it a lot. And what I found was that this frequently didn't stop me passing in an invalid date. What I noticed is that if you run it in quick succession there are still snapshots from the **previous** instance of `foo` hanging around. The only way to tell which snapshots belong to **this** instance is to filter on the `InstanceCreateTime`:

```python
result = client.describe_db_snapshots(DBInstanceIdentifier=dbname)
snapshots = result.get('DBSnapshots', [])
snapshots = filter(
    lambda s: s['InstanceCreateTime'] == db['InstanceCreateTime'],
    snapshots,
)
snapshots.sort(key=lambda snapshot: snapshot['SnapshotCreateTime'])
if not snapshots or target < snapshots[0]['SnapshotCreateTime']:
    raise ValueError("Can't restore before the first backup")
```

Grim.
