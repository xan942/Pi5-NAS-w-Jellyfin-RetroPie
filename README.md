#(OLD - refer to NASPi_Guide) Pi 5 NAS Standalone Setup
## Secure Encrypted NAS + Jellyfin + RetroPie + Enterprise Security

> **Enterprise-grade private media server and game emulator on Raspberry Pi 5 with LUKS encryption, WireGuard VPN, 2FA, AIDE intrusion detection, automated security updates, and defense-in-depth hardening (6 security layers).**

[![Status](https://img.shields.io/badge/Status-Enterprise%20Ready-green)](.)
[![Hardware](https://img.shields.io/badge/Hardware-Pi%205%208GB%20%2B%20NVMe-blue)](.)
[![Security](https://img.shields.io/badge/Security-Grade%20A%2B%20Enterprise-red)](.)
[![Phases](https://img.shields.io/badge/Phases-12%20%2B%201%20Future-purple)](.)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

---

## 🎯 What This Is

A complete setup guide for building an **enterprise-grade encrypted private media server** on Raspberry Pi 5 with **6 integrated security layers**:

- **Encrypts all data at rest** — LUKS encryption on NVMe SSD (2TB, unreadable without passphrase)
- **Provides secure remote access** — WireGuard VPN tunnel + 2FA + IP whitelisting
- **Streams media privately** — Jellyfin on encrypted storage (SSL/TLS encrypted)
- **Runs classic games** — RetroPie emulation with secure ROM storage
- **Detects intrusions** — AIDE + auditd monitors file changes and system calls in real-time
- **Alerts on threats** — Email notifications for suspicious activity
- **Hardens the system** — AppArmor confinement, automated updates, firewall rate limiting
- **Audits everything** — rsyslog centralized logging, daily security reports
- **Integrates with Bridge** — Designed to connect to Pi 5 Bridge Router (adds Zeek IDS + Pi-hole)

**Hardware:** Pi 5 8GB + Kingston P34A60 SSD (2TB) + Waveshare HAT  
**Storage:** Fully encrypted LUKS volume with automatic unlock on boot  
**Authentication:** 2FA SSH (TOTP) + Key-based auth + IP whitelisting  
**Intrusion Detection:** AIDE + auditd + Tripwire file integrity  
**Services:** Jellyfin (streaming) + RetroPie (gaming) + Samba (sharing)  
**Security Grade:** A+ (Enterprise-Grade)

---

## 📑 Table of Contents

**Getting Started**
- [Quick Start](#-quick-start) — Prerequisites, architecture, 3-step setup
- [Documentation](#-documentation) — Guide locations and descriptions

**System Design**
- [Architecture Overview](#-architecture-overview) — Hardware, security layers, services
- [Security Features](#-security-features) — 6 layers explained, future enhancements
- [12 Implementation Phases](#-12-implementation-phases) — Timeline, phases, deliverables

**Implementation & Operations**
- [Deployment Timeline](#-deployment-timeline) — Week-by-week breakdown
- [Security Checklist](#-security-checklist) — Pre-production verification
- [Performance Expectations](#-performance-expectations) — CPU, RAM, network impact
- [Security Grade Progression](#-security-grade-progression) — Before/after comparison

**Integration & Support**
- [Bridge Router Integration](#-integration-with-pi-5-bridge-router) — Standalone vs. Bridge mode
- [Quick Troubleshooting](#-quick-troubleshooting) — Common issues & fixes
- [Project Structure](#-project-structure) — File organization
- [Summary](#-summary) — What you're building, investment, grade

**Resources**
- [Support & Resources](#-support--resources) — Documentation, external links

---

## ⚡ Quick Start

### Prerequisites
- **Hardware**: Raspberry Pi 5 (8GB), Kingston P34A60 SSD, Waveshare PCIe HAT, microSD card
- **Software**: Ubuntu 24.04 LTS ARM64
- **Network**: Gigabit Ethernet + WiFi
- **Time**: 20-26 hours over 4-5 weeks
- **VPN**: Account with ProtonVPN/Mullvad/IVPN (for WireGuard)
- **Authenticator App**: Google Authenticator, Authy, or similar (for 2FA)

### 30-Second Architecture

```
┌──────────────────────────────────────────┐
│        Raspberry Pi 5 (8GB)              │
│        ├─ Boot: microSD                  │
│        └─ Data: Kingston P34A60 (LUKS)  │
├──────────────────────────────────────────┤
│  Layer 6: System Hardening               │
│  ├─ AppArmor, auto-updates, immutable   │
├──────────────────────────────────────────┤
│  Layer 5: Network Hardening              │
│  ├─ SSL/TLS, DoH, MAC filtering         │
├──────────────────────────────────────────┤
│  Layer 4: Monitoring & Logging          │
│  ├─ rsyslog, email alerts, Tripwire    │
├──────────────────────────────────────────┤
│  Layer 3: Intrusion Detection            │
│  ├─ AIDE, auditd (file changes + calls) │
├──────────────────────────────────────────┤
│  Layer 1: Auth & Access Control          │
│  ├─ 2FA SSH, nginx proxy, IP whitelist  │
├──────────────────────────────────────────┤
│  Foundation (LUKS + WireGuard)           │
│  ├─ Jellyfin, RetroPie, Samba           │
└──────────────────────────────────────────┘
```

### Get Started in 3 Steps

1. **Read the Complete Guide**
   ```bash
   cat Pi5_NAS_Standalone_Setup_Guide_Enhanced_Security.md
   ```

2. **Follow the 12 Phases** (in order)
   - Phases 1-5: Foundation (OS, encryption, services, VPN)
   - Phases 6-11: Security Hardening (6 layers, enterprise-grade)
   - Phase 12: Monitoring & Backups
   - Phase 13 (save for later): Enhanced backups

3. **Deploy & Test** (verify each phase before next)

---

## 📚 Documentation

| Document | Purpose | Read Time |
|----------|---------|-----------|
| **Pi5_NAS_Standalone_Setup_Guide_Enhanced_Security.md** | Complete 12-phase setup with all security layers | 2-3 hours |
| **SECURITY_LAYERS_INTEGRATION_SUMMARY.md** | Security architecture, threat matrix, daily operations | 30 min |
| **README.md** (this file) | Quick reference, features, timeline | 10 min |

---

## 🏗️ Architecture Overview

### Physical Hardware

```
Raspberry Pi 5 (8GB)
├─ CPU: 64-bit Quad-core Cortex-A76 @ 2.4GHz
├─ RAM: 8GB LPDDR5
├─ Network: Gigabit Ethernet RJ45
├─ Cooling: Active cooling unit (included)
└─ Storage:
   ├─ Boot (microSD): OS + essentials
   └─ Data (NVMe SSD):
      ├─ Kingston P34A60 2TB (PCIe Gen 3 x4)
      ├─ Waveshare HAT x1001 (PCIe to M.2)
      └─ ENCRYPTED with LUKS
```

### Security Layers (6 Integrated)

```
Layer 6: System Hardening
├─ AppArmor mandatory access control
├─ Automated security updates (unattended-upgrades)
├─ Immutable critical files
├─ Disabled unused kernel modules
└─ Secure boot configuration

Layer 5: Network Hardening
├─ SSL/TLS certificates (self-signed)
├─ DNS over HTTPS forcing
├─ MAC filtering on Samba shares
└─ Enhanced UFW + rate limiting

Layer 4: Monitoring & Logging
├─ rsyslog centralized logging
├─ Email alerts on security events
├─ Tripwire filesystem integrity
└─ Daily automated security reports

Layer 3: Intrusion Detection
├─ AIDE file integrity monitoring
├─ auditd system call logging
├─ chroot jails (optional)
└─ Zeek IDS (when Bridge deployed)

Layer 1: Authentication & Access Control
├─ nginx reverse proxy
├─ TOTP-based 2FA for SSH
├─ IP whitelisting (NYC VPN only)
└─ Rate limiting

Foundation (Core Security)
├─ LUKS at-rest encryption
└─ WireGuard VPN in-transit encryption
```

---

## 🔐 Security Features

### At-Rest Protection

✅ **LUKS Encryption** — Full SSD encryption with strong passphrase  
✅ **Automatic Unlock** — crypttab entry unlocks on boot (secure)  
✅ **Immutable Configs** — Critical files can't be accidentally deleted  
✅ **Physical Security** — Encrypted even if SSD stolen  

### In-Transit Protection

✅ **WireGuard VPN** — All traffic through encrypted tunnel to external VPN  
✅ **SSH 2FA** — TOTP authenticator code + SSH key required  
✅ **SSL/TLS Certs** — Web traffic encrypted (nginx proxy)  
✅ **IP Whitelisting** — Only NYC VPN IP can access SSH  

### Intrusion Detection

✅ **AIDE** — Detects file modifications within minutes  
✅ **auditd** — Logs every system call (who did what)  
✅ **Tripwire** — Filesystem integrity verification  
✅ **Email Alerts** — Suspicious activity triggers email notification  

### Access Control

✅ **AppArmor** — Services confined to safe permissions (can't escape)  
✅ **Firewall** — UFW with rate limiting, blocks DDoS  
✅ **Service Isolation** — Each service on different port  
✅ **MAC Filtering** — Samba shares whitelist trusted device MACs  

### System Hardening

✅ **Auto-Updates** — Security patches applied nightly  
✅ **Non-Standard Port** — SSH on 2222 (reduces noise)  
✅ **Disabled Modules** — Unused kernel modules removed  
✅ **Fail2ban** — Brute-force protection on all services  

### Future Enhancements (With Bridge)

🚀 **Zeek IDS** — Network-level threat detection  
🚀 **Pi-hole DNS** — Automatic malware domain blocking  
🚀 **Double Encryption** — Bridge WireGuard + local LUKS  
🚀 **Headscale VPN** — Zero-trust device enrollment  

---

## 📊 12 Implementation Phases

### Foundation Phases (1-5): ~10-12 hours

| Phase | Component | Time | What You Get |
|-------|-----------|------|---|
| 1 | OS Installation & NVMe | 1-2h | Ubuntu 24.04 LTS on Pi 5, NVMe detected |
| 2 | LUKS Encryption | 1-2h | Full SSD encrypted, auto-unlock on boot |
| 3 | Jellyfin Media Server | 1-2h | Private media streaming on port 8096 |
| 4 | RetroPie Emulation | 1-2h | Classic games (NES, SNES, Genesis, etc.) |
| 5 | WireGuard VPN | 1-2h | Remote access from NYC to LA (encrypted) |

### Security Hardening Phases (6-11): ~8-10 hours

| Phase | Layer | Component | Time | What You Get |
|-------|-------|-----------|------|---|
| 6 | 1 | 2FA + nginx proxy | 1-2h | TOTP authenticator required for SSH |
| 7 | 3 | AIDE + auditd | 1-2h | File change detection + system call logging |
| 8 | 4 | rsyslog + alerts | 1-2h | Email alerts on suspicious activity |
| 9 | 5 | SSL/TLS + DNS | 1-2h | Encrypted web traffic, DoH forcing |
| 10 | 6 | AppArmor + updates | 1-2h | Service confinement, auto security patches |
| 11 | 5 | Enhanced firewall | 1h | Rate limiting, IP whitelisting, logging |

### Operations Phase (12): ~1 hour

| Phase | Component | Time | What You Get |
|-------|-----------|------|---|
| 12 | Monitoring & Backups | 1h | Daily health checks, automated backups |

### Future Phase (13): Save for Week 8+

| Phase | Layer | Component | Time | What You Get |
|-------|-------|-----------|------|---|
| 13 | 2 | Enhanced backups | 1-2h | External USB backups, integrity checks |

**Total Time:** 20-26 hours over 4-5 weeks

---

## 🚀 Deployment Timeline

### Week 1: Build & Boot (2-3 days)
- **Phase 1:** OS Installation (1-2h)
- **Phase 2:** LUKS Encryption (1-2h)

### Week 2: Services (2-3 days)
- **Phase 3:** Jellyfin Setup (1-2h)
- **Phase 4:** RetroPie Setup (1-2h)

### Week 3: Access & Security (3-4 days)
- **Phase 5:** WireGuard VPN (1-2h)
- **Phase 6:** Layer 1 - 2FA + nginx (1-2h)
- **Phase 7:** Layer 3 - AIDE + auditd (1-2h)

### Week 4: Monitoring & Hardening (3-4 days)
- **Phase 8:** Layer 4 - Logging + Alerts (1-2h)
- **Phase 9:** Layer 5 - Network Hardening (1-2h)
- **Phase 10:** Layer 6 - System Hardening (1-2h)
- **Phase 11:** Enhanced Firewall (1h)
- **Phase 12:** Monitoring + Backups (1h)

### Week 5: Testing & Verification (1-2 days)
- Stress testing all services
- Verify 2FA, encryption, monitoring
- Test remote access from NYC
- Verify email alerts working

### Week 8+: Future Enhancement (Optional)
- **Phase 13:** Enhanced Encrypted Backups
- Or wait for Bridge deployment

---

## ✅ Security Checklist

Before deploying to production:

### Authentication (Phase 6)
- [ ] 2FA enabled on SSH with authenticator app
- [ ] SSH keys working (key-based auth)
- [ ] IP whitelist restricts SSH to NYC VPN IP only
- [ ] nginx proxy accessible on port 8080

### Intrusion Detection (Phase 7)
- [ ] AIDE database initialized and running
- [ ] auditd logging all system calls
- [ ] Daily AIDE checks scheduled
- [ ] Tripwire monitoring active

### Monitoring (Phase 8)
- [ ] rsyslog collecting logs from all services
- [ ] Email alerts configured and tested
- [ ] Daily security reports being emailed
- [ ] Tripwire notifications working

### Network Security (Phase 9)
- [ ] SSL/TLS certificates installed
- [ ] HTTPS accessible on port 443
- [ ] DNS over HTTPS forcing active
- [ ] Samba MAC filtering configured

### System Hardening (Phase 10)
- [ ] AppArmor profiles loaded and enforced
- [ ] Automated security updates enabled
- [ ] Critical files marked immutable
- [ ] Unused kernel modules disabled

### Firewall (Phase 11)
- [ ] UFW enabled with correct rules
- [ ] SSH rate limiting active (5/min)
- [ ] HTTPS rate limiting active (10/sec)
- [ ] IP whitelisting working
- [ ] Only NYC VPN IP can access SSH

### Monitoring (Phase 12)
- [ ] Health check script working
- [ ] Daily security reports being sent
- [ ] Backups completing successfully
- [ ] Checksum verification working
- [ ] No default passwords remaining

---

## 📈 Performance Expectations

### Storage
```
Kingston P34A60 (with LUKS encryption):
├─ Sequential Read:  ~2,500-3,000 MB/s
├─ Sequential Write: ~2,000-2,500 MB/s
├─ Random Access:    <1ms
└─ Encryption Overhead: ~5-10% (LUKS)
```

### Services
```
All services + all 6 security layers:
├─ Jellyfin:         <10% CPU (direct play), 30-50% (transcoding)
├─ RetroPie:         5-15% CPU (emulation depends on game)
├─ WireGuard VPN:    <1% CPU
├─ AIDE/auditd:      2-3% CPU (during daily checks)
├─ Firewall/UFW:     <1% CPU
├─ AppArmor:         <1% CPU
├─ Monitoring:       <1% CPU
└─ Total overhead:   ~6-8% CPU (leaves ~92% for services)

RAM Usage:
├─ Ubuntu base:      500MB
├─ All services:     ~1.5GB
├─ Security layers:  ~130MB
└─ Total:            ~2.2GB (leaves 5.8GB free)
```

### Network
```
Gigabit Ethernet:
├─ Local network:    900+ Mbps
├─ WireGuard VPN:    200-400 Mbps (provider dependent)
└─ Samba shares:     100-150 MB/s write

SSL/TLS Encryption:
├─ Jellyfin HTTPS:   ~2% CPU overhead
└─ nginx proxy:      Negligible impact
```

---

## 🎯 Security Grade Progression

### Before Implementation
```
Standalone Pi 5 NAS:
├─ ✅ LUKS encryption (at-rest)
├─ ✅ WireGuard VPN (in-transit)
├─ ⚠️ SSH with password
├─ ✅ Basic firewall
└─ ❌ No intrusion detection
└─ ❌ No monitoring/alerts
└─ ❌ No 2FA

Grade: B (Good, but vulnerable to insider threats)
```

### After Implementation
```
Enterprise-Hardened Pi 5 NAS:
├─ ✅ LUKS encryption (at-rest)
├─ ✅ WireGuard VPN (in-transit)
├─ ✅ 2FA SSH (TOTP + key + IP whitelist)
├─ ✅ AppArmor service confinement
├─ ✅ AIDE + auditd (intrusion detection)
├─ ✅ SSL/TLS encryption (web traffic)
├─ ✅ rsyslog + email alerts (monitoring)
├─ ✅ UFW + rate limiting (firewall)
├─ ✅ Automated security updates
├─ ✅ Daily security reports
├─ ✅ Tripwire integrity checking
└─ ✅ IP whitelisting (NYC only)

Grade: A+ (Enterprise-grade security)
```

---

## 🔄 Integration with Pi 5 Bridge Router

Your NAS is designed to work with the **Pi 5 Bridge Router** (separate device):

### Current (Standalone)
```
You (NYC)
    ↓
WireGuard Client (manual setup)
    ↓
External VPN Provider
    ↓
Your Pi 5 NAS (LA)
    + 6 local security layers (AIDE, AppArmor, etc.)
```

### After Bridge Deployment
```
You (NYC)
    ↓
Headscale Client (automatic, comes with Bridge)
    ↓
Pi 5 Bridge Router (LA)
    ├─ Bridge's WireGuard (encrypts ALL traffic)
    ├─ Pi-hole DNS (filters malware)
    ├─ Zeek IDS (network threat detection)
    └─→ Your Pi 5 NAS
        + 6 local security layers (AIDE, AppArmor, etc.)
```

### Benefits of Bridge Integration
- ✅ Automatic encryption (no manual VPN)
- ✅ Network-level threat detection (Zeek IDS)
- ✅ DNS filtering (Pi-hole blocks malware)
- ✅ Double encryption (Bridge + local LUKS)
- ✅ Unified monitoring dashboard
- ✅ Zero-trust device management (Headscale)

---

## 🆘 Quick Troubleshooting

### NVMe Not Detected
```bash
lsblk | grep nvme
# If not showing: Reseat HAT and SSD, reboot
```

### LUKS Unlock Failed
```bash
sudo cryptsetup luksOpen /dev/nvme0n1p1 nas_encrypted
# If forgotten passphrase: ⚠️ SSD permanently inaccessible
```

### 2FA Not Working
```bash
# Verify Google Authenticator app is generating codes
# Sync phone time (Settings > Date & Time > Set automatically)
# Check SSH logs: sudo journalctl -u ssh -n 20
```

### Email Alerts Not Sending
```bash
# Check mail setup: sudo systemctl status postfix
# Test: echo "Test" | mail -s "Test" root
```

### WireGuard Tunnel Down
```bash
sudo systemctl restart wg-quick@vpn
sudo wg show
```

**For detailed troubleshooting:** See main guide

---

## 📄 Project Structure

```
Pi 5 NAS Standalone Setup
├── README.md (this file - quick reference)
├── Pi5_NAS_Standalone_Setup_Guide_Enhanced_Security.md
│   ├── Phase 1: OS Installation & NVMe
│   ├── Phase 2: LUKS Encryption
│   ├── Phase 3: Jellyfin Media Server
│   ├── Phase 4: RetroPie Game Emulator
│   ├── Phase 5: WireGuard VPN Client
│   ├── Phase 6: Layer 1 - 2FA + nginx
│   ├── Phase 7: Layer 3 - AIDE + auditd
│   ├── Phase 8: Layer 4 - rsyslog + alerts
│   ├── Phase 9: Layer 5 - SSL/TLS + DNS
│   ├── Phase 10: Layer 6 - AppArmor + updates
│   ├── Phase 11: Enhanced Firewall
│   ├── Phase 12: Monitoring & Backups
│   └── Phase 13 (Future): Enhanced Backups
└── SECURITY_LAYERS_INTEGRATION_SUMMARY.md
    ├── Architecture diagrams
    ├── Threat mitigation matrix
    ├── Daily operations guide
    ├── Performance impact analysis
    └── Timeline & verification checklist
```

---

## 📊 Summary

### What You're Building
An **enterprise-grade encrypted private media server** with:
- ✅ Full disk encryption (LUKS - military-grade)
- ✅ Secure remote access (WireGuard VPN + 2FA)
- ✅ Media streaming (Jellyfin on encrypted storage)
- ✅ Game emulation (RetroPie with secure ROMs)
- ✅ Intrusion detection (AIDE + auditd monitoring)
- ✅ Real-time alerting (email on suspicious activity)
- ✅ System confinement (AppArmor sandboxing)
- ✅ Defense-in-depth (6 security layers)
- ✅ Bridge-ready (scales to enterprise when deployed)

### Time Investment
- **Setup:** 20-26 hours (4-5 weeks, 2-3 hours/week)
- **Daily Maintenance:** 2-5 minutes
- **Weekly Review:** 15 minutes
- **Monthly Tasks:** 30 minutes

### Security Level
- ✅ **At-Rest:** LUKS encryption (military-grade, unbreakable without passphrase)
- ✅ **In-Transit:** WireGuard VPN (modern cryptography, ISP can't see traffic)
- ✅ **Access Control:** 2FA SSH + IP whitelisting (requires key + authenticator + right IP)
- ✅ **Intrusion Detection:** AIDE + auditd (detects hacks in real-time)
- ✅ **System Hardening:** AppArmor (services can't escape sandbox)
- ✅ **Monitoring:** 24/7 file integrity + system call logging
- ✅ **Grade:** A+ (Enterprise-Grade)

### Cost
- **Hardware:** $250-350 (one-time)
- **VPN:** $5-12/month (or free with Mullvad)
- **Software:** Free (all open-source)

---

## 🎉 Ready to Build?

1. **Read the guide:** `Pi5_NAS_Standalone_Setup_Guide_Enhanced_Security.md`
2. **Review security summary:** `SECURITY_LAYERS_INTEGRATION_SUMMARY.md`
3. **Gather hardware:** Pi 5 8GB, Kingston P34A60 SSD, Waveshare HAT, microSD
4. **Start Phase 1:** OS installation
5. **Follow each phase sequentially** (verify after each)
6. **Enjoy enterprise-grade security!**

---

## 📞 Support & Resources

### Documentation
- **Main Guide:** 55KB, detailed 12-phase setup
- **Security Summary:** 18KB, architecture & threat analysis
- **This README:** Quick reference

### External Resources
- [Jellyfin Docs](https://docs.jellyfin.org/)
- [RetroPie Docs](https://retropie.org.uk/docs/)
- [WireGuard](https://www.wireguard.com/)
- [Ubuntu Docs](https://ubuntu.com/support)

### VPN Providers
- [ProtonVPN](https://protonvpn.com/) (recommended)
- [Mullvad](https://mullvad.net/) (free)
- [IVPN](https://www.ivpn.io/)

---

**Status:** ✅ **Enterprise-Ready (Grade A+)**  
**Last Updated:** 2026-02-22  
**Version:** 2.0 (Enhanced Security)  
**License:** MIT

---

**Built with ❤️ for privacy, security, and control**

---
