---
layout: post
title: Forefront Protection 2010 for Exchange Blocking winmail.dat
---

When deploying Forefront Protection 2010 for Exchange and enabling file filter the majority if not all emails being sent and received had the body of the message replaced with:

FILE QUARANTINED

Microsoft Forefront Protection for Exchange Server removed a file since it was found to match a filter.
File name: “winmail.dat”
Filter name: “FILE FILTER= Blocked File Extensions: *”

It turns out that the winmail.dat file is how Exchange handles rich text emails.  So there are a couple of options.  Exchange or Outlook can be set to only send plain text emails, which users won’t like, or in the file filter make sure the checkmark for Microsoft Transport Neutral Encapsulation Format is cleared.  This will allow users to send and receive rich text emails and not get the above message.
