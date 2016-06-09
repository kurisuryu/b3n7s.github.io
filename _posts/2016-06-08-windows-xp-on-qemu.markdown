---
layout: post
title:  "Virtualising Windows XP in QEMU"
date:   2016-06-08 12:22:10 +0430
categories: update 
---

Today I am going to share with you how to set up a Windows XP virtual machine in QEMU. Since everything did not went straight away in my case, I might help you to save some time by explaining you the different issues I had.

First of all, why the hell would you, anno 2016, want to set up a Windows XP virtual machine? Well, there might be plenty of reasons actually. But in my case, I was starting the book "Metasploit: The Penetration Tester's Guide" (which looks very promising BTW). In this book, as in many other books on the subject of penetration testing, a vulnerable target system is required to test the vulnerability assessment tools in the book. This is why they ask you to set up an unpatched Windows XP SP2 machine, with no antivirus software and with the firewall turned off.

One way of obtaining this off course is by use of a virtual machine. This saves you from dedicating a physical machine to your setup.

For virtualisation software you have quite some choices, depending on your OS: VirtualBox, VMWare Player,... and QEMU. As I am always trying to head towards the more "power user designed" tools, I wanted to set this whole thing up in QEMU.

So, for the sake of completeness, this is my current environment in which I executed the different commands:

- Host OS: **Arch Linux 4.5.4-1**
- Virtualisation software: **QEMU 2.5.2-1**
- Guest OS: **Windows XP Professional SP2 32-bit**

As mentioned, you will need an old image of Windows XP, as these are not available anymore on Microsoft's website. You will need it under the ISO format. If you have it on CD or DVD, you can find plenty of tutorials online to convert your CD/DVD to an ISO.

So, first up, you will need QEMU of course:

```
$ sudo pacman -S qemu
```

If you are working under another &#42;nix variant, consult its documentation on how to set up QEMU on it.

Now to stay organised when working with QEMU virtual machines, I have my own method. You are of course free to choose yours. What I will do is create a folder called _virtual-machines_ in my home directory. Next I will create a different folder for each virtual machine, in this case called _windows-xp-pro-sp2-32-bit_. In this folder we will put our first file: the ISO of our Windows XP installation DVD.

```
$ mkdir ~/virtual-machines
$ cd ~/virtual-machines
$ mkdir windows-xp-pro-sp2-32-bit
$ cd windows-xp-pro-sp2-32-bit
$ mv <path-to-windows-xp-iso-file> ./windows-xp-pro-sp2-32-bit.iso
```
Now we should create a hard drive for our virtual machine. We will use the _qcow2_ format which has some advantages and disadvantages over the _raw_ format (mainly storage space reduction) which we will not discuss here.

```
$ qemu-img create -f qcow2 windows-xp-pro-sp2-32-bit.img 10G
```
As you can see, we have created a dynamically expendable virtual hard disk, which can expand up to 10 GB, enough for a Windows XP installation.

Now, we will set up all the networking, as we want our virtual machine to be able to communicate with our host machine. This is because we will be running our vulnerability assessment tools on our host machine, and we want to be able to attack our target machine. We only want host-to-target communication (and vice versa). Our target machine does not need to be able to go on the internet.

First we create a bridged interface called _br0_. We will assign an IP address of 192.168.0.1 to this interface and bring it up. We do not care about a default gateway as we will only be communicating on this network. Next, we add a tap interface, called _tap0_, bring it up and add it to our bridged interface. It is this _tap0_ interface that we will connect later on to our virtual machine.

```
$ sudo brctl addbr br0
$ sudo ip addr add 192.168.0.1/24 dev br0
$ sudo ip link set br0 up
$ sudo ip tuntap add dev tap0 mode tap
$ sudo ip link set tap0 up
$ sudo brctl addif br0 tap0
```



When starting off with QEMU, you should understand well that each time that you will launch a virtual machine, you will actually launch a command. This command can have one or more flags, which will mainly determine the conditions of the virtual machine, as its hardware for example. This is why a QEMU command for launching a virtual machine can become quite scary. But do not worry, I will explain the different flags.

Moreover, what I will always do is put my QEMU command in a shell script. This way I don't have to remember the finetuned flags on each launch and it save me a lot of time. To create this shell script:

```
$ cat > windows-xp-pro-sp2-32-bit.sh << EOF
#i/bin/sh
qemu-system-i386 -net tap,ifname=tap0,script=no,downscript=no -net nic,model=rtl8139 -cdrom ../../operating-systems/windows-xp-pro-sp2-32bit.iso  -enable-kvm -drive file=windows-xp-sp2.img,format=qcow2 -m 2G
EOF
$ chmod +x windows-xp-pro-sp2-32-bit.sh
```
Waw, what a command right? So let's go through it:

- **qemu-system-i386**: this binary will launch a 32-bit virtual machine
- **-net tap,ifname=tap0,script=no,downscript=no**: remember *tap0*? Here we are telling Qemu to connect our virtual machine to this interface.
- **-net nic,model=rtl8139**: this one resolved a headache. Without specifying this Realtek NIC card model, another NIC card model will be used for the virtual machine. My Windows XP had no driver for the default NIC, so networking was impossible. However, this Realtek interface card was detected and worked with my Windows XP image.
- **-cdrom windows-xp-pro-sp2-32bit.iso**: this will put the installation ISO in the virtual DVD reader. This is technically only mandatory on the first boot to install the OS.
- **-enable-kvm**: this enables KVM (duh!), which makes for better performance.
- **-drive file=windows-xp-sp2.img,format=qcow2**: here we specify which hard drive image to use
- **-m 2G**: we will give 2 GB of RAM to our virtual machine, which is more than enough for a basic Windows XP installation. Feel free to adapt this value according to your needs. You can go very low with this value, but it will slow your guest OS down.

So, here we are, ready to launch our virtual machine:

```
$ ./windows-xp-pro-sp2-32-bit.sh
```

Now go ahead and install Windows XP. At the end of the installation you can reboot into the freshly installed OS. As mentioned before, after the installation is finished you can safely remove the option **-cdrom ...** from the script. Once you find yourself in the installed Windows XP, go ahead and assign a static IP address of 192.168.0.2/24 to its Ethernet card. Next, try a ping to our host machine:

```
> ping 192.168.0.1
```

You should have 4 replies coming back, which means everything is working well! If not, read everything over again to make sure you have got it all right. If that does not solve your problem, you can always email me for any question.

Enjoy your Windows box ! (Not easy, I know...)
