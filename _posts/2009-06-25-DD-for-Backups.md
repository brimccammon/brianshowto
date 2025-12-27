---
layout: post
title: DD for Backups
---

I wanted a way to create a backup of my whole hard drive or just a partition of the drive.  In windows I use Ghost or Acronis but for Linux I wanted something free and easy to use.  DD is build into Linux and works great.  I like that it is possible to compress the image with gzip.  Here is how to do backups with DD.

Compressed Backup

```
dd if=/dev/hdx | gzip > /path/to/image.gz
```

Change hdx for the hard drive to backup.

Restore Backup of hard disk copy

```
dd if=/path/to/image of=/dev/hdx

gzip -dc /path/to/image.gz | dd of=/dev/hdx
```

MBR backup

Backup MBR and partition table.

```
dd if=/dev/hdx of=/path/to/image count=1 bs=512
```

MBR restore

```
dd if=/path/to/image of=/dev/hdx
```

Add “count=1 bs=446″ to exclude the partition table from being written to the disk

