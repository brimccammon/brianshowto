---
layout: post
title: OpenFiler as Fiber Channel Target
---

I wanted to post my findings on how to use OpenFiler as a Fiber Channel SAN.  This process isn’t documented as well as I hoped it would have been but the forums on OpenFiler helped a lot but still were lacking.  This post will go from blank box to Fiber Channel LUN being presented by OpenFiler.  My goal was for a faster SAN as I was using iSCSI but it just didn’t meet my expectations.  For the SAN I am using a Qlogic QLA-2342 fiber card and a Brocade Silkworm 3200 fiber switch.

1. Download the latest version of OpenFiler.  In this example I used 2.3.
2. Install OpenFiler with out the fiber channel card installed.
3. Once installed update OpenFiler using conary updateall.
4. Reboot once the updates have completed and make sure everything is working ok.
5. Shutdown the machine and install the fiber card.
6. Get the WWN of the fiber card port(s) by showing the content of /sys/class/fc_host/hostX/port_name
cat /sys/class/fc_host/hostX/port_name
NOTE: X in hostX is the number of the adapter.  If there is more than one port check all WWNs.
7. Edit /etc/init.d/scst and change the chkconfig to # chkconfig: –  99 36
8. Have the scst server run on startup with chkconfig scst on
9. Start the scst service.  service scst start
10. Write the blank config to file with scstadmin –writeconfig /etc/scst.conf
11. Create a logical volume for the LUN.
    1. This example we added a drive just for data /dev/sdb this will make prepare the drive to hold logical volumes for LUNs.  First get the geometry for the disk
    ````
    parted /dev/sdb print
    ````
    2. Label the partition
    ````
    parted /dev/sdb mklabel msdos
    ````
    3. Create the partition, 0.000 is the beginning and 1907328.000 is the end of the geometry from step 1
    ````
    parted /dev/sdb mkpart primary 0.000 1907328.000
    ````
    4. Enable lvm
    ````
    parted /dev/sdb set 1 lvm on
    ````
    5. Create the physical disk
    ````
    pvcreate /dev/sdb1
    ````
    6. Create the group, luns is the example name of the group
    ````
    vgcreate luns /dev/sdb1
    ````
    7. Create the logical volume, 50G is the size in GB, lun1 is the name of the logical volume and luns is the group
    ````
    lvcreate –L 50G –n lun1 luns
    ````
12. Edit /etc/scst.conf.
````
vim /etc/scst.conf
````
    1. Under **[TARGETS disable]** move the HOST entries under **[TARGETS enable]**.  NOTE: The HOST entry WWN should match the WWN(s) from step 6.
    2. Under **[HANDLER vdisk]** add the logical volume created in step 11.
    **DEVICE lun1,/dev/luns/lun1,WRITE_THROUGH,512**
    3. Under **[GROUP Default]** add the USER WWN.  This is the server you want to be able to access the LUN.
    **USER 21:00:00:1b:32:17:87:7e**
    4. Under **[ASSIGNMENT Default]** add DEVICE then the device created in step 12.2 then the LUN number for the device.
    **DEVICE lun1,0**
13. Edit modprobe.conf 
````
vim /etc/modprobe.conf
````
and change **qla2xxx** to **qla2x00tgt**
14. Reboot the OpenFiler machine and when it starts the LUN should be presented to the server.
