# Threat Modeling with MITRE ATT\&CK

This document outlines how the home lab uses MITRE ATT\&CK as a framework for understanding, simulating, and detecting adversarial behavior. It includes tools like ATT\&CK Navigator, MITRE D3FEND, and the open-source DETTECT framework created by Rabobank.

---

## 🎯 Purpose

* Use MITRE ATT\&CK to guide red/blue team simulations
* Identify relevant TTPs (tactics, techniques, and procedures)
* Map detections in Wazuh to real-world attack behaviors
* Create a repeatable workflow for purple teaming and detection validation

---

## 🧰 Tools Used

### 🔍 MITRE ATT\&CK Navigator

* Web-based matrix visualizer for tracking coverage
* Self-hosted in lab via Docker using the [official GitHub repo](https://github.com/mitre-attack/attack-navigator)
* Allows layering TTPs, detections, mitigations, etc.
* Used to visually track techniques executed in lab scenarios

#### 🐳 Docker Setup for Navigator

```bash
# Clone the Navigator repo
git clone https://github.com/mitre-attack/attack-navigator.git
cd attack-navigator/nav-app

# Build and run the container
docker build -t attack-navigator .
docker run -d -p 4200:4200 attack-navigator
```

* Access via: `http://<your-lab-ip>:4200`

---

### 🧠 MITRE D3FEND

* Defensive counterpart to ATT\&CK
* Used to identify detection and mitigation strategies
* Official site: [https://d3fend.mitre.org](https://d3fend.mitre.org)

---

### 📘 DETTECT (By Rabobank)

* YAML-based detection-as-code framework
* Maps telemetry, detection logic, and ATT\&CK techniques
* Repository: [https://github.com/rabobank-cdc/DETTECT](https://github.com/rabobank-cdc/DETTECT)

#### 🐳 Docker Setup for DETTECT Web UI

```bash
# Clone the DETTECT repo
git clone https://github.com/rabobank-cdc/DETTECT.git
cd DETTECT

# Start via Docker Compose
docker-compose up -d

# Access via:
http://localhost:8050 or http://<your-lab-ip>:8050
```

* Use this interface to browse techniques, log sources, telemetry, and detection ideas for each ATT\&CK ID

---

## 🧪 Lab Application Examples

| Technique ID | Name                    | Simulated Via          | Detection in Wazuh    |
| ------------ | ----------------------- | ---------------------- | --------------------- |
| T1059        | Command & Scripting     | PowerShell on endpoint | Wazuh Sigma rules     |
| T1078        | Valid Accounts          | AD credential use      | Event ID 4624/4625    |
| T1021.001    | Remote Desktop Protocol | RDP from Kali → target | Auth logs + firewall  |
| T1003.001    | LSASS Dumping           | Procdump from admin    | Sysmon + Wazuh alerts |

---

## 🗺️ Workflow

1. **Select TTPs** from MITRE ATT\&CK matrix
2. **Use Navigator** to visualize selection
3. **Simulate activity** using attacker VM (Kali)
4. **Monitor logs** in Wazuh (Winlogbeat, Sysmon, etc.)
5. **Correlate detections** to ATT\&CK techniques
6. **Refine GPOs, Sigma rules, alerting** accordingly

---

## 📎 Resources

* MITRE ATT\&CK Navigator: [https://github.com/mitre-attack/attack-navigator](https://github.com/mitre-attack/attack-navigator)
* MITRE D3FEND: [https://d3fend.mitre.org](https://d3fend.mitre.org)
* DETTECT (Rabobank): [https://github.com/rabobank-cdc/DETTECT](https://github.com/rabobank-cdc/DETTECT)

---

> 💡 This threat modeling approach ties detection engineering to real-world adversary behavior and helps make security outcomes measurable and tactical in the lab.
