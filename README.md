<img width="2150" height="1260" alt="image" src="https://github.com/user-attachments/assets/ea56823f-873f-4b65-9798-393ea9c8cd03" />

# Azure SOC Honeypot 🍯

In this project, I deployed a Honeypot in Microsft Azure by exposing two virtual machines (Windows & Linux) to the public internet. I then configured a central log repository so logs can be ingested into Log Analytics workspace, which is then used by Microsoft Sentinel to build attack maps. I allowed the Honeypot to be exposed to the internet for 24 hours so that I could capture and observe live brute force attack data.

The main objective of this lab is to: 
- collect and analyze logs using KQL
-  Built an attack map workbook to visualize attacker IPs and their countries
-   Create a custom detection rule and responded to a confirmed brute-force attack incident

the two main metrics that I will focus on are:

- SecurityEvent (Windows Event Logs)
- Syslog (Linux Event log)


---

## ⚙️ Project Overview
- **Environment:** Microsoft Azure 
- **Objective:** Collect and analyze malicious traffic and creating attack maps using Microsoft Sentinel 
- **Focus Areas:** Honeypot setup, log forwarding, KQL queries, attack visualization

## 🏗️ Lab Architecture

<img width="1272" height="756" alt="image" src="https://github.com/user-attachments/assets/4ab3a224-3b6d-4346-b9b6-c23d7614c5a6" />

The architecutre consists of:
- Cloud platform (Microsoft Azure)
- Virtual Network (VNet)
- Network Security Group (NSG)
- Virtual Machines ( 1 windows, 1 linux)
- Log Analytics Workspace
- Microsoft Sentinel
- Kusto Query Language (KQL)
---
## 🐝 Honeypot?

A **honeypot** is a decoy system made to look vulnerable so attackers interact with it.  
It helps security teams **attract and detect threats, gather intelligence, test defenses, and study real-world attack behavior** without risking production systems.

In this project, the Azure VMs were configured as a honeypot by exposing it to the public internet.  
Attackers attempted brute-force logins, and these events were captured, forwarded, and analyzed in **Microsoft Sentinel**.  

---

## 📌 Lab Checklist

1. **Create two virtual machines in Azure (1 Windows, 1 Linux)** - *Honeypot*
   - Opened the VM to the public internet by configuring the **Network Security Group (NSG)** to allow all inbound traffic and Disabling the **Windows Firewall** inside the VM.
   - *I tried to use authentic sounding VM names that would look like real infrastructure to make the honeypot more attractive to attackers. eg. CORP-NET-DEV (for Windows) and CORPNET (for Linux)*

Review of important components of set up 

**Resource Group Creation**

<img width="626" height="279" alt="Extra 1" src="https://github.com/user-attachments/assets/e879227c-64d5-4437-8d15-4df415684957" />

```
Basics

Resource Group
  └─ Subscription: Azure Subscription 1  
  └─ Resource Group Name: RG-willis 
  └─ Region: (US) East US 2
Review + Create
  └─ Create (When Prompted)
```
**Virtual Network Creation**

<img width="626" height="279" alt="Extra 1" src="https://github.com/user-attachments/assets/3ad66e12-4e04-4654-b7cf-7f832f85da22" />

```
Basics

Project Details
  └─ Subscription: Azure Subscription 1  
  └─ Resource Group Name: RG-willis 
Instance Details
  └─ Virtual Network Name: vnet-willis
  └─ Region: (US) East US 2
Review + Create
  └─ Create (When Prompted)
```

**VM Creation**

<img width="2360" height="750" alt="image" src="https://github.com/user-attachments/assets/a99a66e6-a1a9-4f4b-84e2-c1f9abed98d5" />

***Windows***

CORP-NET-DEV
Windows 10 Pro, version 22H2 - x64 Gen2
Size:  Standard_D2s_v3 - 2 vcpus, 8 GiB memory 
Type: Premium SSD

