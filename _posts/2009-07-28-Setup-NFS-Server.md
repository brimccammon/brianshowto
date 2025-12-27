---
layout: post
title: Setup NFS Server
---

This is based on CentOS but should be very similar for other distros as well.

If NFS is not installed install it

````
yum install nfs-utils
````

Have NFS start at startup

````
chkconfig nfs on
````

Create a directory to share

````
mkdir /nfs
````

Change the rights on the directory to allow users to create and delete data

````
chmod 777 /nfs
````

Edit /etc/exports to share the NFS directory

````
vim /etc/exports
````

add the following to allow the subnet 192.168.0.0 read/write access to /nfs

````
/nfs        192.168.0.0/24(rw)
````

Restart nfs service

````
service nfs restart
````