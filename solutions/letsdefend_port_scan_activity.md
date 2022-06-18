# Let's Defend Port Scan Activity - https://app.letsdefend.io/challenge/port-scan-activity/

This challenge entails detecting a malicious user performing a port scan on a network by using Wireshark!

1. What is the IP address scanning the environment?

`10.42.42.253`

In order to connect to a host via TCP, a three-way handshake must be completed which begins with an initiating host beginning the connection by sending a *TCP SYN* packet.

If we skim through the packet capture, it becomes apparent that there's mainly 1 host who is initiating connections by sending multiple *TCP SYN* packets to different destination IP addresses:

![](https://i.imgur.com/RFBaZet.png)

By looking at this, we can determine that the host `10.42.42.253` is initiating these connections, but never completing these connections by sending a final *ACK* packet. The malicious user never completes the connection because they simply want to check whether a host exists at the destination IP address. 

If the host responds, it exists. Otherwise if there's no response, the host doesn't exist.

2. What is the IP address found as a result of the scan?

`10.42.42.50`

The first host that responds to the *TCP SYN* packets would be a discovered host, even if that response is a reset packet.

If we look at the packet capture, we can see that the first *TCP SYN* packet is sent to a host at `10.42.42.50` who then immediately responds with a reset packet:

![](https://i.imgur.com/RnCDvzr.png)

Since `10.42.42.50` responds with this reset packet, the attacker can determine that a host exists at this IP address, otherwise there would be no response.

3. What is the MAC address of the Apple system it finds?

`00:16:cb:92:6e:dc`

This task can be solved by looking at the Ethernet section of each packet which shows the MAC address, and searching for the string *Apple*:

* Wireshark resolves the first half of the MAC address to the manufacturer of the hardware so we don't have to!

![](https://i.imgur.com/hDzpKiP.png)

And by doing so, we'll find that in packet 4, the attacker initiates the connection with a host at `10.42.42.25`, and this device's MAC address is identified as being an Apple device:

![](https://i.imgur.com/OTiYJyW.png)

4. What is the IP address of the detected Windows system?

`10.42.42.50`

If we click the *Protocol* column in Wireshark to sort the packets in alphabetical order by the protocol that each packet is classified as, we can gather a quick list of the protocols that are in this packet capture:

* DCERPC
* ICMP
* NBNS
* NBSS
* TCP
* UDP

*NetBIOS Name Service (NBNS)* appears to be similar to DNS for legacy Windows environments to my understanding. You can read more about it [here](https://wiki.wireshark.org/NetBIOS/NBNS)

So now we know that NBNS is mainly used on Windows systems, and the 2 main IP addresses which are using NBSS to communicate include:

* 10.42.42.25
* 10.42.42.50

And if we look through the NBSS packets, we can see that some of them are session requests to an SMB server. I'm aware that SMB is a Windows protocol for printer and file sharing across a network, so the SMB server is our Windows system. 

And according to packet 6134, the destination that the packet is being sent to is the SMB server, `10.42.42.50`, and it's being sent from `MAC0016CB926EDC`, `10.42.42.25` (the Apple device we discovered earlier).

![](https://i.imgur.com/NXhSyTs.png)

## References

* https://wiki.wireshark.org/NetBIOS/NBNS
