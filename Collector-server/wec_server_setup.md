# Windows Event Collector (WEC) Setup

This document outlines the configuration of the Windows Event Collector (WEC) server in the home lab. The WEC server centralizes event logs from domain-joined endpoints for monitoring and analysis using tools like Wazuh.

---

## Purpose

* Centralize and normalize Windows event logs
* Forward logs from all domain-joined endpoints (servers and workstations)
* Integrate with Wazuh for real-time alerting and SIEM correlation

---

## Host Info

* **Hostname:** `LAB-WEC`
* **OS:** Windows Server 2022
* **VLAN:** Lab (VLAN 20)
* **Domain Joined:** Yes (`corp.local`)

---

## Setup Steps

### 1. Install WEC Role

```powershell
# Open PowerShell as Administrator
Install-WindowsFeature -Name Windows-Server-Backup, EventCollector -IncludeManagementTools
```

### 2. Start & Configure Windows Event Collector Service

```powershell
sc config Wecsvc start= auto
Start-Service Wecsvc
```

### 3. Create Event Subscription

1. Open **Event Viewer** → **Subscriptions** tab
2. Click **Create Subscription**
3. Choose a name like `Forwarded Events`
4. Select **Collector initiated** (or source initiated if using GPO)
5. Define criteria (e.g., Security logs, Event ID filters)
6. Assign to a computer group (e.g., `Domain Computers`)

### 4. Configure GPO for Source Computers

* Open **Group Policy Management** → Create a new GPO (e.g., `Event Log Forwarding`)
* Apply to target OU (e.g., workstations or servers)

#### GPO Settings:

* `Computer Configuration` → `Policies` → `Administrative Templates` → `Windows Components` → `Event Forwarding`

  * Enable **Configure target subscription manager**

    * Value: `Server=https://<WEC-IP>:5985/wsman/SubscriptionManager/WEC,Refresh=5`
  * Enable **Windows Event Collector** and set to **Start Automatically (delayed start)**

* Link GPO to desired OU and run `gpupdate /force` on client

---

## Validation

* On client: Check `wevtutil ep` to verify connection to the subscription
* On WEC: Navigate to `Event Viewer` → `Forwarded Events`
* Logs should start appearing from domain endpoints

---

## Integration with Wazuh

* Wazuh agent installed on `LAB-WEC`
* WEC forwards logs to `/var/ossec/logs/` via the Wazuh agent
* Used for correlation, alerting, and MITRE ATT\&CK mapping

---

> 🧠 This WEC setup provides centralized visibility into endpoint activity across the domain and serves as a crucial log source for SIEM and threat detection efforts.
