---
tags:
- A+
---

# Tech Talk - Basic Network Troubleshooting Tools

Describe the various hardware and software tools designed to troubleshoot and optimize networks. Give definitions and examples of usage for each tool.


## Hardware tools

* <b class="malachite">Crimper</b> - used to attach a connector to a cable’s end e.g. a RJ45 connector at the end of a patch cable
* <b class="malachite">Cable tester</b> - verifies continuity for each wire in the cable to ensure there are no breaks and that the wire is cabled correctly eg. identifies missing pins/crossed wires
* <b class="malachite">Punchdown tool</b> - used w/66 or 110 blocks, network jacks and patch panels to terminate wires on a punchdown block without stripping off the insulation e.g forces wire into wiring block
* <b class="malachite">OTDR</b> - Time Domain Reflectometer for fiber optic cables (O for optical) - locates breaks in copper cables and provides an estimate of severity and distance to break. eg. estimate fiber lengths, measure signal loss, determine light reflection
* <b class="malachite">Light meter</b> -sends light from one side to measure light power
* <b class="malachite">Tone generator</b> - allows technicians to generate a tone at one end of a connection and use the probe to audibly detect the wire pair connected to the tone generator e.g cable hunting
* <b class="malachite">Loopback adapter</b> - device that connects transmit pins (or fibers) to receive pins (or fiber) to test network interfaces e.g test RJ45 or fiber ports to detect a bad module or physical port
* <b class="malachite">Multimeter</b> - used with copper cabling to verify continuity, resistance, amperage or voltage e.g. used to test device’s power supply or as a make-shift cable tester
* <b class="malachite">Spectrum analyzer</b> - measures/compares frequency in relation to the full frequency range of an instrument e.g used to identify frequency conflicts, display full frequency spectrum

## Software tools

* <b class="malachite">Packet sniffer</b> - captures and analyzes packets that flow through a network
* <b class="malachite">Port scanner</b> - scans for open ports and IP addresses e.g. rogue system detection, visually map a network
* <b class="malachite">Protocol analyzer</b> - captures traffic and signals as they travel across communication channels e.g. looking for problems in communication between devices 
* <b class="malachite">WiFi analyzer</b> - specialized software that can conduct wireless surveys to ensure proper coverage and prevent non-desired overlap e.g. view wireless information, signal to noise ratio channel information
* <b class="malachite">Bandwidth speed tester</b> - verifies throughput from client to the internet and determines overall connection. e.g speed to the internet bay downloading a random large file from a test server and uploading the file back to the server
* <b class="malachite">Command line</b> - used to configure and troubleshoot networks by issuing text based commands at an operating system prompt e.g. every tool listed below
    * <b class="malachite">ping</b> - checks connectivity between devices e.g. test a device for internet connection or connection between devices on a network
    * <b class="malachite">tracert</b> - displays the path between your device and the destination IP, showing each router hop along the way e.g. determine the route a packet takes to a destination
    * <b class="malachite">traceroute</b> - tracert for unix
    * <b class="malachite">nslookup</b> - resolves a FQDN to an IP address. e.g find the IP for a domain name. 
    * <b class="malachite">ipconfig</b> - shows network/IP configuration information e.g. check the IP address, default gateway etc
    * <b class="malachite">ifconfig</b> - ipconfig for unix
    * <b class="malachite">iptables</b> -used for packet filtering, command-line firewall utility that uses policy chains to allow or block traffic. 
    * <b class="malachite">netstat</b> - (network statistics) - displays information for IP-based connections on a PC e.g if you want to find out current sessions, source and destination IP address and port numbers on a device
    * <b class="malachite">tcpdump</b> - CLI packet sniffer - captures packets from the command line 
    * <b class="malachite">pathping</b> - combines tracert and ping
    * <b class="malachite">nmap</b> - network vulnerability scanner e.g find network devices, identify open ports, discover a devices OS
    * <b class="malachite">route</b> - used to change or display the contents of the PCs current IP routing tables
    * <b class="malachite">arp</b> - shows the MAC address table for a known IP addresses e.g. view the corresponding MAC addresses for every device the client has communicated with
    * <b class="malachite">dig</b> - non-interactive but more robust form of nslookup

