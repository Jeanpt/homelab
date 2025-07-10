# Active Directory + DNS Server Setup (Windows Server 2022)

This guide documents the initial setup of a Domain Controller (DC) with Active Directory Domain Services (ADDS) and DNS in a home lab using Windows Server 2022.

---

## Overview

* **VM Host:** Proxmox (or any hypervisor)
* **OS:** Windows Server 2022 (GUI)
* **Services Installed:**

  * Active Directory Domain Services (ADDS)
  * DNS Server
* **Domain:** `corp.local`
* **Hostname:** `LAB-DC01` (example)
* **IP Address:** Static IP (e.g., `10.10.10.10`) on Lab VLAN (e.g., `10.10.10.0/24`)

---

## Step 1: Create the VM

* Create a new VM in Proxmox

  * Assign at least 2 vCPUs, 4–8 GB RAM, 60+ GB disk
  * Mount Windows Server 2022 ISO
* Install with **Desktop Experience**
* Complete setup with admin password and basic system config

---

## Step 2: Configure Static IP

1. Open `Control Panel` → `Network and Sharing Center` → `Change adapter settings`
2. Right-click `Ethernet` → Properties → `Internet Protocol Version 4 (TCP/IPv4)`
3. Set a static IP (example):

   ```
   IP Address:     10.10.10.10
   Subnet Mask:    255.255.255.0
   Default Gateway:10.10.10.1
   Preferred DNS:  10.10.10.10  (point to itself)
   ```

---

## Step 3: Rename Server

1. Open `Server Manager` → `Local Server`
2. Click the computer name to rename the server
3. Use a hostname like `LAB-DC01`
4. Reboot when prompted

---

## Step 4: Install ADDS and DNS Roles

1. Open **Server Manager** → `Add Roles and Features`
2. Proceed through the wizard:

   * Role-based or feature-based installation
   * Select the local server
   * Select roles:

     *  Active Directory Domain Services
     *  DNS Server
3. Continue through the wizard and click **Install**

---

## Step 5: Promote to Domain Controller

1. After installation, click **“Promote this server to a domain controller”**
2. Select **Add a new forest** → Root domain: `corp.local`
3. Set Directory Services Restore Mode (DSRM) password
4. Accept default settings for DNS, NetBIOS, paths, etc.
5. Review and install
6. The server will automatically reboot after promotion

---

## Step 6: Post-Promotion Tasks

1. Log in using the domain account: `corp\Administrator`
2. Verify services:

   * Open `dsa.msc` (Active Directory Users and Computers)
   * Open `dnsmgmt.msc` (DNS Manager)
   * Open `gpmc.msc` (Group Policy Management)

---

## Step 7: (Optional) Group Policy Setup

1. Open `gpmc.msc`
2. Create new GPOs under `corp.local`
3. Link GPOs to desired OUs (Organizational Units)

---

## Notes

* This Domain Controller acts as the identity and DNS backbone of your lab
* It can be expanded with DHCP, CA (Certificate Authority), NPS, etc.
* Ensure it is always online — if it goes down, domain login and name resolution will fail

---

## Next Steps

* Join domain clients (e.g., LAB-ADMIN1)
* Deploy Windows Event Forwarding (WEC)
* Push GPOs for log collection, security baselines, etc.

---

> ✅ This configuration lays the foundation for a realistic enterprise-style home lab.
