---
layout: post
title: Visual Studio Code with Visual Studio Team Services (VSTS)
---

I have been doing a ton of PowerShell scripts lately and I was wanting to find a way to tie into a version control system.  Originally I wanted to do something like GitHub but since I want to do a free option GitHub was out as everything is public on their free option.  So I looked at Visual Studio Team Services (VSTS).  They offer 5 users for free and the repositories can be setup as either git or TFS.  So VSTS was it.

My next challenge was PowerShell ISE.  I was wanting to interrelate VSTS into ISE but I did not find a solution that I liked but I did stuble across Visual Studio Code.  VS Code is pretty neat.  It has a ton of addons and can support many languages including PowerShell.  Also there is an addon for VSTS which is nice.

Now that I picked my tools I just had to get them to work together.  The information I could find left a lot to be desired.  There are articles on how to setup and create a project in VSTS.  But what now, once I had my project setup I was stuck.  So this is the reason for this post.  So to be clear right now I have Visual Studio Code installed with the VSTS addon and Git for Windows.  I also have a project setup on the VSTS site.

In VS Code open the Command Palette (Ctrl + Shift + P) and type:

    Git: Clone

Enter the Repository URL which can be found on the project page on VSTS it should be something like https://yourname.visualstudio.com/_git/ProjectName.

Next enter the parent directory.  If you want your project to be stored at C:\git\ProjectName just enter:

    C:\git

Once prompted to login enter the credentials to the Microsoft account that has access to the project.

A prompt asking to Open Repository will come up, click Open Repository.  Now that the project repository has been cloned it is time to signin to VSTS.  To do this open the Command Palette (Ctrl + Shift + P) then type:

    Team: Signin

For ease of use select Authenticate and get an access token automatically, a code will appear so copy that and then click the link https://aka.ms/devicelog…

Paste the code and click continue then sign in using the credentials to the Microsoft account that has access to the project.

VS Code is now connected to a project repository in VSTS.  Now once any changes are made they will show up in Source Control (Ctrl + Shift + G)

![VSCode SCM]({{ "/assets/images/VSCode-SCM-300x227.jpg" }})

and the changes can be staged by clicking the plus next to the file

![VSCode Stage]({{ "/assets/images/VSCode-Stage-300x21.jpg" }})

then they can be committed by clicking the check mark at the top

![VSCode Commit]({{ "/assets/images/VSCode-Commit-300x36.jpg" }})

then sync the commits by clicking the arrows at the bottom.

![VSCode Sync]{{ "(/assets/images/VSCode-Sync.jpg" }})
