---
layout: post
title: SSH Without a Password
---


On the client generate a new rsa ssh key.

NOTE: When prompted for a passphrase leave blank (just hit enter).

````
ssh-keygen -t rsa
````

From the client append the new key to the servers .ssh/authorized_keys file.

````
cat ~/.ssh/id_rsa.pub | ssh root@hostname_or_ip ‘cat >> ~/.ssh/authorized_keys’
````

NOTE: Enter the root password on the server for the last time.

Now ssh into the server

````
ssh -l root hostname
````

It should log in without any prompt for a password.