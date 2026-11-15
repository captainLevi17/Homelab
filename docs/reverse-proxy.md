# Reverse Proxy

## Overview

The homelab uses two **Nginx Proxy Manager (NPM)** instances for separate purposes:

| Instance | Location | Role |
| --- | --- | --- |
| Internal NPM | Proxmox host environment | Routes internal hostnames to services on the home network and presents the internal wildcard certificate. |
| Public NPM | Oracle Cloud Always Free instance | Provides public ingress for selected home-hosted services and reaches their backends over Tailscale. |

The public proxy allows selected services to remain hosted at home without requiring direct inbound connections to the home network.

## Internal Requests

Internal DNS resolves service names under `local.mlsg.net` to the local NPM instance. NPM uses the requested hostname to route the request to the corresponding internal service.


Internal client
    -> Internal DNS
    -> Internal NPM
    -> Home service


## Public Requests

Cloudflare manages public DNS for `mlsg.net`. Public service records direct clients to the Oracle Cloud NPM instance. That instance forwards selected requests over Tailscale to the corresponding service at home.


Public client
    -> Oracle Cloud NPM
    -> Tailscale
    -> Selected home service

## TLS Certificates

NPM uses the **Cloudflare DNS-01 challenge** when requesting or renewing the certificate. This proves control of the domain by creating a DNS challenge record; it does not require the internal services to be publicly reachable, and it does not route their normal traffic through Cloudflare.

Public-facing services use certificates configured on the public NPM instance.