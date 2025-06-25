# Home Lab Network Topology & Infrastructure

This document outlines the physical and logical network layout of the home lab, including firewall, switches, VLANs, and device segmentation.

---

## 🔧 Physical Hardware

### 🧱 Firewall:

* **Model:** Protectli Vault VP2420
* **Firewall Software:** OPNsense
* **Purpose:** Gateway, routing, VLAN management, DHCP (per VLAN), DNS (optional), internet egress control

### 🔌 Switches:

* **Downstairs Core Switch:** D-Link DGS-1510-52 (Layer 3)
* **Upstairs Access Switch:** Netgear ProSAFE GS108E (smart switch)
* **Purpose:** VLAN-aware switching and trunking between rooms/devices

---

## 🌐 VLAN Configuration

| VLAN ID | Name  | Subnet          | Purpose                  |
| ------- | ----- | --------------- | ------------------------ |
| 10      | Admin | 192.168.10.0/24 | Personal PC, mgmt access |
| 20      | Lab   | 10.10.10.0/24   | Home lab + test VMs      |

### 💡 Key Concepts:

* VLANs are trunked across switches
* Inter-VLAN routing is handled by OPNsense
* DHCP is assigned per VLAN on OPNsense
* Devices are assigned VLANs via port configuration

---

## 🧵 Switch Port Layout

### Downstairs D-Link DGS-1510-52:

* **Port 1:** Uplink to OPNsense firewall (Tagged: VLAN 10, 20)
* **Port 7:** Uplink to Netgear upstairs (Tagged: VLAN 10, 20)

### Upstairs Netgear GS108E:

* **Port 1:** Uplink to D-Link (Tagged VLAN 10, 20)
* **Port 2:** Personal gaming PC (Untagged VLAN 10)
* **Port 3:** Lab PC / Proxmox host (Untagged VLAN 20)

---

## 🛜 DHCP & Static IPs

* **DHCP:** Managed by OPNsense per VLAN
* **Static IPs:** Reserved for:

  * Domain Controller (e.g., 10.10.10.10)
  * SIEM Server (e.g., 10.10.10.50)
  * OPNsense gateway interfaces (e.g., 192.168.10.1, 10.10.10.1)

---

## 🧪 Network Flow Summary

```
               [ Internet ]
                   |
              [ OPNsense Firewall ]
                   |
           -------------------------
           | VLAN 10 | VLAN 20     |
           |         |             |
      [ D-Link Switch (core) ]
           |
      [ Netgear Switch (access) ]
        |                  |
 [Gaming PC - VLAN 10]   [Lab PC - VLAN 20]
```

---

## ✅ Outcome

* Layered network segmentation via VLANs
* Inter-VLAN routing controlled and logged by firewall
* Personal devices separated from lab VMs
* Secure foundation for enterprise-like scenarios (SIEM, AD, DNS, WEC, etc.)

---

> ⚠️ Note: Use proper trunk/tag settings between switches and on uplinks. Test VLAN routing and isolation with `ping` and firewall rules.
