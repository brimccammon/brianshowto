---
layout: post
title: Hyper-V on Windows 8 – Wireless Network Does Not Work
---

I recently upgraded one of my laptops to Windows 8 and I noticed that it could run Hyper-V, so I enabled the Hyper-V feature and everything went smooth until I setup the Virtual Switch.  I set it up to use my wireless NIC and I left the check mark for “Allow management operating system to share this network adapter”.  Once it was done setting up the Network Bridge and the vEthernet interface I could no longer access the network.  I did some looking and I was not getting an IP address for the vEthernet interface (which I should have been).  After playing with it for a bit I found that if I disabled the vEthernet interface and then enabled it everything worked.  Then after a reboot it would not work again until I disabled and enabled the interface.  My long term solution to this was to create a PowerShell script to disable the interface then enable it.  I then created a scheduled task to run the script at startup, which has worked well.

For this to work first unsigned PowerShell scripts will need to be allowed to run.  Do this by running the following command in a PowerShell windows as Administrator.

````
Set-ExecutionPolicy unrestricted
````

Next find the name of the vEthernet interface by running the following command in a PowerShell windows.

````
Get-NetAdapter -name vEthernet* | fl
````

The Name field is what we are looking for.

Once all that has been done create a text file and name it something like wififix.ps1 then edit the file and enter the following text.

````
Disable-NetAdapter “vEthernet (Wireless)” -Confirm:$false
Start-Sleep -Seconds 5
Enable-NetAdapter “vEthernet (Wireless)”
````

Make sure to change “vEthernet (Wireless)” to match the Name field from the previous command.

Once that is done save the file then create a Scheduled Task to run the script at startup with the highest privileges.
