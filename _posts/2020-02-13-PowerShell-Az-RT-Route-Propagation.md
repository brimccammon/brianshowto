---
layout: post
title: PowerShell – Create Azure Route Table with Virtual network gateway route propagation
---

Recently I was writing a script to create Azure Route Tables and I kept having issue with getting virtual network gateway route propagation to be enabled.

The example I was working from was:

````
New-AzRouteTable -Name $RouteTableName -ResourceGroupName $ResourceGroupName -Location $Location -DisableBgpRoutePropagation
````

I was finally able to track down what was causing my issue. The DisableBgpRoutePropagation equates to disabling virtual network gateway route propagation in the Azure Portal. So the correct command to have virtual network gateway route propagation enabled is:

````
New-AzRouteTable -Name $RouteTableName -ResourceGroupName $ResourceGroupName -Location $Location
````

There was not a lot of information about this that I could find so I hope this helps whom ever finds it.
