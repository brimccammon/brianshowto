---
layout: post
title: Snort Inline on CentOS
---



I have been wanting to setup Snort on a CentOS based firewall for a while and I finally got around to it.  The good thing is I finally got it working thanks to a blog Dennis Panagiotopoulos here, I have confirmed this works for CentOS 6.6 and 7.1.  The problem is as with getting Snort to run inline.  I was unable to find any thing on getting this to work correctly.  So here is what I had to do in addition to Dennis’s blog post:

First thing is to edit /etc/sysconfig/snort so it has both the internal and external interfaces like the following:

INTERFACE=”eth0:eth1″

For me this was line 15 in the file

 

Second thing is to edit /etc/snort/snort.conf uncomment the daq lines and make them like the following:

config daq: afpacket
config daq_dir: /usr/local/lib/daq
config daq_mode: inline
config daq_var: buffer_size_mb=128

For me these lines were 155 though 158 in the file.

 

The last thing I had to do was to edit the service at /etc/init.d/snort to add -Q to the end of the following lines under start like the following:

daemon /usr/sbin/snort $ALERTMODE $BINARY_LOG $NO_PACKET_LOG $DUMP_APP -D $PRINT_INTERFACE -i $i -u $USER -g $GROUP $CONF -l $LOGDIR/$i $PASS_FIRST $BPFFILE $BPF –Q
daemon /usr/sbin/snort $ALERTMODE $BINARY_LOG $NO_PACKET_LOG $DUMP_APP -D $PRINT_INTERFACE $INTERFACE -u $USER -g $GROUP $CONF -l $LOGDIR $PASS_FIRST $BPFFILE $BPF -Q

For me these lines were 115 and 119 in the file.

Once I made the changes I restarted the Snort service and then tested and confirmed Snort was running inline.

 

An easy way to test is to edit /etc/snort/rules/icmp.rules and add:

drop icmp any any -> any any (msg:”ICMP Packet”; sid:477; rev:3;)

Once the rule is added restart snort and then ping from a machine that is on the inside out to the outside and it should return “Destination port unreachable”