```
Basics

Project Details
  └─ Subscription: Azure Subscription 1  
  └─ Resource Group: RG-willis
Instance Details 
  └─ Virtual Machine Name: CORP-NET-EAST  
  └─ Region: (US) East US 2
  └─ Image: Windows 10 Pro, Version 22H2 - x64 Gen2  
Size
  └─ Standard_B1s 1 vCPU, 1 GiB RAM  
Administrator Account
  └─ Username: YourUsername
  └─ Password: YourPassword
  └─ Confirm Password: YourPassword  
Licensing  
  └─ ☑ I confirm I have an eligible Windows 10/11 license
```

```
Disks

OS Disk
  └─ OS Disk Type: Standard HDD 
```

```
Networking

Network Interface
  └─ Virtual Network: Create New 
  └─ Subnet: Default 
  └─ Public IP: (new)  
  └─ NIC Network Security Group: Basic
  └─ ☑ Delete public IP and NIC when VM is deleted 
```

```
Monitoring

Diagnostics
  └─ Boot Diagnostics: Disable 
```

```
Review + Create
  └─ Create (When Prompted)
```

<img width="2868" height="1080" alt="image" src="https://github.com/user-attachments/assets/11665d2a-2edc-4687-8773-6442b4dcb9ad" />

<img width="2838" height="1184" alt="image" src="https://github.com/user-attachments/assets/e4ee1adb-a541-4a57-8a6c-eb7ca444b377" />

<img width="2878" height="1210" alt="image" src="https://github.com/user-attachments/assets/c6633f62-cf50-435a-b549-cdbbdab8776a" />

***Linux***

CORPNET
Ubuntu Server 22.04 LTS - x64 Gen2
Size: Image default
Type: premium SSD


2. **NSG Configuration**  

- Configure the NSG for each vm to allow all onbound traffic.
- Turn off Firewall in windows vm
- Note: The Ubuntu VM firewall (UFW) is disabled by default on most cloud platforms. Network security is handled by the cloud provider (e.g., Azure NSG, AWS Security Group).
- To check the firewall status in linux vm: sudo ufw status
If needed, you can turn it off manually: sudo ufw disable


<img width="1387" height="704" alt="image" src="https://github.com/user-attachments/assets/392d91c8-917f-4b8e-ba04-9854c594dff3" />

---
<img width="2340" height="340" alt="image" src="https://github.com/user-attachments/assets/ae83edfb-33e5-4771-bab1-fd2fefd7ad1b" />

---

2. **Configure log forwarding**  
   - Set up an **Azure Log Analytics Workspace**  
   - Configured the **Azure Monitoring Agent** to forward security logs from the VM
  
<img width="2870" height="1654" alt="image" src="https://github.com/user-attachments/assets/ad6dbfdf-e67b-4ef7-97a6-e90c8381a8c9" />

---
<img width="2874" height="1566" alt="image" src="https://github.com/user-attachments/assets/a976dc74-d701-4978-b9f0-cf59b53e53a4" />


3. **Connect to a SIEM (Microsoft Sentinel)**  
   - Created a **Microsoft Sentinel** instance  
   - Connected Sentinel to the Log Analytics workspace


### 4. Query failed login attempts (raw logs + SIEM)
- Reviewed raw logs in **Windows Event Viewer** to confirm multiple Event ID **4625** (failed logon) records.
- Ran KQL queries in Sentinel to extract attacker IPs and correlate them with geographic data.

**Windows Event Viewer – Multiple Failed Logins:**  
<img width="2574" height="1554" alt="image" src="https://github.com/user-attachments/assets/5a53120f-63d4-4b00-9437-613f5c207bc5" />


**KQL Query Output in Sentinel:**  
<img width="1435" height="458" alt="image" src="https://github.com/user-attachments/assets/a0aca62f-1249-4733-a748-31e26f121374" />

---

We will use these logs to plot this map of where all the different attackers are coming from. In order to do that we need some type of geographic data - Import the spreadsheet we are going to use to resolve IP addresses to actual physical locations on a map.

