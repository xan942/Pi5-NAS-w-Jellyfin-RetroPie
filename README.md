# NASPi — Raspberry Pi 5 Home NAS

> Self-hosted NAS on Raspberry Pi 5 with Nextcloud file sync, Jellyfin media streaming,
> RetroArch + EmulationStation gaming, WireGuard VPN, and layered security.
> Complete step-by-step guide from bare hardware to production system.

[![Status](https://img.shields.io/badge/Status-Production--Ready-brightgreen)](NASPi_Guide_v1.md)
[![Platform](https://img.shields.io/badge/Platform-Pi%205%20%2B%20Ubuntu%2024.04%20LTS-blue)](NASPi_Guide_v1.md)
[![Phases](https://img.shields.io/badge/Phases-1--9-purple)](NASPi_Guide_v1.md)
[![License](https://img.shields.io/badge/License-MIT-green)]()

---

## What This Is

A complete, opinionated implementation guide for building a self-hosted home NAS on Raspberry Pi 5. Starts from a box of hardware and ends with a running production system.

**What you get:**

- **Nextcloud** — active file sync across all devices (documents, photos), self-hosted and open-source
- **Jellyfin** — private 4K media streaming with no accounts, no paywall, no data sent anywhere
- **RetroArch + EmulationStation** — 50+ console systems, ROMs stored on encrypted NVMe
- **WireGuard VPN** — encrypted remote access, standalone server on the NASPi, future router-ready
- **LUKS2 encryption** — AES-256-XTS on all user data, encrypted at rest on NVMe SSD
- **Layered security** — UFW, SSH hardening, AppArmor, Fail2ban, AIDE, auditd
- **Monitoring** — rsyslog, daily health checks, email alerts, Cockpit, Netdata

All from bare hardware (Pi 5 + Geekworm X1001 + Silicon Power A60 256GB NVMe) in ~20 hours across Phases 1–9.

---

## Hardware

| Component | Details |
|---|---|
| SBC | Raspberry Pi 5 (8GB) |
| Storage interface | Geekworm X1001 PCIe HAT (PCIe Gen 3 x4 — supports M.2 2230/2242/2260/2280) |
| SSD | Silicon Power A60 256GB NVMe M.2 PCIe Gen3x4 2280 (SP256GBP34A60M28) |
| Cooling | Active (heatsink + fan — required under sustained load) |
| Network | Gigabit Ethernet (required for reliable 4K streaming) |
| Power | 27W official Raspberry Pi PSU |
| Boot | microSD — initial setup only; OS migrates to NVMe in Phase 1 |

---

## Storage Layout

```
nvme0n1  (Silicon Power A60 256GB)
├─ p1    512M   FAT32   /boot/firmware   (EFI/boot partition)
├─ p2     50G   ext4    /                (Ubuntu OS root)
└─ p3   ~200G   LUKS2   /mnt/data        (AES-256-XTS encrypted user data)
         └─ /dev/mapper/data → /mnt/data
            ├─ nextcloud/     (Nextcloud data root — documents, photos)
            │  └─ <user>/files/
            │     └─ Media/   (Jellyfin reads here via ACL)
            │        ├─ Movies/
            │        ├─ TV/
            │        └─ Music/
            └─ retroarch/     (ROMs, saves, BIOS, configs)
```

p1 and p2 are created in Phase 1 (OS migration). p3 is added in Phase 2 (LUKS setup).

---

## Software Stack

```
Applications     Nextcloud (snap)  |  Jellyfin  |  RetroArch + EmulationStation
Reverse Proxy    nginx — HTTPS :8920 → Jellyfin :8096 (localhost-only, never exposed)
Remote Access    WireGuard VPN (UDP :51820, standalone server, 100.64.0.1/24)
Firewall         UFW — default deny, LAN (192.168.1.0/24) + VPN (100.64.0.0/24) allowlist
SSH Hardening    Port 2222 | key-only auth | no root login | Fail2ban jail
AppArmor         Enforcing mode (Ubuntu 24.04 default)
IDS              AIDE (file integrity monitoring) + auditd (kernel syscall logging)
Monitoring       rsyslog → /var/log/nas/ | Postfix/Gmail relay | daily health-check.sh
Dashboards       Cockpit (:9090 HTTPS) + Netdata (:19999 HTTP, LAN-only)
OS               Ubuntu 24.04 LTS ARM64
```

---

## Implementation Phases

| Phase | Component | Time | Result |
|-------|-----------|------|--------|
| 1 | Ubuntu Installation & NVMe Migration | 2–3 hrs | Ubuntu 24.04 LTS booting from NVMe |
| 2 | Data Partition & LUKS Encryption | 1–2 hrs | Encrypted data volume (p3) mounted at /mnt/data |
| 3 | Nextcloud + Jellyfin Setup | 3 hrs | File sync + media streaming on encrypted volume |
| 3.5 | Jellyfin Security Hardening | 2–3 hrs | nginx reverse proxy, HTTPS, Fail2ban jail |
| 4 | RetroArch + EmulationStation | 1.5 hrs | 50+ system emulation, ROMs on encrypted NVMe |
| 5 | WireGuard VPN | 1.5 hrs | Standalone VPN server, split-tunnel client config |
| 6 | Firewall & Hardening | 2 hrs | UFW default-deny, SSH hardening, AppArmor verify |
| 7 | Monitoring & Logging | 1.5 hrs | rsyslog, health-check.sh, email alerts via Postfix |
| 8 | Intrusion Detection | 1.5 hrs | AIDE file integrity + auditd syscall audit logging |
| 9 | Management Dashboard | 30 min | Cockpit admin UI + Netdata real-time metrics |

**Total: ~20 hours**

Each phase has a **Verify** section to confirm the setup before moving on, and a **Future Router Integration Note** where relevant.

---

## The Guide

Everything is in [`NASPi_Guide_v1.md`](NASPi_Guide_v1.md) — one document, start to finish.

```
NASPi_Guide_v1.md
├── Project Summary
├── Architecture & Design     — hardware diagram, software layers, security model
├── Technology Reasoning      — why every tool was chosen over the alternatives
├── Phases 1–9 Configuration  — step-by-step with copy-paste commands
├── Troubleshooting           — common issues with accurate debug steps
├── References                — command cheatsheet, firewall rules, external docs
└── Summary                   — what you've built, community links
```

---

## Future: Privacy Router Integration

The NASPi is designed to eventually sit behind a dedicated **Pi 5 privacy router**. Each relevant phase includes a "Future Router Integration Note" documenting exactly what changes when the router is deployed — nothing needs to be torn down or rebuilt.

| Phase | Router adds | NASPi change required |
|---|---|---|
| 5 — WireGuard | Router becomes VPN server | Remove NAT from wg0.conf, reconfigure NASPi as VPN peer |
| 7 — Monitoring | Centralised syslog server + Grafana | Enable rsyslog forwarding, install node_exporter |
| 8 — IDS | Snort3 or Suricata NIDS at network boundary | None — AIDE/auditd remain host-level layer |
| 9 — Dashboard | Grafana pulls metrics from all devices | Add node_exporter, add Prometheus scrape target |

```
Internet
    │
[ Pi 5 Privacy Router ]  ← WireGuard server | Snort3/Suricata NIDS | Grafana
    │
[ NASPi ]                ← Nextcloud | Jellyfin | RetroArch | AIDE | auditd
```

---

## Project Structure

```
Pi5-NAS-w-Jellyfin-RetroPie/
├── README.md           ← This file
├── NASPi_Guide_v1.md  ← Complete implementation guide (Phases 1–9)
└── Docs/              ← Legacy reference documents (pre-rewrite)
```

---

## Resources

| Project | Documentation |
|---|---|
| Nextcloud | https://docs.nextcloud.com/ |
| Jellyfin | https://jellyfin.org/docs/ |
| RetroArch / Libretro | https://docs.libretro.com/ |
| WireGuard | https://www.wireguard.com/ |
| Cockpit | https://cockpit-project.org/ |
| Netdata | https://learn.netdata.cloud/ |
| Ubuntu on Pi | https://ubuntu.com/download/raspberry-pi |
| LUKS / cryptsetup | https://gitlab.com/cryptsetup/cryptsetup |
| AIDE | https://aide.github.io/ |
| Fail2ban | https://www.fail2ban.org/ |

**Community:**

| | |
|---|---|
| Jellyfin | https://forum.jellyfin.org/ |
| Nextcloud | https://help.nextcloud.com/ |
| Raspberry Pi | https://forums.raspberrypi.com/ |
| WireGuard | https://www.wireguard.com/#community |
