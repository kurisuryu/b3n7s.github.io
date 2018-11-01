---
layout: post
title:  "PsExec on Windows 10"
date:   2018-10-28 00:00:00 +0000
categories: 
---

Just some notes on making PsList and PsExec work on Windows 10 (between computers not in domain). 

# PsList

* Enable Remote Registry service on target machine (will be started automatically when PsList executes)
* Turn on File and Printer sharing in "Advanced sharing settings" in Network and Sharing center on target machine
* Create registry value `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\system\LocalAccountTokenFilterPolicy` with value 1.
* On host machine: 

```
pslist -u desktop-3733rlc\benoit \\172.16.52.185
```

# PsExec

* Turn on File and Printer sharing in "Advanced sharing settings" in Network and Sharing center on target machine
* Create registry value `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\system\LocalAccountTokenFilterPolicy` with value 1.
* On host machine: 

```
psexec -u desktop-3733rlc\benoit \\172.16.52.185 cmd
```


