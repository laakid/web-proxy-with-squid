# Securing Network Access using Squid Web Proxy & iptables Gateway

![Squid Proxy](https://img.shields.io/badge/Proxy-Squid-blue.svg)
![iptables](https://img.shields.io/badge/Firewall-iptables-red.svg)
![Ubuntu Server](https://img.shields.io/badge/Gateway-Ubuntu_Server-E95420.svg)
![Kali Linux](https://img.shields.io/badge/Client-Kali_Linux-557CDA.svg)
![License](https://img.shields.io/badge/license-MIT-green.svg)

This repository documents the architectural design, security policy implementation, and verification of an edge security gateway built on **Ubuntu Server**. Utilizing **iptables** for Layer 3/4 packet filtering/NAT and **Squid Proxy** for Layer 7 content control, this system regulates network traffic, enforces internet usage policies, and blocks restricted web domains for internal LAN workstations.

--

### IP Addressing Table

| Device / Host | Interface | IP Address | Subnet Mask | Default Gateway | Role / Function |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Ubuntu Gateway** | `ens33` | `192.168.183.140` | `255.255.255.0` | `192.168.183.1` | WAN / Internet Interface |
| **Ubuntu Gateway** | `ens37` | `192.168.10.1` | `255.255.255.0` | N/A | LAN Gateway & Proxy Server |
| **Kali Client** | `eth0` | `192.168.10.10` | `255.255.255.0` | `192.168.10.1` | Internal Workstation |

---

## Security Policies Implemented

### 1. iptables Rules (Network/Transport Layer)
* **NAT / Masquerading**: Enables outbound packet forwarding from private LAN (`192.168.10.0/24`) through `ens33`.
* **Port Ingestion**: Permits internal client traffic to access Squid Proxy on TCP port `3128`.
* **DNS Resolution**: Allows TCP/UDP port `53` traffic required for domain name lookup.
* **Protocol Restricting**: Permits standard web egress (HTTP port `80` and HTTPS port `443`).
* **Explicit IP Block**: Drops outbound communication directed to specific external IPs (e.g., `8.8.4.4`).
* **Default Egress Policy**: Drops all unapproved forward traffic.

### 2. Squid Proxy Rules (Application Layer)
* **Subnet Access**: Authorizes the private LAN segment (`192.168.10.0/24`).
* **Domain Control**: Enforces ACL blocks (`dstdomain`) against restricted social media and streaming domains (`.tiktok.com`, `.netflix.com`).

---

## Verification & Test Results

| Test Target | Destination / Domain | Expected Policy Result | Observed Outcome | Status |
| :--- | :--- | :--- | :--- | :--- |
| **Internal Egress Ping** | `192.168.10.1` | Reply from LAN Gateway | 0% Packet Loss | **PASS** |
| **Allowed Website** | `NeverSSL` / `GMI VLU` | HTTP/HTTPS Access Allowed | Page Loaded Successfully | **PASS** |
| **Restricted Domain** | `netflix.com` | Web Content Blocked | Squid Error: Access Denied | **PASS** |
| **Restricted Domain** | `tiktok.com` | Web Content Blocked | Squid Error: Access Denied | **PASS** |

---

### Prerequisites

* **Hypervisor**: VMware ESXi (or VMware Workstation / VirtualBox) to host isolated virtual switches and VM instances[.: 3].
* **Ubuntu Server VM (Gateway & Security Server)**: 
  * Dual Virtual Network Interfaces: `ens33` (WAN interface connected to external internet/DHCP) and `ens37` (LAN interface configured with static IP `192.168.10.1`).
  * Installed Packages: `iptables`, `squid`.
* **Kali Linux VM (Client Workstation)**: 
  * Single Network Interface: `eth0` configured with static IP `192.168.10.10` pointing to `192.168.10.1` as the default gateway.
  * Configured Browser Proxy: HTTP/HTTPS proxy directed to `192.168.10.1` on port `3128`.

---

## Acknowledgments & Course Details

This project was developed as part of the CBS 2413 Securing Network at the German-Malaysian Institute (GMI).

* **Course**: CBS 2413 Securing Network
* **Institution**: German-Malaysian Institute (GMI).
* **Contributors**:
  - Al Akid Bin Al Addy
  - Muhammad Syazzanie Bin Abdul Rahim
  - Muhammad Ammar Hadi Bin Sazri
