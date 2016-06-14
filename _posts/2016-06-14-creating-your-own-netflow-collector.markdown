---
layout: post
title:  "Creating your own NetFlow monitoring server"
date:   2016-06-14 12:22:10 +0430
categories: update 
---
In this post, I will be explaining you how you can set up your own Netflow monitoring server. From Wikipedia:

> NetFlow is a feature that was introduced on Cisco routers that provides the ability to collect IP network traffic as it enters or exits an interface. By analyzing the data provided by NetFlow, a network administrator can determine things such as the source and destination of traffic, class of service, and the causes of congestion. A typical flow monitoring setup (using NetFlow) consists of three main components:
>
> - Flow exporter: aggregates packets into flows and exports flow records towards one or more flow collectors.
> - Flow collector: responsible for reception, storage and pre-processing of flow data received from a flow exporter.
> - Analysis application: analyzes received flow data in the context of intrusion detection or traffic profiling, for example.

NetFlow allows you thus to get more insight in what is happening on your network. In my case, I had to turn to it for a site I was administrating, which had a very limited bandwidth and a lot of users on it. To detect excessive download and upload behavior, we had to implement a NetFlow solution.

If we want to get a NetFlow environment up and running, we will need the three pieces mentioned above. In my test setup the **flow exporter** will be a Cisco 2811 router. These are the lines I have added to my router configuration:

```
router(config)#ip flow-export version 9
router(config)#ip flow-export destination 192.168.10.253 34343 
router(config)#interface FastEthernet0/1
router(config-if)#ip flow ingress
router(config-if)#ip flow egress
```

You need a working router setup of course before adding these lines. Some important parameters are:

- **The destination IP:** in this case 192.168.10.253
- **The destination UDP port:** the NetFlow data will be sent on this UDP port, in this case port 34343
- **The interface to monitor:** in the case of a NAT-enabled router, you should pick the inside interface as in the inverse case, you would not be able to see the different inside addresses. Under this interface you specify in what direction you want to monitor the flows. We pick both directions.

This is about it for the flow exporter configuration. The **flow collector** will take a little bit more energy.

In our case we will use a Linux server as a flow collector. If you do not have one up and running, you can create one by installing Arch Linux on a computer. Arch Linux is an excellent, minimalist (which is perfect for servers in particular) distribution with great documentation. For an excellent beginners guide head to the [Arch wiki](https://wiki.archlinux.org/index.php/beginners%27_guide). Follow the guidelines in this article to install your server. After completing the beginner's guide, you can configure your server as an SSH server (to configure it over the network) and add a less privileged account, which you put in a sudoers group.

The rest of the tutorial will presume you are working in Arch Linux. You should be able to execute similar commands on other Linux variants. 

To collect the NetFlow data, we will make use of **nfdump**, available in the Arch Linux package repositories. Install it with:

```
$ sudo pacman -S nfdump
```

Next we want to create a folder where we will store all our collected NetFlow data:

```
$ sudo mkdir -p /srv/netflow
```

In this folder, we will create, for the sake of organisation, a folder per device from which we are capturing:

```
$ sudo mkdir -p /srv/netflow/cisco
```
You can name these folders however you want of course.

The daemon that can capture NetFlow date is called **nfcapd**. The basic command to launch a NetFlow capture daemon is:

```
$ /usr/bin/nfcapd -w -D -l /srv/netflow/cisco -p 34343 
```

So, a little explanation of the different options:

- **-w**: Sync file rotation with next 5min (default) interval
- **-D**: Fork to background
- **-l /srv/netflow/cisco**: set the output directory to /srv/netflow/cisco
- **-p 34343**: listen on port 34343

If we run this command, the first files should start appearing in the folder /srv/netflow/cisco, one per every 5 minutes. **nfcapd** will keep the current data in a file named **nfcapd.current**. The past data is stored each time in a file called **nfcapd.YYYYMMDDHHmm**.

Now we want to make a service of this command to enable it (so that it runs at startup). The proper way of doing this is by creating a systemd file:

```
$ sudo cat > /etc/systemd/system/nfcapd-cisco.service << EOF
[Unit]
Description=nfcapd

[Service]
Type=forking
ExecStart=/usr/bin/nfcapd -w -D -l /srv/netflow/cisco -p 34343

[Install]
WantedBy=multi-user.target
EOF
```

Starting this service, will run the nfcapd daemon and enabling it will ensure that nfcapd is launched at every startup (in case of a server reboot):

```
$ sudo systemctl start nfcapd-cisco
$ sudo systemctl enable nfcapd-cisco 
```

Our collector is set up! Now we will analyse our data. For this, we can use the command-line tool **nfdump**, shipped with the **nfdump** package, to do some quick queries. Let's give some examples. To list the top 10 downloaders (measured in bytes downloaded) on your network from 12:00 on April 10th 2016 to 12:00 on April 17th 2016:

```
$ nfdump -R /srv/netflow/cisco/nfcapd.201604101200:nfcapd.201604171200 -s dstip -O bytes -n 10
```

To check to 50 most accessed servers (in terms of bytes downloaded), for a particular IP address (in this case 192.168.10.57):

```
$  nfdump -R /srv/netflow/cisco/nfcapd.201604101200:nfcapd.201604171200 -s srcip -O bytes -n 50 "dst ip 192.168.10.57"
```

We can also export the results to a csv file, which we can then import in a another application, as for example Microsoft Excel (or even better, gNumeric), for further analysis:

```
nfdump -R /srv/netflow/cisco-2811/nfcapd.201605220730:nfcapd.201605270730 -s dstip -O bytes -n 100  -o csv "dst net 192.168.10.0/24" > 20160527-cisco.csv
```
Here, we exported the top 100 downloaders (in terms of downloaded bytes) from 07:30 on May 22nd to 07:30 on May 27th to a csv file. Moreover we add a filter to only see local IP addresses (in the range of 192.168.10.0/24), to filter out upload traffic.

Once you have the NetFlow data, there are almost no limits in the queries you can perform. So play with it. It will learn you more about networking and especially about the network you are monitoring.






