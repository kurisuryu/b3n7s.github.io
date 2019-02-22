---
layout: post
title:  "WinDbg default workspace"
date:   2018-11-15 00:00:00 +0000
categories: 
---

In order to set up a default WinDbg workspace:

* Open WinDbg. Do NOT attach to a process, launch an executable, etc.
* Set up everything as you want.
* Select `File -> Save Workspace`
* If you want a backup of your setup, export the registry key under `HKCU\Software\Microsoft\WinDbg\Workspaces\Default`

My current setup is this one:

```
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\Software\Microsoft\Windbg\Workspaces]
"Default"=hex:57,44,57,53,01,00,00,00,44,00,00,00,10,00,04,00,01,00,00,00,4c,\
  00,55,00,04,00,03,00,10,00,04,00,00,00,00,00,00,00,00,00,04,00,01,00,70,02,\
  68,02,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\
  00,00,00,00,00,00,00,00,00,ff,ff,ff,0f,00,00,00,00,00,00,00,00,01,00,00,00,\
  01,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,05,\
  00,00,00,ff,ff,ff,0f,01,00,00,00,00,00,00,00,02,00,00,00,00,00,00,00,00,00,\
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,ff,ff,ff,\
  0f,00,00,00,00,00,00,00,00,03,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,ff,ff,ff,0f,00,00,00,00,00,\
  00,00,00,04,00,00,00,01,00,00,00,01,00,00,00,20,00,00,00,4a,00,00,00,dc,02,\
  00,00,da,01,00,00,03,00,00,00,00,00,00,80,00,00,00,00,00,00,00,00,05,00,00,\
  00,01,00,00,00,01,00,00,00,a6,ff,ff,ff,87,01,00,00,62,02,00,00,17,03,00,00,\
  01,00,00,00,00,00,00,80,00,00,00,00,00,00,00,00,06,00,00,00,01,00,00,00,01,\
  00,00,00,10,00,00,00,3a,00,00,00,cc,02,00,00,ca,01,00,00,03,00,00,00,00,00,\
  00,00,05,00,00,00,00,00,00,00,07,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,ff,ff,ff,0f,00,00,00,00,\
  00,00,00,00,08,00,00,00,01,00,00,00,01,00,00,00,30,00,00,00,5a,00,00,00,ec,\
  02,00,00,ea,01,00,00,04,00,00,00,02,00,00,00,04,00,00,00,00,00,00,00,09,00,\
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\
  00,00,00,00,00,ff,ff,ff,0f,00,00,00,00,00,00,00,00,0a,00,00,00,00,00,00,00,\
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,ff,\
  ff,ff,0f,00,00,00,00,00,00,00,00,0b,00,00,00,00,00,00,00,00,00,00,00,00,00,\
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,ff,ff,ff,0f,00,00,00,\
  00,00,00,00,00,0c,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\
  00,00,00,00,00,00,00,00,00,00,00,00,ff,ff,ff,0f,00,00,00,00,00,00,00,00,0d,\
  00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,00,\
  00,00,00,00,00,00,ff,ff,ff,0f,00,00,00,00,00,00,00,00,00,00,03,00,10,00,08,\
  00,04,00,00,00,70,00,00,00,01,00,01,00,38,00,2c,00,2c,00,00,00,02,00,00,00,\
  03,00,00,00,ff,ff,ff,ff,ff,ff,ff,ff,ff,ff,ff,ff,ff,ff,ff,ff,00,00,00,00,ac,\
  00,00,00,80,04,00,00,5c,03,00,00,65,00,72,00,03,00,03,00,80,00,78,00,05,00,\
  00,00,00,00,00,00,64,00,00,00,14,00,00,00,00,00,00,80,01,00,00,40,07,00,00,\
  00,00,00,00,00,00,00,00,00,96,02,00,00,fa,02,00,00,05,00,00,00,01,00,00,00,\
  01,00,00,00,a6,ff,ff,ff,87,01,00,00,62,02,00,00,17,03,00,00,01,00,00,00,00,\
  00,00,80,00,00,00,00,00,00,00,00,ff,ff,ff,0f,ff,ff,ff,0f,ff,ff,ff,0f,ff,ff,\
  ff,0f,00,00,00,00,01,00,00,00,00,00,00,00,01,00,00,00,03,00,03,00,80,00,74,\
  00,06,00,00,00,01,00,00,00,64,00,00,00,14,00,00,00,00,00,00,80,01,00,00,40,\
  06,00,00,00,96,02,00,00,00,00,00,00,ea,04,00,00,fa,02,00,00,06,00,00,00,01,\
  00,00,00,01,00,00,00,10,00,00,00,3a,00,00,00,cc,02,00,00,ca,01,00,00,03,00,\
  00,00,00,00,00,00,05,00,00,00,00,00,00,00,00,00,00,80,02,00,00,40,00,00,00,\
  00,01,00,00,00,01,00,00,00,ec,ff,ff,ff,00,00,00,00,3b,00,43,00,03,00,03,00,\
  78,00,6c,00,04,00,00,00,02,00,00,00,64,00,00,00,14,00,00,00,00,00,00,80,03,\
  00,00,40,07,00,00,00,ea,04,00,00,00,00,00,00,9c,05,00,00,5c,01,00,00,04,00,\
  00,00,01,00,00,00,01,00,00,00,20,00,00,00,4a,00,00,00,dc,02,00,00,da,01,00,\
  00,03,00,00,00,00,00,00,80,00,00,00,00,00,00,00,00,00,00,00,80,ff,ff,ff,0f,\
  01,00,00,40,03,00,00,40,01,00,00,00,00,00,00,00,03,00,03,00,90,00,84,00,08,\
  00,00,00,03,00,00,00,64,00,00,00,14,00,00,00,00,00,00,80,03,00,00,40,07,00,\
  00,00,ea,04,00,00,5c,01,00,00,9c,05,00,00,fa,02,00,00,08,00,00,00,01,00,00,\
  00,01,00,00,00,30,00,00,00,5a,00,00,00,ec,02,00,00,ea,01,00,00,04,00,00,00,\
  02,00,00,00,04,00,00,00,00,00,00,00,00,00,00,80,02,00,00,40,02,00,00,00,03,\
  00,00,00,02,00,00,00,00,00,00,00,08,00,00,00,01,00,00,00,01,00,00,00,65,00,\
  73,00,70,00,00,00,00,00,00,00,01,00,03,00,10,00,04,00,04,00,00,00,00,00,00,\
  00
```

