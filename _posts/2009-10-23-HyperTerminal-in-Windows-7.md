---
layout: post
title: HyperTerminal in Windows 7
---

I have talked to a lot of professionals and one of the big complaints about Windows 7 is the lack of HyperTerminal.  So here is how to get HyperTerminal on to Windows 7 (32 or 64-bit).

On the Windows 7 box make a new folder under C:\Program Files\HyperTerminal for 32-bit and for 64-bit make a new folder C:\Program Files (x86)\HyperTerminal

From a Windows XP box and copy the following 3 files to the folder that was just created on the Windows 7 box:

C:\Program Files\Windows NT\hypertrm.exe
C:\WINODWS\system32\hypertrm.dll
C:\WINODWS\Help\hypertrm.chm

Now just run hypertrm.exe and HyperTerminal is on Windows 7.

If you want to have HyperTerminal on your Start Menu just create a shortcut to hypertrm.exe and put it inC:\ProgramData\Microsoft\Windows\Start Menu\Programs and when you go to All Programs under the Start Menu HyperTerminal will be there.

The only issue that I have noticed is that when creating a connection the icons do not show, but in my opinion that is a non issue.

A quick side note if you have problems getting your USB to Serial connection working under Windows 7 try getting drivers from http://www.prolific.com.tw/eng/downloads.asp?ID=31.  The PL2303_Prolific_DriverInstaller_v10518.zip worked great for me.
