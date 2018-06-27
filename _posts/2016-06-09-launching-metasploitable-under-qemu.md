---
layout: post
title:  "Launching Metasploitable under Qemu"
date:   2016-06-09 12:22:10 +0430
categories: update 
---

Today's post is going to be a slight continuation of yesterday's post, called *Virtualising Windows XP in Qemu*.
In that post we saw how to get an old Windows XP up and running under Qemu. Here, we are gonna do the same, but with a "distribution" called Metasploitable, which is Linux image full of vulnerabilities for pentesting purposes.

It might be interesting for you to read my post of yesterday, as a lot of stuff will be coming back and I will cover it more quickly.

What do you need?

- Host OS: in my case, **Arch Linux**
- Virtualisation software: here **Qemu**
- **Metasploitable** Linux, freely available at [sourceforge.net](https://sourceforge.net/projects/metasploitable)

After downloading Metasploitable, you will obtain a ZIP file called **metasploitable-linux-2.0.0.zip**. You will need this later on.

As in yesterday's post, let's start of with making a specific directory for our virtual machine:

```
$ mkdir -p ~/virtual-machines/metasploitable
```

Next we will unzip our downloaded archive file. As Metasploitable is an OS, I like to keep it in a dedicated directory, called **operating-systems**:

```
$ mkdir ~/operating-systems
$ mv ~/Downloads/metasploitable-linux-2.0.0.zip ~/operating-systems/
$ cd ~/operating-systems/
$ unzip metasploitable-linux-2.0.0.zip
```
This will create a directory **Metasploitable2-Linux**. Let's see what is in this directory:

```
$ ls Metasploitable2-Linux/
Metasploitable.nvram  Metasploitable.vmdk  Metasploitable.vmsd  Metasploitable.vmx  Metasploitable.vmxf
```

What is great with Qemu, is that we will only need the vmdk file, which is the hard drive. The other files contain settings for running Metasploitable under other virtualisers. So let's move our virtual hard disk to our virtual machine folder and remove the other files:

```
$ mv Metasploitable2-Linux/Metasploitable.vmdk ~/virtual-machines/metasploitable/metasploitable.vmdk
$ rm -rf Metasploitable2-Linux
```
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

Now let's move to our virtual machine folder and create a launch script for our VM:

```
$ cd ~/virtual-machines/metasploitable
$ cat > metasploitable.sh << EOF
#!/bin/sh
qemu-system-i386 -drive file=metasploitable.vmdk -enable-kvm -m 2G  -net tap,ifname=tap0,script=no,downscript=no -net nic,model=rtl8139 
EOF
$ chmod +x metasploitable.sh
```

The different options for our virtual machine are:

- **-drive file=metasploitable.vmdk**: use the freshly downloaded vmdk as our virtual hard disk drive
- **-enable-kvm**: for better performance
- **-m 2G**: dedicate 2 GB of RAM to our VM
- **-net tap,ifname=tap0,script=no,downscript=no**: connect our VM to the tap0 interface
- **-net nic,model=rtl8139**: use the NIC card model RTL8139 for our VM

Now we can launch Metasploitable:

```
$ ./metasploitable.sh
```

Once Metasploitable has finished booting, you can log in with the username "msfadmin" and the password "msfadmin". Be careful though, it is by default configured for a QWERTY keyboard. Once you log in, you can change this to a Belgian keyboard for example:

```
msfadmin@metasploitable:~$ sudo loadkeys be
```
When asked for a password, simply enter "msfadmin".

Now we will configure a static IP address for our VM:

```
msfadmin@metasploitable:~$ sudo ip addr add 192.168.0.3/24 dev eth0
```

You should be able to ping successfully your host machine at 192.168.0.1.



