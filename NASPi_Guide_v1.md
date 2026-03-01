# NASPi_Guide_v1: Comprehensive Raspberry Pi 5 Home NAS Implementation Manual
## Master Guide - Option 2 (Nextcloud Edition) - Complete in One Document

**Version:** 1.0 (Production-Ready)  
**Status:** ✅ Enterprise-Grade Security + File Sync + Professional Monitoring  
**Total Scope:** 3,500+ lines (consolidated from all guides)  
**Primary Guide:** YES (start here - everything you need)  
**Reference Guides:** Available in /outputs for deep dives  

---

## ⚡ PROJECT SUMMARY

```
WHAT:     Secure home NAS with Nextcloud file sync, Jellyfin media,
          RetroArch + EmulationStation gaming, WireGuard VPN, and professional monitoring

WHY:      Production-grade security, privacy-focused, self-hosted,
          fully documented, cost-effective ($300 vs $3,000+)

HOW:      6-7 weeks, 17-22 hours total
          Phases 1-9: Core NAS (from Enhanced guide)
          Jellyfin security hardening (this guide)
          Nextcloud + Cockpit + Netdata (this guide)
          
RESULT:   ✅ LUKS encryption (AES-256)
          ✅ 6 overlapping security layers
          ✅ Nextcloud auto file sync
          ✅ Jellyfin 4K media streaming  
          ✅ RetroArch + EmulationStation gaming (50+ systems)
          ✅ WireGuard VPN (NYC ↔ LA)
          ✅ Professional monitoring dashboards
          ✅ Intrusion detection (AIDE/auditd/Tripwire)
          ✅ Zero-trust firewall architecture
```

---

## NAVIGATION - THIS IS YOUR MASTER GUIDE

**Read this guide if:**
- ✅ You want complete end-to-end implementation
- ✅ You want everything in one document
- ✅ You want architecture + reasoning + configuration + troubleshooting
- ✅ You want copy-paste ready commands

**Reference these guides for deep dives:**
- `Pi5_NAS_Standalone_Setup_Guide_ENHANCED.md` - Detailed Phase 1-9 steps
- `Jellyfin_Web_Access_Security_Hardening.md` - Advanced Jellyfin security
- `MANAGEMENT.md` - Detailed Cockpit/Netdata setup
- `NEXTCLOUD_TAILSCALE_ANALYSIS.md` - Nextcloud implementation details

**Quick Links Below:**

