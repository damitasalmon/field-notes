---
tags: 
- routing
- swtitching
- study-guide
- A+
---

# A+ Study Guide - Routing and Switch Characteristics

### IPv6 concepts

  * <b class="malachite">Addressing</b> - 128-bit hexadecimal address - abbreviation => omit leading 0’s & contigous fields of 0’s replaced with :: (once per address)
      * IP6 uses auto configuration to discover its current network and selects its own host ID based on its MAC using EUI64 process - doesn't need DHCP but can use DHCPv6
      * no broadcasts
      * no fragmentation (no MTU)
      * simplified header (5 fields)
  * <b class="malachite">Tunneling</b> - provisions access to something not locally available
      * <b class="malachite">Teredo tunneling</b> - IPv6 host provides IPv6 connectivity when host is connected to an IPv4 only network also called 6to4 or 4to6 tunneling
  * <b class="malachite">Dual stack</b> - IPv6 can coexist with IPv4 -> running IPv4 and IPv6 simultaneously on a network interface or device
  * <b class="malachite">Neighbor discovery** protocol (NDP) - leans the Layer 2 addresses on the network (features below):
      * <b class="malachite">Router Solicitation - Hosts send messages to locate routers on the link
      * <b class="malachite">Router Advertisement** - router advertise their presence periodically and also in response to solicitation
      * <b class="malachite">Neighbor Solicitation - Used by notes to determine link layer address
      * <b class="malachite">Neighbor advertisement - used by nodes to respond to solicitation messages
      * Redirect - routers inform host of better first-hop routers
  
### Performance concepts
  * <b class="malachite">Traffic shaping** - allows buffer to delay traffic from exceeding configured rate limit (speed limit) when there’s space available, it starts pushing it over that empty space and starts shaping out the packets
      * maximizes bandwidth on slow speed interfaces: T1, dial-up, satellite connections, ISDN
  * <b class="malachite">QoS** - Quality of Service - enables strategic optimization of network performance for different types of traffic
      * <b class="malachite">Diffserv** - Differentiated Services
          * Soft QoS
          * Differentiation of data types
          * routers and switches make decisions based on markings and fluctuates traffic if needed
      * CoS - Class of Service - QoS at Layer 2 - gives preferential treatment to voice traffic 
* **NAT/PAT**
    * Network Address Translation - allows for the mapping of one public IP to multiple private IP’s - used to conserve IPv4 addresses
    * Port Address Translation - a variation of address translation that utilizes port numbers instead of IP addresses for translation
*  **Port forwarding** - where specific destination ports can be configured to always be delivered to specific nodes. Allows for complete IP masquerading while still having services that can respond to incoming traffic. 
* **Access control list** - sets of rules typically applied to router interfaces or firewalls that permit or deny certain traffic based on its IP address, MAC address or its port depending on what device you’re dealing with. 
* **Distributed switching** - a virtual network distributed accoss all physical platforms - removes physical segmentation
* **Packet-switched vs. circuit- switched network**
    * Packet-switched connection - always on like a dedicated leaded line, but multiple customers share the bandwidth. SLAs used to guarantee a certain quality. 
    * Circuit-switched - connection is brought up only when needed, like making a phone call.
* **Software-defined networking** - allows administrators to programmatically initialize, control, change, and manage network behavior dynamically via open interfaces and provide abstraction of lower-level functionality.

