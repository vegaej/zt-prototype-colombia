# Zero Trust Prototype for Colombian SMEs

An open-source Zero Trust architecture prototype built with Pomerium, Keycloak, and Docker on standard off-the-shelf hardware. Developed as an undergraduate thesis project at Pontificia Universidad Javeriana.

## Authors

- Juan Diego Tenjo Borda
- Eugenio José Vega Contreras

**Advisor:** Ing. Luis Carlos Trujillo Arboleda, MSc.
**Faculty of Engineering — Electronics Department, 2026**

---

## Architecture

The prototype distributes the roles defined by NIST SP 800-207 across two physical nodes:

| Node | Role | Components |
|---|---|---|
| Edge server | PEP (Policy Enforcement Point) | Pomerium proxy, inter-VLAN router, NAT |
| Management server | PDP (Policy Decision Point) + IdP | Pomerium mgmt, Keycloak, PostgreSQL |
| Web server (cloud) | Protected resource | Apache, PHP |
| Database server (cloud) | Protected resource | MySQL 8.0 |

The local network is segmented into two VLANs:
- **VLAN 10** — internal users
- **VLAN 30** — administration (PDP + IdP)

---

## Repository Structure

```
zta-pyme/
├── sr530/                  # Edge server (PEP)
│   ├── docker-compose.yml
│   ├── config/
│   │   └── config.yaml
│   └── hosts.txt
├── pc-gestion/             # Management server (PDP + IdP)
│   ├── docker-compose.yml
│   ├── config/
│   │   └── config.yaml
│   └── hosts.txt
└── docs/
    └── architecture.md
```

---

## Requirements

- Docker Engine >= 24.0 and Docker Compose >= 2.0
- Ubuntu Server 22.04 LTS (edge server and cloud servers)
- Ubuntu 22.04 Desktop or Linux Mint (management server)
- Managed switch with 802.1Q VLAN support
- Server with two Gigabit network interfaces (edge server)

## Software Stack (open source, no licensing cost)

- [Pomerium v0.32.4](https://www.pomerium.com)
- [Keycloak 24.0](https://www.keycloak.org)
- [PostgreSQL 15](https://www.postgresql.org)
- [Docker Engine](https://www.docker.com)

---

## Getting Started

1. Generate the internal PKI (see installation manual — section 4).
2. Copy the files from `sr530/` to the edge server.
3. Copy the files from `pc-gestion/` to the management server.
4. Create the `.env` and `.env.mgmt` credential files (see security notice below).
5. Follow the startup sequence in the installation manual.

---

## Security Notice

The following files are **not included** in this repository:

- `.env` and `.env.mgmt` (credentials and secrets)
- PKI certificates (`*.pem`, `*.crt`, `*.key`)
- Combined CA bundle (`combined-ca.crt`)

To generate them, follow section 4 of the installation manual included in the thesis document.

---

## Reference

> Tenjo Borda, J. D. & Vega Contreras, E. J. (2026). *Design and Implementation of a Scalable Zero Trust Prototype for Colombian SMEs with Validation in Test Environments and Secure Cloud Access.* Undergraduate Thesis, Pontificia Universidad Javeriana, Bogotá D.C.