1. [Architecture & Design](#architecture-and-design) - Understand the system
2. [Technology Reasoning](#technology-reasoning) - Why each choice
3. [Phases 1-9 Configuration](#phases-1-9-configuration) - Core NAS setup
   - [Phase 1: Ubuntu Installation & NVMe Migration](#phase-1-ubuntu-installation--nvme-migration-2-3-hours)
   - [Phase 2: Data Partition & LUKS Encryption](#phase-2-data-partition--luks-encryption-1-2-hours)
   - [Phase 3: Nextcloud + Jellyfin Practical Setup](#phase-3-nextcloud--jellyfin-practical-setup-3-hours)
   - [Phase 3.5: Jellyfin Security Hardening](#phase-35-jellyfin-security-hardening-2-3-hours)
   - [Phase 4: RetroArch + EmulationStation](#phase-4-retroarch--emulationstation-15-hours)
   - [Phase 5: WireGuard VPN](#phase-5-wireguard-vpn-15-hours)
   - [Phase 6: Firewall & Hardening](#phase-6-firewall--hardening-2-hours)
   - [Phase 7: Monitoring & Logging](#phase-7-monitoring--logging-15-hours)
   - [Phase 8: Intrusion Detection](#phase-8-intrusion-detection-15-hours)
   - [Phase 9: Management Dashboard](#phase-9-management-dashboard-15-min)
4. [Troubleshooting](#troubleshooting) - Solutions for common issues
5. [References](#references) - Commands, links, resources
6. [Summary](#summary) - What you've built, next steps

---

## Architecture and Design

### Hardware Setup

```
Raspberry Pi 5 (8GB RAM)
├─ Waveshare PCIe HAT (PCIe Gen 3 x4 — native NVMe, no USB bottleneck)
├─ NVMe SSD
│  ├─ nvme0n1p1  512M   vfat        /boot/firmware   (EFI/boot)
│  ├─ nvme0n1p2   50G   ext4        /                (Ubuntu OS root)
│  └─ nvme0n1p3  rest   LUKS2/ext4  /mnt/data        (encrypted user data)
│     └─ /dev/mapper/data → /mnt/data
│        ├─ nextcloud/   (Nextcloud data root — documents, photos, media)
│        ├─ jellyfin/    (Jellyfin metadata cache)
│        └─ retroarch/   (ROMs, saves, BIOS, configs)
├─ microSD card          (initial boot / fallback only — OS lives on NVMe)
├─ Active cooling        (heatsink + fan)
├─ Gigabit Ethernet      (wired — required for 4K streaming reliability)
└─ 27W official PSU
```

### Software Architecture

```
LAYER 1: Applications
├─ Nextcloud (port 80/443) - File sync via snap (Apache + MariaDB built-in)
├─ Jellyfin  (port 8096)   - Media server (internal, not exposed directly)
├─ RetroArch + EmulationStation - Game emulation, data on /mnt/data
└─ SSH (port 22 → 2222 after Phase 6 hardening)

LAYER 2: Reverse Proxy & HTTPS
└─ nginx (port 8920 HTTPS) - TLS termination + security headers for Jellyfin
                              (proxies to Jellyfin on localhost:8096)

LAYER 3: Monitoring Dashboards
├─ Cockpit (port 9090 HTTPS) - System administration UI
└─ Netdata (port 19999 HTTP) - Real-time metrics (LAN-only via UFW)

LAYER 4: Security Tools
├─ Fail2ban  - Brute-force protection (Jellyfin + SSH jails)
├─ AIDE      - File integrity monitoring (daily scan, email report)
├─ auditd    - Syscall-level audit logging
├─ AppArmor  - Mandatory access control (enforcing mode)
└─ UFW       - Stateful firewall (default deny inbound)

LAYER 5: Network Security
├─ WireGuard VPN (port 51820 UDP) - Encrypted remote access tunnel
├─ Self-signed TLS (nginx)        - HTTPS for Jellyfin on LAN/VPN
└─ rsyslog                        - Centralized log aggregation

LAYER 6: Encryption at Rest
└─ LUKS2 (nvme0n1p3) - AES-256-XTS, unlocked at boot via crypttab

BASE: OS + Hardware
├─ Ubuntu 24.04 LTS ARM64
└─ Raspberry Pi 5 + Waveshare PCIe HAT + NVMe SSD
```

### Security Layers (5-Deep)

```
LAYER 5: System Hardening
├─ AppArmor (enforcing mode)
├─ unattended-upgrades (automatic security patches)
└─ UFW default-deny inbound

LAYER 4: Monitoring & Alerting
├─ Centralized rsyslog
├─ Netdata email alerts (disk, CPU, memory thresholds)
└─ Jellyfin auth failure log monitoring

LAYER 3: Intrusion Detection
├─ AIDE daily file integrity checks
├─ auditd syscall logging
└─ Fail2ban active banning (Jellyfin + SSH)

LAYER 2: In-Transit Encryption
├─ WireGuard VPN (all remote access)
├─ nginx TLS (HTTPS for Jellyfin on port 8920)
└─ Nextcloud HTTPS (snap-managed certificate)

LAYER 1: At-Rest Encryption & Access Control
├─ LUKS2 data volume (AES-256-XTS, passphrase-protected)
├─ SSH key-based authentication
├─ UFW IP whitelisting (LAN + VPN ranges only)
└─ Fail2ban brute-force lockout
```

---

## Technology Reasoning

### Why Raspberry Pi 5?

**PCIe Gen 3 x4 changes everything:**
- Pi 4: USB 3.0 bottleneck (400 MB/s max regardless of SSD speed)
- Pi 5: Native PCIe Gen 3 x4 via Waveshare HAT (theoretical 3,500 MB/s — SSD is the real limit)
- Result: NVMe performs at actual NVMe speeds, not USB speeds

**Why not an x86 server?**
- 150W idle draw = ~$130/year in electricity (Pi 5 draws ~5-10W idle)
- Fan noise in a home environment
- Larger physical footprint
- Pi 5 runs Ubuntu 24.04 LTS with full apt ecosystem — it's not a toy

### Why Waveshare PCIe HAT?

- Exposes the Pi 5's PCIe Gen 3 interface to a standard M.2 NVMe slot
- Passive design — no additional power draw beyond the HAT connector
- Widely tested with Pi 5; known-good compatibility with Ubuntu

### Why NVMe over USB SSD or microSD?

- MicroSD: ~50 MB/s sequential, high failure rate under sustained writes — unsuitable as a NAS drive
- USB SSD: ~400 MB/s cap due to USB 3.0 controller, adds latency
- NVMe on PCIe: 1,000–3,500 MB/s sequential — no artificial bottleneck, durable under continuous workload

### Why Ubuntu 24.04 LTS?

- **LTS = 5-year security support** (until April 2029)
- Official Raspberry Pi 5 ARM64 image with maintained kernel
- Full apt ecosystem: Cockpit, Netdata, Fail2ban, AIDE, auditd all available as packages
- Familiar environment for server administration (not Pi-OS specific)

### Why LUKS2 (Single Volume on p3)?

- **One passphrase, one partition** — simple to manage, nothing spread across multiple volumes
- **Standard:** cryptsetup/LUKS is built into every major Linux distribution
- **Portable:** If the SSD is moved to another machine, any Linux system can unlock it
- **Separation from OS:** p1 and p2 (boot/root) are unencrypted so the system boots normally; only user data requires the passphrase
- **AES-256-XTS** is hardware-accelerated on the Pi 5's Cortex-A76 cores

### Why Nextcloud (not Samba/SMB)?

- **Active sync:** Samba is a file share you mount; Nextcloud actively syncs to devices like Dropbox
- **Mobile apps:** Native iOS and Android clients with automatic photo backup
- **File versioning:** Recover previous versions of any file
- **Selective sync:** Sync only what you need (documents), not your entire media library
- **Self-hosted and open-source:** No vendor lock-in, no cloud subscription

### Why Nextcloud via snap?

- Single command install — Apache, PHP, and MariaDB are bundled and pre-configured inside the snap
- Automatic internal updates managed by the snap
- Isolated from the host system's PHP/Apache stack (no version conflicts)
- Trade-off: slightly less flexible than a manual LAMP stack, but the reduced complexity is worth it for a home NAS

### Why Jellyfin (not Plex)?

- **No account required:** Plex requires a plex.tv account even for local streaming
- **No paywall:** Plex locks 4K transcoding and downloads behind Plex Pass
- **Open-source:** Jellyfin is fully community-maintained; no proprietary relay servers
- **Privacy:** Media activity stays on your hardware — Plex phones home

### Why nginx as Reverse Proxy for Jellyfin?

- Jellyfin's built-in HTTPS requires a PKCS12 (.pfx) certificate — nginx handles standard PEM certs directly
- nginx adds security headers (HSTS, X-Frame-Options, CSP) that Jellyfin doesn't set natively
- WebSocket proxying for Jellyfin's real-time session updates is straightforward in nginx
- Keeps Jellyfin itself on localhost:8096 (not exposed) while nginx handles the public-facing port 8920

### Why RetroArch + EmulationStation (not RetroPie)?

- RetroPie is built for Raspberry Pi OS and is not officially supported on Ubuntu 24.04
- RetroArch is available in the Ubuntu apt repository and runs natively on ARM64
- EmulationStation provides the same game-browsing frontend experience
- ROMs and saves stored on /mnt/data (encrypted NVMe) rather than microSD — faster loading, more storage

### Why WireGuard VPN?

- **Fast:** Consistently 300–400 MB/s on Pi-class hardware (vs OpenVPN ~80 MB/s)
- **Simple:** ~4,000 lines of code in the kernel module (vs ~400,000 for OpenVPN)
- **Modern cryptography:** ChaCha20-Poly1305, Curve25519 — not legacy cipher suites
- **Low overhead:** <1% CPU at idle on the Pi 5; OpenVPN requires continuous userspace processing

### Why Cockpit + Netdata?

- **Different roles:** Cockpit is for administration (manage services, storage, users, firewall); Netdata is for real-time observability (CPU, disk I/O, memory, per-process)
- **Zero configuration:** Both work immediately after `apt install`
- **Lightweight:** Cockpit is socket-activated (zero overhead when not in use); Netdata uses ~50–80MB RAM

---

## PHASES 1-9 CONFIGURATION

### Quick Overview Table

| Phase | Component | Time | Result |
|-------|-----------|------|--------|
| 1 | Ubuntu Installation & NVMe Migration | 2-3 hrs | Ubuntu 24.04 LTS booting from NVMe |
| 2 | Data Partition & LUKS Encryption | 1-2 hrs | Encrypted data volume (p3) added, mounted at /mnt/data |
| 3 | Nextcloud + Jellyfin | 3 hrs | File sync + media streaming, shared directory on encrypted volume |
| 3.5 | Jellyfin Security Hardening | 2-3 hrs | HTTPS via nginx, Fail2ban, audit logging |
| 4 | RetroArch + EmulationStation | 1.5 hrs | 50+ system game emulation on NVMe |
| 5 | WireGuard VPN | 1.5 hrs | Encrypted remote access tunnel |
| 6 | Firewall & Hardening | 2 hrs | UFW default-deny, TLS certs, AppArmor |
| 7 | Monitoring & Logging | 1.5 hrs | rsyslog, health checks, email alerts |
| 8 | Intrusion Detection | 1.5 hrs | AIDE file integrity, auditd syscall logging |
| 9 | Management Dashboard | 15 min | Cockpit admin UI + Netdata real-time metrics |

### Phase 1: Ubuntu Installation & NVMe Migration (2-3 hours)

**Starting point:** Raspberry Pi 5 assembled with NVMe HAT and SSD, powered off. MicroSD card available for initial boot.

---

#### Step 1: Flash Ubuntu to microSD

On a separate computer:

1. Download **Raspberry Pi Imager** from [raspberrypi.com/software](https://www.raspberrypi.com/software/)
2. Open Imager → **Choose OS** → Other general-purpose OS → Ubuntu → **Ubuntu Server 24.04 LTS (64-bit)**
3. **Choose Storage** → select your microSD card
4. Click the **gear icon** (advanced options) and configure:
   ```
   Set hostname:        naspi
   Enable SSH:          ✅ (Use password authentication)
   Set username:        pi   (or your preferred username)
   Set password:        [strong password]
   ```
5. **Write** — this flashes and verifies the image

Insert the microSD into the Pi 5. Do **not** boot yet.

---

#### Step 2: First Boot from microSD

Connect the Pi to your router via Ethernet, then power on. Wait 60-90 seconds for cloud-init to complete first-boot setup.

SSH in from another machine on your network:

```bash
ssh pi@naspi.local
# If .local resolution doesn't work, find the Pi's IP via your router's device list
```

Update packages and verify the kernel:

```bash
sudo apt-get update && sudo apt-get upgrade -y
uname -r  # Should show 6.8 or later
```

---

#### Step 3: Set a Static IP

```bash
# Find your current network interface name (usually eth0)
ip link show

sudo nano /etc/netplan/50-cloud-init.yaml
```

Replace the contents with:

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: no
      addresses:
        - 192.168.1.x/24        # choose a free IP on your LAN
      routes:
        - to: default
          via: 192.168.1.1      # your router IP
      nameservers:
        addresses: [1.1.1.1, 8.8.8.8]
```

```bash
sudo netplan apply

# Reconnect SSH using the new static IP
ssh pi@192.168.1.x
```

---

#### Step 4: Verify the NVMe SSD is Detected

```bash
lsblk
# nvme0n1 should appear — if not, check the HAT seating and run:
sudo apt-get install -y nvme-cli
sudo nvme list
```

If the NVMe does not appear, power off and reseat the HAT connection before continuing.

---

#### Step 5: Partition the NVMe for OS (p1 and p2)

This creates only the boot and root partitions. The remaining space is left unpartitioned — Phase 2 will add the encrypted data partition (p3).

```bash
sudo apt-get install -y parted

# Create a fresh GPT partition table
sudo parted /dev/nvme0n1 --script mklabel gpt

# p1: EFI/boot partition (512MB, FAT32)
sudo parted /dev/nvme0n1 --script mkpart primary fat32 1MiB 513MiB
sudo parted /dev/nvme0n1 --script set 1 esp on

# p2: OS root partition (50GB, ext4)
sudo parted /dev/nvme0n1 --script mkpart primary ext4 513MiB 51713MiB

# Confirm
sudo parted /dev/nvme0n1 print
```

Format the partitions:

```bash
sudo mkfs.fat -F32 -n system-boot /dev/nvme0n1p1
sudo mkfs.ext4 -L os-root /dev/nvme0n1p2
```

---

#### Step 6: Clone the OS from microSD to NVMe

Mount the NVMe root partition:

```bash
sudo mkdir -p /mnt/nvme-root
sudo mount /dev/nvme0n1p2 /mnt/nvme-root

sudo mkdir -p /mnt/nvme-root/boot/firmware
sudo mount /dev/nvme0n1p1 /mnt/nvme-root/boot/firmware
```

Copy the root filesystem (excluding virtual and temporary directories):

```bash
sudo rsync -axv --progress / /mnt/nvme-root/ \
  --exclude=/proc \
  --exclude=/sys \
  --exclude=/dev \
  --exclude=/run \
  --exclude=/tmp \
  --exclude=/mnt \
  --exclude=/media \
  --exclude=/lost+found \
  --exclude=/boot/firmware
```

Copy the boot files:

```bash
sudo rsync -axv --progress /boot/firmware/ /mnt/nvme-root/boot/firmware/
```

---

#### Step 7: Update Boot Configuration on the NVMe

The cloned OS still references the microSD's partition UUIDs. Update them to point to the NVMe.

```bash
# Get NVMe partition identifiers
BOOT_UUID=$(sudo blkid -o value -s UUID /dev/nvme0n1p1)
ROOT_UUID=$(sudo blkid -o value -s UUID /dev/nvme0n1p2)
ROOT_PARTUUID=$(sudo blkid -o value -s PARTUUID /dev/nvme0n1p2)

echo "Boot UUID:    $BOOT_UUID"
echo "Root UUID:    $ROOT_UUID"
echo "Root PARTUUID: $ROOT_PARTUUID"
```

Update **fstab** on the NVMe root:

```bash
sudo nano /mnt/nvme-root/etc/fstab
```

The file will have entries for `/` and `/boot/firmware` referencing the microSD's UUIDs or labels. Update them to:

```
UUID=<ROOT_UUID>   /               ext4   defaults        0 1
UUID=<BOOT_UUID>   /boot/firmware  vfat   defaults        0 1
```

Update **cmdline.txt** on the NVMe boot partition (replaces the microSD's root PARTUUID):

```bash
# Get the microSD root PARTUUID (what's currently referenced in cmdline.txt)
SD_PARTUUID=$(sudo blkid -o value -s PARTUUID /dev/mmcblk0p2)

# Replace it with the NVMe root PARTUUID
sudo sed -i "s/$SD_PARTUUID/$ROOT_PARTUUID/g" \
  /mnt/nvme-root/boot/firmware/cmdline.txt

# Verify the change looks correct
cat /mnt/nvme-root/boot/firmware/cmdline.txt
# root=PARTUUID=... should now show the NVMe PARTUUID
```

---

#### Step 8: Configure Pi 5 to Boot from NVMe

The Pi 5 boot order is stored in EEPROM. Update it to try NVMe first, falling back to microSD.

```bash
sudo apt-get install -y rpi-eeprom

# Check current EEPROM config
sudo rpi-eeprom-config

# Write updated boot order
# 0xf16 = NVMe (6) → SD card (1) → stop (f), read right to left
cat > /tmp/nvme_boot.conf << 'EOF'
BOOT_ORDER=0xf16
EOF

sudo rpi-eeprom-config --apply /tmp/nvme_boot.conf
```

Unmount the NVMe and reboot:

```bash
sudo umount /mnt/nvme-root/boot/firmware
sudo umount /mnt/nvme-root

sudo reboot
```

---

#### Step 9: Verify Boot from NVMe

After reboot, SSH back in and confirm the OS is running from the NVMe:

```bash
lsblk
# Root (/) should be mounted on nvme0n1p2, not mmcblk0p2

findmnt /
# Should show: /dev/nvme0n1p2

cat /proc/cmdline
# Should show root=PARTUUID=<nvme-partuuid>
```

The microSD can remain inserted as a fallback boot device or be removed once you're confident the NVMe is stable.

**Phase 2 will add and encrypt the data partition (p3) in the remaining NVMe space.**

### Phase 2: Data Partition & LUKS Encryption (1-2 hours)

**Starting point:** Pi 5 booted from NVMe (Phase 1 complete). p1 and p2 are in use by the OS. The rest of the drive is unpartitioned.

```
nvme0n1  (coming out of Phase 1)
├─ nvme0n1p1   512M   FAT32   /boot/firmware   ← exists
├─ nvme0n1p2    50G   ext4    /                ← exists
└─ [remaining space unpartitioned]              ← Phase 2 adds p3 here
```

#### Step 1: Confirm free space

```bash
sudo parted /dev/nvme0n1 print free
# Look for "Free Space" after nvme0n1p2 — that's where p3 will go
```

#### Step 2: Create the data partition (p3)

```bash
# Add p3 using all remaining space (starts at 51713MiB where p2 ends)
sudo parted /dev/nvme0n1 --script mkpart primary 51713MiB 100%

# Confirm p3 appears
lsblk /dev/nvme0n1
```

#### Step 3: Encrypt the data partition with LUKS

```bash
sudo apt-get install -y cryptsetup

# Format p3 as a LUKS2 container (AES-256-XTS)
# When prompted: type YES (uppercase), then set a strong passphrase
# Store this passphrase in a password manager — if lost, data is unrecoverable
sudo cryptsetup luksFormat --type luks2 /dev/nvme0n1p3

# Verify the LUKS header
sudo cryptsetup luksDump /dev/nvme0n1p3 | head -10

# Unlock the container — maps it to /dev/mapper/data
sudo cryptsetup luksOpen /dev/nvme0n1p3 data

# Create ext4 filesystem inside the LUKS container
sudo mkfs.ext4 -L data /dev/mapper/data
```

#### Step 5: Mount and configure auto-unlock on boot

```bash
# Create mount point and mount
sudo mkdir -p /mnt/data
sudo mount /dev/mapper/data /mnt/data

# Set ownership to your user
sudo chown $USER:$USER /mnt/data

# Get the UUID of the raw LUKS partition (not the mapper)
sudo blkid /dev/nvme0n1p3
# Note the UUID="xxxx..." TYPE="crypto_LUKS" value

# Register with crypttab — will prompt for passphrase at each boot
echo "data UUID=<your-uuid-here> none luks,discard" | sudo tee -a /etc/crypttab

# Register with fstab — mounts after LUKS unlocks
echo "/dev/mapper/data /mnt/data ext4 defaults,nofail 0 2" | sudo tee -a /etc/fstab

# Reload and verify
sudo systemctl daemon-reload
sudo mount -a
```

#### Verify

```bash
lsblk -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINT,LABEL
df -h | grep /mnt/data
sudo cryptsetup status data
```

Expected output:
```
NAME        SIZE  TYPE  FSTYPE      MOUNTPOINT     LABEL
nvme0n1p1   512M  part  vfat        /boot/firmware
nvme0n1p2    50G  part  ext4        /              os-root
nvme0n1p3  [xG]  part  crypto_LUKS
└─data      [xG]  crypt ext4        /mnt/data      data
```

### Phase 3: Nextcloud + Jellyfin Practical Setup (3 hours)

#### How They Work Together

Nextcloud manages your files. Jellyfin streams your media. Both operate on the same underlying directories on the encrypted volume — no duplication, no syncing between services.

```
/mnt/data/nextcloud/<username>/files/
├── Documents/        ← sync to devices via Nextcloud client
├── Photos/           ← sync to devices via Nextcloud client
└── Media/            ← do NOT sync (too large); stream via Jellyfin instead
    ├── Movies/
    ├── TV/
    └── Music/
```

When you add a file through Nextcloud (web, desktop, or mobile), Jellyfin sees it automatically once it scans. When you delete through Nextcloud, it disappears from Jellyfin on next scan. One source of truth.

---

#### Part A: Install and Configure Nextcloud (1.5 hrs)

```bash
# Install Nextcloud via snap (includes Apache, PHP, MariaDB — no manual stack needed)
sudo snap install nextcloud

# Allow the snap to access /mnt paths (required to use our encrypted volume)
sudo snap connect nextcloud:removable-media

# Initial setup — creates the admin account and default data directory
# Use a strong password; this is your Nextcloud admin login
sudo nextcloud.manual-install <admin-username> <strong-password>
```

**Move data directory to the encrypted volume:**

```bash
# Stop Nextcloud before moving data
sudo snap stop nextcloud

# Create the target data directory
sudo mkdir -p /mnt/data/nextcloud

# Copy existing data (the initial admin skeleton files)
sudo cp -a /var/snap/nextcloud/common/nextcloud/data/. /mnt/data/nextcloud/

# Tell Nextcloud the new data location
sudo nextcloud.occ config:system:set datadirectory --value="/mnt/data/nextcloud"

# Restart
sudo snap start nextcloud

# Verify Nextcloud can see its data
sudo nextcloud.occ status
```

**Configure trusted domain (your Pi's LAN IP):**

```bash
# Replace 192.168.1.x with your Pi's actual static IP
sudo nextcloud.occ config:system:set trusted_domains 1 --value="192.168.1.x"

# Access Nextcloud at:
# http://192.168.1.x
# Log in with the admin credentials set above
```

**Create the directory structure for media and documents:**

```bash
# Replace <admin-username> with the username you set above
NC_FILES="/mnt/data/nextcloud/<admin-username>/files"

sudo mkdir -p "$NC_FILES/Documents"
sudo mkdir -p "$NC_FILES/Photos"
sudo mkdir -p "$NC_FILES/Media/Movies"
sudo mkdir -p "$NC_FILES/Media/TV"
sudo mkdir -p "$NC_FILES/Media/Music"

# Register the new folders with Nextcloud's file index
sudo nextcloud.occ files:scan --all
```

---

#### Part B: Install and Configure Jellyfin (1 hr)

```bash
# Install Jellyfin from the official repository
curl -fsSL https://repo.jellyfin.org/install-debuntu.sh | sudo bash

sudo systemctl enable --now jellyfin

# Confirm it's running
sudo systemctl status jellyfin

# Access the setup wizard at:
# http://192.168.1.x:8096
```

**Give Jellyfin read access to Nextcloud's media files:**

Nextcloud owns its data directory. Jellyfin runs as the `jellyfin` user. Use ACLs to grant read access without changing ownership.

```bash
sudo apt-get install -y acl

# Replace <admin-username> with your Nextcloud admin username
MEDIA_DIR="/mnt/data/nextcloud/<admin-username>/files/Media"

# Grant jellyfin user read + execute on existing files
sudo setfacl -R -m u:jellyfin:rx "$MEDIA_DIR"

# Set default ACL so new files added via Nextcloud inherit the permission
sudo setfacl -R -d -m u:jellyfin:rx "$MEDIA_DIR"

# Verify
getfacl "$MEDIA_DIR"
```

**Add libraries in the Jellyfin setup wizard:**

```
In the browser at http://192.168.1.x:8096 → initial setup wizard:

Add media library → Movies
  Folder: /mnt/data/nextcloud/<admin-username>/files/Media/Movies

Add media library → Shows
  Folder: /mnt/data/nextcloud/<admin-username>/files/Media/TV

Add media library → Music
  Folder: /mnt/data/nextcloud/<admin-username>/files/Media/Music
```

Complete the wizard. Jellyfin will scan the folders — they will be empty until you add media files.

---

#### Part C: Selective Sync Strategy

The Nextcloud desktop and mobile clients support selective sync. Configure clients to sync only lightweight folders:

```
SYNC these folders (small, frequently accessed):
  ✅ Documents/
  ✅ Photos/

DO NOT SYNC (too large — access via Jellyfin streaming instead):
  ❌ Media/Movies
  ❌ Media/TV
  ❌ Media/Music
```

To add media: copy files directly to the NAS via SFTP, or upload through the Nextcloud web interface, then trigger a Jellyfin library scan:

```bash
# Manual scan after adding media files
sudo nextcloud.occ files:scan --all        # update Nextcloud index
# Then in Jellyfin: Dashboard → Libraries → [library] → Scan library
```

---

#### Verify

```bash
# Nextcloud is running
sudo snap services nextcloud

# Jellyfin is running
sudo systemctl status jellyfin

# Data lives on the encrypted volume
df -h /mnt/data
du -sh /mnt/data/nextcloud/

# ACLs are set on Media directory
getfacl /mnt/data/nextcloud/<admin-username>/files/Media
# Should show: user:jellyfin:r-x
```

### Phase 3.5: Jellyfin Security Hardening (2-3 hours)

**Why this phase exists:** Jellyfin ships with sensible defaults for a home network but needs deliberate hardening before any remote or VPN access. These steps lock down authentication, enable HTTPS via nginx, and set up logging and brute-force protection.

#### Security Issues Addressed

| Issue | Severity | Solution |
|-------|----------|----------|
| Default admin user | 🔴 Critical | Delete default account, create named admin |
| Weak password | 🔴 Critical | 20+ char password enforced |
| HTTP unencrypted | 🔴 Critical | HTTPS via nginx reverse proxy (port 8920) |
| No rate limiting | 🟡 High | Fail2ban (5 attempts → 1hr ban) |
| No 2FA | 🟡 High | oauth2-proxy (optional) |
| No audit logging | 🟡 Medium | Jellyfin log monitoring enabled |
| Sessions unmanaged | 🟡 Medium | Login attempt lockout configured |
| Clickjacking | 🟢 Low | X-Frame-Options header via nginx |
| MIME sniffing | 🟢 Low | X-Content-Type-Options header via nginx |

#### Step 1: Basic Hardening in the UI (30 min)

Do this immediately after the first boot of Jellyfin, before opening any ports.

```
# Access admin dashboard at:
http://192.168.1.x:8096/web/index.html#!/dashboard
```

In the browser UI:

```
Dashboard → Users
├─ Delete the default 'Administrator' account  ← CRITICAL, do this first
└─ Create a new admin with a unique username + 20+ char password

Dashboard → Playback
└─ Max concurrent streams: 3-5 (prevents resource exhaustion)

Dashboard → Networking
├─ Allow remote connections to this server: ON  (required for VPN access later)
├─ Enable automatic port mapping (UPnP): OFF
└─ External server registration: OFF

Dashboard → Logs
├─ Log level: Information
└─ (Logs are retained at /var/log/jellyfin/ — 30 days by default)
```

#### Step 2: HTTPS via nginx Reverse Proxy (45 min)

Jellyfin's built-in HTTPS requires a PKCS12 (.pfx) certificate, which is an extra conversion step. It is simpler to run nginx as a reverse proxy — nginx handles TLS with standard PEM certs and Jellyfin stays on HTTP internally (localhost only).

```bash
# Install nginx
sudo apt-get install -y nginx

# Generate a self-signed cert for the NAS (reuse this cert for Nextcloud too)
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/private/nas-key.pem \
  -out /etc/ssl/certs/nas-cert.pem \
  -subj "/CN=naspi.local"

# Create the nginx site config for Jellyfin
sudo nano /etc/nginx/sites-available/jellyfin
```

Paste the following nginx config (not a bash script):

```nginx
upstream jellyfin_backend {
    server 127.0.0.1:8096;
    keepalive 10;
}

# Redirect HTTP → HTTPS
server {
    listen 80;
    server_name naspi.local 192.168.1.x;
    return 301 https://$host:8920$request_uri;
}

server {
    listen 8920 ssl http2;
    server_name naspi.local 192.168.1.x;

    ssl_certificate     /etc/ssl/certs/nas-cert.pem;
    ssl_certificate_key /etc/ssl/private/nas-key.pem;
    ssl_protocols       TLSv1.2 TLSv1.3;
    ssl_ciphers         HIGH:!aNULL:!MD5;

    # Security headers
    add_header Strict-Transport-Security "max-age=31536000" always;
    add_header X-Content-Type-Options    "nosniff"          always;
    add_header X-Frame-Options           "SAMEORIGIN"       always;
    add_header X-XSS-Protection          "1; mode=block"    always;
    add_header Referrer-Policy           "strict-origin"    always;

    # WebSocket support (required for Jellyfin)
    location /socket {
        proxy_pass         http://jellyfin_backend;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade    $http_upgrade;
        proxy_set_header   Connection "upgrade";
        proxy_set_header   Host       $host;
        proxy_set_header   X-Real-IP  $remote_addr;
    }

    location / {
        proxy_pass            http://jellyfin_backend;
        proxy_http_version    1.1;
        proxy_set_header      Connection         "";
        proxy_set_header      Host               $host;
        proxy_set_header      X-Real-IP          $remote_addr;
        proxy_set_header      X-Forwarded-For    $proxy_add_x_forwarded_for;
        proxy_set_header      X-Forwarded-Proto  https;
        proxy_buffering       off;
    }
}
```

```bash
# Enable site and test
sudo ln -s /etc/nginx/sites-available/jellyfin /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl enable nginx
sudo systemctl restart nginx

# Block direct external access to Jellyfin's raw HTTP port
# (nginx handles the public-facing traffic; 8096 stays localhost-only)
sudo ufw deny in on eth0 to any port 8096

# Open the HTTPS port for LAN and VPN
sudo ufw allow from 192.168.1.0/24 to any port 8920
sudo ufw allow from 100.64.0.0/10  to any port 8920

# Verify: open https://192.168.1.x:8920 in a browser
# Accept the self-signed cert warning — this is expected
sudo ufw status
```

#### Step 3: Fail2ban for Brute-Force Protection (20 min)

```bash
sudo apt-get install -y fail2ban

# Create the Jellyfin jail
sudo nano /etc/fail2ban/jail.d/jellyfin.conf
```

```ini
[jellyfin]
enabled  = true
port     = 8920
filter   = jellyfin
logpath  = /var/log/jellyfin/log*.log
maxretry = 5
findtime = 600
bantime  = 3600
```

```bash
# Create the filter that matches Jellyfin's auth failure log format
sudo nano /etc/fail2ban/filter.d/jellyfin.conf
```

```ini
[Definition]
failregex = ^.*Authentication request for .* has been denied \(IP: "<HOST>"\).*$
ignoreregex =
```

```bash
sudo systemctl restart fail2ban

# Confirm the jail is running
sudo fail2ban-client status jellyfin
```

#### Step 4: Log Monitoring (15 min)

```bash
# Jellyfin log directory
ls /var/log/jellyfin/

# Watch live for authentication events
sudo tail -f /var/log/jellyfin/log*.log | grep -i "auth\|denied\|failed"

# Spot-check for failed logins
grep -i "denied\|failed" /var/log/jellyfin/log*.log | tail -50

# Schedule a nightly auth report (optional)
crontab -e
# Add:  0 7 * * * grep -i "denied\|failed" /var/log/jellyfin/log*.log | mail -s "Jellyfin Auth Report" your@email.com
```

**Verification checklist:**
```
☐ https://192.168.1.x:8920 loads Jellyfin
☐ http://192.168.1.x:8096 is NOT reachable from another machine on the LAN
☐ Default 'Administrator' account is deleted
☐ sudo fail2ban-client status jellyfin → shows jail active
☐ sudo nginx -t → no errors
☐ Security headers visible in browser DevTools → Network → Response Headers
```

### Phase 4: RetroArch + EmulationStation (1.5 hours)

**What:** Install RetroArch emulator + EmulationStation frontend on Ubuntu  
**Why:** Game emulation (50+ systems) on NVMe storage (faster than microSD)  
**Time:** 1.5 hours  
**Difficulty:** Intermediate

**Note:** Unlike RetroPie (not officially supported on Ubuntu 24.04), RetroArch + EmulationStation is the manual but fully functional approach on Ubuntu.

```bash
# Create gaming directory structure on encrypted NVMe
mkdir -p /mnt/data/retroarch/{roms,saves,configs,bios}
mkdir -p /mnt/data/emulationstation

# Install RetroArch and dependencies
sudo apt-get install -y retroarch retroarch-assets
sudo apt-get install -y libretro-{nestopia,snes9x,genesis-plus-gx,mupen64plus,dolphin}

# Install EmulationStation
sudo apt-get install -y emulationstation

# Configure RetroArch to use NVMe storage
nano ~/.config/retroarch/retroarch.cfg

# Set these paths:
# savefile_directory = /mnt/data/retroarch/saves
# savestate_directory = /mnt/data/retroarch/saves
# system_directory = /mnt/data/retroarch/bios

# Create symbolic link for ROMs
ln -s /mnt/data/retroarch/roms ~/.config/emulationstation/roms

# Configure EmulationStation
emulationstation --help

# Add ROM files to /mnt/data/retroarch/roms/[system]/
# Supported systems: nes, snes, genesis, n64, psx, etc.

# Start EmulationStation
emulationstation

# Optional: Create systemd service for auto-start
sudo nano /etc/systemd/system/emulationstation.service
# [Unit]
# Description=EmulationStation
# After=network.target
# 
# [Service]
# Type=simple
# User=ubuntu
# ExecStart=/usr/bin/emulationstation
# 
# [Install]
# WantedBy=multi-user.target

# Enable service
sudo systemctl enable emulationstation
```

**Storage Structure:**
```
/mnt/data/retroarch/
├─ roms/               (Game ROM files)
│  ├─ nes/
│  ├─ snes/
│  ├─ genesis/
│  ├─ n64/
│  ├─ psx/
│  └─ [other systems]/
├─ saves/              (Game save files)
├─ configs/            (RetroArch configurations)
└─ bios/               (BIOS files for systems needing them)
```

**Performance on Pi5 with NVMe:**
```
✅ 8/16-bit systems: Full speed (NES, SNES, Genesis)
✅ PlayStation 1: Mostly playable (NVMe fast loading helps)
✅ Nintendo 64: 30-40 FPS on most games
✅ NVMe advantage: Faster ROM loading than microSD
❌ GameCube/Wii: Not supported (too demanding)
```

**Verification:**
```bash
# Test RetroArch
retroarch --version

# Check EmulationStation
emulationstation --version

# Verify storage paths are correct
ls -la /mnt/data/retroarch/
```

### Phase 5: WireGuard VPN (1.5 hours)

> **Scope of this phase:** NASPi is configured as a standalone WireGuard server — remote devices connect directly to it over the internet. This is the correct setup for the current single-Pi network topology.
> See the [Future Router Note](#future-router-note) at the end of this phase before a Pi 5 router is added to the network.

#### Step 1: Enable IP Forwarding

WireGuard needs the kernel to forward packets between interfaces. Without this, connected clients can reach NASPi but not the LAN behind it.

```bash
echo "net.ipv4.ip_forward = 1" | sudo tee /etc/sysctl.d/99-wireguard.conf
sudo sysctl -p /etc/sysctl.d/99-wireguard.conf

# Verify
sysctl net.ipv4.ip_forward
# Expected: net.ipv4.ip_forward = 1
```

#### Step 2: Install WireGuard

```bash
sudo apt-get install -y wireguard wireguard-tools
```

#### Step 3: Generate Keys

Generate a keypair for the server (NASPi) and one per client device.

```bash
# Server keys
sudo wg genkey | sudo tee /etc/wireguard/server.key | sudo wg pubkey | sudo tee /etc/wireguard/server.pub
sudo chmod 600 /etc/wireguard/server.key

# Client keys — repeat for each device (laptop, phone, etc.)
sudo wg genkey | sudo tee /etc/wireguard/client1.key | sudo wg pubkey | sudo tee /etc/wireguard/client1.pub
sudo chmod 600 /etc/wireguard/client1.key

# View the keys you'll need for the config files
sudo cat /etc/wireguard/server.key   # → SERVER_PRIVATE_KEY
sudo cat /etc/wireguard/server.pub   # → SERVER_PUBLIC_KEY
sudo cat /etc/wireguard/client1.key  # → CLIENT_PRIVATE_KEY
sudo cat /etc/wireguard/client1.pub  # → CLIENT_PUBLIC_KEY
```

#### Step 4: Create the Server Config

```bash
sudo nano /etc/wireguard/wg0.conf
```

```ini
[Interface]
PrivateKey = <SERVER_PRIVATE_KEY>
Address    = 100.64.0.1/24
ListenPort = 51820

# NAT: routes VPN client traffic out through eth0 to the LAN
PostUp   = iptables -A FORWARD -i %i -j ACCEPT; \
           iptables -A FORWARD -o %i -j ACCEPT; \
           iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; \
           iptables -D FORWARD -o %i -j ACCEPT; \
           iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

# --- Client devices (add one [Peer] block per device) ---

# Client 1 (e.g., laptop)
[Peer]
PublicKey  = <CLIENT_PUBLIC_KEY>
AllowedIPs = 100.64.0.2/32
```

```bash
sudo chmod 600 /etc/wireguard/wg0.conf
```

#### Step 5: Create the Client Config

Copy this to the client device (import into the WireGuard app on phone/laptop). Generate one config per device, incrementing the client IP (`.2`, `.3`, etc.).

```ini
[Interface]
PrivateKey = <CLIENT_PRIVATE_KEY>
Address    = 100.64.0.2/32
DNS        = 1.1.1.1

[Peer]
PublicKey           = <SERVER_PUBLIC_KEY>
Endpoint            = <YOUR_PUBLIC_IP_OR_DDNS>:51820
AllowedIPs          = 192.168.1.0/24, 100.64.0.0/24
PersistentKeepalive = 25
```

> **AllowedIPs explained:** `192.168.1.0/24, 100.64.0.0/24` is a split tunnel — only LAN and VPN traffic goes through NASPi. The client's regular internet traffic goes directly. This is preferred for a NAS access use case (not a full privacy VPN).

#### Step 6: Enable and Start WireGuard

```bash
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0

# Verify the interface is up and the peer is listed
sudo wg show
```

#### Step 7: Home Router Port Forwarding

For remote access to work, your current home router (not the future Pi 5 router) must forward inbound UDP traffic to NASPi:

```
Protocol : UDP
Port     : 51820
Forward to: 192.168.1.x (NASPi's static LAN IP)
```

This is configured in your router's admin UI (typically at 192.168.1.1). The exact steps vary by router model.

#### Verify

```bash
# Interface is up, peer is listed
sudo wg show

# wg0 has the expected IP
ip addr show wg0

# From a connected client device, ping NASPi's VPN IP
ping 100.64.0.1

# From a connected client, reach NASPi's Jellyfin
# https://100.64.0.1:8920
```

---

#### Future Router Note

> **Read this before adding a Pi 5 router to the network.**
>
> This phase configures NASPi as a **standalone WireGuard server** — it accepts incoming VPN connections directly and NATs traffic to the LAN. When a Pi 5 privacy router is introduced between NASPi and the internet, this configuration will need to change.
>
> **What will change:**
> - The Pi 5 router becomes the WireGuard server; NASPi becomes a **peer/client** on the router's VPN network
> - `ListenPort` is removed from NASPi's `wg0.conf` (NASPi no longer accepts direct connections)
> - The `PostUp`/`PostDown` NAT masquerading lines are removed (the router handles routing)
> - A `[Peer]` block pointing to the Pi 5 router replaces the current client peer block
> - The UFW rule `allow 51820/udp` on NASPi is removed or restricted to the router's LAN IP
>
> **What stays the same:**
> - The WireGuard package — no reinstall needed
> - NASPi's keypair — keys can be reused or re-registered on the router
> - All other phases — Jellyfin, Nextcloud, nginx, UFW rules for LAN services are unaffected
>
> The reconfiguration is a `wg0.conf` edit and two UFW rule changes. No data is at risk.

### Phase 6: Firewall & Hardening (2 hours)

#### Step 1: UFW Firewall

```bash
# Default policy: deny all inbound, allow all outbound
sudo ufw default deny incoming
sudo ufw default allow outgoing

# SSH — LAN only, non-standard port (rate-limited)
# Note: sshd is reconfigured to port 2222 in Step 2 below
sudo ufw limit from 192.168.1.0/24 to any port 2222/tcp

# WireGuard — open to internet (traffic is encrypted end-to-end)
sudo ufw allow 51820/udp

# Jellyfin via nginx — LAN and VPN
sudo ufw allow from 192.168.1.0/24 to any port 8920/tcp
sudo ufw allow from 100.64.0.0/24  to any port 8920/tcp

# Block raw Jellyfin HTTP — should not be reachable outside localhost
# (also set in Phase 3.5; included here as canonical firewall rule)
sudo ufw deny in on eth0 to any port 8096

# Nextcloud snap — LAN and VPN (both HTTP and HTTPS; snap handles redirect)
sudo ufw allow from 192.168.1.0/24 to any port 80/tcp
sudo ufw allow from 192.168.1.0/24 to any port 443/tcp
sudo ufw allow from 100.64.0.0/24  to any port 80/tcp
sudo ufw allow from 100.64.0.0/24  to any port 443/tcp

# Cockpit + Netdata ports are added in Phase 9 when those services are installed

# Enable UFW
sudo ufw enable

# Verify — review every rule before continuing
sudo ufw status verbose
```

---

#### Step 2: SSH Hardening

**Important:** Complete all three steps in the same session. Keep your current SSH connection open until you have verified the new port works from a second connection.

```bash
sudo nano /etc/ssh/sshd_config
```

Find and update these lines (uncomment if needed):

```
Port 2222
PermitRootLogin no
PasswordAuthentication no
AuthenticationMethods publickey
MaxAuthTries 3
LoginGraceTime 30
```

```bash
# Test the config for syntax errors before restarting
sudo sshd -t

# Restart SSH
sudo systemctl restart ssh

# Verify sshd is listening on 2222
sudo ss -tlnp | grep 2222
```

**Open a new terminal and confirm you can SSH in on port 2222 before closing the original session:**

```bash
ssh -p 2222 pi@192.168.1.x
```

> **If password auth is disabled and you don't have an SSH key set up yet:**
> Set up key-based auth first — copy your public key to the Pi before disabling password auth:
> ```bash
> # Run this from your local machine (not the Pi)
> ssh-copy-id -p 22 pi@192.168.1.x
> # Then disable PasswordAuthentication and restart sshd
> ```

---

#### Step 3: AppArmor

AppArmor is installed and active by default on Ubuntu 24.04. Verify it is in enforcing mode:

```bash
sudo aa-status
# Expected: "apparmor module is loaded" with profiles in enforce mode

sudo systemctl is-enabled apparmor
# Expected: enabled

sudo systemctl status apparmor
# Expected: active (exited)
```

If any profiles are in complain mode rather than enforce, switch them:

```bash
sudo aa-enforce /etc/apparmor.d/*
```

---

#### Step 4: Automatic Security Updates

```bash
sudo apt-get install -y unattended-upgrades

# Configure — select Yes when prompted
sudo dpkg-reconfigure -plow unattended-upgrades

# Verify the auto-upgrade timer is active
sudo systemctl is-enabled unattended-upgrades
sudo systemctl status apt-daily-upgrade.timer

# Confirm security updates are enabled in the config
grep "security" /etc/apt/apt.conf.d/50unattended-upgrades | grep -v "^//"
```

---

#### Verify

```bash
# Firewall rules
sudo ufw status verbose

# SSH on new port
sudo ss -tlnp | grep 2222

# AppArmor enforcing
sudo aa-status | head -5

# Auto-updates active
systemctl is-active unattended-upgrades
```

### Phase 7: Monitoring & Logging (1.5 hours)

#### Step 1: Centralised Security Logging with rsyslog

rsyslog is already running on Ubuntu 24.04. Add a drop-in config to route NAS-relevant events to dedicated log files.

```bash
sudo mkdir -p /var/log/nas
sudo chown root:adm /var/log/nas

sudo nano /etc/rsyslog.d/99-nas.conf
```

```
# /etc/rsyslog.d/99-nas.conf

# SSH logins, sudo, PAM authentication events
auth,authpriv.*                     /var/log/nas/auth.log

# UFW firewall blocks
:msg, contains, "[UFW BLOCK]"       /var/log/nas/firewall.log
& stop

# Fail2ban bans and unbans
:programname, isequal, "fail2ban"   /var/log/nas/fail2ban.log
& stop

# nginx errors
:programname, isequal, "nginx"      /var/log/nas/nginx.log
& stop
```

```bash
sudo systemctl restart rsyslog

# Confirm logs are being written (may take a moment for events to arrive)
ls -lh /var/log/nas/

# Trigger a test entry
logger -p auth.info "rsyslog test entry"
grep "rsyslog test" /var/log/nas/auth.log
```

---

#### Step 2: Log Rotation

Prevent the NAS log files from consuming unbounded disk space.

```bash
sudo nano /etc/logrotate.d/nas
```

```
/var/log/nas/*.log {
    daily
    rotate 30
    compress
    delaycompress
    missingok
    notifempty
    create 640 root adm
    sharedscripts
    postrotate
        systemctl reload rsyslog > /dev/null 2>&1 || true
    endscript
}
```

```bash
# Test rotation config for syntax errors
sudo logrotate --debug /etc/logrotate.d/nas
```

---

#### Step 3: Email Alerts

Install mailutils with Postfix configured as a Gmail SMTP relay. You need a **Gmail App Password** (not your regular Gmail password) — generate one at https://myaccount.google.com/apppasswords.

```bash
sudo apt-get install -y mailutils
# When the Postfix installer prompts: select "Internet Site" → hostname: naspi
```

Configure Postfix to relay through Gmail:

```bash
sudo nano /etc/postfix/main.cf
```

Add or update these lines at the bottom:

```
relayhost = [smtp.gmail.com]:587
smtp_use_tls = yes
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_security_options = noanonymous
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
```

```bash
# Store Gmail credentials
echo "[smtp.gmail.com]:587 your@gmail.com:your-app-password" | sudo tee /etc/postfix/sasl_passwd
sudo postmap /etc/postfix/sasl_passwd
sudo chmod 600 /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db

sudo systemctl restart postfix

# Test — should arrive in your inbox within a minute
echo "NASPi email test" | mail -s "NASPi Test" your@gmail.com
```

---

#### Step 4: Health Check Script

```bash
nano ~/health-check.sh
```

```bash
#!/bin/bash
# NASPi Daily Health Check
ALERT_EMAIL="your@gmail.com"
SUBJECT="NASPi Health Report - $(date '+%Y-%m-%d')"

report() {
  echo "=== NASPi Health Report - $(date) ==="
  echo ""

  echo "--- Disk Usage ---"
  df -h / /boot/firmware /mnt/data 2>/dev/null
  echo ""

  echo "--- LUKS Volume ---"
  sudo cryptsetup status data 2>/dev/null | grep -E "type|cipher|keysize|device|size"
  echo ""

  echo "--- Memory ---"
  free -h
  echo ""

  echo "--- CPU Temperature ---"
  awk '{printf "%.1f°C\n", $1/1000}' /sys/class/thermal/thermal_zone0/temp
  echo ""

  echo "--- System Uptime ---"
  uptime
  echo ""

  echo "--- Service Status ---"
  for svc in ssh nginx jellyfin fail2ban wg-quick@wg0 unattended-upgrades; do
    status=$(systemctl is-active "$svc" 2>/dev/null)
    echo "  $svc: $status"
  done
  echo ""

  echo "--- Nextcloud Status ---"
  sudo nextcloud.occ status 2>/dev/null | head -4
  echo ""

  echo "--- Recent Auth Failures (last 24h) ---"
  grep -i "failed\|invalid\|denied" /var/log/nas/auth.log 2>/dev/null \
    | grep "$(date '+%b %e')" | tail -10 \
    || echo "  None"
  echo ""

  echo "--- Active Fail2ban Bans ---"
  sudo fail2ban-client status 2>/dev/null | grep "Jail list" || echo "  fail2ban not running"
  echo ""
}

report | tee /tmp/naspi-health-report.txt | mail -s "$SUBJECT" "$ALERT_EMAIL"
```

```bash
chmod +x ~/health-check.sh

# Run once manually to verify output and confirm email arrives
~/health-check.sh
```

---

#### Step 5: Schedule Cron Jobs

```bash
crontab -e
```

Add these entries:

```cron
# Daily health report at 6:00 AM
0 6 * * * /home/pi/health-check.sh

# Weekly: check for UFW blocks from overnight
0 7 * * 1 grep "[UFW BLOCK]" /var/log/nas/firewall.log | tail -20 | mail -s "NASPi Weekly Firewall Report" your@gmail.com
```

---

#### Verify

```bash
# rsyslog routing is active
sudo systemctl status rsyslog

# Logs directory populated
ls -lh /var/log/nas/

# Postfix is running and relaying
sudo systemctl status postfix
sudo tail -5 /var/log/mail.log

# Crontab saved correctly
crontab -l
```

---

#### Future Router Integration Note

> **Planned:** The NASPi will eventually sit behind a dedicated Pi 5 privacy router with its own monitoring and logging stack.

**Two monitoring layers — not duplicates:**

```
[ Pi 5 Router ]  ← Network-level monitoring: traffic flows, DNS queries,
    │               bandwidth per device, WAN health, inter-device comms
    │
[ NASPi ]        ← Host-level monitoring: disk, LUKS, CPU temp, services,
                    Nextcloud status, auth failures, Fail2ban bans
```

The NASPi's health-check.sh and rsyslog setup report on things the router cannot see (LUKS unlock status, Nextcloud internals, Jellyfin service state). The router sees things the NASPi cannot (traffic leaving the network, DNS lookups, bandwidth by device). These layers complement rather than duplicate each other.

**What changes when the router is added:**

| Component | Change |
|---|---|
| rsyslog | Forward NASPi logs to router's centralized syslog server |
| health-check.sh | Add router connectivity check; simplify network sections if router covers them |
| Postfix relay | No change — Gmail relay continues working through the router |
| UFW | Allow syslog forwarding port (514/tcp) inbound from router |
| Metrics (optional) | Install `prometheus-node-exporter` so router's Prometheus can scrape NASPi |

**1. Centralized log forwarding to router syslog**

When the router is running rsyslog or syslog-ng as a central log server, add one line to the NASPi's drop-in config:

```bash
sudo tee -a /etc/rsyslog.d/99-nas.conf <<'EOF'

# Forward all NASPi logs to router's central syslog (enable when router is ready)
# Replace 192.168.1.1 with router's actual LAN IP
# *.* @@192.168.1.1:514    # TCP (reliable)
# *.* @192.168.1.1:514     # UDP (lower overhead)
EOF
```

Allow the outbound port through UFW:

```bash
# Uncomment and run when router syslog server is configured
# sudo ufw allow out to 192.168.1.1 port 514
```

**2. Metrics scraping with Prometheus node_exporter (optional)**

If the router runs Prometheus + Grafana for a unified metrics dashboard, install node_exporter on the NASPi so the router can pull hardware metrics (CPU, disk I/O, memory, network) directly:

```bash
# Install when router's Prometheus is configured
sudo apt install -y prometheus-node-exporter

# Restrict node_exporter to LAN only (never expose to WAN)
sudo ufw allow from 192.168.1.0/24 to any port 9100/tcp

# Router's prometheus.yml scrape target:
# - job_name: 'naspi'
#   static_configs:
#     - targets: ['192.168.1.x:9100']
```

With this in place, the router's Grafana dashboard can display NASPi disk usage, temperature, and service metrics alongside router-level traffic graphs — a single pane of glass for the whole setup.

**3. Health check script evolution**

Once the router is monitoring network-level health, add a router connectivity check to the NASPi script so it can flag if the router goes down:

```bash
# Add to health-check.sh report() function when router is deployed:
echo "--- Router Connectivity ---"
if ping -c 2 -W 2 192.168.1.1 &>/dev/null; then
  echo "  Router: reachable"
else
  echo "  WARNING: Router unreachable"
fi
echo ""
```

### Phase 8: Intrusion Detection (1.5 hours)

Two complementary tools:
- **AIDE** — file integrity monitoring: detects unauthorized changes to system files
- **auditd** — kernel-level audit logging: records who did what, when

---

#### Step 1: AIDE — File Integrity Monitoring

AIDE builds a cryptographic database of your system files and alerts you when anything changes.

```bash
sudo apt install -y aide aide-common
```

AIDE's default config (`/etc/aide/aide.conf`) already covers standard system paths. Add a drop-in rule to also monitor NAS-specific configuration:

```bash
sudo tee /etc/aide/aide.conf.d/99-nas.conf <<'EOF'
# Monitor WireGuard config
/etc/wireguard/wg0.conf CONTENT_EX

# Monitor Nextcloud snap config
/var/snap/nextcloud/current/nextcloud/config/config.php CONTENT_EX

# Monitor nginx config
/etc/nginx/sites-enabled/ CONTENT_EX

# Exclude volatile data paths
!/mnt/data
!/var/lib/nextcloud
EOF
```

Initialize the database. This scans your entire system and takes 5–10 minutes:

```bash
sudo aideinit
```

When complete, move the new database into place for future checks:

```bash
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db
```

Schedule a daily integrity check. The output is emailed via Postfix (configured in Phase 7):

```bash
crontab -e
```

Add this line:

```
# AIDE daily file integrity check (2:30 AM)
30 2 * * * /usr/bin/aide --check 2>&1 | mail -s "[NASPi] AIDE Integrity Report $(date +\%F)" your@email.com
```

> **Note:** AIDE only emails when it finds changes. If you receive no email, the check ran clean. Run `sudo aide --check` manually to confirm.

After any intentional system changes (package upgrades, config edits), update the database so AIDE stops flagging legitimate changes:

```bash
sudo aide --update
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db
```

---

#### Step 2: auditd — Kernel Audit Logging

auditd records privileged actions at the kernel level — file access, auth events, privilege escalation, and configuration changes.

```bash
sudo apt install -y auditd audispd-plugins
```

Create audit rules targeting NAS-relevant events:

```bash
sudo tee /etc/audit/rules.d/nas.rules <<'EOF'
# Delete all existing rules on load
-D

# Increase buffer for busy systems
-b 8192

# Failure mode: 1 = log to kernel (safe default)
-f 1

# --- Identity & authentication files ---
-w /etc/passwd -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/group -p wa -k identity
-w /etc/sudoers -p wa -k identity
-w /etc/sudoers.d/ -p wa -k identity

# --- SSH configuration ---
-w /etc/ssh/sshd_config -p wa -k sshd_config

# --- Cron jobs ---
-w /etc/crontab -p wa -k cron
-w /etc/cron.d/ -p wa -k cron
-w /var/spool/cron/crontabs/ -p wa -k cron

# --- LUKS / cryptsetup ---
-w /etc/crypttab -p wa -k luks
-a always,exit -F path=/usr/sbin/cryptsetup -F perm=x -k luks

# --- WireGuard configuration ---
-w /etc/wireguard/ -p wa -k wireguard

# --- Privilege escalation ---
-w /usr/bin/sudo -p x -k privilege_escalation
-w /bin/su -p x -k privilege_escalation

# --- Login events ---
-w /var/log/faillog -p wa -k auth
-w /var/log/lastlog -p wa -k auth
-w /var/run/faillock/ -p wa -k auth
EOF
```

Load the rules and enable auditd on boot:

```bash
sudo systemctl enable --now auditd
sudo auditctl -R /etc/audit/rules.d/nas.rules
```

Verify rules are active:

```bash
sudo auditctl -l
```

---

#### Reviewing Audit Logs

Search logs by key to investigate specific events:

```bash
# See all identity-related changes
sudo ausearch -k identity --interpret

# See all privilege escalation (sudo/su use)
sudo ausearch -k privilege_escalation --interpret

# See all WireGuard config changes
sudo ausearch -k wireguard --interpret

# Summary of all authentication events
sudo aureport --auth

# Summary of all failed events
sudo aureport --failed
```

Generate a weekly audit summary via cron:

```bash
crontab -e
```

Add:

```
# Weekly auditd summary report (Monday 3 AM)
0 3 * * 1 sudo aureport --summary 2>&1 | mail -s "[NASPi] Weekly Audit Report $(date +\%F)" your@email.com
```

---

#### Verify

```bash
# AIDE database exists
ls -lh /var/lib/aide/aide.db

# AIDE manual check runs without errors
sudo aide --check

# auditd is running
sudo systemctl status auditd

# Audit rules loaded
sudo auditctl -l

# Trigger a test event and confirm it's logged
sudo touch /etc/sudoers.d/test-audit && sudo rm /etc/sudoers.d/test-audit
sudo ausearch -k identity --interpret | tail -20
```

---

#### Future Router Integration Note

> **Planned:** The NASPi will eventually sit behind a dedicated Pi 5 privacy router running Snort3 or Suricata for network-level IDS/IPS.

**How they work together:**

AIDE and auditd are *host-based* intrusion detection (HIDS) — they monitor the NASPi's own files and system calls from the inside. Snort3/Suricata on the router is *network-based* intrusion detection (NIDS) — it inspects all traffic crossing the network boundary before it reaches any device. These two layers are complementary and do not conflict.

```
Internet
    │
[ Pi 5 Router ]  ← Snort3/Suricata: detects malicious traffic patterns,
    │               port scans, exploit attempts, C2 beacons in-flight
    │
[ NASPi ]        ← AIDE + auditd: detects file tampering, config changes,
                    privilege escalation, and local auth events
```

**What changes when the router is added:**

| Component | Change needed |
|---|---|
| AIDE | None — continues monitoring system file integrity |
| auditd rules | None — kernel audit logging is host-local |
| auditd log forwarding | Optional: forward audit events to router's syslog for centralized review |
| UFW rules | Consider tightening — router's IPS provides an additional perimeter layer |

**Optional: forward NASPi audit logs to router syslog**

When the router is running a syslog server (e.g., rsyslog), you can ship NASPi's audit events to it for centralized review:

```bash
# /etc/audit/auditd.conf — add remote logging
sudo tee -a /etc/audit/auditd.conf <<'EOF'
##
# Remote syslog forwarding (enable when router syslog is configured)
# name_format = HOSTNAME
EOF

# Or ship via rsyslog — add to /etc/rsyslog.d/99-nas.conf:
# *.* @@192.168.1.1:514   # TCP to router syslog
```

**Snort3/Suricata placement considerations:**

- Run in **inline IPS mode** on the router to actively block detected threats before they reach the NASPi
- Add NASPi-specific rules: flag unexpected inbound connections to port 8920, 51820, or 2222 from unknown sources
- Suricata can consume the Emerging Threats ruleset (`emerging-threats.rules`) for broad coverage with minimal configuration

The NASPi's Phase 8 setup requires **no changes** when the router is deployed. The two detection layers strengthen each other automatically.

### Phase 9: Management Dashboard (15 min)

#### Quick Install

```bash
# Install both dashboards
sudo apt-get install -y cockpit cockpit-storaged netdata

# Enable on boot
sudo systemctl enable cockpit.socket netdata

# Start services
sudo systemctl start cockpit.socket netdata

# Configure firewall
sudo ufw allow from 192.168.1.0/24 to any port 9090
sudo ufw allow from 192.168.1.0/24 to any port 19999

# Access in browser
# Cockpit:  https://pi.local:9090  (admin interface)
# Netdata:  http://pi.local:19999  (real-time graphs)
```

#### Cockpit (HTTPS Port 9090)

```
What you get:
├─ System overview (CPU, RAM, disk usage)
├─ Storage management (LUKS, partitions, SMART)
├─ Service start/stop/restart
├─ Firewall GUI (UFW configuration)
├─ User account management
├─ Terminal access in browser
└─ System logs

Access: https://pi.local:9090
Login: Your Pi username + password

Key Features:
├─ System → Overview: Hardware metrics
├─ System → Storage: SSD info + LUKS status
├─ System → Services: All systemd services
├─ Networking → Firewall: UFW rules
└─ Networking → Interfaces: Network status
```

#### Netdata (HTTP Port 19999)

```
What you get:
├─ Real-time graphs (1-second updates)
├─ 7-day historical data
├─ CPU, RAM, disk, network monitoring
├─ Process monitoring (top CPU/memory users)
├─ Service health tracking
├─ Email alerts on problems
└─ Custom dashboards

Access: http://pi.local:19999
No login required (firewall protected)

Key Sections:
├─ Overview: All metrics at glance
├─ System: CPU, memory, load
├─ Disk: Storage usage, I/O
├─ Network: Ethernet traffic
└─ Processes: Top resource users
```

#### Email Alerts Configuration

```bash
# Configure Netdata email alerts
sudo nano /etc/netdata/health_alarm_notify.conf

SEND_EMAIL="YES"
EMAIL_SENDER="netdata@pi.local"
RECIPIENTS_EMAIL="your-email@example.com"

# For Gmail (use an app-specific password, not your Gmail password)
SMTP_SERVER="smtp.gmail.com"
SMTP_PORT="587"
SMTP_AUTH="YES"
SMTP_USER="your-email@gmail.com"
SMTP_PASS="your-app-specific-password"

# Test
sudo /usr/libexec/netdata/plugins.d/alarm-notify.sh test

# Restart Netdata
sudo systemctl restart netdata
```

#### Daily Monitoring Routine

```
Morning Check (2 minutes):
1. Open Cockpit → System → Overview
   ├─ CPU usage normal? (should be <20% idle)
   ├─ Memory available? (should have >4GB free)
   └─ Disk space? (target <80% full)

2. Check Netdata alerts
   ├─ Any red alerts?
   ├─ Disk filling rapidly?
   └─ Network issues?

3. If green: You're good for the day!
   If problems: Investigate in Cockpit/Netdata
```

---

## TROUBLESHOOTING 

### Installation Issues

#### LUKS Passphrase Not Working

**Symptom:** "No key available with this passphrase"

**Solutions:**
```
1. Check CAPS LOCK is off
2. Try another keyfile if configured
3. Verify passphrase stored securely
4. If forgotten → SSD data lost (by design, unrecoverable)
   └─ Reinstall Ubuntu, start over
5. Prevention: Store in password manager
```

#### Jellyfin HTTPS Port Not Accessible

**Symptom:** "Connection refused" on port 8920

**Debug:**
```bash
# Check Jellyfin running
sudo systemctl status jellyfin

# Check port listening
sudo lsof -i :8920

# Check certificate exists
ls -la /etc/jellyfin/ssl-cert.pem

# Check firewall allows
sudo ufw status | grep 8920

# Restart if needed
sudo systemctl restart jellyfin

# Try accessing
https://192.168.1.x:8920 (use IP not hostname)
```

#### WireGuard Won't Connect

**Symptom:** "Connection timeout" from laptop

**Debug:**
```bash
# Check WireGuard running on Pi
sudo systemctl status wg-quick@wg0
sudo wg show

# Check firewall port
sudo ufw status | grep 51820

# Verify keys correct
cat /etc/wireguard/wg0.conf | grep -i key

# Test from Pi
ping 100.64.0.1

# On laptop, test connection
wg-quick up wg0
ping 100.64.0.1

# Check firewall on both ends
```

### Service Issues

#### Service Won't Start

```bash
# Check status and logs
sudo systemctl status jellyfin -n 100
sudo journalctl -u jellyfin -n 50

# Check for obvious errors
sudo jellyfin --version

# Restart and watch logs
sudo systemctl restart jellyfin
sudo journalctl -u jellyfin -f

# Check port conflicts
sudo lsof -i :8096

# Fix permissions if needed
sudo chown -R jellyfin:jellyfin /var/lib/jellyfin
```

### Storage & Encryption Issues

#### LUKS Volume Won't Mount

```bash
# Check status
sudo cryptsetup status data

# Try manual unlock
sudo cryptsetup luksOpen /dev/nvme0n1p3 data

# Manual mount
sudo mount /dev/mapper/data /mnt/data

# Check disk errors
sudo fsck -n /dev/mapper/data
```

#### Disk Space Issues

```bash
# Check usage by directory
du -sh /mnt/data/*

# Find large files
find /mnt/data -size +1G -type f

# Check Jellyfin/Nextcloud data
du -sh /mnt/data/nextcloud/
du -sh /mnt/data/jellyfin/
du -sh /mnt/data/retroarch/
```

### Network & Remote Access Issues

#### Can't Access NAS via VPN

**Symptom:** VPN connected, but services unreachable

```bash
# Verify VPN IP assigned
ip addr show wg0  # should show 100.64.0.x

# Check firewall allows VPN range
sudo ufw status | grep 100.64

# Verify service listening on all IPs
sudo lsof -i :8920  # should show 0.0.0.0

# Test from Pi directly
curl http://127.0.0.1:8096

# Restart firewall
sudo systemctl restart ufw
```

#### Remote Storage Too Slow

**Symptom:** Only 50 MB/s through VPN (should be 300-400)

```bash
# Check WireGuard throughput
sudo wg show  # watch tx/rx bytes

# Check internet speed
speedtest-cli  # from both locations

# Check if using USB drives (slow)
iostat -x 1   # watch disk utilization

# Check CPU throttling (temperature)
vcgencmd measure_temp
vcgencmd measure_clock arm

# If hot:
├─ Improve cooling (fan speed)
└─ Reduce load (stop unnecessary services)
```

### Performance & Monitoring Issues

#### Cockpit/Netdata Using Too Much CPU

```bash
# Check what's consuming
top  # sort by CPU
ps aux | grep -E "cockpit|netdata"

# If Netdata:
sudo nano /etc/netdata/netdata.conf
# Reduce: update every = 2 (from 1)
# Or: history = 1440 (from 10080)
sudo systemctl restart netdata

# If Cockpit:
sudo systemctl restart cockpit.socket
```

#### Netdata Alerts Not Sending

```bash
# Verify email config
grep -i "send_email\|recipients" \
  /etc/netdata/health_alarm_notify.conf

# Test email system
echo "test" | mail -s "Test" your-email@example.com

# Check mail logs
sudo tail -f /var/log/mail.log

# Test Netdata alert
sudo /usr/libexec/netdata/plugins.d/alarm-notify.sh test

# Restart Netdata
sudo systemctl restart netdata
```

---

## REFERENCES 

### All Your Documentation

```
In /mnt/user-data/outputs/:

PRIMARY (Read First):
├─ NASPi_Guide_v1.md ← YOU ARE HERE (this file)
│  Contains everything integrated in one place
│
REFERENCE (Deep Dives):
├─ Pi5_NAS_Standalone_Setup_Guide_ENHANCED.md
│  └─ Detailed Phases 1-9 with all commands
├─ Jellyfin_Web_Access_Security_Hardening.md
│  └─ Advanced Jellyfin security features
├─ MANAGEMENT.md
│  └─ Detailed Cockpit + Netdata setup
├─ NEXTCLOUD_TAILSCALE_ANALYSIS.md
│  └─ Nextcloud implementation options
│
OPTIONAL REFERENCES:
├─ Pi5_NAS_Hardware_Monitoring_Dashboard.md
│  └─ 4 different monitoring tool options
├─ JELLYFIN_SECURITY_ISSUES_MAPPING.md
│  └─ All 10 security issues mapped to solutions
└─ README.md
   └─ Project overview
```

### Essential Command Reference

```bash
# System Status
sudo systemctl status jellyfin smbd sshd ufw fail2ban
sudo df -h
vcgencmd measure_temp
free -h

# Service Management
sudo systemctl restart [service]
sudo systemctl enable [service]
sudo systemctl stop [service]

# Firewall
sudo ufw status
sudo ufw allow from 192.168.1.0/24 to any port 8920
sudo ufw delete allow 8920

# Logs
journalctl -u jellyfin -f
journalctl -n 50
sudo tail -f /var/log/syslog

# WireGuard
sudo wg show
sudo systemctl restart wg-quick@wg0

# LUKS
sudo cryptsetup status data
sudo mount /dev/mapper/data /mnt/data

# Monitoring
top
ps aux | grep jellyfin
lsof -i :8920
```

### Firewall Rules Reference

```bash
# LAN access (192.168.1.0/24)
sudo ufw allow from 192.168.1.0/24 to any port 2222    # SSH
sudo ufw allow from 192.168.1.0/24 to any port 8920    # Jellyfin
sudo ufw allow from 192.168.1.0/24 to any port 443     # Nextcloud
sudo ufw allow from 192.168.1.0/24 to any port 9090    # Cockpit
sudo ufw allow from 192.168.1.0/24 to any port 19999   # Netdata

# WireGuard
sudo ufw allow 51820/udp

# VPN access (100.64.0.0/10)
sudo ufw allow from 100.64.0.0/10 to any port 8920
sudo ufw allow from 100.64.0.0/10 to any port 443
sudo ufw allow from 100.64.0.0/10 to any port 9090
sudo ufw allow from 100.64.0.0/10 to any port 19999
```

### External Resources

**Software Documentation:**
- Ubuntu 24.04: https://ubuntu.com/download/raspberry-pi
- Jellyfin: https://docs.jellyfin.org/
- Nextcloud: https://docs.nextcloud.com/
- RetroArch: https://docs.libretro.com/
- EmulationStation: https://emulationstation.org/
- WireGuard: https://www.wireguard.com/
- Cockpit: https://cockpit-project.org/
- Netdata: https://learn.netdata.cloud/

**Security:**
- LUKS: https://gitlab.com/cryptsetup/cryptsetup
- AIDE: https://aide.github.io/
- Fail2ban: https://www.fail2ban.org/
- AppArmor: https://gitlab.com/apparmor/apparmor/-/wikis/home

**Hardware:**
- Raspberry Pi: https://www.raspberrypi.com/
- Kingston SSD: https://www.kingston.com/en/support
- Waveshare: https://www.waveshare.com/



## SUMMARY 

### What You've Built

```bash
✅ ENTERPRISE SECURITY (6 layers deep)
   ├─ LUKS encryption (AES-256)
   ├─ SSH key-based auth
   ├─ Fail2ban brute-force protection
   ├─ AIDE + auditd + Tripwire
   ├─ AppArmor mandatory access control
   └─ Automatic security updates

✅ MODERN FILE SYNC (Nextcloud)
   ├─ Auto-sync between devices
   ├─ Phone photo backup
   ├─ Document versioning
   ├─ Collaborative editing
   ├─ Web + mobile access
   └─ Stored on encrypted SSD

✅ MEDIA STREAMING (Jellyfin)
   ├─ 4K streaming capability
   ├─ Multi-user library
   ├─ Secure SSL/TLS
   ├─ Hardened web access
   └─ Privacy-respecting

✅ GAMING EMULATION (RetroArch + EmulationStation)
   ├─ 50+ console systems (NES, SNES, Genesis, N64, PSX, etc)
   ├─ Installed on Ubuntu 24.04 (not microSD)
   ├─ Game storage on encrypted NVMe SSD
   ├─ Full controller support
   └─ Zero interference with NAS services

✅ REMOTE ACCESS (WireGuard VPN)
   ├─ Fast (400+ MB/s)
   ├─ Simple (1,000 lines code)
   ├─ Secure (modern crypto)
   └─ Low CPU usage (<1%)

✅ PROFESSIONAL MONITORING
   ├─ Cockpit admin interface
   ├─ Netdata real-time graphs
   ├─ Email alerts
   ├─ 7-day historical data
   └─ Service health tracking

✅ AUTOMATED RELIABILITY
   ├─ AIDE daily integrity checks
   ├─ Netdata + Cockpit health monitoring
   └─ Email alerts on anomalies
```

### Key Achievements

| Metric | Achievement |
|--------|-------------|
| **Security** | Enterprise-grade (6-layer defense) |
| **Cost** | 1/10th of commercial NAS |
| **Learning** | Professional DevOps platform |
| **Privacy** | Complete data control |
| **Performance** | 2.5 TB/s SSD, 400 MB/s VPN |
| **Uptime** | 24/7 self-hosted services |
| **Operations** | 3-5 min daily, 30 min monthly |

### Timeline Summary

```
Week 1-2:  Foundation (OS + Encryption)                        2.5-3 hrs
Week 2-3:  Services (Nextcloud + Jellyfin + RetroArch/ES)      4.5 hrs
Week 3-4:  Remote Access (VPN + Firewall)                      3.5 hrs
Week 4-5:  Advanced Security (Monitoring + IDS + Cockpit/Netdata)  4.75 hrs
────────────────────────────────────────────────────────────────
TOTAL:     5-6 weeks                                   15-19 hrs
```

### Your Next Steps

#### Immediate (Today)
```
1. ✓ Read NASPi_Guide_v1 (this guide) ← You're doing this!
2. ✓ Understand architecture + reasoning
3. ✓ Review technology choices
4. Gather hardware
5. Download Ubuntu 24.04 ARM64 ISO
6. Plan network addressing
7. Set up workspace
```

#### Short Term (This Week)
```
1. Flash Ubuntu to microSD
2. Boot Pi 5 (Phase 1)
3. Encrypt SSD (Phase 2)
4. Start Jellyfin (Phase 3)
```

#### Medium Term (Weeks 2-4)
```
1. Complete Phases 3-6
2. Setup WireGuard VPN
3. Configure security layers
4. Test remote access
```

#### Long Term (Weeks 4-5)
```
1. Complete Phases 7-8
2. Add Cockpit + Netdata
3. Enjoy professional NAS!
```

---

## FINAL NOTES

### This Guide vs Reference Guides

**Use NASPi_Guide_v1.md (this file) for:**
- ✅ Complete end-to-end overview
- ✅ Architecture understanding
- ✅ Technology reasoning
- ✅ Quick command reference
- ✅ Troubleshooting
- ✅ Everything in one place

**Use reference guides for:**
- 🔍 Detailed step-by-step instructions
- 🔍 Advanced configuration options
- 🔍 Additional security hardening
- 🔍 Performance tuning
- 🔍 Specific technology deep dives

### Support Structure

```
If you're stuck:
1. Search NASPi_Guide_v1 (this file)
2. Read relevant reference guide
3. Check troubleshooting section
4. Follow detailed command-by-command
5. Verify with quick reference
```

### You're Not Alone

```
Community:
├─ Jellyfin community: https://jellyfin.org
├─ Raspberry Pi forums: https://forums.raspberrypi.com
├─ Nextcloud community: https://help.nextcloud.com
└─ WireGuard: https://www.wireguard.com/#community

Debugging:
├─ systemctl status [service]
├─ journalctl -u [service] -f
├─ Google error messages
└─ Check official docs
```

---

## DEPLOYMENT CHECKLIST

Before you start, verify you have:

```
Hardware:
☐ Raspberry Pi 5 (8GB)
☐ Kingston P34A60 SSD (2TB)
☐ Waveshare PCIe HAT x1001
☐ Active cooling (heatsink + fan)
☐ 25W official PSU
☐ Gigabit Ethernet cable
☐ microSD card (64GB)

Software:
☐ Ubuntu 24.04 ARM64 ISO
☐ Balena Etcher or similar
☐ Password manager (for passphrases)
☐ Text editor (for config files)
☐ SSH client (for remote access)

Network:
☐ Static IP planned (192.168.1.x)
☐ Router access available
☐ Gigabit network connection
☐ WireGuard keys ready (generated during setup)
☐ Firewall rules documented

Time & Space:
☐ 17-22 hours available
☐ 6-7 weeks timeline
☐ Workspace for hardware assembly
☐ Linux knowledge (or patience to learn)
```

---

**Status:** ✅ Production-Ready  
**Difficulty:** Intermediate  
**Time:** 6-7 weeks  
**Reward:** Professional home NAS with enterprise security  

**Let's build something awesome! 🚀**

