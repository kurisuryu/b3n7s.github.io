---
layout: post
title:  "Malicous CPLs"
date:   2018-10-28 00:00:00 +0000
categories: 
---

As described in the [Mitre ATT&CK](https://attack.mitre.org/techniques/T1196/), CPL files, which represent control panel items in Windows, can be used for malicious purposes.

A way to do this is to make an ordinary DLL, stuff your malicious code in DLLMain, compile it and change the extension of the compiled DLL from `.dll` to `.cpl`.

By double-clicking the CPL file, the code will be executed.

Moreoever, this is an easy trick if you just want to execute the `DLLMain` code in some DLL and you are to lazy to use `rundll32.exe`.
