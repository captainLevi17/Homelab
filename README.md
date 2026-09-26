# Homelab

A homelab built around **Proxmox, OPNsense, local NAS storage, Tailscale, Cloudflare, and Oracle Cloud**.

The environment provides media services, self-hosted applications, DNS, reverse proxying, network experimentation, and selected public-facing services while keeping the majority of workloads and data on-premises.

---

## Architecture

The homelab uses **Starlink** for internet connectivity, with a **TP-Link BE3600 Wi-Fi 7 router** providing the primary home network.

The router connects directly to:

* Desktop PC
* MyCloudPR4100 NAS
* Dell OptiPlex 7040 running Proxmox
* Dell OptiPlex 9020 running OPNsense

The **OptiPlex 9020** provides a separate Lab Network through OPNsense, an HP ProCurve switch, and an HP ProCurve access point.

The **OptiPlex 7040** runs Proxmox and hosts the majority of the homelab services.

An **Oracle Cloud Always Free** instance provides public ingress for selected services. **Tailscale** provides private connectivity between the Oracle Cloud environment and services hosted within the home network.

The physical network topology is documented separately in **draw.io**.

---

## Proxmox

The **Dell OptiPlex 7040** runs Proxmox VE and acts as the primary virtualization host.

### Services

| Service                | Purpose                     |
| ---------------------- | --------------------------- |
| ARR Stack              | Media automation            |
| Jellyfin               | Media streaming             |
| Jellyseerr             | Media requests              |
| qBittorrent            | Downloads                   |
| SABnzbd                | Downloads                   |
| Ubuntu Server          | Testing / development       |
| Web Server             | Self-hosted applications    |
| Nginx Proxy Manager    | Internal reverse proxy      |
| Technitium DNS         | Internal DNS                |
| Other VMs / containers | Testing and experimentation |

---

## Media Stack

The media environment consists of the ARR applications, Jellyseerr, qBittorrent, SABnzbd, and Jellyfin.

* **Jellyseerr** — Media request interface
* **ARR Stack** — Media management and automation
* **qBittorrent / SABnzbd** — Download clients
* **Jellyfin** — Media library and streaming
* **NAS** — Primary media storage

---

## Storage

The **MyCloudPR4100** provides network storage for the homelab.

Primary storage workloads include:

* Movies
* TV shows
* Download storage
* Application data
* Backups
* General file storage

---

## Lab Network

The Lab Network provides an isolated environment for networking experimentation and testing.

### Hardware

* Dell OptiPlex 9020
* OPNsense
* HP ProCurve switch
* HP ProCurve access point

OPNsense provides routing and firewalling for the Lab Network, while the ProCurve switch and access point provide wired and wireless connectivity.

The lab is used for:

* Firewall experimentation
* Routing
* VLANs
* Network service testing
* Lab environments
* Infrastructure learning

---

## DNS and Domain

I own the **`mlsg.net`** domain, which is used for both public-facing services and internal homelab services.

**Cloudflare** manages the domain's DNS.

Public-facing services use subdomains of `mlsg.net`, while internal services use the `.local.mlsg.net` namespace.

For example:

```text
Public:
jellyfin.mlsg.net

Internal:
technitium01.local.mlsg.net
sab.local.mlsg.net
```

This provides a consistent naming scheme while distinguishing internal services from public-facing services.

---

## HTTPS / TLS

Services use **Let's Encrypt** certificates managed through Nginx Proxy Manager.

Certificate issuance uses the **Cloudflare DNS-01 challenge**, allowing certificates to be issued without requiring internal services to be publicly accessible.

This provides trusted HTTPS for both internal and public-facing services.

---

## Reverse Proxy

There are two **Nginx Proxy Manager** instances.

### Internal NPM

The local NPM instance runs within Proxmox and handles reverse proxying and HTTPS for internal services.

Internal DNS resolves `.local.mlsg.net` hostnames to the local infrastructure, allowing clients to access services using their normal HTTPS hostnames.