- Upload the spreadsheet (CSV/XLSX) to Sentinel.
- Create a **Watchlist** from the uploaded file (IP ranges → geo metadata).
- Use the watchlist in KQL queries to enrich attacker IPs with country/region data.

- [Download GeoIP Spreadsheet](https://drive.google.com/file/d/13EfjM_4BohrmaxqXZLB5VUBIz2sv9Siz/view)

<img width="2864" height="1506" alt="image" src="https://github.com/user-attachments/assets/1e5907e2-0473-4678-9571-3210a3252903" />


### 5. Attack map creation
We will use the logs and the watchlist to visualize attacker locations on a map in Microsoft Sentinel.

**Steps**

1. **Create a new Workbook** in Microsoft Sentinel.  

<img width="1396" height="777" alt="image" src="https://github.com/user-attachments/assets/bc2cf5cb-8ffa-467c-a258-bcc2106a2dc1" />

2. **Delete prepopulated elements** and **add a "Query" element**.  

3. **Open the Advanced Editor** tab and paste the workbook JSON.

<img width="2880" height="1596" alt="image" src="https://github.com/user-attachments/assets/36c034f6-2692-4133-bff6-3435c868a44a" />

4. **Observe the workbook — verify the query is working correctly.**

<img width="2880" height="1512" alt="image" src="https://github.com/user-attachments/assets/5a6b258f-4a33-4161-9b0c-3f2aa53a156f" />

5. **Observe map settings — configure data source, aggregation, tooltip fields, and time range.**

<img width="2868" height="1590" alt="image" src="https://github.com/user-attachments/assets/d2ed25c8-9f4f-495f-a255-e66d5a84f1d2" />

6. **Observe the final attack map — attacker IPs resolved to locations plotted on the map.**

<img width="2348" height="1130" alt="image" src="https://github.com/user-attachments/assets/18a56ab6-03b7-407e-9df5-4cad7efdf7a3" />



---

## Conclusion & Lessons Learned

This project provided a practical, end-to-end example of a small SOC pipeline in Azure:
- Deploy a honeypot VM and intentionally expose it to the internet to attract automated scanning and brute-force attempts.
- Forward raw security events to a Log Analytics workspace via the Azure Monitoring Agent.
- Ingest data into Microsoft Sentinel, enrich attacker IPs using a GeoIP watchlist, and visualize activity with a workbook map.

**Key lessons learned**
- Exposed VMs are probed almost immediately — even simple lab VMs attract high-volume automated traffic.
- Raw logs (Windows Event Viewer) are the authoritative source; SIEM dashboards and workbooks provide the analysis layer on top of those logs.
- Enrichment (watchlists, threat intel) dramatically improves situational awareness and makes visualizations meaningful.
- Workbooks are flexible and allow you to combine KQL results and watchlists into a single interactive visualization.
- 
---

## Safety, Responsible Use & Cleanup

**Important — only run this type of honeypot in an isolated lab environment. Never expose production systems.**

**Before finishing the lab**
- Remove or restrict any public-facing NSG rules that allow all inbound traffic.
- Re-enable the Windows firewall on the VM.
- If you no longer need the VM, **decommission and delete** the VM, associated disks, and public IP to avoid ongoing risk and charges.
- Clean up associated resources: Log Analytics workspace, Sentinel workbook (if desired), and watchlists if not needed.

**Cost & security note**
- Leaving cloud VMs or workspaces running can incur charges and create security risks. Confirm deletion or set up automation to tear down lab resources.

## Next Steps

- Add richer telemetry: enable Sysmon, install EDR agents in a safe lab image, capture network flow logs.
- Create custom Sentinel analytic rules to generate alerts for suspicious patterns (e.g., repeated 4625 from multiple IPs).
- Integrate threat intelligence feeds (AbuseIPDB, VirusTotal) for automated enrichment.
- Build a simple playbook (SOAR) to test automated containment/notification actions in the lab.

