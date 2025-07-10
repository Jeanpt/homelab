**Keycloak Identity Provider (IdP) Setup in a Home Lab
This guide documents how I deployed and configured Keycloak as an Identity Provider (IdP) in my segmented cybersecurity home lab. The goal of this project was to implement real-world authentication and authorization flows using OAuth2 and OpenID Connect (OIDC), and to simulate identity federation and application access across VLANs.

Lab Overview
IdP Platform: Keycloak 22.x (Docker-based deployment)

OS: Ubuntu 22.04 Server (on Proxmox VM)

Network: VLAN-segmented lab (LAB VLAN: 10.10.10.0/24)

Protocols: OAuth 2.0, OpenID Connect

Client Apps: Simulated frontend and backend API for token-based access testing

1. VM Setup
Provisioned a new VM in Proxmox:

2 vCPUs, 4 GB RAM, 20 GB disk

Connected to the LAB VLAN

Installed Ubuntu 22.04 and configured a static IP:

bash
Copy
Edit
sudo nano /etc/netplan/00-installer-config.yaml
# Example IP: 10.10.10.30
sudo netplan apply
2. Deployed Keycloak via Docker
I used Docker for a fast and isolated deployment. Running Keycloak in start-dev mode allowed rapid iteration during setup and testing.

bash
Copy
Edit
sudo apt update && sudo apt install docker.io docker-compose -y
docker pull quay.io/keycloak/keycloak:22.0.5

docker run -d \
  --name keycloak \
  -p 8080:8080 \
  -e KEYCLOAK_ADMIN=admin \
  -e KEYCLOAK_ADMIN_PASSWORD=changeme \
  quay.io/keycloak/keycloak:22.0.5 \
  start-dev
Accessed the Keycloak admin console at http://10.10.10.30:8080

3. Initial Configuration
Inside the Keycloak UI:

Created a new realm called lab

Created a test user and set credentials manually

Defined two clients:

frontend-app: Public client using Authorization Code flow

api-server: Confidential client for server-to-server access

Configured redirect URIs and enabled appropriate flows:

Authorization Code

Refresh Token

Optional: Implicit flow for SPA testing

4. Token Customization
Configured the token settings to reflect real-world security practices:

Access Token lifespan: 5 minutes

Refresh Token lifespan: 30 minutes

Refresh token rotation enabled

Added custom claims (roles, email, etc.)

These settings allowed me to test expiration handling, refresh flows, and role-based access control (RBAC) logic in consuming apps.

5. Simulated OAuth2 + OIDC Flow
Used a mock frontend to initiate an OIDC login:

ruby
Copy
Edit
https://10.10.10.30:8080/realms/lab/protocol/openid-connect/auth?
  client_id=frontend-app
  &redirect_uri=http://localhost:3000/callback
  &response_type=code
  &scope=openid email profile
Captured the authorization code and exchanged it using curl:

bash
Copy
Edit
curl -X POST https://10.10.10.30:8080/realms/lab/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "client_id=frontend-app" \
  -d "grant_type=authorization_code" \
  -d "code=<code_here>" \
  -d "redirect_uri=http://localhost:3000/callback"
Verified the returned access_token and id_token via jwt.io to confirm claims and TTL.

6. Backend API Authorization (Bearer Token)
Tested the end-to-end flow by protecting a Node.js API with Keycloak-issued JWTs:

Integrated express-jwt and jwks-rsa libraries

Verified token signature and expiration

Enforced custom scopes and role claims

Tested expired and invalid token scenarios

Key Takeaways
This lab gave me practical experience in:

Deploying and managing an OAuth2/OIDC-compliant Identity Provider

Understanding access tokens, ID tokens, scopes, and role-based claims

Building and testing real SSO flows across frontend and backend components

Performing token validation, rotation, and introspection

Hardening IdP behavior for production-like scenarios
