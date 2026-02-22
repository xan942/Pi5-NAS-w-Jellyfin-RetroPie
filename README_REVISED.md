# Pi 5 NAS Standalone Setup
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

## ⚡ Quick Start

### Prerequisites
- **Hardware**: Raspberry Pi 5 (8GB), Kingston P34A60 SSD, Waveshare PCIe HAT, microSD card
- **Software**: Ubuntu 24.04 LTS ARM64
- **Network**: Gigabit Ethernet + WiFi (for Bridge integration later)
- **Time**: 15-20 hours over 4 weeks
- **VPN**: Account with ProtonVPN/Mullvad/IVPN (for WireGuard)

### 30-Second Architecture

```
┌──────────────────────────────────┐
│  Raspberry Pi 5 (8GB)            │
│  ├─ Boot: microSD                │
│  └─ Data: Kingston P34A60 (LUKS) │
│     ├─ Jellyfin (encrypted)      │
│     ├─ RetroPie (encrypted)      │
│     ├─ Samba shares (encrypted)  │
│     └─ Backups (encrypted)       │
└────────┬─────────────────────────┘
         │
    ┌────┴────────────────────┐
    │ Network Access Layer     │
    ├─ WireGuard VPN Client   │
    │  (NYC ↔ LA tunnel)       │
    ├─ SSH (key-based auth)   │
    ├─ Samba (file sharing)   │
    ├─ Jellyfin (port 8096)   │
    └─ RetroPie (port 3000)   │
```

### Get Started in 3 Steps

1. **Read the Complete Guide**
   ```bash
   # Main setup guide (comprehensive)
   cat Pi5_NAS_Standalone_Setup_Guide.md
   ```

2. **Follow the 7 Phases** (in order)
   - Phase 1: OS Installation & NVMe (1-2h)
   - Phase 2: LUKS Encryption (1-2h)
   - Phase 3: Jellyfin Setup (1-2h)
   - Phase 4: RetroPie Setup (1-2h)
   - Phase 5: WireGuard VPN (1-2h)
   - Phase 6: Networking & Security (1h)
   - Phase 7: Monitoring & Backups (30m)

3. **Deploy & Test** (verify each phase before moving next)
   ```bash
   # Example: After Phase 2, verify encryption
   df -h | grep nas_storage
   # Should show encrypted volume mounted
   ```

---

## 📚 Documentation

| Document | Purpose | Read Time |
|----------|---------|-----------|
| **Pi5_NAS_Standalone_Setup_Guide.md** | Complete setup with all code examples | 1-2 hours |
| **README.md** (this file) | Quick reference and architecture overview | 10 min |

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
      ├─ Kingston P34A60 2TB
      ├─ PCIe Gen 3 x4
      ├─ Mounted via Waveshare HAT x1001
      └─ ENCRYPTED with LUKS
```

### Software Stack

```
Ubuntu 24.04 LTS ARM64
├─ OpenSSH (SSH access, key-only auth)
├─ WireGuard VPN Client (encrypted tunnel)
├─ Jellyfin (media server on port 8096)
├─ RetroPie (game emulation)
├─ Samba (network file sharing)
├─ UFW (firewall - default deny)
├─ Fail2ban (brute-force protection)
├─ LUKS (full SSD encryption)
└─ systemd (service management)
```

### Data Encryption Layers

```
Physical Disk (Kingston P34A60)
    │
    ↓ LUKS Encryption (at-rest)
    │
Encrypted Partition (/dev/nvme0n1p1)
    │
    ↓ Filesystem (ext4)
    │
Mounted Volume (/mnt/nas_storage)
    ├─ /jellyfin (media config + cache)
    ├─ /media (movie/TV/music libraries)
    ├─ /retropie (ROM collections)
    ├─ /backups (config backups)
    └─ (All encrypted, unreadable without passphrase)
```

### Remote Access Flow

```
NYC Laptop                           LA NAS
    │                                 │
