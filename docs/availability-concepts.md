---
tags: 
- availability
- study-guide
---

# Study Guide - Availability Concepts

* <b class="malachite">Fault tolerance</b> - hardware redundancy. The capability of any system to continue functioning after some part of the system has failed. 
* <b class="malachite">High availability</b>  - making sure that our systems are up (measured in uptime) - try to maintain the 5 9’s (99.999%) 
* <b class="malachite">Load balancing</b>  - distributes incoming requests across various server in the incoming requests based on location
* <b class="malachite">NIC teaming</b>  - Connecting multiple NICs in tandem to increase bandwidth in smaller increments.
* <b class="malachite">Port aggregation</b>  - joining multiple network device ports together for increased bandwidth and redundancy.
* <b class="malachite">Clustering</b>  - connecting systems together with the intent of delivering network services from the cluster to increase responsiveness and capacity. This solution also increases availability and redundancy. 
  
### Power management

  * <b class="malachite">Battery backups/UPS</b> - An uninterruptible power supply (UPS) protects your computer in the event of a power sag or power outage. A UPS essentially contains a big battery that provides AC power to your computer regardless of the power coming from the AC outlet. It does not provide enough power for you to continue working.
  * <b class="malachite">Power generators</b> - A generator provides power redundancy. Provides electricity if the power utility fails.
  * <b class="malachite">Dual power supplies</b> - Secondary source of power in the event that primary power fails.
  * <b class="malachite">Redundant circuits</b> - specialized electrical hardware like rack-mounted AC distribution boxes. AC distribution system can supply multiple dedicated AC circuits to handle any challenging setups/systems.

### Recovery
  * <b class="malachite">Cold sites</b> (weeks/months to get back up and running)
      * Building is available, but you may not have any hardware or software in place or configured
      * You need to buy resources (or ship them in), and then configure/restore the network
      * Recovery is possible, but slow and time consuming
  * <b class="malachite">Warm sites</b> (24 hrs - 7 days to get back up and running)
      * Building & equipment is available
      * Software may not be installed and latest data is not available
      * Recovery is fairly quick, but not everything from original site is available for employees
  * <b class="malachite">Hot sites</b> (high cost)
      * Building, equipment, and data is available
      * Software and hardware is configured
      * Basically, people can just walk into the new facility and get back to work
      * Downtime is minimal w/nearly identical service level maintained
  
### Backups

  * <b class="malachite">Full</b> - Complete backup is the safest and most comprehensive, time consuming & costly
  * <b class="malachite">Differential</b> - Only backups data since the last backup
  * <b class="malachite">Incremental</b> - Backup only data changed since last incremental backup - need all backups for full restore
  * <b class="malachite">Snapshots</b> - read-only copy of data frozen in time (VM’s)
---
  * <b class="malachite">MTTR (mean time to repair)</b> - measures the average time it takes to repair a network device
  * <b class="malachite">MTBF (mean time between failures)</b> - measures the average time between failures of a network device 
  * <b class="malachite">SLA requirements</b> - quality availability, specific responsibilities - are agreed upon between the service provider and the service user.

