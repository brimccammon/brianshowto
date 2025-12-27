---
layout: post
title: XenServer - Change Bonding Mode
---

By default XenServer 5.5 uses source-based ARP load balancing (balance-slb or mode 7) for bonding.  If this is needed to be changed to a different mode all that is needed to do is edit the file /opt/xensource/libexec/interface-reconfigure on line 863.  Do this using the following command:

````
vi /opt/xensource/libexec/interface-reconfigure +863
````

once there the value for mode can be changed from “balance-slb” to any supported mode.  Here is a list of the modes:

balance-rr
active-backup
balance-xor
broadcast
802.3ad
balance-tlb
balance-alb
balance-slb

See Linux Channel Bonding for more information on the types of bond modes.  Unfortunately I could not find much information on balance-slb.

To check if the bond mode change worked correctly run

````
cat /proc/net/bonding/bond<bondnumber>
````

Run ifconfig to find the bondnumber.

This will list the bond mode and the slave interfaces to the bond.
