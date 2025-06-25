# Wazuh SIEM Setup in a Home Lab

This guide documents the steps to build a centralized Wazuh SIEM server in a home lab environment using Ubuntu, Proxmox, and Windows domain endpoints.

---

## Lab Overview

* **SIEM Platform**: Wazuh 4.7.x (includes Elasticsearch/OpenSearch, Filebeat, and Kibana-based dashboard)
* **OS**: Ubuntu 22.04 Server (recommended by Wazuh)
* **Virtualization**: Proxmox
* **Endpoints**: Domain Controller (Windows Server), LAB-ADMIN1 (Windows 10), WEC01 (Windows Event Collector)
* **Firewall**: OPNsense
* **Switches**: D-Link + Netgear with VLAN segmentation

---

## 1. Prepare the VM

1. **Create a new VM in Proxmox**

   * 2 CPUs, 4–8 GB RAM, 40+ GB disk
   * Connect to your LAB VLAN

2. **Install Ubuntu 22.04**

   * Use "Erase disk and install Ubuntu"
   * Set a static IP (e.g. `10.10.10.50`) in Netplan or via GUI

---

## 2. Install Wazuh Server

```bash
# Update and install dependencies
sudo apt update && sudo apt upgrade -y
sudo apt install curl unzip gnupg -y

# Download and run Wazuh installation script
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
sudo bash wazuh-install.sh -a
```

> ⚠️ Use Ubuntu 22.04 or 20.04. Do NOT use 24.04 yet (incompatible).

After install:

* The Wazuh dashboard will be available at: `https://<your-server-ip>`
* Default credentials and user roles are stored in:

  ```bash
  ~/wazuh-install-files/wazuh-passwords.txt
  ```

---

## 3. Configure Agent on Windows Endpoints

### On your domain-joined Windows machines:

```powershell
# Download Wazuh agent (replace with latest link if needed)
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.7.2-1.msi -OutFile $env:temp\wazuh-agent.msi

# Install and point to your SIEM server
msiexec.exe /i $env:temp\wazuh-agent.msi /q WAZUH_MANAGER="10.10.10.50"

# Register agent
& "$Env:ProgramFiles\ossec-agent\agent-auth.exe" -m 10.10.10.50

# Restart agent service
Restart-Service -Name wazuh
```

---

## 4. Verify Agent Status

* Log into the Wazuh dashboard (`https://<your-ip>`) with `admin`
* Go to **Agents** → Confirm your machines show as **Active**

---

## 5. Firewall/Network

* Ensure TCP port `1514` (UDP + TCP), `1515`, and `55000` are open between agents and the Wazuh manager
* OPNsense firewall was used to segment VLANs (e.g. VLAN 10 for Admin, VLAN 20 for Lab)

---

## 6. User Roles (from passwords file)

| Username          | Role                                |
| ----------------- | ----------------------------------- |
| `admin`           | Web UI and full access              |
| `kibanaserver`    | Wazuh dashboard ↔ indexer connector |
| `kibanaro`        | Read-only dashboard user            |
| `logstash`        | Handles CRUD operations on indices  |
| `readall`         | Read access to all indices          |
| `snapshotrestore` | Snapshot and restore operator       |
| `wazuh`           | API user for CLI or integrations    |

Use `admin` for most dashboard interactions.

---

## Notes

* This server acts as your centralized log collector and security event analysis platform
* More integrations (e.g. Sysmon, Suricata, OTX, etc.) can be added over time
* Follow up by adding dashboards, detection rules, or agent policies

---

## Coming Next

* API lab
* Hosting a vulnerable web app
* Sysmon + OSSEC customization
* Centralized event log forwarding from WEC to SIEM

---

> 🧠 Built for security engineers who want hands-on enterprise-grade visibility in a home lab.
