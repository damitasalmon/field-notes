---
tags: 
- arp
- mac
---

# Using ARP to Discover a MAC Address

**Look at the arp -s option. How might that be useful?**

<p class="question-response">Suppose you have both a wireless adapter and wired adapter. Both are set to obtain an IP address automatically. It is possible for them both to receive an IP address that begins with 192.168.1.--. This is confusing to the route table. If you need to create a static connection the MAC address of a particular gateway for 192.168.1.1, you could use the arp -s command to associate the right MAC address to 192.168.1.1, thus avoiding route table confusion.</p>

**If you wait about 2 or 3 minutes without really doing anything to activate the NIC into transmitting/receiving any data, it will no longer show any output when you type arp -a. Why do you think this is true?**

<p class="question-response">The ARP cache is a memory location with a timer. If no activity occurs within a certain period of time, the cache is emptied to make room for whatever MAC addresses it will need to learn going forward. This serves to force it to refresh itself if what it thought it knew about the connection has changed. For example, someone could have unplugged his machine from the network and another machine might have taken its place. This would cause the ARP cache to be wrong, sending packets to the wrong interface, a significant security risk in the wrong situation.<p>
