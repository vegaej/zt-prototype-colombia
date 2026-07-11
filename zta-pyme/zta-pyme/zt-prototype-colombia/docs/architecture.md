# Architecture Overview

## Nodes

### Edge Server (PEP)
- **Public IP:** 10.195.26.114/22 (eno1)
- **VLAN 10:** 192.168.10.50/24 (eno2.10)
- **VLAN 30:** 192.168.30.50/24 (eno2.30)
- **Role:** intercepts all incoming traffic and enforces access decisions made by the PDP
- **Service:** `pomerium-proxy` running in Docker (network_mode: host)

### Management Server (PDP + IdP)
- **IP:** 192.168.30.1/24 (VLAN 30)
- **PDP role:** evaluates access policies and issues allow/deny decisions
- **IdP role:** Keycloak authenticates users and issues OIDC tokens with group claims
- **Services:** `pomerium-mgmt`, `keycloak`, `postgres` running in Docker

### Web Server — Cloud (protected resource)
- **IP:** 10.195.23.159/22
- Apache 2 + PHP 8.1
- Accessible only through Pomerium

### Database Server — Cloud (protected resource)
- **IP:** 10.195.23.160/22
- MySQL 8.0
- Accessible only from the web server

---

## Authentication Flow

```
User → [Edge Server: PEP] → redirect
     → [Management Server: authenticate]
     → [Keycloak: login form]
     → User submits credentials
     → [Keycloak: OIDC token with groups claim]
     → [Management Server: authorize] → allow / deny
     → [Edge Server: PEP] → forward to resource
     → [Web / DB Server] → HTTP 200
```

---

## Network Segmentation

```
Internet
    │
[Edge Server eno1: 10.195.26.114/22]
    │
[Switch OS6860E-24]
    ├── VLAN 10 (192.168.10.0/24) → Internal users
    └── VLAN 30 (192.168.30.0/24) → Management server (PDP + IdP)
```

---

## Access Policies

| Route | Target | Policy |
|---|---|---|
| `recurso.empresa.local` | 10.195.23.159/22 | Authenticated + member of `pyme-users` |
| `app.empresa.local` | 192.168.10.100/24 | Any authenticated user |
| `authenticate.empresa.local` | 192.168.30.1/24:443 | OIDC callback (internal) |

---

## Required Credential Files (not included in repo)

### `sr530/.env`
```env
POMERIUM_SHARED_SECRET=<generate with: openssl rand -base64 32>
POMERIUM_COOKIE_SECRET=<generate with: openssl rand -base64 32>
KC_CLIENT_SECRET=<copy from Keycloak Admin → Clients → pomerium → Credentials>
```

### `pc-gestion/.env.mgmt`
```env
KC_DB_PASS=<PostgreSQL password>
KC_ADMIN_PASS=<Keycloak admin password>
KC_CLIENT_SECRET=<same value as sr530/.env>
POMERIUM_SHARED_SECRET=<same value as sr530/.env>
POMERIUM_COOKIE_SECRET=<same value as sr530/.env>
```

---

## Startup Sequence

Start the management server first and wait ~40 seconds for Keycloak to initialize,
then start the edge server. The PEP will fail to sync if the PDP is not ready.

```bash
# 1. Management server
cd ~/keycloak-mgmt
docker compose --env-file .env.mgmt up -d

# 2. Verify Keycloak is ready
curl http://192.168.30.1:8080/realms/empresa

# 3. Edge server
cd ~/pomerium
docker compose up -d
```
