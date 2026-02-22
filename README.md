# Pi 5 NAS Standalone Setup
## Secure Encrypted NAS + Jellyfin Media Server + RetroPie Emulation

> **Enterprise-grade private media server and game emulator on Raspberry Pi 5 with LUKS encryption, WireGuard VPN access, and defense-in-depth security.**

[![Status](https://img.shields.io/badge/Status-Ready%20for%20Deployment-green)](.)
[![Hardware](https://img.shields.io/badge/Hardware-Pi%205%208GB%20%2B%20NVMe-blue)](.)
[![Security](https://img.shields.io/badge/Security-LUKS%20%2B%20WireGuard%20%2B%20UFW-red)](.)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

---

## 🎯 What This Is

A complete setup guide for building a **secure, encrypted private media server** on Raspberry Pi 5 that:

- **Encrypts all data at rest** — LUKS encryption on NVMe SSD (2TB)
- **Provides secure remote access** — WireGuard VPN tunnel from anywhere
- **Streams media privately** — Jellyfin on encrypted storage (no ISP visibility)
- **Runs classic games** — RetroPie emulation with secure ROM storage
- **Defends against attacks** — UFW firewall + Fail2ban + SSH key-only auth
- **Integrates with Bridge** — Designed to connect to Pi 5 Bridge Router later

**Hardware:** Pi 5 8GB + Kingston P34A60 SSD (2TB) + Waveshare HAT  
**Storage:** Fully encrypted LUKS volume with automatic unlock on boot  
**Remote Access:** WireGuard VPN (NYC → LA encrypted tunnel)  
**Services:** Jellyfin (streaming) + RetroPie (gaming) + Samba (file sharing)

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
