# Homelab

A homelab built around **Proxmox, OPNsense, local NAS storage, Tailscale, Cloudflare, and Oracle Cloud**.

The environment provides media services, self-hosted applications, DNS, reverse proxying, network experimentation, and selected public-facing services while keeping the majority of workloads and data on-premises.

---

## Architecture

The physical network uses **Starlink** for internet connectivity. The Starlink router is in passthrough mode and connects to OPNsense on the **Dell OptiPlex 9020**, which provides routing and firewalling for the home network.

OPNsense connects to an **HP ProCurve 2910al** switch. The switch connects the workstation, **MyCloudPR4100 NAS**, and **Dell OptiPlex 7040** running Proxmox. A **TP-Link BE3600** operates in access point mode and provides Wi-Fi; it is not the network's router.

The physical topology is documented in [the draw.io diagram](diagrams/MLSG_home_network.drawio). The diagram shows one switched home network.

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
| Zabbix                 | Network Monitoring          |
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

## Network and Experimentation

OPNsense on the Dell OptiPlex 9020 is the network gateway and firewall. The HP ProCurve 2910al provides wired switching, and the TP-Link BE3600 provides wireless access in AP mode.

---

## DNS and Domain


**Cloudflare** manages the domain's DNS.

Public-facing services use subdomains of `mlsg.net`, while internal services use the `local.mlsg.net` namespace.

For example:

```
Public:
jellyfin.mlsg.net

Internal:
jellyfin.local.mlsg.net
sab.local.mlsg.net
```

This provides a consistent naming scheme while distinguishing internal services from public-facing services.

---

## HTTPS / TLS

Internal services use a **Let's Encrypt wildcard certificate** for `*.local.mlsg.net`, managed by the internal Nginx Proxy Manager instance.

Nginx Proxy Manager uses the **Cloudflare DNS-01 challenge** to prove control of the domain when requesting or renewing the certificate. Cloudflare handles DNS validation; the certificate is issued by Let's Encrypt and presented by the internal reverse proxy. This does not require internal services to be publicly accessible or their traffic to pass through Cloudflare.

Public-facing services use certificates configured on the public Nginx Proxy Manager instance.

---

## Reverse Proxy

There are two **Nginx Proxy Manager** instances.

### Internal NPM

The local NPM instance runs within Proxmox and handles reverse proxying and HTTPS for internal services.

Internal DNS resolves `*.local.mlsg.net` service hostnames to the local infrastructure. The internal NPM presents the wildcard certificate so clients can access those services over HTTPS.

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
* DNS-01 validation for the Let's Encrypt wildcard certificate used by internal services

Cloudflare DNS directs public service hostnames to Oracle Cloud NPM. Requests then follow this general path:

**Client → Oracle Cloud NPM → Tailscale → Home Service**

Internal services use the local `.local.mlsg.net` namespace and are handled by the internal Nginx Proxy Manager instance.

---

## Infrastructure Summary

| Component           | Role                         |
| ------------------- | ---------------------------- |
| Starlink            | Internet connectivity        |
| TP-Link BE3600      | Wi-Fi access point            |
| MyCloudPR4100       | NAS / network storage        |
| Dell OptiPlex 7040  | Proxmox virtualization       |
| Dell OptiPlex 9020  | OPNsense router/firewall      |
| HP ProCurve 2910al  | Network switching             |
| Proxmox VE          | Application hosting          |
| Technitium DNS      | Internal DNS                 |
| Zabbix              | Network and endpoint monitoring |
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
* **OPNsense** → Routing and firewalling
* **ProCurve** → Wired switching
* **TP-Link** → Wi-Fi access point
* **Proxmox** → Compute
* **NAS** → Storage
* **Technitium** → DNS
* **Nginx Proxy Manager** → Reverse proxy and HTTPS
* **Cloudflare** → DNS and certificate validation
* **Tailscale** → Private connectivity
* **Oracle Cloud** → Public ingress


---

## Documentation

Additional documentation can be added under `docs/` as the homelab evolves.

See [Reverse Proxy](docs/reverse-proxy.md) for the current internal and public NPM request flows and certificate handling.

Future documentation:

* `infrastructure/network.md` - Network topology, subnets and VLANs
* `infrastructure/proxmox.md` - VMs, containers and Proxmox configuration
* `infrastructure/storage.md` - Virtualization 
* `services.md` - Application and service inventory
* `networking/dns.md` - Technitium DNS configuration
* `networking/tailscale.md` - Tailscale configuration
* `networking/opnsense.md` - Lab firewall/router configuration
* `cloudflare.md` - DNS and DNS-01 configuration
* `backups.md` - Backup strategy and recovery procedures
