---
tags: 
- A+
categories:
- Study-Guide
---

# A+ Study Guide - DNS & DHCP

### DNS Service
  <b class="malachite">Domain Name Service/System</b> - a global and highly distributed network service that resolves strings of letters into IP addresses for you. 

* <b class="malachite">Internal vs. external DNS</b> - An organization might maintain an externally available authoritative-only DNS server to handle public DNS queries for the domains and zones that it handles. For its internal users, the organization might use a separate DNS server that contains the authoritative information that the public DNS provides, as well as additional information about internal hosts and services. It might also provide additional features, such as recursion and caching for its internal clients.
* <b class="malachite">Third-party/cloud-hosted DNS</b> - Third party DNS servers are name servers specifically configured so that anyone can use them. Most public DNS servers are available globally though anycast.
* <b class="malachite">Hierarchy</b> - DNS uses a hierarchy to manage its distributed database system. The DNS hierarchy, also called the <b class="malachite">domain name space</b>, is an inverted tree structure, much like eDirectory. The DNS tree has a single domain at the top of the structure called the <b class="malachite">root domain</b>. A period or dot (.) is the designation for the root domain. Below the root domain are the <b class="malachite">top-level domains</b> that divide the DNS hierarchy into segments. Below the top-level domains, the domain name space is further divided into <b class="malachite">subdomains</b> representing individual organizations.
* <b class="malachite">Forward vs. reverse zone</b> - A forward DNS lookup is when you resolve the IP address of a given domain. A reverse DNS lookup is when you want to resolve an domain name for a known IP address.

#### Record types
  * <b class="malachite">A</b> - used to point a certain domain name at a certain IPv4 address
      * a single record is configured for a single domain name
      * a single domain name can have multiple records
  * <b class="malachite">AAAA</b> (quad A) - used to point to a domain name at a certain IPv6 address
  * <b class="malachite">TXT</b> (<b class="malachite">SPF</b> - Sender Policy Framework, <b class="malachite">DKIM</b> - DomainKeys Identified Email) - text - used to communicate configuration preferences
  * <b class="malachite">SRV</b> - service record - used to define the location of specific services on a network
  *  using host and port information 
  * <b class="malachite">MX</b> - mail exchange - used to deliver email to the correct server (only for mail services)
  * <b class="malachite">CNAME</b> - canonical name - used to redirect traffic from one domain name to another.
  * <b class="malachite">NS</b> - delegates a DNS zone to use the given authoritative name servers
  * <b class="malachite">PTR</b> - pointer resource record - resolves an IP to a name
  

### DHCP Service

<b class="malachite">Dynamic Host Configuration Protocol</b> - an application layer protocol that automates the configuration process of hosts on a network.

* <b class="malachite">MAC reservations</b> - also DCHP reservation - static addressing approach where a specific MAC address is mapped to a specific IP address.
* <b class="malachite">Pools</b> - Scopes are also known as pools, can contain a range of IP address for a subnet, lease time and options for configuration.
* <b class="malachite">IP exclusions</b> - configuration option to exclude IP addresses from the scope you make available.
  
#### Scope options

|OPTION	|TITLE	|DEFINITION	|
|---	|---	|---	|
|1	|Subnet Mask	|Subnet Mask Value	|
|3	|Router	|Default Gateway addresses	|
|4	|Time Server	|Time server addresses	|
|5	|Name Server	|IEN-116 Server addresses	|
|6	|Domain Server	|DNS Server addresses	|
|43	|Vendor Specific	|Vendor Specific Information	|
|82	|Relay Agent Information	|Used to Identify Client Location	|
|150	|TFTP server address	|Used to Identify Voice Server/Gateway	|

* <b class="malachite">Lease time</b> - the amount of time in minutes or seconds that a network device can use an IP Address in a network.
* <b class="malachite">TTL</b> - a value in seconds that can be configured by the owner of a domain name for how long a name server is allowed to cache an entry before it should discard it and perform a full resolution again
* <b class="malachite">DHCP relay/IP helper</b> - DHCP relay is used to forward DHCP broadcast requests on LAN as a unicast packet to a central server, while ```ip helper-address``` is the command used to enable DHCP relay in routers.
* <b class="malachite">NTP</b> - Network Time Protocol - used to keep all computers on a network synchronized in times.
* <b class="malachite">IPAM</b> - IP address management - a means for planning, tracking and managing the IP address space used in a network. Integrates DNS and DHCP.