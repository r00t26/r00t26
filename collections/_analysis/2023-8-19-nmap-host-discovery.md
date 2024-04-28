---
title: Host Discovery with Nmap
---


## Introduction
Host discovery is a critical aspect of offensive cybersecurity efforts, particularly in network pentesting. It enables us to gain insights into the target network by identifying and mapping out the active hosts operating within it. Understanding the hosts on a network is essential for building an effective attack strategy and prioritizing potential targets.
<br><br>
By conducting host discovery, cybersecurity professionals can assess the network's scope, identify potential vulnerabilities, and develop tailored attack vectors. Without this foundational knowledge, attempting to penetrate a network would be akin to navigating blindly.
<br><br>
While numerous tools are available for host discovery, one of the most widely used and versatile tools is **nmap**, which is what we will be using here. If you are not familiar with nmap, please see[**here**](https://nmap.org/book/man.html#man-description).
<br><br>
*`You should also be familiar with IP addresses and subnetting for this article.`*



## Finding Your Subnet
Before you begin running scans on your network, you need to figure out what the subnet is. You can do this by running this command:
```
ip addr show
```
Look for the network interface that is connected to your network (typically named eth0, enp0s3, wlan0, etc.). You will see information such as the IP address, subnet mask, and broadcast address associated with the interface. The subnet information is usually displayed in CIDR notation, which consists of the IP address followed by a forward slash and the subnet mask length (e.g., **192.168.1.0/24**).
<br>
Here is example output forn the **ip addr show** command:
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
     inet 127.0.0.1/8 scope host lo
        valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
     inet 192.168.1.10/24 brd 192.168.1.255 scope global dynamic eth0
        valid_lft 86385sec preferred_lft 86385sec
3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
     inet 192.168.0.101/24 brd 192.168.0.255 scope global dynamic wlan0
        valid_lft 43227sec preferred_lft 43227sec
```
You can also run ifconfig, but ifconfig is not installed on all Linux distros:
```
ifconfig
```


## Host Discovery Techniques
In the following section we are going to talk about the different host discovery techniques and how to put them into use with nmap. Please note that I am not going to dive into each single concept surrounding these techniques. Rather, I will let you explore that on your own. Remember, just like most other commands, nmap has a man (manual) page that you access by running this command:
```
man nmap
```
Just read the man page for a deeper understanding of every flag/option we use. Explaining them indepth here will be pointless as nmap already covers them extremely well. Here is the[**official nmap**](https://nmap.org/book/man-host-discovery.html)page for host discovery - well worth the read!
<br><br>
All of the commands in this article will be assuming that our target network is is **192.168.1.0/24**. Again, you should be familiar with IP's and subnetting for this.



## Ping Sweep
The old traditional ping sweep utilizing ICMP (type 8) echo request packets to check if hosts are online. If a host is alive it responds with an ICMP echo reply (type 0) packet, (assuming the target doesn't block the request).
<br>
This technique is the most basic form of host discovery and tends to work well against poorly secured networks. However, Windows systems block this by default. And many firewalls IPS/IDS systems will also catch this. 
<br>
**To perform a ping sweep with nmap**, simply run:
```
nmap -sn 192.168.1.0/24
```
One other thing to note is that if you are running this while on a local ethernet connection, nmap will send ARP requests instead of ICMP requests. I am not going to dive into why this is, as this will require me explaining ARP (which is for another article).
<br>
To **run the nmap ping sweep using the ICMP protocol whilst on an ethernet connection**, run:
```
nmap -sn 192.168.1.0/24 --send-ip
```


## TCP SYN Ping
This technique involves sending TCP SYN packets to a specific port (often port 80) to check if a host is alive. If a host responds with a TCP SYN/ACK packet, nmap considers the host alive. If a host responds with a TCP RST packet, nmap also considers the host alive. No response at all can indicate that the host is down, our packets are being blocked, or the host is configured to not respond to TCP SYN packets.
<br>
To **perform a TCP SYN Ping**, simply run:
```
nmap -sn -PS 192.168.1.0/24
```
This will by default send the packets to port 80, but you can change this by specifying a custom port or ports after the -PS flag, like this:
```
nmap -sn -PS22 192.168.1.0/24
```
This will send the packets to port 22 instead. You can also specify multiple ports, e.g. **-PS22,21,443**



## UDP Ping
This technique works by sending UDP packets to a specific port to check if a host is alive. This can be effective for hosts that do not respond to ICMP or TCP probes.
<br>
To perform a UDP Ping scan, simply run:
```
nmap -sn -PU 192.168.1.0/24
```
As with the other technique, we can get nmap to send the packets to a specific port:
```
nmap -sn -PU123 192.168.1.0/24
```



## Other Nmap Options
Since nmap is so versatile, we have the option of modifying our commands to suit our needs. 
<br>
To run a host discovery **scan on a list of hosts from a file**, add the **-iL** option followed by the file name:
```
nmap -sn -iL targets.txt
```
We can also **perform two different host discovery scan types at the same time**, simply by supplying their options like this:
```
nmap -sn -PS,21,22,25 -PU137,138 192.168.1.0/24
```
In the above command we are erunning a SYN Ping and UDP Ping on specific ports
<br>
To run a host discovery scan and **save the results in a file**, do:
```
nmap -sn 192.168.1.0/24 -oN /home/scan_results.txt
```
We can also ask nmap to **be more verbose** and display results as the scan is in progress by using the **-v** flag:
```
nmap -v -sn 192.168.1.0/24 
```
Lastly, we can **adjust the timing template**. This is used to specify the timing and performance options for scans. This flag allows us to control the speed and aggressiveness of the scan. We do this by supplying the **-T<0-5>** flag. **-T0** is the slowest option and it is called "paranoid", whilst **-T5** is the fastest option and it is called "insane". The slowest option will take a very long time to complete, however it is also the stealthiest. It is up to your judgement to choose the best option in your scenario. The following command performs a -T4 (aggressive) scan.
```
nmap -sn -T4 192.168.1.0/24[**official nmap**](https://nmap.org/book/man-host-discovery.html)
```


## Nmap Host Discovery Cheatsheet
```
man nmap : view the nmap manual
nmap -sn 192.168.1.0/24 : perform a ping sweep on the specified subnet
nmap -sn 192.168.1.0-10 : perform a ping scan on only the first 10 IP’s on the subnet
nmap -sn 192.168.1.0/24 --send-ip : run ICMP ping scan on local ethernet network
nmap -sn 192.168.1.23 192.168.1.54 : perform a ping scan on 2 specific hosts
nmap -sn -iL targets.txt : perform a ping scan on a list of targets in the specified file
nmap -sn -PS 192.168.1.0/24 : perform a TCP SYN ping scan by sending SYNs to port 80
nmap -sn -PS22 192.168.1.0/24 : perform a TCP SYN ping scan by sending SYNs to port 22
-PS option can also be set to send packets to a range of ports, e.g. -PS1-1000 or -PS443,22,21
-PA option can also be modified to scan specific ports, e.g. -PA1-1000 or -PA443,22
nmap -sn -PU 192.168.1.0/24 : perform a UDP host discovery scan
nmap -sn -v 192.168.1.0/24 : perform a verbose ping sweep 
nmap -sn -PS,21,22,25 -PU137,138 192.168.1.0/24 : perform a SYN ping and UDP ping scan on the given host by sending packets to the provided ports
```


## Conclusion
In this article, we've covered some fundamental host discovery techniques with Nmap, but this is just the tip of the iceberg. Host discovery is a vast and evolving field in cybersecurity, with new techniques and tools emerging constantly. I encourage you to delve deeper into this topic by exploring additional resources, experimenting with different techniques, and participating in cybersecurity communities and forums. 
To learn more about nmap, please see the official nmap web page[**https://nmap.org**](https://nmap.org).




