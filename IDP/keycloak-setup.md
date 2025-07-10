# Keycloak Identity Provider (IdP) Setup in a Home Lab

This guide documents the steps I took to deploy and configure Keycloak as an Identity Provider (IdP) in a segmented home lab environment using Docker, Ubuntu, and VLAN-based network segmentation. The lab was used to simulate real-world OAuth2 and OpenID Connect (OIDC) authentication and authorization flows.

---

## Lab Overview

* **IdP Platform**: Keycloak 22.x (Docker deployment)
* **OS**: Ubuntu 22.04 Server (minimal install)
* **Virtualization**: Proxmox
* **Network**: LAB VLAN (10.10.10.0/24)
* **Protocols**: OAuth 2.0, OpenID Connect (OIDC)
* **Clients**: Simulated frontend (SPA) and backend API (Node.js)
* **Use Cases**: Single Sign-On, RBAC, token validation, secure access to protected APIs

---

## 1. Prepare the VM

### 1.1 Create a new VM in Proxmox

* 2 CPUs, 4–8 GB RAM, 20+ GB disk
* Connect to your LAB VLAN (e.g., VLAN 20)

### 1.2 Install Ubuntu 22.04

* Use "Erase disk and install Ubuntu"
* Set a static IP using Netplan or GUI (e.g. `10.10.10.30`)

```bash
sudo nano /etc/netplan/00-installer-config.yaml
# Set your static IP, gateway, and DNS

sudo netplan apply
```

---

## 2. Deploy Keycloak via Docker

```bash
# Update and install Docker
sudo apt update && sudo apt install docker.io docker-compose -y

# Pull Keycloak image
docker pull quay.io/keycloak/keycloak:22.0.5

# Run Keycloak container in development mode
docker run -d \
  --name keycloak \
  -p 8080:8080 \
  -e KEYCLOAK_ADMIN=admin \
  -e KEYCLOAK_ADMIN_PASSWORD=changeme \
  quay.io/keycloak/keycloak:22.0.5 \
  start-dev
```

> 📍 After deployment, access the Keycloak admin console at:  
> `http://10.10.10.30:8080`

Login:
```
Username: admin
Password: changeme
```

---

## 3. Keycloak Initial Configuration

### 3.1 Create Realm

* Name: `lab`

### 3.2 Create a Test User

* Username: `testuser`
* Manually set a password
* Disable "Temporary Password"
* Optionally assign built-in or custom roles

### 3.3 Create Clients

**Frontend App (OIDC Public Client)**

* Client ID: `frontend-app`
* Client Type: `Public`
* Redirect URI: `http://localhost:3000/*`
* Enable flows:
  * Authorization Code
  * Refresh Token

**Backend API (OIDC Confidential Client)**

* Client ID: `api-server`
* Client Type: `Confidential`
* Configure client secret
* Enable service account access (optional)

---

## 4. Configure Token Settings

* Access Token lifespan: `5 minutes`
* Refresh Token lifespan: `30 minutes`
* Enable Refresh Token Rotation: `true`
* Enable Client Credentials Flow for machine-to-machine API access (optional)

---

## 5. Test OIDC Authentication Flow

Open a browser and visit the authorization endpoint:

```text
https://10.10.10.30:8080/realms/lab/protocol/openid-connect/auth?
  client_id=frontend-app
  &redirect_uri=http://localhost:3000/callback
  &response_type=code
  &scope=openid email profile
```

Keycloak will prompt for login, then redirect with an auth code.

---

## 6. Exchange Authorization Code for Tokens

```bash
curl -X POST https://10.10.10.30:8080/realms/lab/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "client_id=frontend-app" \
  -d "grant_type=authorization_code" \
  -d "code=<code_here>" \
  -d "redirect_uri=http://localhost:3000/callback"
```

Response:

```json
{
  "access_token": "eyJhbGciOi...",
  "id_token": "eyJhbGciOi...",
  "expires_in": 300,
  "refresh_token": "...",
  ...
}
```

Inspect the token at [https://jwt.io](https://jwt.io) to verify claims such as `sub`, `aud`, `exp`, and `scope`.

---

## 7. Protect Backend API with JWT

* Used `express-jwt` and `jwks-rsa` in a Node.js API
* Validated signature using Keycloak's JWKS endpoint
* Enforced access control based on token claims:
  * `roles`
  * `aud`
  * `scope`
* Simulated token expiration, refresh, and misuse scenarios

---

## Notes

* Keycloak is running in development mode for lab purposes; production setups should use TLS, PostgreSQL backing, and persistent volumes
* This setup provides full simulation of modern OIDC flows with secure token validation
* The configuration can be expanded to integrate with Grafana, Jenkins, or external directories like LDAP

---
