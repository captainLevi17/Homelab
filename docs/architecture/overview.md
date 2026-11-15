# Architecture Overview

## Purpose

This document describes the current home network, virtualization and storage infrastructure, and cloud ingress for externally accessible services. 

---

## High-Level Architecture

The homelab consists of several major components:

* **Home Network** — Current switched network for personal devices and local connectivity.
* **Network Edge** — Starlink internet connection, OPNsense routing/firewalling, wired switching, and Wi-Fi access.
* **Proxmox Infrastructure** — Primary compute and virtualization platform.
* **NAS Storage** — Centralized storage for media, application data, downloads, and backups.
* **Cloud Infrastructure (Oracle Cloud NPM)** — Provides public ingress for selected services.
* **Private Connectivity (Tailscale)** — Connects the cloud proxy to selected services hosted at home.
* **DNS and Reverse Proxying (Technitium and two NPM instances)** — Provides internal name resolution, TLS, and host-based routing.
* **Monitoring (Zabbix)** — Monitors network infrastructure and endpoints.

Internal request: client -> internal DNS -> internal NPM -> home service
Public DNS:       Cloudflare DNS resolves hostname to Oracle Cloud NPM
Public request:   client -> Oracle Cloud NPM -> Tailscale -> selected home service

Cloudflare provides public DNS and DNS-01 certificate validation.

---

## Network Architecture

### Home Network

The primary home network provides connectivity for normal household devices and the core homelab infrastructure.

**Primary responsibilities:**

* Internet connectivity through Starlink and OPNsense
* Local device connectivity
* Access to homelab services
* Connectivity to the virtualization and storage infrastructure through the HP ProCurve switch


The HP ProCurve 2910al provides wired switching, and the TP-Link BE3600 provides Wi-Fi in AP mode. 

---

## Compute Architecture

Proxmox VE provides the primary virtualization layer.

The virtualization host runs a combination of virtual machines and containers for:

* Self-hosted applications
* Network services
* Development and testing
* Infrastructure services
* Experimental workloads

Zabbix runs as a VM on Proxmox and monitors network infrastructure and endpoints.

The virtualization layer allows individual workloads to be isolated while sharing the available physical hardware.

---

## Storage Architecture

A dedicated NAS provides centralized storage for the homelab.

Primary storage workloads include:

* Media
* Application data
* Downloads
* Backups
* General file storage

The NAS is consumed by selected services hosted within the virtualization environment.

---

## Service Architecture

Services are hosted primarily within the local infrastructure.

Examples include:

* Media services
* DNS
* Reverse proxying
* Download services
* Self-hosted applications
* Testing and development environments

See [`services.md`](../services.md) for the complete service inventory.

---

## DNS Architecture

DNS is separated into public and internal responsibilities.

### Public DNS

Cloudflare manages public DNS records for `mlsg.net`. Public service hostnames resolve to the Oracle Cloud NPM instance.

### Internal DNS

Internal services use the `local.mlsg.net` namespace. Internal DNS resolves these names to the internal NPM instance on the home network.

This allows internal services to use consistent hostnames while keeping their requests on the home network.

---

## Reverse Proxy Architecture

Reverse proxying is divided between internal and public infrastructure.

### Internal Reverse Proxy

The internal NPM instance runs within Proxmox and handles services intended for internal access. Internal DNS resolves service hostnames under `local.mlsg.net` to this proxy, which routes requests to the matching home service.

It presents a Let's Encrypt wildcard certificate for `*.local.mlsg.net`. NPM uses the Cloudflare DNS-01 challenge to request or renew the certificate. Cloudflare validates domain control but does not carry the normal internal service traffic.

### Public Reverse Proxy

The public NPM instance runs on Oracle Cloud and acts as the public ingress point for selected home-hosted services. Public DNS is managed through Cloudflare.

Requests are forwarded through Tailscale to services hosted within the home network.

This allows selected services to remain hosted locally without requiring direct inbound access to the home network.

See [`reverse-proxy.md`](../reverse-proxy.md) for more information.

---

## Public Traffic Flow

For services exposed to the internet, Cloudflare DNS resolves the public hostname to the Oracle Cloud NPM endpoint. The request then follows this path:

```
Internet Client
      │
      ▼
Oracle Cloud NPM
      │
      ▼
Tailscale VPN
      │
      ▼
Home Network
      │
      ▼
Internal Service
```

DNS resolution points the hostname to Oracle Cloud NPM. The home-hosted service does not need to accept direct inbound connections from the internet.

---

## Internal Traffic Flow

Internal service requests follow a separate path:

```
Internal Client
      │
      ▼
Internal DNS
      │
      ▼
Internal Reverse Proxy
      │
      ▼
Homelab Service
```

The internal NPM presents the wildcard certificate for `*.local.mlsg.net`, allowing clients to use HTTPS without routing internal requests through the public infrastructure.

---

## Security Boundaries

The architecture uses several logical boundaries:

### Home Network

The primary network contains normal household devices and core infrastructure.

### Network Segmentation

VLANs and OPNsense firewall rules are planned.

### Cloud Infrastructure

The cloud environment acts as the public-facing boundary for selected services.

### Private Connectivity

The private connectivity layer provides communication between selected cloud systems and home infrastructure without exposing the home services directly.

---

## Design Principles

### Local-first

The majority of services, compute, and data remain hosted within the home environment.

### Cloud-assisted

Cloud infrastructure is used where it provides a useful capability that would otherwise require exposing the home network.

### Private connectivity (Tailscale)

Communication between cloud and home infrastructure uses a private connectivity layer rather than direct inbound access.

### Separation of concerns

Different components are responsible for different parts of the infrastructure:

* Compute -> Proxmox
* Storage -> NAS
* Routing and firewalling → OPNsense
* DNS -> Technitium
* Reverse proxying → Nginx Proxy Manager
* Public DNS -> Cloudflare
* Internal DNS -> Technitium
* Private connectivity -> Tailscale
* Public ingress -> Oracle Cloud Always-Free Tier Instance


## Architectural Considerations

### Why keep services local?

Keeping services local provides control over:

* Data
* Hardware
* Storage
* Network configuration
* Service deployment

It also provides a practical environment for learning systems administration and infrastructure.

### Why use cloud infrastructure?

The cloud environment provides a public-facing ingress layer while allowing the actual applications and data to remain on-premises. Overall, the main purpose of the cloud instance is for me to bypass my lack of static public IPV4 problem with Starlink.


---

## Related Documentation

* [`Network`](../infrastructure/network.md)
* [`Proxmox`](../infrastructure/proxmox.md)
* [`Storage`](../infrastructure/storage.md)
* [`Services`](../services.md)
* [`DNS`](../networking/dns.md)
* [`Tailscale`](../networking/tailscale.md)
* [`OPNsense`](../networking/opnsense.md)
* [`Reverse Proxy`](../reverse-proxy.md)
* [`Cloudflare`](../cloudflare.md)
* [`Backups`](../backups.md)
