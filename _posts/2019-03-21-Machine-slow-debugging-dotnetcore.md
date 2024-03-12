---
layout: post
title:  "Debugging a couple of AspNetCore websites and your machine gets slow?"
date:   2019-03-21 10:06:00 +1000
categories: [dotmet, development]
permalink: /blog/debugging-a-couple-of-aspnetcore-websites-on-windows-10-and-your-machine-gets-slow/
---


I am developing an asp.net core application comprised of 4 microservices.

Every day at about 3pm my machine gets a little slow.

I thought it may be antivirus or something but checked and nothing was running.

looking in task manager I could see that there was a number of **.NET Core Host** processes still running.

![task manager showing .net core host background processes][1]

Usually I run 2 of the asp.net web aps from the commandline with ```dotnet watch run```
The other 2 I run with Visual Studio debug because I want to be able to set breakpoints and catch errors.
I always assumed that when you stop debugging it killed the processes but it seems that they don't

Luckily with a bit of powershell it is easy enough to kill the processes.
You can see what dotnet processes are running with:

```
Get-Process dotnet
```


That result can then be piped to the **Stop-Process** cmdlet:

```
Get-Process dotnet* | Stop-Process
```

You will need powershell running as administrator. I usually have the PowerShell ISE running to run azure command like restarting app service etc.
It will also kill any other apps running in open command windows so you will have to run those again.



I realise this is another reason to switch to using docker but I will have to put it to the team first.


  [1]: /media/1022/k2tv7033uth21.png