---
layout: post
title: Ubuntu Server Disk Resize
---

I have had the need to expand the disk an Ubuntu Server so I thought I would lay out the steps.  First step which I won't cover is to expand the disk, for me this is though my hypervisor.  Once the virtual disk has been expanded proceed with the following steps.

1. Verify the OS is seeing the drive has extra space on it
````
lsblk
````
    Example output
    ````
    NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS  
    loop0                       7:0    0  55.5M  1 loop /snap/core18/2959  
    loop1                       7:1    0  55.5M  1 loop /snap/core18/2976  
    loop2                       7:2    0  63.8M  1 loop /snap/core20/2682  
    loop3                       7:3    0  63.8M  1 loop /snap/core20/2686  
    loop4                       7:4    0    74M  1 loop /snap/core22/2163  
    loop5                       7:5    0  91.4M  1 loop /snap/lxd/36558  
    loop6                       7:6    0    74M  1 loop /snap/core22/2193  
    loop7                       7:7    0  91.4M  1 loop /snap/lxd/36918  
    loop8                       7:8    0   375M  1 loop /snap/nextcloud/51274  
    loop9                       7:9    0 375.4M  1 loop /snap/nextcloud/51617  
    loop10                      7:10   0  76.6M  1 loop /snap/powershell/313  
    loop11                      7:11   0  76.6M  1 loop /snap/powershell/316  
    loop12                      7:12   0  50.8M  1 loop /snap/snapd/25202  
    loop13                      7:13   0  50.9M  1 loop /snap/snapd/25577  
    sda                         8:0    0   500G  0 disk    
    ├─sda1                      8:1    0     1G  0 part /boot/efi  
    ├─sda2                      8:2    0     2G  0 part /boot  
    └─sda3                      8:3    0 396.9G  0 part    
     └─ubuntu--vg-ubuntu--lv 252:0    0 396.9G  0 lvm  /  
    sr0                        11:0    1  1024M  0 rom     
    ````
    On device sda the size is 500G but the partitions only add up to 399.9G, so we have 100G of unpartitioned space.

2. Now use growpart to expand the sda3 partition
````
sudo growpart /dev/sda 3
````
    Example Output
    ````
    CHANGED: partition=3 start=6397952 old: size=832462815 end=838860766 new: size=1042178015 end=1048575966  
    ````
    It shows that partition 3 was changed to the new size.

3. Once the partiton has been expanded use pvresize to update the physical volume.
````
sudo pvresize /dev/sda3
````
    Example Output
    ````
    Physical volume "/dev/sda3" changed
    1 physical volume(s) resized or updated / 0 physical volume(s) not resized
    ````

4. Once the volume has been resized extend the logical volume to 100%
````
sudo lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
````
    Example Output
    ````
    Size of logical volume ubuntu-vg/ubuntu-lv changed from <396.95 GiB (101618 extents) to <496.95 GiB (127218 extents).  
    Logical volume ubuntu-vg/ubuntu-lv successfully resized.  
    ````

5. Finally it's time to grow the file system.
````
sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
````
    Example Output
    ````
    resize2fs 1.47.0 (5-Feb-2023)  
    Filesystem at /dev/mapper/ubuntu--vg-ubuntu--lv is mounted on /; on-line resizing required  
    old_desc_blocks = 50, new_desc_blocks = 63  
    The filesystem on /dev/mapper/ubuntu--vg-ubuntu--lv is now 130271232 (4k) blocks long.  
    ````

6. Lastly verify the free space is being shown
````
df -h
````
    Example Output
    ````
    Filesystem                         Size  Used Avail Use% Mounted on  
    tmpfs                              392M  988K  391M   1% /run  
    efivarfs                           128M   26K  128M   1% /sys/firmware/efi/efivars  
    /dev/mapper/ubuntu--vg-ubuntu--lv  489G  391G   78G  84% /  
    tmpfs                              2.0G     0  2.0G   0% /dev/shm  
    tmpfs                              5.0M     0  5.0M   0% /run/lock  
    /dev/sda2                          2.0G  198M  1.6G  11% /boot  
    /dev/sda1                          1.1G  6.2M  1.1G   1% /boot/efi  
    tmpfs                              392M   12K  392M   1% /run/user/1000
    ````
    On the partiton mounted as / it shows the new size of 489G.