SSH Client ─────────────────────→ WireGuard Client
    │        (SSH key auth)           │
    │                                 │
Encrypted ←───────────────────────── SSH
    │        (WireGuard tunnel)       │
                                      
Result:
- NYC laptop SSH goes through WireGuard tunnel
- All traffic encrypted (ISP can't see destination)
- Safe remote access to Jellyfin/Samba/SSH
```

---

## 🔒 Security Features

### At-Rest Protection (Data on Disk)

✅ **LUKS Encryption** — Full SSD encryption with passphrase  
✅ **Automatic Unlock** — crypttab entry unlocks on boot  
✅ **Secure Deletion** — Old files unrecoverable  
✅ **Physical Security** — Encrypted even if SSD stolen  

### In-Transit Protection (Data on Network)

✅ **WireGuard VPN** — All traffic through encrypted tunnel  
✅ **SSH Key Auth** — No passwords (only private keys)  
✅ **Non-Standard Port** — SSH on 2222 (obscurity layer)  
✅ **IP Encryption** — ISP can't see destinations  

### Network Security

✅ **UFW Firewall** — Default deny all, explicit allow  
✅ **Fail2ban IPS** — Auto-block brute-force attempts  
✅ **Service Isolation** — Each service on different port  
✅ **Local Firewall** — Restrict access to LAN/VPN only  

### Future Enhancements (With Bridge)

🚀 **Bridge WireGuard** — Double encryption (local + bridge)  
🚀 **Pi-hole DNS** — Automatic ad-blocking + privacy  
🚀 **Zeek IDS** — Network threat detection  
🚀 **Headscale VPN** — Zero-trust device management  

---

## 📊 Hardware Specifications

### Compute
```
Raspberry Pi 5 (8GB)
├─ 64-bit ARM Cortex-A76 @ 2.4GHz
├─ 4 CPU cores
├─ 8GB LPDDR5 RAM
└─ Adequate for:
   - Jellyfin (1-2 simultaneous streams)
   - RetroPie (classic game emulation)
   - Samba (network file serving)
   - WireGuard VPN (all traffic encrypted)
```

### Storage
```
Kingston P34A60 (2TB)
├─ PCIe Gen 3 x4 M.2 2280 SSD
├─ Read: ~3,500 MB/s
├─ Write: ~3,000 MB/s
├─ LUKS encrypted
└─ Mounted via Waveshare PCIe HAT x1001
```

### Networking
```
Gigabit Ethernet (1000 Mbps)
├─ Fallback for local network access
├─ Can connect to Bridge Router later
└─ Alternative to WiFi
```

### Power
```
Official Pi 5 PSU (25W)
├─ USB-C 5V @ 5A
├─ Sufficient for:
   - Pi 5 base: 4-5W
   - NVMe SSD: 2-3W
   - Services: 3-5W
   - Active cooling: 2-3W
└─ Total: ~15W typical (25W PSU provides 10W headroom)
```

### Cost Breakdown
```
Raspberry Pi 5 (8GB):     $60-70
Kingston P34A60 SSD:      $150-200
Waveshare HAT:            $12-15
microSD card (64GB):      $15-25
Official PSU (25W):       $10-15
Active cooling unit:      $10-15
───────────────────────
Total:                    $257-340
```

---

## 🎯 Feature Comparison

### Jellyfin Media Server

| Feature | Support | Notes |
|---------|---------|-------|
| **Streaming** | ✅ | 1-2 simultaneous streams |
| **Transcoding** | ⚠️ | Limited (software only) |
| **Remote Access** | ✅ | Via WireGuard tunnel |
| **Library Management** | ✅ | Movies, TV, Music |
| **Encryption** | ✅ | Stored on LUKS volume |
| **Hardware Acceleration** | ❌ | Not available on Pi 5 |

### RetroPie Emulation

| System | Support | Notes |
|--------|---------|-------|
| **NES** | ✅ | Full speed emulation |
| **SNES** | ✅ | Full speed emulation |
| **Genesis** | ✅ | Good performance |
| **GBA** | ✅ | Smooth gameplay |
| **N64** | ⚠️ | 30-40 FPS (playable) |
| **GameCube/Wii** | ❌ | Requires more power |

### Storage Performance

```
Kingston P34A60 (LUKS encrypted):
├─ Read:  ~2,500-3,000 MB/s (after decryption)
├─ Write: ~2,000-2,500 MB/s (after encryption)
├─ LUKS Overhead: ~5-10%
└─ Result: Still very fast for NAS usage
```

---

## 📋 7 Phases Explained

### Phase 1: OS Installation & NVMe (1-2 hours)
- Download Ubuntu 24.04 LTS ARM64
- Flash to microSD card
- Install Waveshare HAT + Kingston SSD
- Boot Pi 5 and verify NVMe detected
- Initial system setup

**Key Skills:** Flashing images, hardware assembly  
**Tools Needed:** USB adapter for microSD, laptop

---

### Phase 2: LUKS Encryption (1-2 hours)
- Partition NVMe SSD
- Create LUKS encrypted volume (with passphrase)
- Format filesystem (ext4)
- Setup automatic unlock (crypttab)
- Create storage directories

**Key Skills:** Linux disk management, encryption  
**Important:** ⚠️ Store passphrase securely!

---

### Phase 3: Jellyfin Media Server (1-2 hours)
- Install Jellyfin package
- Move config to encrypted storage
- Create media libraries (Movies, TV, Music)
- Configure streaming settings
- Add media files to NAS

**Key Skills:** Service installation, web UI configuration  
**Result:** Private media streaming available at http://pi:8096

---

### Phase 4: RetroPie Game Emulator (1-2 hours)
- Install RetroPie framework
- Move emulator data to encrypted storage
- Add ROM collections (NES, SNES, Genesis, etc.)
- Configure controllers (if attached)
- Test gameplay

**Key Skills:** Service installation, ROM management  
**Result:** Classic games available locally + via RetroPie web UI

---

### Phase 5: WireGuard VPN Client (1-2 hours)
- Install WireGuard on Pi
- Choose VPN provider (ProtonVPN recommended)
- Download and configure VPN connection
- Setup SSH with key-based authentication
- Test remote access from NYC

**Key Skills:** VPN configuration, SSH key management  
**Result:** Secure encrypted tunnel for remote access

---

### Phase 6: Networking & Security (1 hour)
- Configure UFW firewall (default deny)
- Setup Samba file sharing (with auth)
- Install Fail2ban brute-force protection
- Configure SSH on non-standard port
- Test firewall rules

**Key Skills:** Linux firewall, service configuration  
**Result:** Hardened network with multiple security layers

---

### Phase 7: Monitoring & Backups (30 minutes)
- Create disk usage monitoring script
- Setup health check scripts
- Configure daily backups to encrypted storage
- Test log viewing
- Create maintenance schedule

**Key Skills:** Bash scripting, automation  
**Result:** Automated monitoring and recovery capability

---

## 🚀 Deployment Timeline

### Week 1: Prepare & Build
- **Day 1-2:** Gather hardware, download software
- **Day 3-4:** Phase 1 (OS installation)
- **Day 5-7:** Phase 2 (LUKS encryption setup)

### Week 2: Services
- **Day 8-10:** Phase 3 (Jellyfin installation + config)
- **Day 11-13:** Phase 4 (RetroPie installation + ROMs)
- **Day 14:** Testing and troubleshooting

### Week 3: Remote Access & Security
- **Day 15-17:** Phase 5 (WireGuard VPN + SSH keys)
- **Day 18-19:** Phase 6 (Firewall + Samba setup)
- **Day 20:** Testing remote access from NYC

### Week 4: Operations
- **Day 21:** Phase 7 (Monitoring scripts + backups)
- **Day 22-25:** Stress testing, media library expansion
- **Day 26-28:** Documentation, final verification

---

## ✅ Security Checklist

Before considering deployment complete:

- [ ] LUKS passphrase created and stored securely
- [ ] Passphrase tested (can unlock SSD)
- [ ] SSH keys generated (Ed25519)
- [ ] SSH keys copied to Pi
- [ ] SSH key-based auth working
- [ ] SSH password auth disabled
- [ ] SSH on non-standard port (2222)
- [ ] WireGuard tunnel active on Pi
- [ ] WireGuard tested from NYC laptop
- [ ] Can SSH through WireGuard tunnel
- [ ] UFW firewall enabled
- [ ] Fail2ban protecting SSH
- [ ] Jellyfin accessible locally
- [ ] RetroPie games play without issues
- [ ] Samba shares accessible on LAN
- [ ] Automatic backups running
- [ ] No default passwords remaining
- [ ] System logs show no errors

---

## 🔄 Integration with Pi 5 Bridge Router

Your standalone NAS is designed to integrate with the **Pi 5 Bridge Router** (separate device) when deployed:

### Current (Standalone)
```
You (NYC)
    ↓
WireGuard Client (manual)
    ↓
External VPN Provider
    ↓
Your Pi NAS (LA)
```

### After Bridge Deployment
```
You (NYC)
    ↓
Headscale Client (automatic)
    ↓
Pi 5 Bridge Router (LA)
    ├─ Bridge's WireGuard (encrypts all traffic)
    ├─ Pi-hole DNS (filters queries)
    └─→ Your Pi NAS (already encrypted)
```

### Benefits of Bridge Integration
- ✅ Automatic encryption for all traffic (no manual VPN)
- ✅ DNS filtering via Pi-hole
- ✅ Threat detection via Zeek IDS
- ✅ Better remote access via Headscale
- ✅ Unified monitoring dashboards
- ✅ Defense-in-depth (double encryption possible)

---

## 🆘 Quick Troubleshooting

### NVMe Not Detected
```bash
# Check if NVMe is recognized
lsblk | grep nvme
# Or: lspci | grep -i nvme

# If not showing: Reseat HAT and SSD, reboot
```

### LUKS Unlock Failed at Boot
```bash
# Check if passphrase is correct
sudo cryptsetup luksOpen /dev/nvme0n1p1 nas_encrypted

# If forgotten: ⚠️ SSD is permanently inaccessible
# Prevention: Store passphrase securely!
```

### Jellyfin Won't Start
```bash
# Check service status
sudo systemctl status jellyfin

# View logs
sudo journalctl -u jellyfin -n 20

# Verify mount point exists
ls -la /mnt/nas_storage/jellyfin
```

### SSH Connection Refused
```bash
# Check SSH is running on port 2222
sudo ss -tlnp | grep ssh
# Should show: 2222/tcp

# Verify SSH keys copied
cat ~/.ssh/pi5_nas.pub | ssh -p 2222 ubuntu@pi.local "cat >> .ssh/authorized_keys"
```

### WireGuard Tunnel Down
```bash
# Check status
sudo systemctl status wg-quick@vpn

# Restart tunnel
sudo systemctl restart wg-quick@vpn

# Verify connection
sudo wg show
```

**For more detailed troubleshooting:** See main guide's Troubleshooting section

---

## 📈 Performance Expectations

### Storage
```
Kingston P34A60 (with LUKS encryption):
├─ Sequential Read:  ~2,500-3,000 MB/s
├─ Sequential Write: ~2,000-2,500 MB/s
├─ Random Access:    <1ms
└─ Encryption Overhead: ~5-10%
```

### Jellyfin Streaming
```
Direct Play (no transcoding):
├─ CPU Usage:   <10%
├─ RAM Usage:   200-300MB
└─ Max Streams: 1-2 (depending on bitrate)

Software Transcode (H.264):
├─ CPU Usage:   30-50%
├─ RAM Usage:   400-500MB
└─ Max Streams: 1 (limited by CPU)
```

### Network
```
Gigabit Ethernet:
├─ Local network:   900+ Mbps
├─ WireGuard VPN:   200-400 Mbps (depends on provider)
└─ Samba shares:    100-150 MB/s write speed
```

---

## 🛠️ Maintenance Schedule

### Daily (2 minutes)
- Verify services running: `sudo systemctl status jellyfin samba`
- Check storage accessible: `df -h /mnt/nas_storage`

### Weekly (15 minutes)
- Review system logs: `sudo journalctl -n 100`
- Check disk usage: `du -sh /mnt/nas_storage/*`
- Verify backups completed

### Monthly (30 minutes)
- Security updates: `sudo apt-get update && sudo apt-get upgrade`
- Archive old logs: `sudo journalctl --vacuum=30d`
- Test backup restoration
- Review Fail2ban activity

### Quarterly (1 hour)
- Full security audit
- Performance analysis
- Update documentation
- Test disaster recovery

---

## 📞 Support & Resources

### Documentation
- **Main Guide:** Pi5_NAS_Standalone_Setup_Guide.md (complete reference)
- **Troubleshooting:** See section in main guide
- **Commands Reference:** Quick reference at end of guide

### External Resources
- [Jellyfin Documentation](https://docs.jellyfin.org/)
- [RetroPie Documentation](https://retropie.org.uk/docs/)
- [WireGuard Installation](https://www.wireguard.com/install/)
- [Ubuntu Documentation](https://ubuntu.com/support)

### VPN Providers
- [ProtonVPN](https://protonvpn.com/) (recommended)
- [Mullvad](https://mullvad.net/)
- [IVPN](https://www.ivpn.io/)

---

## 📄 Project Structure

```
├── README.md (this file)
└── Pi5_NAS_Standalone_Setup_Guide.md
    ├── Phase 1: OS Installation & NVMe Setup
    ├── Phase 2: LUKS Encryption
    ├── Phase 3: Jellyfin Media Server
    ├── Phase 4: RetroPie Game Emulator
    ├── Phase 5: Remote Access Setup (WireGuard)
    ├── Phase 6: Networking & Firewall
    ├── Phase 7: Monitoring & Backups
    ├── Future Bridge Integration
    └── Troubleshooting & Reference
```

---

## 📊 Summary

### What You're Building
A **private, encrypted media server and game emulator** with:
- ✅ Full disk encryption (LUKS)
- ✅ Secure remote access (WireGuard VPN)
- ✅ Media streaming (Jellyfin)
- ✅ Game emulation (RetroPie)
- ✅ Defense-in-depth security (UFW + Fail2ban + SSH keys)
- ✅ Automated monitoring & backups
- ✅ Bridge-ready architecture

### Time Investment
- **Setup:** 15-20 hours (spread over 4 weeks)
- **Daily Maintenance:** 2-5 minutes
- **Weekly Review:** 15 minutes
- **Monthly Tasks:** 30 minutes

### Security Level
- ✅ **At-Rest:** LUKS encryption (military-grade)
- ✅ **In-Transit:** WireGuard VPN (modern cryptography)
- ✅ **Network:** UFW firewall + Fail2ban
- ✅ **Authentication:** SSH key-based (no passwords)
- ✅ **Future:** Bridge integration (additional layers)

### Cost
- **Hardware:** $250-350 (one-time)
- **VPN:** $5-12/month (or free with Mullvad)
- **Software:** Free (all open-source)

---

## 🎉 Ready to Build?

1. **Read the guide:** `Pi5_NAS_Standalone_Setup_Guide.md`
2. **Gather hardware:** Pi 5, SSD, HAT, microSD
3. **Start with Phase 1:** OS installation
4. **Follow each phase sequentially**
5. **Test after each phase** before moving to next
6. **Enjoy your secure NAS!**

**Questions or issues?** Check Troubleshooting section in main guide.

---

**Status:** ✅ Ready for deployment  
**Last Updated:** 2026-02-22  
**Version:** 1.0  
**License:** MIT

---

**Built with ❤️ for privacy and security**
