---
title: "Azure SOC Honeynet MOC"
tags: 
- azure
- lab
- SOC
- honeynet
- sentinel
- log-analytics
- microsoft-defender
- NIST-SP-800-53-r4
- PowerShell
- KQL
- azure-bastion
- rdp
- virtualization
---

# Building a (mini) SOC & Honeynet in Azure (w/Live Traffic)
![Architecture Diagram](./assets/images/soc-honeynet/topology-diagram-2.png)

!!! note
    This is a not a necessarily complete walk-though but it is a more detailed iteration of the Github/Gitlab repo. This page is a work in progress and the documentation for this lab is egreiously long. **Click on images to expand.**

## Overview

In this project, I built a small-scale honeynet and SOC in Azure. Log Analytics was used to ingest logs from various sources that Microsoft Sentinel would leverage to build attack maps, trigger alerts, and create incidents. Microsoft Defender for Cloud was used as a data source for LAW and to assess the VM configuration relative to regulatory frameworks/security controls. I configured log collection on the insecure environment, set security metrics then observed the environment for 24 hours. After investigating the incidents that Microsoft Sentinel generated during that period, security controls were applied to address the incidents and harden the environment based on recommendations from Microsoft Defender. After a second 24-hour observation new metrics were collected on the environment post-remediation. 

Collected metrics: 

- SecurityEvent (Windows Event Logs)
- Syslog (Linux Event Logs)
- SecurityAlert (Log Analytics Alerts Triggered)
- SecurityIncident (Incidents created by Sentinel)
- AzureNetworkAnalytics_CL (Malicious Flows allowed into my honeynet)

## Architecture Before Hardening / Security Controls
![Architecture Diagram](./assets/images/soc-honeynet/architecture-before.png)<br>

The architecture of the mini honeynet in Azure consists of the following tools and components:

- Virtual Network (VNet)
- Network Security Group (NSG)
- Virtual Machines (2 Windows, 1 Linux)
- Azure Key Vault
- Azure Storage Account
- Microsoft SQL Server
- SQL Server Management Studio (SSMS)
- Azure Active Directory

Additionally, the SOC utilized the following tools, components and regulations: 

