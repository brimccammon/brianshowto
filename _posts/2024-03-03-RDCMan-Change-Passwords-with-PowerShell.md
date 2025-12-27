---
layout: post
title: RDCMan Change Passwords with PowerShell
---

I have been using different RDP managers over the years and the last one I was using was Terminals. but the project has not been supported since 2019 and so security issues that come up will never be fixed. I recently moved to a different laptop and decided it was time to look for a different RDP manager. I landed on RDCMan from Sysinternals.

One of the things I like a lot is that I can encrypt the passwords using a cert instead of it being tied to my user and computer. That way I can have it on multiple computers and its always up to date, because I sync the config via OneDrive

The issue I ran into was our PAM solution has a unique username string for every server. RDCMan does not have a feature that will allow you to change just the password to everything in the group. So I started to look into solutions for this. I came across a few GitHub RDCMan Password cracking projects and decided that if they could use PowerShell to decrypt the passwords then I could use PowerShell to encrypt password then replace them in the cfg file.

I was surprised to find out that to get the methods for RDCMan all that I had to do was to rename the RDCMan.exe to RDCMan.dll and import the DLL like a module. So I started out with a variable with the path to my RDCMan.exe and then did a copy and then imported it.

````
$RDCManPath = "C:\Program Files\RDCMan\RDCMan.exe"
Copy-Item -Path $RDCManPath -Destination "$env:TEMP\RDCMan.dll" -Force
Import-Module "$env:TEMP\RDCMan.dll"
````

Once I had that I could encrypt the password with the cert. I just needed the cert thumbprint, create a new object, set the encryption settings and encrypt the password.

````
$Password = "Password1234"
$EncryptionSettings = New-Object -TypeName RdcMan.EncryptionSettings
$EncryptionSettings.EncryptionMethod.Value = 'Certificate'
If ([string]::IsNullOrEmpty($CertThumbprint))
{
    $CertThumbprint = (Get-ChildItem Cert:\CurrentUser\My | Out-GridView -Title "Select Cert for RDC Encryption" -PassThru).Thumbprint
}
$EncryptionSettings.CredentialData.Value = $CertThumbprint

$RDCEncPass = [RdcMan.Encryption]::EncryptString($Password,$EncryptionSettings)
````

Perfect, now I have a password that was encrypted with our cert that can be added to my cfg file. I just need to find the existing passwords and replace them. To do that I need to import the cfg file as XML then I can get the existing entries under my PAM group that the password is not inherited. Then once all the entries are updated I just write the XML back

````
$RDGPath = "C:\Users\User\Documents\RDC.rdg"

$XML = [xml](Get-Content -Path $RDGPath) 

$NeedsChanged = ($XML.RDCMan.file.group | Where-Object {$_.properties.name -eq "PAM"}).group.server.logonCredentials | Where-Object {$_.inherit -eq "none"}
ForEach ($Change in $NeedsChanged)
{
    $Change.password = $RDCEncPass
}

$XML.OuterXml | Set-Content $RDGPath
````

That’s the basics of the script but to make things more secure I wanted to prompt for the password as a secure string and then convert it to plain text so it can be encrypted by RDCMan.

````
$Password = Read-Host "Enter new password" -AsSecureString
$Password = [Net.NetworkCredential]::new('', $Password).Password
````

Other things I wanted to add to make this more versatile are variable for the cert thumbprint, file picker for RDCMan.exe, cfg and an Out-GridView for when the variables are not set. After all that was done here is the full script.

````
$RDCManPath = ""
$RDGPath = ""
$CertThumbprint = ""

$Password = Read-Host "Enter new password" -AsSecureString
$Password = [Net.NetworkCredential]::new('', $Password).Password

If (-not ([AppDomain]::CurrentDomain.GetAssemblies() | Where-Object {$_.Location -like "*RDCMan.dll"}))
{
    If ([string]::IsNullOrEmpty($RDCManPath))
    {
        Add-Type -AssemblyName System.Windows.Forms
        Write-Host "Select the RDCMan.exe from the File Browser dialog box"
        $FileBrowser = New-Object System.Windows.Forms.OpenFileDialog -Property @{ 
            InitialDirectory = [Environment]::GetFolderPath('Desktop') 
            Filter = 'EXE |*.exe'
        }
        $null = $FileBrowser.ShowDialog()
        $RDCManPath = $FileBrowser.FileName
    }
    Copy-Item -Path $RDCManPath -Destination "$env:TEMP\RDCMan.dll" -Force
    Import-Module "$env:TEMP\RDCMan.dll"
}
$EncryptionSettings = New-Object -TypeName RdcMan.EncryptionSettings
$EncryptionSettings.EncryptionMethod.Value = 'Certificate'
If ([string]::IsNullOrEmpty($CertThumbprint))
{
    $CertThumbprint = (Get-ChildItem Cert:\CurrentUser\My | Out-GridView -Title "Select Cert for RDC Encryption" -PassThru).Thumbprint
}
$EncryptionSettings.CredentialData.Value = $CertThumbprint

$RDCEncPass = [RdcMan.Encryption]::EncryptString($Password,$EncryptionSettings)

If ([string]::IsNullOrEmpty($RDGPath))
{
    Add-Type -AssemblyName System.Windows.Forms
    Write-Host "Select the RDG file to update from the File Browser dialog box"
    $FileBrowser = New-Object System.Windows.Forms.OpenFileDialog -Property @{ 
        InitialDirectory = [Environment]::GetFolderPath('Desktop') 
        Filter = 'RDG |*.rdg'
    }
    $null = $FileBrowser.ShowDialog()
    $RDGPath = $FileBrowser.FileName
}

$XML = [xml](Get-Content -Path $RDGPath) 

$NeedsChanged = ($XML.RDCMan.file.group | Where-Object {$_.properties.name -eq "PAM"}).group.server.logonCredentials | Where-Object {$_.inherit -eq "none"}
ForEach ($Change in $NeedsChanged)
{
    $Change.password = $RDCEncPass
}

$XML.OuterXml | Set-Content $RDGPath

Write-Host "$RDGPath PAM group servers has been updated with new password"
````