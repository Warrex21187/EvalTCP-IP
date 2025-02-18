# **ASSIGNMENT TCP/IP**
## 
### In this ReadMe you will find the summary of all the exercices we had the last 4 weeks. With the differents steps and explanations.
- - -
# Table of Contents
1. [Introduction](#introduction) 

2. [Preparation](#preparation)

3. [Simulation of a complete LAN with routing and DHCP](#simulation-of-a-complete-lan-with-routing-and-dhcp)

4. [Network performance analysis with Wireshark](#network-performance-analysis-with-wireshark)

5. [Comparative study: TCP vs UDP in a simulated environment](#comparative-study-tcp-vs-udp-in-a-simulated-environment)

6. [Resolving a network problem with ICMP and ARP](#resolving-a-network-problem-with-icmp-and-arp)

7. [Theorical Analysis](#theorical-analysis)

8. [Conclusion](#conclusion)

9. [Quizz](#quizz)

10. [Ressources](#ressources)

# Introduction
Following the lesson about TCP/IP protocol we had 4 exercices to practice more on the notions we talked about. The aim of this activity was to look more in depth how protocols worked, how devices communicates between them and how to properly setup a network. In this file you will find all the steps I followed to complete the exercices, the difficulties I faced and the solutions I found. I will then proceed to a critical analysis about the differents protocols (TCP/IP, UDP), and then I will conclude with the knowledge I acquired and the impact of TCP/IP on a network.

# Preparation
First thing I had to do was to prepare 4 virtual machines, one Windows Server 2016, two windows 11 client, and one ubuntu client. For that I had to download the respective ISO for each machine and create a virtual machine on VirtualBox. I also needed to install Cisco Packet Tracer to pursue the last exercices. To use it we had a small lesson by our teacher, I then asked CHATGPT for further informations.

# Simulation of a complete LAN with routing and DHCP
The goal of this exercice was to create a LAN with multiple subnets, all connected with multiples routers, including a DHCP server to dynamicly attribute IP addresses.
So the first step of the operation was to setup the Windows server, so he can has the DHCP and the routing. I started with the DHCP, in network manager you go to the **"manage"** tab, **"Add features and roles"** and in "Role Based Installation" add the DHCP. After the installation is completed, go into the DHCP app. Now we can add a new DHCP scope for each subnet:
- Name: ADMIN, ip range: 192.168.1.100 -> 192.168.1.200 , length: 24, mask: 255.255.255.0, Gateway: 192.168.1.10, DNS: 8.8.8.8, 8.8.4.4
- Name: TECH, ip range: 192.168.2.100 -> 192.168.2.200, length: 24, mask: 255.255.255.0, Gateway: 192.168.2.10, DNS: 8.8.8.8, 8.8.4.4
- Name: COMM, ip range: 192.168.3.100 -> 192.168.3.200, length: 24, mask: 255.255.255.0, Gateway: 192.168.3.10, DNS: 8.8.8.8, 8.8.4.4

  Now that DHCP is set we need to give the server the ability to act like a router. First we need to go back again to **"Add features and roles"** and in "Role Based Installation" select **Routing and remote access**. 
Once the download is completed go into the tools tab and Routing and Remote Access, you got to configure. Select NAT, select the NAT interface of your server, and save the configuration. 
Now in static routing you need to add 3 routes:
- Destination: 192.168.1.0, mask: 255.255.255.0, gateway: 192.168.1.10 
- Destination: 192.168.2.0, mask: 255.255.255.0, gateway: 192.168.2.10
- Destination: 192.168.3.0, mask: 255.255.255.0, gateway: 192.168.3.10
It ensures that each subnet can communicate via the server.
  Now that the router and DHCP are setup, I started my client VMs to see if the DHCP worked and attributed IP to them. But no.
There was one issue because we are using Virtual Machines. We actually needed to change the network interface in each VMs. Without that VMs are by default in NAT so not in the private network
For the Windows Server, the first interface was set to **NAT**, so the access to internet is garanteed for the server, that will be able to give access to internet to the other VMs. The three other interfaces were set to **Internal Network**, and I gave them the same names as the DHCP scopes (ADMIN, TECH and COMM).
So now for each of my client VMs, I set their first and only interface to **Internal Network**, and giving them the same name as the server's interfaces (ADMIN for the first one, TECH for the second, and COMM for the last one.)
Then to check if everything was alright I started each VM, checked if the IP was set by DHCP
```
ip a (for linux)
```
and
```
ipconfig (for windows)
```

Also to check if the routing is working we can ping between the machines using the *ping* command.

Now that the network is fully completed and functionnal let's proceed to the next exercice.
- - -
# Network performance analysis with Wireshark
The aim of this exercice was to analyse TCP and UPC packets to understand their differences and in performance and behaviour.
For this exercice we just have to use the VMs of the previous exercise.
On my windows client I installed Wireshark, then I wanted to listen on the ethernet interface, and in the filtering field I typed the following line to only listen on TCP:
```
tcp.port==80
```
However I had no HTTP address to test on, so I had to create one.
I then opened an HTTP server on my ubuntu VM using the following command:
```
python3 -m http.server 80
```
I then sent a request to connect to the website
```
http://192.168.3.100
```
Here are the Wireshark screenshots:
![Capture d’écran (23)](https://github.com/user-attachments/assets/f55f7026-6677-4d11-9d9b-4e77c00122c7)
![Capture d’écran (25)](https://github.com/user-attachments/assets/7270bd2c-937e-4e13-9ded-5fc96f3b80b9)

We can see there is traffic between 192.168.2.100 (the client where I am requesting to access the website), 192.168.3.100 (the server), and 192.168.3.1 (the gateway).
TCP ensures reliable delivery of packets by establishing a connection (3-way handshake: SYN, SYN-ACK, ACK). 
- You have SYN packets, which are the start of a TCP connection.
- FIN, ACK packets indicate closing of a connection.
- GET / HTTP/1.1 means a client is requesting a web page from a server.
- PSH, ACK packets mean that data is being transmitted (likely the webpage content).
Analyzing performance we can see it's pretty quick, after the get HTTP, the answer is nearly instant.

Now I'm editing the filter bar with:
```
udp.port==5000
```
And with the linux device I am sending UDP packets with nc command:
```
echo "UDP Test" | nc -u 192.168.1.10 5000
```
Here's the wireshark results:

![Capture d’écran (25)](https://github.com/user-attachments/assets/531be553-753f-4556-9cbb-0594ac23c455)

The raw packets been successfully sent.
Unlike TCP, UDP is connectionless, meaning there’s no handshake, no acknowledgment, and no guarantee of delivery. This makes it ideal for real-time applications (e.g., VoIP, gaming, streaming).
So basically TCP is more reliable than UDP because it send again the packet that were lost, while UDP just let the packet get lost. 
TCP also ensure packet come in order, while in UDP packet may come in disorder.
However UDP is way faster than TCP.

- - -
# Comparative study: TCP vs UDP in a simulated environment 

The next exercise needed us to simulate a scenario where TCP and UDP are used by differents application and analyze their impact on the network.
First thing first was to create our network using Cisco Packet Tracer.

I first set up a router with on it's first interface the address 192.168.1.1 and on the second 192.168.1.2.
Then I connected a computer (192.168.1.20) and a server (192.168.1.10) to a switch, connected to the first interface. Then I connected another computer (192.168.2.20) and another server (192.168.2.10) to a switch connected to the other interface. But the devices couldn't communicate and there was only one router so I had to find a way to route. I then added two static route on the router (192.168.1.0 via FastEthernet0/0 and 192.168.2.0 via FastEthernet0/1), now devices could communicate between them.
I then went to the server 192.168.1.10 and in services activated HTTP. I then went into the computer 192.168.2.20, and sent a request to access the HTTP website.
It worked perfectly.
![Capture d’écran (26)](https://github.com/user-attachments/assets/54005c9d-2057-4119-a051-e21062fe3d3d)

Now to simulate a RTP packet (based on UDP), it was a bit more tricky because they were no service on the server to simulate it directly, we had to access the traffic simulator and fill in the fields like this:
![Capture d’écran (27)](https://github.com/user-attachments/assets/dd302245-6645-438f-bf41-75186ca482d0)

Then the packets were sent, however we can see they are ICMP protocol involved, meaning some packet were sent but lost.
![Capture d’écran (28)](https://github.com/user-attachments/assets/a9428257-93bd-42c5-95b9-46a46e4e9cb0)

As we seen before UDP is slower and keep sending packet even if they are not received, while TCP is a bit slower and resend the lost packets.

So TCP is better for bank transaction because it needs to be 100% sure and reliable, while UDP is perfect for streaming because the flow need to be continuous.

- - -
# Resolving a network problem with ICMP and ARP
The aim of this exercise is to use ICMP and ARP to diagnostic and resolve a connectivity problem

First I removed my other device (IP 192.168.1.10) from the DHCP.

A user reports that they cannot access a local server with the IP address 192.168.1.10.  
I ran the following command to check
   ```
   ping 192.168.1.10
   ```
If a response is received: the server is reachable, and the issue may be related to an application or service.  
If no response is received: I proceed to the ARP analysis. (here they were no response)
I started by checking the ARP table:  
   ```
   arp -a
   ```
If a MAC address is associated with `192.168.1.10`, the IP is being resolved, but another issue (firewall, service down, etc.) might be blocking communication.  
If no MAC address is found, send a manual ARP request.  (Here the device isn't on the network so there is no MAC address found)
   
- On **Windows**:  
     ```
     arp -d 192.168.1.10  # Deletes the entry to force a new request
     ```

 - On **Linux/macOS**:  
     ```
     arp -n 192.168.1.10
     ```
     Or send a direct ARP request with:  
     ```
     arping -c 3 -I eth0 192.168.1.10
     ```
     Then ping again to see if the ARP table refreshed


if no MAC address was found check if the server has the correct IP configuration.  
- Ensure the server is powered on and connected to the network.  
- Restart the network interface (`ipconfig /renew` on Windows, `systemctl restart networking` on Linux).  
- Check the DHCP server and manually assign an IP if needed.  
Here the device is just not connected to the DHCP, so to fix it you just have to reconnect it.
---
# Problems and Solutions (for more information refer yourself to the part following the explanation)
In the first exercice I had a problem where my VMs won't connect to DHCP, so I had to change directly their network interfaces on VirtualBox software.( [Simulation of a complete LAN with routing and DHCP](#simulation-of-a-complete-lan-with-routing-and-dhcp) )

In the second exercice the first issue was that I hadn't a HTTP website to access, so I simply check on internet how to create one quickly with a bash command with my Linux VM.
Then I had to find a simple way to analyze UDP packet, so instead of accessing a streaming platform I just used the "nc" command to send packets to the VM I had Wireshark on. ( [Network performance analysis with Wireshark](#network-performance-analysis-with-wireshark) )

In the third exercice my only issue was finding a way to simulate a RTP packet as I didn't knew how to use the packet traffic generator, so I simply checked how to use it on the internet. ( [Comparative study: TCP vs UDP in a simulated environment](#comparative-study-tcp-vs-udp-in-a-simulated-environment) )

# Theorical analysis

TCP/IP is very important in today's networks, it's hardware and OS-independent, allowing devices from different manufacturers and architectures to communicate without issues (here I integrated Windows and Linux devices in my virtual network). It also provides an efficient data routing as IP enables network segmentation into subnets and uses routers to efficiently direct packets across local and global networks. Its also very reliable as TCP ensures reliable transmission by guaranteeing that packets arrive in the correct order and without errors through error-checking mechanisms. It's also felxible with using differents transports protocols suited for different communication types.
TCP usually used for web browsing, emails and file transfers because it's more reliable and garantees file integrity.
And TCP faster but less reliable is used for online gaming, streaming, and VoIP as it need to be continuous and fast.

# Conclusion

This course with these exercises allowed me to plainfully understand this notion of TCP/IP that I only seen theorically at work or during my own research session. I now understand precisely how it works, the pros and cons of using the differents protocols, creating subnets and DHCP. I am now able to fully setup a whole network using TCP/IP methods, as well as maintaining it to be operative.

# QUIZZ

1) A
2) C
3) C
4) C
5) B

# RESSOURCES

[Digital Ocean](https://www.digitalocean.com)

[ChatGPT](https://chatgpt.com)

[WikiPedia](https://wikipedia.org)

[StackOverflow Forums](stackoverflow.com)
