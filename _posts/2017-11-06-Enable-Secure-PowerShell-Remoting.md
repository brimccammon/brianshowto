---
layout: post
title: Enable Secure PowerShell Remoting
---

Working with a lot of VMs in Azure it has be come more essential to be able to run my scripts on many VMs at a time.  The first step in doing this is to enable remote PowerShell and keep it secure.  Below is the script that I use to enable remote PowerShell and generate a cert to use for encrypting the communication.

````
#Enable PSRemoting and trust all hosts then restart the service
Enable-PSRemoting -SkipNetworkProfileCheck -Force
Set-Item WSMan:\localhost\Client\TrustedHosts -Value * -Force
Restart-Service WinRM

#Create self-signed cert
$cert = New-SelfSignedCertificate -DnsName “$env:computername” -CertStoreLocation cert:\LocalMachine\My

#Create a listener using the new cert
New-Item -Path WSMan:\Localhost\Listener -Transport HTTPS -Address * -CertificateThumbprint $cert.Thumbprint -Force

#Create an entry in Windows Firewall to allow Remote PowerShell over SSL
New-NetFirewallRule -DisplayName “WinRM – 5986” -Direction Inbound -LocalPort 5986 -Protocol TCP -Action Allow
````

Run this on both your source and destination machines.  To run a command remotely run the following:

````
#User Creds
$Domain = “DomainName”
$AdminUser = “UserName”
$AdminPass = “SuperSecretPass”

#Create secure creds
$AdminSecurePass = $AdminPass | ConvertTo-SecureString -asPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential(“$Domain\$AdminUser”,$AdminSecurePass)

#Create session options to work with the self-signed cert
$SessionOptions = New-PSSessionOption –SkipCACheck –SkipCNCheck –SkipRevocationCheck

#Run the Get-Process command on the remote server
Invoke-Command -SessionOption $SessionOptions -UseSSL -ComputerName “ServerFQDN” -Credential $Cred -ScriptBlock {Get-Process}
````