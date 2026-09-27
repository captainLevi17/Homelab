# Architecture Overview

## Purpose

This document describes the overall architecture of the homelab, including the relationship between the home network, lab network, virtualization infrastructure, storage, cloud infrastructure, and externally accessible services.

The goal of the architecture is to provide a local-first environment for self-hosting services, infrastructure experimentation, networking, and learning while keeping the primary home network isolated from lab experimentation.

---

## High-Level Architecture

The homelab consists of several major components:

* **Home Network** — Primary network for personal devices and local connectivity.
* **Lab Network** — Isolated environment for networking and infrastructure experimentation.
* **Proxmox Infrastructure** — Primary compute and virtualization platform.
* **NAS Storage** — Centralized storage for media, application data, downloads, and backups.
* **Cloud Infrastructure (Oracle NGINX Proxy Manager Instance)** — Provides public ingress for selected services.
* **Private Connectivity (Tailscale)** — Provides secure connectivity between cloud infrastructure and selected services hosted at home.
* **DNS and Reverse Proxying (Technitium & Local NPM instance)** — Provides service discovery, HTTPS, and traffic routing.

At a high level, the architecture can be represented as:

```
                         Internet
                            │
                        DNS / CDN
                            │
                     Cloud Infrastructure
                            │
                       Reverse Proxy
                            │
                     Private Network
                        Connectivity
                            │
                     ┌──────▼──────┐
                     │ Home Network │
                     └──────┬──────┘
                            │
                 ┌──────────┴──────────┐
                 │                     │
             Proxmox                  NAS
                 │
          ┌──────┴──────┐
          │             │
       Services       Testing
```

---

## Network Architecture

### Home Network

The primary home network provides connectivity for normal household devices and the core homelab infrastructure.

**Primary responsibilities:**

* Internet connectivity
* Local device connectivity
* Access to homelab services
* Connectivity to the virtualization and storage infrastructure

### Lab Network

The Lab Network is separated from the primary home network and is used for experimentation.

It provides an environment for:

* Firewall experimentation
* Routing
* VLAN configuration
* Network service testing
* Infrastructure experimentation
* Learning and testing potentially disruptive configurations

The lab environment is managed by OPNsense and dedicated switching and wireless infrastructure.

---

## Compute Architecture

Proxmox VE provides the primary virtualization layer.

The virtualization host runs a combination of virtual machines and containers for:

* Self-hosted applications
* Network services
* Development and testing
* Infrastructure services
* Experimental workloads

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

Public DNS records are managed through the external DNS provider (Cloudflare in my case)  and are used for services that need to be reachable from the Internet.

### Internal DNS

Internal services use a separate namespace that resolves to local infrastructure.

This allows internal services to use consistent hostnames while keeping their traffic within the home network.

---

## Reverse Proxy Architecture

Reverse proxying is divided between internal and public infrastructure.

### Internal Reverse Proxy

The internal reverse proxy handles services that are intended to remain accessible only from the home network.

It provides:

* HTTPS
* Host-based routing
* Internal service access

### Public Reverse Proxy

A reverse proxy hosted in the cloud (Oracle) acts as the public ingress point for selected services.

Requests are forwarded through the private connectivity layer (Tailscale VPN) to services hosted within the home network.

This allows selected services to remain hosted locally without requiring direct inbound access to the home network.

See [`reverse-proxy.md`](../reverse-proxy.md) for more information.

---

## Public Traffic Flow

For services exposed to the Internet, traffic follows this general path:

```
Internet Client
      │
      ▼
Public DNS / Cloudflare
      │
      ▼
Cloud Reverse Proxy
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

The home-hosted service does not need to accept direct inbound connections from the Internet.

---

## Internal Traffic Flow

Internal services follow a separate path:

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

This allows internal services to use HTTPS and consistent hostnames without routing internal traffic through the public infrastructure.

---

## Security Boundaries

The architecture uses several logical boundaries:

### Home Network

The primary network contains normal household devices and core infrastructure.

### Lab Network

The lab network provides separation for experimental infrastructure and networking changes.

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
* Lab routing/firewalling → OPNsense
* DNS -> Technitium
* Reverse proxying → Nginx Proxy Manager
* Public DNS -> Cloudflare
* Private DNS ->
* Private connectivity -> Tailscale
* Public ingress -> Oracle Cloud Always-Free Tier Instance

### Experimentation

The dedicated lab environment provides a place to experiment with networking and infrastructure without making the primary home environment the testing ground.

---

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

### Why separate the lab network?

The separate lab environment allows networking and firewall configurations to be changed and tested without unnecessarily disrupting the primary home network.

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