### Public NPM

The Oracle Cloud NPM instance acts as the public ingress point for selected services.

Requests are forwarded through **Tailscale** to the corresponding service within the home network.

This allows services to remain hosted locally without requiring direct inbound access to the home network.

---

## Oracle Cloud

An **Oracle Cloud Always Free** instance provides the public-facing infrastructure.

It runs:

* Nginx Proxy Manager
* Tailscale

The Oracle instance is used as a public ingress point for services that need to be accessible from the internet.

---

## Tailscale

**Tailscale** provides private connectivity between participating systems across the homelab and Oracle Cloud.

Its primary role is connecting the public Nginx Proxy Manager instance in Oracle Cloud to services hosted within the home network.

This allows traffic to travel from the public NPM instance to internal services without exposing those services directly through inbound port forwarding.

Tailscale can also provide private administrative access to participating homelab systems.

---

## Cloudflare

Cloudflare provides:

* DNS hosting for `mlsg.net`
* Public DNS records
* DNS-01 validation for Let's Encrypt certificates

Public service traffic can therefore use the following general flow:

**Client → Cloudflare → Oracle Cloud NPM → Tailscale → Home Service**

Internal services use the local `.local.mlsg.net` namespace and are handled by the internal Nginx Proxy Manager instance.

---

## Infrastructure Summary

| Component           | Role                         |
| ------------------- | ---------------------------- |
| Starlink            | Internet connectivity        |
| TP-Link BE3600      | Primary home router          |
| MyCloudPR4100       | NAS / network storage        |
| Dell OptiPlex 7040  | Proxmox virtualization       |
| Dell OptiPlex 9020  | OPNsense lab router/firewall |
| HP ProCurve Switch  | Lab switching                |
| HP ProCurve AP      | Lab wireless                 |
| Proxmox VE          | Application hosting          |
| Technitium DNS      | Internal DNS                 |
| Nginx Proxy Manager | Reverse proxy / HTTPS        |
| Let's Encrypt       | TLS certificates             |
| Cloudflare          | DNS / DNS-01 validation      |
| Tailscale           | Private connectivity         |
| Oracle Cloud        | Public ingress               |
| Jellyfin            | Media streaming              |
| Jellyseerr          | Media requests               |
| ARR Stack           | Media automation             |
| qBittorrent         | Downloads                    |
| SABnzbd             | Downloads                    |

---

## Design Philosophy

### Local-first

The majority of services, compute, and data remain within the home environment.

### Cloud-assisted

Oracle Cloud provides public ingress without requiring services to be directly exposed from the home network.

### Private connectivity

Tailscale provides the private connection between cloud infrastructure and selected home services.

### Separation of concerns

Infrastructure is divided by responsibility:

* **Starlink** → Internet
* **TP-Link** → Home networking
* **Proxmox** → Compute
* **NAS** → Storage
* **OPNsense** → Lab routing and firewalling
* **ProCurve** → Lab switching and wireless
* **Technitium** → DNS
* **Nginx Proxy Manager** → Reverse proxy and HTTPS
* **Cloudflare** → DNS and certificate validation
* **Tailscale** → Private connectivity
* **Oracle Cloud** → Public ingress

### Experimentation

The dedicated Lab Network provides a safe environment for experimenting with networking, routing, VLANs, firewalls, and other infrastructure without making the primary home network the testing environment.

---

## Documentation

Additional documentation can be added under `docs/` as the homelab evolves.

Future documentation:

* `network.md` — Network topology, subnets and VLANs
* `proxmox.md` — VMs, containers and Proxmox configuration
* `services.md` — Application and service inventory
* `dns.md` — Technitium DNS configuration
* `reverse-proxy.md` — Nginx Proxy Manager configuration
* `tailscale.md` — Tailscale configuration
* `cloudflare.md` — DNS and DNS-01 configuration
* `opnsense.md` — Lab firewall/router configuration
* `backups.md` — Backup strategy and recovery procedures
