---
layout: post
title:  "Windows kernel debugging under VMWare Fusion"
date:   2017-11-01 15:24:00 +0100
categories: 
---

I will explain here how to set up your lab for debugging the Windows kernel on VMWare Fusion (on macOS). As the debugger, I will use a Windows 10 x64 machine and as the debuggee a Windows 7 x86 machine. These instructions might work with other versions of Windows with no, minor or major adjustments. Let's go:

1. Shut down both virtual machines. We will edit the VMX files of these VM's and I have noticed unwanted behavior when you edit these files while the corresponding virtual machine is on.

2. Edit the VMX file of the debugger machine (Windows 10 x64 in our case). Remove all lines containing 'serial0'. Add the following lines:

	```
	serial0.present = "TRUE"
	serial0.fileType = "pipe"
	serial0.fileName = "/tmp/serial"
	serial0.tryNoRxLoss = "FALSE"
	serial0.pipe.endPoint = "client"
	```

3. Edit the VMX file of the debuggee machine (Windows 7 x86 in our case). Remove all lines containing 'serial0'. Add the following lines:


	```
	serial0.present = "TRUE"
	serial0.fileType = "pipe"
	serial0.fileName = "/tmp/serial"
	serial0.tryNoRxLoss = "FALSE"
	serial0.pipe.endPoint = "server"
	```

4. Boot the debuggee machine. Launch an administrative command prompt. Type following commands:

	```
	bcdedit.exe /debug {current} on 
	bcdedit /dbgsettings serial debugport:1 baudrate:115200
	```

  Shut down the debuggee machine.

5. Boot the debugger machine. Download the Windows 10 SDK from the [official Microsoft website](https://developer.microsoft.com/en-us/windows/downloads/windows-10-sdk). Launch the downloaded installer. You should only check 'Debugging tools for Windows' during installation. This will install WinDbg.

6. On the debugger machine, launch WinDbg x86 (the 32-bit version). Select File > Kernel Debug... . Go to the tab 'COM'. Make sure baud rate is set to 115200 and port is set to com1. Pipe and reconnect should NOT be checked. Click 'Ok'.

7. Boot the debuggee machine. Head back to the debugger machine. You should see some output on your WinDbg console, which indicates that the kernel debugger connection was established.

Now you can start debugging the kernel of your Windows 7. Have fun!