- Microsoft Sentinel (SIEM)
- Microsoft Defender for Cloud
  - [NIST SP 800-53 Revision 4](https://csrc.nist.gov/publications/detail/sp/800-53/rev-4/archive/2015-01-22)
  - [PCI DSS 3.2.1](https://listings.pcisecuritystandards.org/documents/PCI_DSS-QRG-v3_2_1.pdf) 
- Log Analytics Workspace
- Windows Event Viewer
- Kusto Query Language (KQL)
- PowerShell

To collect the metrics for the insecure environment, all resources were originally deployed, exposed to the  public internet. The Virtual Machines had their Network Security Groups open (allowing all traffic) and built-in firewalls disabled. All other resources were deployed with endpoints visible to the public Internet.

## Stage I - Building the honeynet

This project consists of two target virtual machines and one threat VM - two Windows VMs (one used for attack) and one Linux VM.

### Creating Resources

#### Create the Subscription

I already had an Azure account, a tenant (which I renamed + added a custom domain before this run) and a subscription from a previous run of the project. Here is an overview for the next few steps in this section: 

![stage 1 overview](./assets/images/soc-honeynet/stage-one-overview.png)<br>

For the sake of screenshots, I'm starting at creating the subscription. Don't forget to set a budget! 

![Creating Subscription](./assets/images/soc-honeynet/create-sub-01.png)<br>

![Creating Subscription Advanced](./assets/images/soc-honeynet/create-sub-02.png)<br>

![Creating Subscription Budget](./assets/images/soc-honeynet/create-sub-03.png)<br>

Later, I actually changed the alert threshold to something more reasonable like 90%. </br>

#### Create the first resource group

Next, create the first resource group (RG-Cyber-Lab). This one will house the resources that will be exposed to attack (the honeynet). Technically, this can be created at the same time that you create your VMs. 

![Create the First Resource Group](./assets/images/soc-honeynet/create-rg-one.png)<br>

#### Create Virtual Machines

Next, create two VMs. One Windows VM and one Linux VM, using mostly default settings. Add both to the **RG-Cyber-Lab** resource group. Create a new Virtual Network the honeynet (Lab-VNet). 

##### Create Windows VM 

Basics tab:

![Basics tab - Windows VM Wizard](./assets/images/soc-honeynet/create-win-vm-azure-basics-tab.png)<br>

Create a new Virtual Network:

![Create a new VNet](assets/images/soc-honeynet/create-win-vm-azure-vnet.png)<br>

Networking tab:

![Network tab - Windows VM Wizard](assets/images/soc-honeynet/create-win-vm-azure-networking.png) <br>


##### Create Linux VM

Create the Linux VM with the same user, region, resource group and networking settings. 

Basics tab:

![Basics tab - Linux VM Wizard](assets/images/soc-honeynet/create-linux-vm-azure-basics-tab.png) <br>

Networking tab:

![Networking tab - Linux VM Wizard](assets/images/soc-honeynet/create-linux-vm-azure-networking.png) <br>

### Exposing the resources

After both VMs are deployed, change both Network Security Groups (NSGs) to allow all inbound traffic. Removing rules for RDP and SSH and replacing with the custom inbound rule. 

Windows NSG before:

![Windows NSG before](assets/images/soc-honeynet/windows-vm-nsg-before.png) <br>

Linux NSG before:

![Linux NSG before](assets/images/soc-honeynet/linux-vm-nsg-before.png)  <br>

Custom Inbound Rule:

![Add custom Inbound Rule](assets/images/soc-honeynet/Add-inbound-security-rule.png) <br>

Windows NSG after: 

![Windows NSG after](assets/images/soc-honeynet/windows-vm-nsg-after.png) <br>

At this point both NSGs are identical. 

### Create Vulnerabilities

#### Disable Windows Firewall

The goal is to expose these VMs to threat actors and to make them both discoverable and reachable so that we can monitor, log and investigate incidents later. By default Windows Firewall, blocks ICMP packets from the internet. You can see that from another network the windows-vm is unreachable. 

![Ping Windows VM with Firewall ON](assets/images/soc-honeynet/ping-win-vm-before-firewall-change.png)

From here, disable the firewall (wf.msc). 

**Before**

![Windows Firewall Before](assets/images/soc-honeynet/windows-vm-firewall-before.png)


**After**
![Windows Firewall After](assets/images/soc-honeynet/windows-vm-firewall-after-disable.png)

Pinging windows-vm again to test success

![Ping win-vm after firewall change](assets/images/soc-honeynet/ping-win-vm-after-firewall-change.png)

Connecting to the linux-vm via SSH using PuTTy

![Connect to linux-vm using PuTTy](assets/images/soc-honeynet/ssh-putty-linux.png)

![Connected to linux-vm](assets/images/soc-honeynet/ssh-into-linux-success.png)

Test pinging linux-vm externally 

![Ping linux-vm](assets/images/soc-honeynet/test-ping-linux-vm.png)


#### Install MS SQL Server + Utilities

Next, download and install [SQL Server Evaluation](ttps://www.microsoft.com/en-us/evalcenter/evaluate-sql-server-2019). 

Select Download Media. 

![Install MS SQL Server](assets/images/soc-honeynet/install-ms-sql-server.png)

Select ISO and download location.

![Install MS SQL Server](assets/images/soc-honeynet/install-ms-sql-server-2.png)

Once the download completes, go to download location.

![Install MS SQL Server](assets/images/soc-honeynet/install-ms-sql-server-3.png)

Mount the ISO.

![Install MS SQL Server](assets/images/soc-honeynet/install-ms-sql-server-4.png)

Run the installer.
![Install MS SQL Server](assets/images/soc-honeynet/install-ms-sql-server-5.png)

Select New SQL Server

![Install MS SQL Server](assets/images/soc-honeynet/install-ms-sql-server-6.jpg)

Include Database Engine Service

![Install MS SQL Server](assets/images/soc-honeynet/install-ms-sql-server-7.png)

Select Mixed Mode, create a user and select add current user to also allow for Windows authentication using labuser. 

![Install MS SQL Server](assets/images/soc-honeynet/install-ms-sql-server-8.png)

Once the installation completes, download and install [MS SQL Management Studio (SMSS)](https://learn.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms).

![Install SMSS](assets/images/soc-honeynet/install-ms-sql-ssms-2.jpg)

![Install SMSS](assets/images/soc-honeynet/install-ms-sql-ssms-3.jpg)

Once the installation completes, restart the VM. 


##### Enable Logging on SQL Server

After restarting, follow the [Microsoft documentation](https://learn.microsoft.com/en-us/sql/relational-databases/security/auditing/write-sql-server-audit-events-to-the-security-log?view=sql-server-ver16) for adjusting settings to allow SQL Server logs to be ported to Windows Event Viewer. 

Provide full permission for the SQL Server service account (NETWORK SERVICE) to the registry hive. 

![Edit Registry Key](assets/images/soc-honeynet/windows-vm-edit-registry-key.png)

![Grant MS SQL Server Full Control](assets/images/soc-honeynet/windows-vm-edit-registry-network-fullctrl.png)

Configure the audit object access setting in Windows using auditpol by executing the provided command line statement. 

![Execute auditpol statement in cmd](assets/images/soc-honeynet/enable-sql-logging-cmd.jpg)

Launch SSMS and log in to the SQL Server. Then go to Properties > Security > Enable both

**Security Settings Before**

![SQL Server Security Settings Before](assets/images/soc-honeynet/enable-sql-logging-before.png)

**Security Settings After**
![SQL Server Security Settings After](assets/images/soc-honeynet/enable-sql-logging-after.png)

Restart the SQL Server and try to generate some failed authentication logs by trying log into the SQL server with the wrong password. 

![Restart SQL Server](assets/images/soc-honeynet/restart-sql-server.png)

Check Event Viewer to make sure the logs are properly enabled and porting to Event Viewer successfully. 

![Check Event Viewer for Failed Auth Log](assets/images/soc-honeynet/test-sql-login-bad-pw.png)


It was at this time I learned a valuable lesson about **Azure Bastion**. What I had not realized is that once you deploy the bastion instance it is perpetually in a running-state. I was under the impression that once the VM was deallocated/stopped, so was the Bastion. This is not the case. To the best of my knowledge the only way to stop Bastion is to delete it. Luckily, I check cost management semi-neurotically so I caught this before I had so sell any organs. 

I also realized that I could use the Microsoft Remote Desktop on iOS (on iPad) the same way you can with macOS. I was trilled. 

#### Create Attack (Threat) VM

Create another Windows VM in a different resource group, region, and virtual network. All other settings can be the same or similar. 

![Attack VM Overview](assets/images/soc-honeynet/attack-vm-overview.png)

#### Generate Logs 

To make sure everything is working as expected, log into the attack-vm to generate failed authentication logs on both vms.

![generating logs overview](assets/images/soc-honeynet/logging-overview.png)

Generating failed RDP logs on windows-vm: 

![generate failed login w/rdp](assets/images/soc-honeynet/generate-failed-login-rdp.jpg)

Using PowerShell to generate failed login logs on linux-vm:

![generate failed login with ssh on powershell](assets/images/soc-honeynet/generate-failed-login-linux.jpg)

Installing SMSS on attack-vm: 

![install smss on attack-vm](assets/images/soc-honeynet/install-smss-attack-vm.jpg)

Generating  failed login logs for MS SQL Server on windows-vm:

![generate failed sql server login](assets/images/soc-honeynet/generate-failed-login-sql_001.jpg) 

![generate failed sql server login](assets/images/soc-honeynet/generate-failed-login-sql.jpg)

##### Check Logs

Using PowerShell to SSH into linux-vm: 

![ssh into linux-vm with PS](assets/images/soc-honeynet/check-linux-logs_001.jpg)

Investigating the logs at /var/log/auth.log for failed authentication

![the logs in auth.log file](assets/images/soc-honeynet/check-linux-logs_003.jpg)

**Checking Event Viewer on windows-vm:**

In Windows Event Viewer, there are normally a lot of logs, in the screenshots below the logs are filtered by the specific events we're looking for. 

Windows Logs > Security, filtered by Event ID: [4625](https://learn.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625)

![failed login logs in windows event viewer](assets/images/soc-honeynet/failed-login-rdp-eventviewer.jpg)

Windows Logs > Application, filtered by Event ID: [18456](https://learn.microsoft.com/en-us/sql/relational-databases/errors-events/mssqlserver-18456-database-engine-error?view=sql-server-ver16)

![failed sql login logs in windows event viewer](assets/images/soc-honeynet/failed-login-sql-eventviewer.jpg)


## Stage II - Building the SOC

### Log Analytics and Microsoft Sentinel (SIEM) Setup + Data Ingestion

Create Log Analytics Workspace.

![Create LAW](assets/images/soc-honeynet/create-LAW.png)

Add Sentinel to the workspace. 

![Add Sentinel](assets/images/soc-honeynet/add-sentinel-to-a-workspace.png)

![Sentinel & LAW Connected](assets/images/soc-honeynet/Microsoft-Sentinel-Microsoft-Azure.png)

#### Create Sentinel Watchlist 

In Sentinel, a new watchlist for Geo IP Data. This watchlist will help us correlate security events to geographic locations later in the lab. 

![Create GeoIP sentinel Watchlist](assets/images/soc-honeynet/create-sentinel-watchlist-01.png)

Upload the geoIP data and set the search key. 

![Watchlist Wizard](assets/images/soc-honeynet/Watchlist-wizard-Microsoft-Azure.png)

Once the watchlist begins uploading, sentinel will start ingesting the data and it will be available for query, even before ingestion completes. 

![Checking Watchlist ingestion](<assets/images/soc-honeynet/Microsoft-Sentinel-Microsoft-Azure-checking upload.png>)

Querying GeoIP data using KQL.

![Query GeoIP Watchlist](assets/images/soc-honeynet/LAW-Cyber-Lab-Microsoft-Azure-testquery-2.png)

Upload complete.

![GeoIP Ingestion Complete](assets/images/soc-honeynet/Microsoft-Sentinel-Microsoft-Azure-Watchlist-Ingestion-Complete.png)

#### Enable Microsoft Defender for Cloud 

Open MDC: 

![Enable MDC](assets/images/soc-honeynet/enable-mdc_001.jpg) 

Go to Environment Settings > Drill down to the LAW > click the three dots corresponding to LAW > Edit Settings 

![Enable MDC](assets/images/soc-honeynet/enable-mdc_003.jpg) 

Enable Data Collection
Click Data Collection (in sidebar) > All Events > Save

![Enable MDC](assets/images/soc-honeynet/enable-mdc_004.jpg) 

Back in MDC, go to Environment Settings > click the three dots corresponding to the Subscription > Edit Settings 

![Enable MDC](assets/images/soc-honeynet/enable-mdc_005.jpg) 

Under Defender Plans, toggle **ON**: Servers, Databases, Storage and Key Vault

![Enable MDC](assets/images/soc-honeynet/enable-mdc_006.jpg) 

Next to Databases > Select Types > make sure 'SQL servers on Machines' is toggled **ON**, all else toggled **OFF** > Continue

![Enable MDC](assets/images/soc-honeynet/enable-mdc_007.jpg) 

Next to Servers > Under Monitoring Coverage,  click Settings > make sure everything is toggled **ON**

![Enable MDC](assets/images/soc-honeynet/enable-mdc_008.jpg) 

Next to Log Analytics Agent > under Configuration > Edit Configuration. 
Change the workspace selection to Custom and select the LAW created and configured earlier > Apply > Continue > Save

![Enable MDC](assets/images/soc-honeynet/enable-mdc_009.jpg)

!!! note
    If you accidentally saved before configuring the LAW agent: Go back and change to custom, then go through your resources and delete resources that were automatically provisioned in the processes. To avoid future mixups, make sure there is only ONE LAW. 

Click Continuous Export in the sidebar > Select Log Analytics Workspace at the top > toggle **ON** 
Select everything (will fine tune later). 

Make sure export Export Configuration points to the resource group where the LAW is stored and Export target points to the appropriate subscription and LAW. Click Save. 

![Enable Continuous export to LAW](assets/images/soc-honeynet/Settings-ContinousExport-MDC.png)


#### Configure Log Collection for Virtual Machines

Create a Storage Account for Azure to place NSG flow logs later. 

!!! note
    Storage Account name must be globally unique. 

![Create a storage account](assets/images/soc-honeynet/create-a-storage-account.png)

The important part is cut off here, but make sure the storage account is in the **same region** as the target VMs.

![create a storage account wizard](assets/images/soc-honeynet/Create-a-storage-account-2.png)

##### Enable NSG flow logs for target VMs

Go to Network Security Groups, pick one (any but preferably one attached to a target VM) > Under Monitoring, click NSG flow logs > Create flow log

![Create NSG Flow Logs](assets/images/soc-honeynet/windows-vm-nsg-Microsoft-Azure.png) 

Click +Resource > Select the target VM's > Confirm Selection

![Create a flow log wizard](assets/images/soc-honeynet/Create-a-flow-log-Microsoft-Azure-2.png)

![Select NSG](assets/images/soc-honeynet/Select-network-security-group-Microsoft-Azure-3.png)

##### Create a Data Collection Rule for target VMs

First make sure target VMs are running. The Microsoft Defender or will automatically install the agent to the VMs once they are running, if not, you can manually install later. 

Next, go to LAW > Agents > Data Collection Rules > Create Data Collection Rule 

![Data Collection Rule page in LAW](assets/images/soc-honeynet/add-data-collection-001.jpg)

![Create DCR](assets/images/soc-honeynet/add-data-collection-002.jpg)

![Create DCR wizard](assets/images/soc-honeynet/add-data-collection-003.jpg)

Click + Add Resources, and select target VMs > Apply.

![Select resources for DCR](assets/images/soc-honeynet/add-data-collection-004.jpg)

![Select target VMs](assets/images/soc-honeynet/add-data-collection-005.jpg)

![DCR Resources](assets/images/soc-honeynet/add-data-collection-006.jpg)

Click 'Next: Collect and deliver >' + Data Source and add select Linux Syslog from the dropdown for Data Source Type. Then only collect logs for LOG_AUTH (set all other logs to 'none' value) > Next: Destination

![Add Linux Syslog Data Source](assets/images/soc-honeynet/add-data-collection-007.jpg)

Set the destination to the Log Analytics Workspace > Add data source

![Set dest for DCR to LAW](assets/images/soc-honeynet/add-data-collection-008.jpg)

Add another data source, this time for Windows logs.

![Data Collection Summary Page](assets/images/soc-honeynet/add-data-collection-009.jpg)

Select Windows Event Logs from the data source dropdown. Under Basic > Application, Select Information. Under Security, select both Audit success and Audit failure to pull in failed authentication logs from RDP and the SQL Server. 


![DCR Wizard - Windows Event Logs ](assets/images/soc-honeynet/add-data-collection-010.jpg)

As you can see the options here are pretty, well, basic. In order to retrieve (filter) specific data from Windows event logs, XPath queries must be used. Switch over to Custom and add the following X-Path Queries:

For Windows Defender Malware Detection:
```Microsoft-Windows-Windows Defender/Operational!*[System[(EventID=1116 or EventID=1117)]]```

For Windows Firewall Tampering Detection:
```Microsoft-Windows-Windows Firewall With Advanced Security/Firewall!*[System[(EventID=2003)]]```

![DCR Wizard, Windows Event Logs Custom Configuration](assets/images/soc-honeynet/add-data-collection-014.jpg)

Save > Next: Destination and check that the logs are going to the appropriate LAW > Add data source > Review + Create > Create

![DCR Wizard - Data Sources Summary Page](assets/images/soc-honeynet/add-data-collection-011.jpg)

Data Collection Rule complete and deployed

![Data Collection Rules in LAW](assets/images/soc-honeynet/add-data-collection-012.jpg)

If you check back to LAW > Agents, the agents should have deployed and installed on the target VMs

![Windows LAW Agent Installed and Connected](assets/images/soc-honeynet/add-data-collection-015.jpg)

![Linux LAW Agent Installed and Connected](assets/images/soc-honeynet/add-data-collection-016.jpg)

Ideally, at this point you would start querying Log Analytics for logs from the VMs and NSGs to make sure the logs are coming in:
- Syslog (linux)
- SecurityEvent (windows)
- AzureNetworkAnalytics_CL (Network Security Groups/NSGs)

SecurityEvent:
![SecurityEvent query in LAW](assets/images/soc-honeynet/LAW-Cyber-Lab-Microsoft-Azure-SecurityEvent.png)

Syslog:
![Syslog query in LAW](assets/images/soc-honeynet/LAW-Cyber-Lab-Microsoft-Azure-Syslog.png)

AzureNetworkAnalytics_CL:
![AzureNetworkAnalytics in LAW](assets/images/soc-honeynet/LAW-Cyber-Lab-Microsoft-Azure_AzureCL.png)


##### Tenant Level Logging

Create diagnostic settings in Azure Active Directory (Microsoft Entra ID) that allows us to ingest logs. 

![AAD Diagnostic Setting](assets/images/soc-honeynet/Diagnostic-setting-Microsoft-Azure.png)

Create a user, assign global admin and use Log Analytics to check that logs are properly being ingested. 

![Create a Dummy User](assets/images/soc-honeynet/Create-new-user-Microsoft-Azure.png)

![Assign Global Admin](assets/images/soc-honeynet/Directory-roles-Microsoft-Azure.png)

Create an "attacker" user and generate some failed authentication logs by failing to log in 10-20 times. 

![Create an attacker user](assets/images/soc-honeynet/Create-new-user-Microsoft-Azure-2.png)

!!! note
    The AuditLogs come in pretty quickly, but the SigninLogs and AzureActivity take a while to come into the Log Analytics Workspace. Generate the logs, then take a 20-30 minute coffee break and query the LAW after. 

Audit Logs

![Audit Logs](assets/images/soc-honeynet/LAW-Cyber-Lab-Microsoft-Azure.png)

Signin Logs

![Signin Logs](assets/images/soc-honeynet/LAW-Cyber-Lab-Microsoft-Azure-SigninLogs.png)

##### Subscription Level Logging

Export Azure Activity Logs to Log Analytics Workspace. 

Go to Azure Monitor > Activity Log > Export Activity Logs, create diagnostic settings. 

![Az Monitor](assets/images/soc-honeynet/Monitor-Microsoft-Azure.png)

![Diagnostic Setting Az Monitor](assets/images/soc-honeynet/Diagnostic-setting-Microsoft-Azure-Monitor.png)

Generate some logs to confirm functionality. Here, I'm creating two resource groups and changing a NSG, then deleting them **after** confirming the logs are flowing into the LAW properly. 

New Resource Groups

![Test Resource Groups](assets/images/soc-honeynet/Resource-groups-Microsoft-Azure.png)

New inbound security rule in attack-vm-nsg

![Create Test Rule for Logs](assets/images/soc-honeynet/Change_NSG_Test_Activity_Logs.png)

Activity Logs 

 ![NSG Logs](assets/images/soc-honeynet/LAW-Cyber-Lab-Microsoft-Azure-NSG_logs.png)

 Checking resource group deletion in logs

 ![Resource Group deletion in LAW](assets/images/soc-honeynet/Delete-Resource-Group-Logs.png)

##### Resource Level Logging (Data Plane Logs)

###### Storage Account 

Enable logs for storage account and key vault. 

Storage Accounts > Select Storage Account > Under Monitoring, select Diagnostic Settings 

![Create a Diagnostic Setting AZ Storage](assets/images/soc-honeynet/sacyberlab009-Microsoft-Azure-Storage-01.png)

Click 'Disabled' next to 'blob' > Add Diagnostic setting > Select Audit, create name for setting > Save

![Create a Diagnostic Setting AZ Storage](assets/images/soc-honeynet/sacyberlab009-Microsoft-Azure-Storage-02.png)

![Diagnostic Setting Az Storage](assets/images/soc-honeynet/Diagnostic-setting-Microsoft-Azure-Storage.png)

###### Azure Key Vault

Navigate to Azure Key Vault > Create a Key Vault or use one you already have, just make sure it is in the same resource group as the other resources. Name must be globally unique. 

![Create a key vault](<assets/images/soc-honeynet/Create-a-key-vault-Microsoft-Azure -02.png>)

![Create a key vault - Change Access](assets/images/soc-honeynet/Create-a-key-vault-Microsoft-Azure.png)



Create a Diagnostic Setting for LAW

![Create Diagnostic Setting in AKV](assets/images/soc-honeynet/ak-cyber-lab-Microsoft-Azure-diag-setting.png)

![Diagnostic Setting AKV](assets/images/soc-honeynet/Diagnostic-setting-Microsoft-Azure-Key-Vault.png)

Next, generate some logs by creating a few secrets and viewing them. 

![Create secrets in AKV for Logs](assets/images/soc-honeynet/Create-a-secret-Microsoft-Azure.png)

![Create secrets in AKV for Logs](<assets/images/soc-honeynet/Create-a-secret-Microsoft-Azure-2nd One.png>)

![Secrets Overview in AKV](assets/images/soc-honeynet/ak-cyber-lab-Microsoft-Azure-after2ndSecret.png)


Query LAW to make sure logs are flowing from resources:

![Confirm logs in LAW](assets/images/soc-honeynet/Confirm-Azure-Activity-logs.png)


### Configure Microsoft Sentinel

#### World Attack Maps Construction

This is the point where we can create attack maps for the following use cases: 
- Failed Authentication in Windows (RDP/SMB)
- Failed Authentication in Linux (SSH)
- Failed Authentication to the Microsoft SQL Server
- Malicious Flows to the Network Security Groups

Go to Microsoft Sentinel > Workbooks > Add Workbook > Edit > Delete default analytics query > Add query > Advanced Editor > Add json query here > Save and name workbook.

![Sentinel After Workbooks Creation](assets/images/soc-honeynet/Microsoft-Sentinel-Microsoft-Azure-Workbooks-Creation.png)

If you want to test the query, to see if events will be plotted on the map. Exit the Advanced Editor and grab the query from the main Edit page, then use it to query the logs in LAW. 

For example, this is the query from the Microsoft SQL Server failed authentication workbook in LAW:

![MSSQL Failed Auth in LAW](assets/images/soc-honeynet/LAW-Cyber-Lab-Microsoft-Azure-MSSQL.png)

By doing this, you can test changes to the query and make sure it works and is generating desired results before updating the workbooks in Sentinel. 

#### Analytics, Alerting and Incident Generation

#### Manual Alert Creation 

Here we're using KQL queries to trigger alerts and spin up incidents in Microsoft Sentinel. 

Sentinel > Analytics > Create scheduled query rule

![Create Alerts in Sentinel](assets/images/soc-honeynet/Microsoft-Sentinel-Microsoft-Alerts-Creation.png)

![Analytics Rule Wizard Part 1](assets/images/soc-honeynet/Analytics-rule-wizard-01.png) 

![Analytics Rule Wizard Part 2](assets/images/soc-honeynet/Analytics-rule-wizard-02.png) 

![Analytics Rule Wizard Part 3](assets/images/soc-honeynet/Analytics-rule-wizard-03.png) 

![Analytics Rule Wizard Part 4](assets/images/soc-honeynet/Analytics-rule-wizard-04.png)


#### Attack Traffic Generation (Simulated Attacks)

To test your alerts and incidents rule configuration, simulate some attacks on the VMs and see if they show up in Sentinel (generate alerts and incidents). We have to make sure these work before the first observation period. Here are some tests to run: 
- **Trigger AAD Brute Force Success:** Simulate brute force success against Azure AD with your attacker account (from attack-vm). Either use PowerShell or an incognito window to fail 10-11 consecutive logins, followed by one successful login.
- **Trigger MSSQL Brute Force Attempt**: Using the attack-vm, use PowerShell of SSMS to simulate brute force attempt against your SQL Server by failing 10-11 consecutive logins.
- **Trigger Malware Outbreak**: In windows-vm generate a malware alert by using PowerShell to create 1 or more EICAR files. You can also do this manually by creating a text file with an EICAR string in it. 
- **Trigger Possible Privilege Escalation (AKV Critical Credential Retrieval or Update)**: Manually read Key Vault Secret “Tenant-Global-Admin-Password” in the Azure portal.
- **Trigger Windows Host Firewall Tampering**: Manually Enable and Disable the windows-vm Firewall.
- **Trigger Excessive Password Resets**: Reset a users’ password in the Azure portal 10-11 times.

After each attach, wait 10-20 minutes, then check Sentinel to see if you have any incidents. This can also help you with incident investigation later on in the lab. 

Incidents in Sentinel after simulating some attacks: 

![Test Traffic in Sentinel](assets/images/soc-honeynet/Test-Sentinel-Alerts.png)


### Run Insecure Environment (24H)

The following table shows the measurements taken from the insecure environment after the initial 24 hour observation period: <br>

#### Metrics - Before Securing Environment
	
**Start Time**:	12/11/2023, 9:40:11 AM  <br>
**Stop Time**:	12/12/2023, 9:40:11 AM

|Metric | Count |
|------|---------|
|Security Events (Windows VMs)| 39687|
|Syslog (Linux VMs)| 16562|
|SecurityAlert (Microsoft Defender for Cloud) |8|
|SecurityIncident (Sentinel Incidents) |248|
|NSG Inbound Malicious Flows Allowed |2252|
|NSG Inbound Malicious Flows Blocked |0|

#### Attack Maps Before Hardening / Security 

Before taking the screenshots, the workbooks need to be edited to only show the last 24 hours. The query runs over the last 30 days by default. 

**To edit the maps**:

![Generate Maps 1](assets/images/soc-honeynet/generate-attack-maps-01.png) 
![Generate Maps 2](assets/images/soc-honeynet/generate-attack-maps-02.png) 
![Generate Maps 3](assets/images/soc-honeynet/generate-attack-maps-03.png) 
![Generate Maps 4](assets/images/soc-honeynet/generate-attack-maps-04.png)

##### NSG Allowed Malicious Inbound Flows
![NSG Allowed Inbound Malicious Flows](assets/images/soc-honeynet/nsg-malicious-allowed-in-before.png)<br>

##### Linux SSH Authentication Failures
![Linux Syslog Auth Fail](./assets/images/soc-honeynet/linux-ssh-auth-fail-before.png)<br>


##### Windows RDP/SMB Authentication Failures
![Windows RDP/SMB Auth Fail](./assets/images/soc-honeynet/windows-rdp-auth-fail-before.png)<br>

##### MS SQL Server Authentication Failures
![MSSQL Server Auth Fail](assets/images/soc-honeynet/mssql-auth-fail-before.png)


## Stage III - Working Incidents and Incident Response

Next, investigate and work the incidents being generated within Azure Sentinel, in accordance with the [NIST 800-61](https://nvlpubs.nist.gov/nistpubs/specialpublications/nist.sp.800-61r2.pdf) Incident Management Lifecycle. 

![Incident Management Lifecycle](assets/images/soc-honeynet/incident-response-lifecycle.png)

**Step 1: Preparation**

We initiated this already by ingesting all of the logs into Log Analytics Workspace and Sentinel and configuring alert rules. 

**Step 2: Detection & Analysis**
    1. Set Severity, Status, Owner
    2. View Full Details 
    3. Observe the Activity Log
    4. Observe Entities and Incident Timelines
    5. “Investigate” the incident and continue trying to determine the scope
    6. Inspect the entities and see if there are any related events
    7. Determine legitimacy of the incident 
    8. If True Positive, continue, if False positive, close it out

**Step 3: Containment, Eradication, and Recovery**

Use this simple [Incident Response PlayBook](https://docs.google.com/document/d/1EQ5MzN95POLaRIMulYg3PIH3UGHtDNcGdkFvgOXyEXQ/edit#heading=h.85wjqnr3yu96) to remediate the incident.

**Step 4: Post-Incident Activity**

Document Findings and Close out the Incident in Sentinel

### Incident 1 - Brute Force Success (Windows) 

![Incident 1 - Brute Force Overview](assets/images/soc-honeynet/indident-1-brute-force.png)

**Step 2: Detection & Analysis**

    1. Set Severity, Status, Owner
   
   ![Brute Force - Set Status](assets/images/soc-honeynet/indident-1-brute-force-1-set-status.png)

    2. View Full Details 
   
   ![Brute Force - Full Details](assets/images/soc-honeynet/indident-1-brute-force-2-full-details.png)
   
    3. Observe the Activity Log
    4. Observe Entities and Incident Timelines
    5. “Investigate” the incident and continue trying to determine the scope
    6. Inspect the entities and see if there are any related events
    7. Determine legitimacy of the incident 
    8. If True Positive, continue, if False positive, close it out

**Step 3: Containment, Eradication, and Recovery**

**Step 4: Post-Incident Activity**

Document Findings and Close out the Incident in Sentinel


<!--
### Incident 2 - Possible Privilege Escalation

Step 2: Detection & Analysis
    1. Set Severity, Status, Owner
    2. View Full Details 
    3. Observe the Activity Log
    4. Observe Entities and Incident Timelines
    5. “Investigate” the incident and continue trying to determine the scope
    6. Inspect the entities and see if there are any related events
    7. Determine legitimacy of the incident 
    8. If True Positive, continue, if False positive, close it out

Step 3: Containment, Eradication, and Recovery



**Step 4: Post-Incident Activity**

Document Findings and Close out the Incident in Sentinel

### Incident 3 - Brute Force Success (Linux)


Step 2: Detection & Analysis
    1. Set Severity, Status, Owner
    2. View Full Details 
    3. Observe the Activity Log
    4. Observe Entities and Incident Timelines
    5. “Investigate” the incident and continue trying to determine the scope
    6. Inspect the entities and see if there are any related events
    7. Determine legitimacy of the incident 
    8. If True Positive, continue, if False positive, close it out

Step 3: Containment, Eradication, and Recovery



**Step 4: Post-Incident Activity**

Document Findings and Close out the Incident in Sentinel

### Incident 4 - Possible Malware Outbreak

Step 2: Detection & Analysis
    1. Set Severity, Status, Owner
    2. View Full Details 
    3. Observe the Activity Log
    4. Observe Entities and Incident Timelines
    5. “Investigate” the incident and continue trying to determine the scope
    6. Inspect the entities and see if there are any related events
    7. Determine legitimacy of the incident 
    8. If True Positive, continue, if False positive, close it out

Step 3: Containment, Eradication, and Recovery



**Step 4: Post-Incident Activity**

Document Findings and Close out the Incident in Sentinel

## Stage IV - Secure Cloud Configuration

### Implementing Security Controls

### Metrics Before Hardening / Security Controls
To collect the metrics for the secured environment, Network Security Groups were hardened by blocking ALL traffic (with the exception of my workstation), and built-in firewalls enabled. Azure Key Vault and Storage Container were protected by disabling access to public endpoints and replacing them with rivate endpoints.

### Enabling Regulatory Compliance in MDC

## Architecture After Hardening / Security Controls
![Architecture Diagram](./assets/images/soc-honeynet/architecture-after-5.png)<br>


## Attack Maps After Hardening / Security Controls

```All map queries returned no results due to no instances of malicious activity for the 24-hour period after hardening.```

### NSG Allowed Malicious Inbound Flows
![NSG Allowed Inbound Malicious Flows](./assets/images/soc-honeynet/nsg-after.png)<br>



### Linux SSH Authentication Failures
![Linux Syslog Auth Fail](./assets/images/soc-honeynet/syslog-after.png)<br>



### Windows RDP/SMB Authentication Failures
![Windows RDP/SMB Auth Fail](./assets/images/soc-honeynet/windows-rdp-smb-after.png)<br>



### MS SQL Server Authentication Failures
![MSSQL Server Auth Fail](./assets/images/soc-honeynet/mssql-after.png)<br>


## Metrics After Hardening 

The following table shows the measurements taken after applying the security controls the environment and observing for another 24 hours: <br />
2023-07-11 22:15<br />
2023-07-12 22:15

| Metric                   | Count
| ------------------------ | -----
| SecurityEvent            | 22668
| Syslog                   | 24
| SecurityAlert            | 0
| SecurityIncident         | 0
| AzureNetworkAnalytics_CL | 0

### Impact of Security Controls 

| Metric                                       | Change post-hardening
| -------------------------------------------- | -----
| SecurityEvent (Windows VMs)                  | 82.78%
| Syslog (Linux Vms)                           | 99.50%
| SecurityAlert (Microsoft Defender for Cloud) | 100%
| SecurityIncident (Sentinel Incidents)        | 100%
| AzureNetworkAnalytics_CL                     | 100%

## Conclusion

In this project, a mini SOC honeynet was constructed in Microsoft Azure utilizing Log Analytics with Microsoft Sentinel. Sentinel used logs ingested by a Log Analytics workspace to trigger alerts and create incidents. Next, logging was enabled and data collected on the insecure environment based on established security metrics, before applying security controls. The logs and data were reassessed after implementing security measures. As a result, the number of security events and incidents were drastically reduced after the security controls were applied. 

It is worth noting that if the resources within the network were heavily utilized by regular users, it is likely that more security events and alerts may have been generated within the 24-hour period following the implementation of the security controls.


---->