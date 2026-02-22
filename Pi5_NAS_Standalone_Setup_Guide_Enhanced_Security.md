# Pi 5 NAS Standalone Setup Guide
## Secure Encrypted NAS + Jellyfin + RetroPie

**Hardware Configuration:**
- Raspberry Pi 5 8GB RAM
- Waveshare PCIe to M.2 NVMe HAT (x1001)
- Kingston P34A60 SSD (PCIe Gen 3 x4 M.2 2280)
- Active cooling unit
- Gigabit Ethernet (fallback network)

**Status:** Standalone configuration (will integrate with Pi 5 Bridge later)

---

## Table of Contents

### Core Phases (Foundation)
1. [Architecture Overview](#architecture-overview)
2. [Phase 1: OS Installation & NVMe Setup](#phase-1-os-installation--nvme-setup)
3. [Phase 2: LUKS Encryption (At-Rest Security)](#phase-2-luks-encryption-at-rest-security)
4. [Phase 3: Jellyfin Media Server](#phase-3-jellyfin-media-server)
5. [Phase 4: RetroPie Game Emulator](#phase-4-retropie-game-emulator)
6. [Phase 5: Remote Access Setup (WireGuard VPN)](#phase-5-remote-access-setup-standalone---wireguard-vpn)

### Security Hardening Layers (Enterprise-Grade)
7. [Phase 6: Layer 1 - Authentication & Access Control](#phase-6-layer-1-authentication--access-control)
8. [Phase 7: Layer 3 - Intrusion Detection](#phase-7-layer-3-intrusion-detection)
9. [Phase 8: Layer 4 - Monitoring & Logging](#phase-8-layer-4-monitoring--logging)
10. [Phase 9: Layer 5 - Network Hardening](#phase-9-layer-5-network-hardening)
11. [Phase 10: Layer 6 - System Hardening](#phase-10-layer-6-system-hardening)
12. [Phase 11: Networking & Firewall (Enhanced)](#phase-11-networking--firewall-enhanced)

### Monitoring & Backups
13. [Phase 12: Monitoring & Backups (Basic)](#phase-12-monitoring--backups-basic)

### Future Implementation
14. [Phase 13 (Save for Later): Layer 2 - Enhanced Encrypted Backups](#phase-13-save-for-later-layer-2-enhanced-encrypted-backups)
15. [Future Bridge Integration](#future-bridge-integration)

---

## Architecture Overview

### Hardware Layout
```
┌──────────────────────────────────┐
│    Raspberry Pi 5 (8GB)           │
├──────────────────────────────────┤
│ PCIe Slot:                       │
│  └─ Waveshare HAT x1001          │
│     └─ Kingston P34A60 SSD       │
│        (2TB PCIe Gen 3 x4)       │
│                                  │
│ Network: Gigabit Ethernet        │
│ Storage: microSD (boot) + SSD    │
│ Cooling: Active cooling unit     │
└──────────────────────────────────┘
```

### Service Architecture (Standalone)
```
┌──────────────────────────────────────┐
│     Ubuntu 24.04 LTS ARM64           │
├──────────────────────────────────────┤
│ Boot (microSD):                      │
│  └─ /boot, /root, OS                │
│                                      │
│ NVMe SSD (Encrypted with LUKS):      │
│  ├─ /media (Jellyfin library)        │
│  ├─ /home/pi (User data)             │
│  ├─ /opt/retropie (Games)            │
│  └─ /opt/jellyfin (Jellyfin data)    │
│                                      │
│ Running Services:                    │
│  ├─ Jellyfin (Media streaming)       │
│  ├─ RetroPie (Game emulation)        │
│  ├─ SSH (Remote terminal)            │
│  ├─ Samba (NAS file sharing)         │
│  ├─ ufw (Firewall)                   │
│  └─ fail2ban (Brute-force protection)│
│                                      │
│ Remote Access (Temporary):           │
│  ├─ SSH via public IP + key          │
│  └─ (Later: Headscale VPN from bridge)│
└──────────────────────────────────────┘
```

### Future Bridge Integration
```
When Pi 5 Bridge is deployed:

Your Pi 5 NAS (Current)
    │
    └─→ Connected to Bridge WiFi AP (wlan0)
        │
        ├─→ WireGuard VPN (automatic encryption)
        ├─→ Pi-hole DNS (automatic filtering)
        ├─→ Headscale VPN (secure remote access)
        ├─→ Zeek IDS (threat detection)
        └─→ UFW Firewall (network isolation)

Result: All traffic encrypted, no ISP visibility
```

---

## Phase 1: OS Installation & NVMe Setup

### Step 1.1: Download Ubuntu 24.04 LTS ARM64

```bash
# On your laptop, download the image
wget https://cdimage.ubuntu.com/releases/noble/release/ubuntu-24.04-preinstalled-server-arm64+raspi.img.xz

# Verify download
sha256sum ubuntu-24.04-preinstalled-server-arm64+raspi.img.xz
```

### Step 1.2: Flash to microSD Card

```bash
# Insert microSD card via USB adapter into laptop

# Find microSD device (be careful!)
lsblk
# Look for your USB adapter device (e.g., /dev/sdb)

# Flash (replace sdX with your device, e.g., sdb)
xzcat ubuntu-24.04-preinstalled-server-arm64+raspi.img.xz | \
  sudo dd of=/dev/sdX bs=4M conv=fsync status=progress

# Eject
sudo eject /dev/sdX
```

### Step 1.3: Install Pi 5 Hardware

1. **Power off completely** - unplug PSU
2. **Install Waveshare HAT + NVMe SSD**
   - Open Waveshare box
   - Align HAT with GPIO header (golden contacts face down)
   - Insert gently until fully seated
   - Mount with provided screws
   - Insert Kingston P34A60 into M.2 slot (perpendicular angle)
   - Secure with M.2 screw
3. **Install cooling unit**
   - Apply thermal paste to CPU
   - Mount per cooling unit instructions
   - Ensure airflow not blocked
4. **Insert microSD card** into microSD slot
5. **Connect power** - use official 25W PSU

### Step 1.4: Boot and Initial Setup

```bash
# Power on Pi 5
# Wait 2-3 minutes for first boot (cloud-init)

# Once booted, SSH from your laptop
ssh ubuntu@pi.local
# or: ssh ubuntu@[your-pi-ip]

# Default password (change immediately):
# Password: ubuntu

# Change password on first login
passwd
```

### Step 1.5: Verify NVMe is Detected

```bash
# List all block devices
lsblk
# Should show: nvme0n1 (Kingston SSD)

# Example output:
# NAME         MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
# sda            8:0    0 29.8G  0 disk 
# └─sda1         8:1    0 29.8G  0 part /
# nvme0n1      259:0    0  1.9T  0 disk   ← Kingston P34A60

# Get NVMe details
sudo nvme list
# Should show Kingston P34A60, PCIe Gen 3

# Check NVMe speed
lspci -v | grep -i nvme
```

### Step 1.6: System Update

```bash
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get install -y curl wget git build-essential

# Reboot if kernel updated
sudo reboot
```

---

## Phase 2: LUKS Encryption (At-Rest Security)

**Purpose:** Encrypt entire NVMe SSD so data is unreadable without passphrase

### Step 2.1: Partition NVMe

```bash
# Identify NVMe device
lsblk | grep nvme
# Should be: /dev/nvme0n1

# Create partition table (GPT)
sudo parted /dev/nvme0n1 mklabel gpt

# Create single partition using all space
sudo parted /dev/nvme0n1 mkpart primary ext4 1MiB 100%

# Verify
sudo parted /dev/nvme0n1 print
# Should show: /dev/nvme0n1p1
```

### Step 2.2: Setup LUKS Encryption

```bash
# Create encrypted volume
# WARNING: This will prompt for passphrase
sudo cryptsetup luksFormat /dev/nvme0n1p1

# Enter passphrase (use strong passphrase, 16+ characters)
# Passphrase recommendation:
#   - Mix upper/lowercase
#   - Include numbers & special chars
#   - Remember it! (write down securely)
# Example: MyNAS!2026#Secure

# Verify LUKS header
sudo cryptsetup luksDump /dev/nvme0n1p1
```

### Step 2.3: Open Encrypted Volume

```bash
# Unlock the encrypted partition
sudo cryptsetup luksOpen /dev/nvme0n1p1 nas_encrypted

# Verify it's open
sudo lsblk
# Should show: /dev/mapper/nas_encrypted

# Get UUID for auto-mount later
sudo cryptsetup luksUUID /dev/nvme0n1p1
# Save this UUID (you'll need it)
```

### Step 2.4: Create Filesystem

```bash
# Format the unlocked volume as ext4
sudo mkfs.ext4 /dev/mapper/nas_encrypted

# Create mount point
sudo mkdir -p /mnt/nas_storage

# Mount it
sudo mount /dev/mapper/nas_encrypted /mnt/nas_storage

# Verify
df -h | grep nas_storage
# Should show: /dev/mapper/nas_encrypted with size ~1.9T
```

### Step 2.5: Setup Auto-Mount on Boot

```bash
# Get UUID of encrypted partition
LUKS_UUID=$(sudo cryptsetup luksUUID /dev/nvme0n1p1)
echo "LUKS UUID: $LUKS_UUID"

# Edit /etc/crypttab (for LUKS auto-unlock)
sudo nano /etc/crypttab

# Add this line:
# nas_encrypted UUID=[paste-your-LUKS-UUID] none luks

# Example:
# nas_encrypted UUID=a1b2c3d4-e5f6-7890-abcd-ef1234567890 none luks

# Save (Ctrl+O, Enter, Ctrl+X)

# Edit /etc/fstab (for filesystem auto-mount)
sudo nano /etc/fstab

# Add this line at end:
# /dev/mapper/nas_encrypted  /mnt/nas_storage  ext4  defaults  0  2

# Save (Ctrl+O, Enter, Ctrl+X)

# Verify syntax
sudo mount -a
# If no errors, you're good
```

### Step 2.6: Create Storage Directories

```bash
# Create subdirectories for different data
sudo mkdir -p /mnt/nas_storage/jellyfin
sudo mkdir -p /mnt/nas_storage/retropie
sudo mkdir -p /mnt/nas_storage/media
sudo mkdir -p /mnt/nas_storage/backups

# Set permissions
sudo chown -R ubuntu:ubuntu /mnt/nas_storage
sudo chmod -R 755 /mnt/nas_storage

# Verify
ls -la /mnt/nas_storage/
```

### Step 2.7: Test Reboot

```bash
# Reboot to verify auto-mount works
sudo reboot

# SSH back in
ssh ubuntu@pi.local

# Check if encrypted volume mounted automatically
df -h | grep nas_storage
# Should show mounted without prompting for passphrase

# If you see it, encryption is working! ✓
```

**⚠️ IMPORTANT: Backup Passphrase**
- Store passphrase securely (password manager recommended)
- Without it, SSD is permanently inaccessible if forgotten
- Consider storing written copy in safe deposit box

---

## Phase 3: Jellyfin Media Server

**Purpose:** Private media streaming server for your collection

### Step 3.1: Install Jellyfin

```bash
# Add Jellyfin repository
curl https://repo.jellyfin.org/install-debuntu.sh | sudo bash

# Install Jellyfin
sudo apt-get install -y jellyfin

# Enable and start service
sudo systemctl enable jellyfin
sudo systemctl start jellyfin

# Verify it's running
sudo systemctl status jellyfin
```

### Step 3.2: Move Jellyfin Data to Encrypted NVMe

```bash
# Stop Jellyfin
sudo systemctl stop jellyfin

# Move default Jellyfin config to encrypted storage
sudo mv /var/lib/jellyfin /mnt/nas_storage/jellyfin/config
sudo mv /var/cache/jellyfin /mnt/nas_storage/jellyfin/cache

# Create symlinks to maintain paths
sudo ln -s /mnt/nas_storage/jellyfin/config /var/lib/jellyfin
sudo ln -s /mnt/nas_storage/jellyfin/cache /var/cache/jellyfin

# Set correct permissions
sudo chown -R jellyfin:jellyfin /mnt/nas_storage/jellyfin

# Start Jellyfin
sudo systemctl start jellyfin

# Verify
sudo systemctl status jellyfin
```

### Step 3.3: Initial Jellyfin Configuration

```bash
# Access Jellyfin web UI
# From your laptop browser: http://pi.local:8096
# Or: http://[your-pi-ip]:8096

# Go through setup wizard:
# 1. Username: admin
# 2. Password: [create strong password]
# 3. Server name: "NAS-Jellyfin"
# 4. Allow remote connections: NO (for now - will use VPN later)
# 5. Preferred metadata language: English
# 6. Click "Start"
```

### Step 3.4: Add Media Libraries

```bash
# In Jellyfin web UI:
# 1. Click "Manage" (top right) → Libraries
# 2. Click "Add Media Library"
# 3. Set up libraries:

Library 1: Movies
- Name: Movies
- Type: Movies
- Folder: /mnt/nas_storage/media/movies
- Create folder first: mkdir -p /mnt/nas_storage/media/movies

Library 2: TV Shows
- Name: TV Shows
- Type: TV Shows
- Folder: /mnt/nas_storage/media/tvshows
- mkdir -p /mnt/nas_storage/media/tvshows

Library 3: Music
- Name: Music
- Type: Music
- Folder: /mnt/nas_storage/media/music
- mkdir -p /mnt/nas_storage/media/music

# 4. For each library, click "Add Folder"
# 5. Type path, click "OK"
# 6. Click "Save"
```

### Step 3.5: Configure Jellyfin Settings

```bash
# In Jellyfin web UI:
# 1. Admin → Dashboard (gear icon)
# 2. Playback:
#    - Transcoding quality: Medium (to save CPU)
#    - Allowed simultaneous transcode jobs: 2
#    - Transcoding temp directory: /mnt/nas_storage/jellyfin/transcode

# 3. Networking:
#    - Public server: Disabled (for now)
#    - HTTPS: Disabled (standalone - use VPN later)
#    - Port: 8096 (default)

# 4. Library:
#    - Automatic metadata refresh: Enabled
#    - Run metadata update: Weekly

# 5. Users:
#    - Create user for family members
#    - Set parental controls if needed
```

### Step 3.6: Add Content

```bash
# Copy your media to encrypted storage
# From your laptop, use SCP or Samba (see Phase 6)

# Via SCP (from laptop):
scp -r ~/Videos/Movies/* ubuntu@pi.local:/mnt/nas_storage/media/movies/
scp -r ~/Videos/TVShows/* ubuntu@pi.local:/mnt/nas_storage/media/tvshows/

# Jellyfin will auto-scan and index
# Go to Jellyfin → Libraries → Scan
```

### Step 3.7: Test Jellyfin

```bash
# Access from laptop: http://pi.local:8096
# 1. Sign in with admin user
# 2. Go to Libraries
# 3. Verify content appears
# 4. Click on movie/show
# 5. Click Play
# 6. Verify playback works
```

**Jellyfin Notes:**
- Hardware transcoding on Pi 5 is limited (use software transcoding)
- Direct play (no transcoding) is best for performance
- Transcode jobs will use encrypted storage automatically

---

## Phase 4: RetroPie Game Emulator

**Purpose:** Classic game emulation with secure ROM storage

### Step 4.1: Install RetroPie

```bash
# Download and run RetroPie installer
git clone --depth=1 https://github.com/RetroPie/RetroPie-Setup.git
cd RetroPie-Setup

# Run installer
sudo chmod +x retropie_setup.sh
sudo ./retropie_setup.sh

# In the menu that appears:
# 1. Basic Install (option 1)
# 2. Choose all default options
# 3. Wait 30-60 minutes for compilation
```

### Step 4.2: Move RetroPie to Encrypted Storage

```bash
# Stop EmulationStation if running
sudo systemctl stop emulationstation

# Move RetroPie to encrypted storage
sudo mv ~/.emulationstation /mnt/nas_storage/retropie/emulationstation
sudo mv ~/.config/retropie /mnt/nas_storage/retropie/config
sudo mv ~/RetroPie /mnt/nas_storage/retropie/roms

# Create symlinks
ln -s /mnt/nas_storage/retropie/emulationstation ~/.emulationstation
ln -s /mnt/nas_storage/retropie/config ~/.config/retropie
ln -s /mnt/nas_storage/retropie/roms ~/RetroPie

# Set permissions
sudo chown -R ubuntu:ubuntu /mnt/nas_storage/retropie
```

### Step 4.3: Configure RetroPie

```bash
# Enable RetroPie service
sudo systemctl enable emulationstation
sudo systemctl start emulationstation

# Connect via SSH and run RetroPie configuration
# ssh ubuntu@pi.local
# emulationstation --help

# Or access via RetroPie web interface:
# http://pi.local:3000/
```

### Step 4.4: Add ROM Files

```bash
# Create ROM directories
mkdir -p /mnt/nas_storage/retropie/roms/{nes,snes,genesis,gba,etc}

# Copy ROM files from laptop via SCP:
scp ~/Roms/nes/*.nes ubuntu@pi.local:/mnt/nas_storage/retropie/roms/nes/

# Or use Samba (see Phase 6) to drag-and-drop
```

### Step 4.5: Test RetroPie

```bash
# Access RetroPie: http://pi.local:3000/
# Or connect to display:
#   1. Connect Pi 5 to HDMI display
#   2. Reboot
#   3. EmulationStation should launch
#   4. You'll see ROM collections
#   5. Select ROM and play

# Or SSH and test CLI:
ssh ubuntu@pi.local
emulationstation --collection all
```

**RetroPie Notes:**
- ROMs stored on encrypted NVMe are automatically protected
- All game saves go to encrypted storage
- Can play simultaneously with Jellyfin (different services)

---

## Phase 5: Remote Access Setup (Standalone) - WireGuard VPN

**Purpose:** Secure encrypted remote access from NYC to your NAS in LA

**IMPORTANT:** This WireGuard setup is temporary (standalone). When Pi 5 Bridge is deployed, Bridge's WireGuard will encrypt ALL network traffic automatically, making this redundant but compatible.

### Step 5.1: Install WireGuard on Pi 5

```bash
# Update packages
sudo apt-get update

# Install WireGuard
sudo apt-get install -y wireguard wireguard-tools resolvconf

# Verify installation
which wg
which wg-quick
```

### Step 5.2: Choose & Download VPN Provider Config

Choose one of these recommended providers (all support WireGuard):

**Option A: ProtonVPN** (Recommended - based in Switzerland, no-logs)
```bash
# 1. Create account at protonvpn.com
# 2. Download WireGuard config for US server
# 3. Get config file (e.g., us-la-01.conf)
# 4. Unzip if needed
```

**Option B: Mullvad** (Anonymous, no account)
```bash
# 1. Go to mullvad.net/wireguard-config/
# 2. Generate WireGuard config (auto-generates)
# 3. Download config file
```

**Option C: IVPN** (No-logs, privacy-focused)
```bash
# 1. Create account at ivpn.net
# 2. Generate WireGuard config
# 3. Download config file
```

**Comparison:**
| Provider | Speed | Privacy | Cost | Logs |
|----------|-------|---------|------|------|
| ProtonVPN | Fast | Very High | $5-12/mo | None |
| Mullvad | Very Fast | Maximum | Free | None |
| IVPN | Fast | Very High | $6-60/year | None |

**Recommendation:** ProtonVPN (good speed + privacy + support)

### Step 5.3: Copy Config to Pi

```bash
# From your laptop, copy the downloaded config to Pi
scp ~/Downloads/us-la-01.conf ubuntu@pi.local:~

# SSH into Pi
ssh ubuntu@pi.local

# Move config to WireGuard directory
sudo mv ~/us-la-01.conf /etc/wireguard/vpn.conf

# Set permissions
sudo chmod 600 /etc/wireguard/vpn.conf

# Verify
ls -la /etc/wireguard/vpn.conf
```

### Step 5.4: Configure WireGuard to Start on Boot

```bash
# Enable WireGuard service
sudo systemctl enable wg-quick@vpn

# Start WireGuard
sudo systemctl start wg-quick@vpn

# Verify it's running
sudo systemctl status wg-quick@vpn
# Should show: active (exited) ✓

# Check WireGuard tunnel
sudo wg show
# Output should show:
# - interface: wg0
# - peer: [VPN provider endpoint]
# - allowed ips: 0.0.0.0/0
# - latest handshake: [recent timestamp]
```

### Step 5.5: Verify VPN is Working

```bash
# Test 1: Check your public IP changed
curl ifconfig.me
# Should return VPN provider's IP (NOT your ISP IP)

# Example:
# Before: 203.0.113.42 (your ISP)
# After: 185.220.101.50 (ProtonVPN server)

# Test 2: Check WireGuard interface
ip addr show wg0
# Should show: inet 10.x.x.x/32 dev wg0

# Test 3: Verify all traffic routed through VPN
sudo wg show
# Look for "endpoint:" - should show VPN server address
# Look for "allowed ips: 0.0.0.0/0" - means ALL traffic

# Test 4: Check DNS leaks (optional)
curl https://dns.mullvad.com/api/ip
# Should return same IP as ifconfig.me (no DNS leaks)
```

### Step 5.6: Setup SSH Over WireGuard

**On Pi (Configure SSH):**

```bash
# Change SSH port to non-standard (defense-in-depth)
sudo nano /etc/ssh/sshd_config

# Find: #Port 22
# Change to: Port 2222

# Save (Ctrl+O, Enter, Ctrl+X)

# Restart SSH
sudo systemctl restart sshd

# Verify on new port
ssh -p 2222 localhost
# Should work
```

**Generate SSH Keys (On Your NYC Laptop):**

```bash
# Generate Ed25519 key pair (if you don't have one)
ssh-keygen -t ed25519 -f ~/.ssh/pi5_nas -N "your-passphrase"

# This creates:
# ~/.ssh/pi5_nas (private key - keep secure!)
# ~/.ssh/pi5_nas.pub (public key - share with Pi)

# Copy public key to Pi
ssh-copy-id -i ~/.ssh/pi5_nas.pub -p 2222 ubuntu@pi.local

# Test key auth works
ssh -i ~/.ssh/pi5_nas -p 2222 ubuntu@pi.local
# Should NOT ask for password
```

**Disable Password Auth (For Security):**

```bash
# On Pi, edit SSH config
sudo nano /etc/ssh/sshd_config

# Find: #PasswordAuthentication yes
# Change to: PasswordAuthentication no

# Find: #PubkeyAuthentication yes
# Change to: PubkeyAuthentication yes

# Save (Ctrl+O, Enter, Ctrl+X)

# Restart SSH
sudo systemctl restart sshd

# Test - this should fail (no password allowed):
ssh -p 2222 ubuntu@pi.local
# Error: Permission denied (publickey)

# This should work (key auth):
ssh -i ~/.ssh/pi5_nas -p 2222 ubuntu@pi.local
# Should connect ✓
```

### Step 5.7: Remote Access from NYC

**From Your NYC Laptop:**

```bash
# Step 1: Install WireGuard on laptop
# macOS: brew install wireguard-tools
# Linux: sudo apt-get install wireguard wireguard-tools
# Windows: Download from wireguard.com

# Step 2: Get same VPN config from provider
# Copy the same config file you used on Pi
# (or generate a new one if provider supports multiple connections)

# Step 3: Start WireGuard on laptop
# macOS/Linux:
sudo wg-quick up /path/to/your/config.conf

# Windows: Use WireGuard GUI, import config

# Step 4: Verify connection
curl ifconfig.me
# Should show VPN provider IP (same as Pi's)

# Step 5: SSH to Pi through VPN tunnel
ssh -i ~/.ssh/pi5_nas -p 2222 ubuntu@pi.local
# Should connect! (even though you're in NYC)

# Step 6: Access Jellyfin
# Browser: http://pi.local:8096
# Or: http://[pi-local-ip]:8096
# (Jellyfin is accessible within VPN tunnel)
```

### Step 5.8: Create SSH Config for Convenience

**On NYC Laptop:**

```bash
# Create SSH config
nano ~/.ssh/config

# Add this:
Host pi-nas
  HostName pi.local
  User ubuntu
  IdentityFile ~/.ssh/pi5_nas
  Port 2222
  # After connecting to VPN, you can:
  # ssh pi-nas (instead of long command)

# Save (Ctrl+O, Enter, Ctrl+X)

# Now you can simply:
ssh pi-nas
```

### Step 5.9: WireGuard on Laptop - Auto-Connect Script (Optional)

**Create connection script on NYC laptop:**

```bash
# macOS/Linux
nano ~/wireguard_connect.sh

#!/bin/bash
# Connect to VPN and SSH to Pi

echo "Connecting to WireGuard..."
sudo wg-quick up /path/to/config.conf

echo "Waiting for tunnel..."
sleep 3

echo "Connecting to Pi NAS..."
ssh -i ~/.ssh/pi5_nas -p 2222 ubuntu@pi.local

echo "Disconnecting from WireGuard..."
sudo wg-quick down /path/to/config.conf

# Save and make executable
chmod +x ~/wireguard_connect.sh

# Then simply run:
./wireguard_connect.sh
```

### Step 5.10: Monitor WireGuard Connection

```bash
# On Pi, check tunnel status
sudo wg show

# Expected output:
# interface: wg0
#   public key: [key]
#   private key: (hidden)
#   listening port: [port]
#
# peer: [VPN-public-key]
#   endpoint: [VPN-IP]:[VPN-port]
#   allowed ips: 0.0.0.0/0
#   latest handshake: [time]
#   transfer: [sent] received, [received] sent

# View continuous traffic
watch -n 1 'sudo wg show'

# Check if tunnel is up
ip link show wg0
# Should show: wg0: <POINTOPOINT,NOARP,UP,LOWER_UP>
```

### Step 5.11: Troubleshooting WireGuard

**Tunnel not connecting:**
```bash
# Check WireGuard service status
sudo systemctl status wg-quick@vpn

# View logs
sudo journalctl -u wg-quick@vpn -n 20

# Try restarting
sudo systemctl restart wg-quick@vpn

# Verify config syntax
sudo wg-quick strip /etc/wireguard/vpn.conf
```

**Can't reach Pi over VPN:**
```bash
# Check if both are connected to VPN
# On Pi: curl ifconfig.me
# On Laptop: curl ifconfig.me
# Should return SAME IP

# If different: One isn't connected to VPN

# Try ping over VPN
ping pi.local
# May not work (depends on VPN config)

# Try SSH with verbose output
ssh -vvv -i ~/.ssh/pi5_nas -p 2222 ubuntu@pi.local
```

**High latency/Slow speed:**
```bash
# Test speed
iperf3 -c [server-in-vlan] (if available)

# Switch to closer VPN server
# On Pi, try different VPN location
# Download new config for closer server
```

### Step 5.12: Verify Complete Setup

```bash
# Checklist:
# [ ] WireGuard running on Pi: sudo systemctl status wg-quick@vpn
# [ ] WireGuard tunnel active: sudo wg show (shows peer connection)
# [ ] Pi's public IP changed: curl ifconfig.me (shows VPN IP)
# [ ] SSH key generated: ls ~/.ssh/pi5_nas
# [ ] SSH key copied to Pi: ssh-copy-id worked
# [ ] SSH key auth works: ssh -i ~/.ssh/pi5_nas -p 2222 ubuntu@pi.local
# [ ] SSH password disabled: PasswordAuthentication no
# [ ] WireGuard running on laptop: wg show (your laptop)
# [ ] Same VPN IP as Pi: curl ifconfig.me
# [ ] Can SSH from laptop: ssh pi-nas (via WireGuard)
# [ ] Can reach Jellyfin: http://pi.local:8096 (over VPN)
# [ ] Can reach Samba: \\pi.local (over VPN)
```

---

## WireGuard Benefits & Architecture

**Why WireGuard for your setup:**
- ✅ You control the VPN endpoint (external provider)
- ✅ Works seamlessly with Bridge's WireGuard later
- ✅ Faster than alternative VPN protocols (modern, minimal)
- ✅ Stronger encryption (Noise protocol, elliptic-curve cryptography)
- ✅ Industry-standard protocol
- ✅ Lightweight on Pi 5 resources (~1MB memory, <1% CPU)
- ✅ Works at tunnel level (transparent to applications)

**Traffic Flow:**
```
NYC Laptop          Pi NAS               VPN Provider
    │                  │                      │
SSH Client ──────→ WireGuard Client ──────→ VPN Endpoint
    │        (encryption)    │        (encrypted)
Encrypted ←───────────────── Internet ─────→ Returns data
```

---

## After Bridge Deployment

When your Pi 5 Bridge is ready:

```
Current (Standalone):
You → Your Laptop WireGuard → External VPN → Pi NAS
(Manual VPN on both sides)

After Bridge:
You → Your Laptop Headscale → Pi 5 Bridge Headscale → Pi NAS
(Automatic, Bridge handles all VPN + traffic encryption)

Your current WireGuard setup:
- Continues to work (optional redundancy)
- Or can be disabled (Bridge WireGuard takes over)
```

---

---

## Phase 6: Layer 1 - Authentication & Access Control

**Purpose:** Enterprise-grade access control with nginx reverse proxy, 2FA, and IP whitelisting

### Step 6.1: Install nginx Reverse Proxy

```bash
# Install nginx
sudo apt-get install -y nginx

# Create nginx configuration directory
sudo mkdir -p /etc/nginx/sites-available /etc/nginx/sites-enabled

# Enable nginx
sudo systemctl enable nginx
sudo systemctl start nginx
```

### Step 6.2: Install 2FA (TOTP-based Authentication)

```bash
# Install libpam-google-authenticator
sudo apt-get install -y libpam-google-authenticator qrencode

# Generate 2FA secret for ubuntu user
google-authenticator -t -d -w 3 -r 3 -R 30

# Follow the prompts:
# 1. Scan QR code with authenticator app (Google Authenticator, Authy, etc.)
# 2. Save emergency scratch codes securely
# 3. Confirm by entering a code from your app

# Save configuration
# (This creates ~/.google_authenticator with your secret)
```

### Step 6.3: Enable 2FA for SSH

```bash
# Edit PAM SSH configuration
sudo nano /etc/pam.d/sshd

# Add at the top:
auth required pam_google_authenticator.so nullok sequential=deny

# Save (Ctrl+O, Enter, Ctrl+X)

# Edit SSH configuration
sudo nano /etc/ssh/sshd_config

# Find: ChallengeResponseAuthentication no
# Change to: ChallengeResponseAuthentication yes

# Save and restart SSH
sudo systemctl restart sshd

# Test 2FA: ssh -p 2222 ubuntu@pi.local
# Will prompt for password, then TOTP code
```

### Step 6.4: Configure nginx with 2FA Reverse Proxy

```bash
# Create nginx config for Jellyfin
sudo nano /etc/nginx/sites-available/jellyfin

upstream jellyfin {
    server 127.0.0.1:8096;
}

# Rate limiting
limit_req_zone $binary_remote_addr zone=jellyfin_limit:10m rate=10r/s;

server {
    listen 8080;
    server_name pi.local;
    
    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    
    # Rate limiting
    limit_req zone=jellyfin_limit burst=20 nodelay;
    
    location / {
        proxy_pass http://jellyfin;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Timeouts for streaming
        proxy_connect_timeout 600s;
        proxy_send_timeout 600s;
        proxy_read_timeout 600s;
    }
}

# Enable site
sudo ln -s /etc/nginx/sites-available/jellyfin /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### Step 6.5: IP Whitelisting (Only Allow NYC IP through VPN)

```bash
# Get your NYC public IP when connected to WireGuard VPN
# From your NYC laptop while VPN is on:
curl ifconfig.me
# Example: 185.220.101.50

# Edit UFW to whitelist only your VPN IP for SSH
sudo ufw delete allow 2222/tcp
sudo ufw allow from 185.220.101.50 to any port 2222 comment "SSH from NYC via WireGuard"

# Verify
sudo ufw status | grep 2222
# Should show: 185.220.101.50 2222/tcp ...

# For Jellyfin, allow from VPN network range (if using Headscale later)
# Allow from WireGuard VPN interface
sudo ufw allow in on wg0 to any port 8096 comment "Jellyfin via WireGuard"
```

### Step 6.6: Verify 2FA + Reverse Proxy Setup

```bash
# Test Jellyfin through nginx proxy (from local network)
curl -v http://pi.local:8080
# Should show nginx proxy response

# Test SSH with 2FA
ssh -p 2222 ubuntu@pi.local
# Should prompt for password, then TOTP code

# Test from NYC (through WireGuard)
# 1. Connect WireGuard on NYC laptop
# 2. SSH with 2FA: ssh -p 2222 ubuntu@pi.local
```

---

## Phase 7: Layer 3 - Intrusion Detection

**Purpose:** Detect unauthorized file changes and system intrusions

### Step 7.1: Install AIDE (File Integrity Monitoring)

```bash
# Install AIDE
sudo apt-get install -y aide aide-common

# Initialize AIDE database (takes 5-10 minutes)
sudo aideinit

# Move database to secure location
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Create daily check script
sudo nano /etc/cron.daily/aide-check

#!/bin/bash
/usr/bin/aide --check > /var/log/aide/daily_check.log 2>&1
if [ $? -ne 0 ]; then
    echo "AIDE detected file changes" | mail -s "AIDE Alert: File Integrity Violation" root
fi

# Make executable
sudo chmod +x /etc/cron.daily/aide-check
```

### Step 7.2: Install auditd (System Call Logging)

```bash
# Install audit daemon
sudo apt-get install -y auditd audispd-plugins

# Enable auditd
sudo systemctl enable auditd
sudo systemctl start auditd

# Add audit rules for critical files
sudo nano /etc/audit/rules.d/pi-nas.rules

# Monitor SSH directory
-w /etc/ssh/ -p wa -k ssh_config_changes

# Monitor system calls
-a always,exit -F arch=b64 -S execve -k exec

# Monitor user/group changes
-w /etc/passwd -p wa -k passwd_changes
-w /etc/group -p wa -k group_changes

# Monitor sudo usage
-w /var/log/auth.log -p wa -k auth_log_changes

# Save and load rules
sudo service auditd restart

# View audit logs
sudo tail -f /var/log/audit/audit.log
```

### Step 7.3: Configure chroot Jails for Services (Optional - Advanced)

```bash
# Create chroot environment for Samba (optional)
sudo mkdir -p /var/chroot/samba

# Basic chroot setup (advanced - only if needed)
# Most users can skip this for initial deployment
```

---

## Phase 8: Layer 4 - Monitoring & Logging

**Purpose:** Centralized logging, security alerts, and log analysis

### Step 8.1: Install rsyslog (Centralized Logging)

```bash
# Install rsyslog
sudo apt-get install -y rsyslog

# Configure rsyslog for centralized collection
sudo nano /etc/rsyslog.d/99-pi-nas.conf

# Collect logs from all services
:programname, isequal, "jellyfin" /var/log/jellyfin.log
:programname, isequal, "sshd" /var/log/ssh-auth.log
:programname, isequal, "sudo" /var/log/sudo.log
:programname, isequal, "fail2ban" /var/log/fail2ban.log

# Security event logging
auth,authpriv.* /var/log/auth-security.log
:msg, contains, "AIDE" /var/log/aide-alerts.log
:msg, contains, "auditd" /var/log/audit.log

# Enable rsyslog
sudo systemctl enable rsyslog
sudo systemctl restart rsyslog
```

### Step 8.2: Security Event Alerting

```bash
# Install mail utilities for alerts
sudo apt-get install -y mailutils

# Create alert script
sudo nano /usr/local/bin/security-alert.sh

#!/bin/bash
# Monitor for security events

LOG_FILE="/var/log/auth-security.log"
ALERT_EMAIL="root"

# Check for failed SSH attempts
tail -f $LOG_FILE | grep "Failed password" | while read line
do
    echo "SSH Attack Detected: $line" | \
    mail -s "🚨 Security Alert: SSH Intrusion Attempt" $ALERT_EMAIL
done

# Check AIDE alerts
tail -f /var/log/aide-alerts.log | while read line
do
    echo "File Integrity Violation: $line" | \
    mail -s "🚨 Security Alert: File Changed" $ALERT_EMAIL
done

# Make executable
sudo chmod +x /usr/local/bin/security-alert.sh

# Run as service
sudo nano /etc/systemd/system/security-alerts.service

[Unit]
Description=Security Event Alerting
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/bin/security-alert.sh
Restart=always

[Install]
WantedBy=multi-user.target

# Enable and start
sudo systemctl enable security-alerts
sudo systemctl start security-alerts
```

### Step 8.3: Install Tripwire (Filesystem Integrity)

```bash
# Install Tripwire
sudo apt-get install -y tripwire

# Initialize Tripwire
sudo tripwire --init
# Follow prompts to create database

# Run daily check
sudo nano /etc/cron.daily/tripwire-check

#!/bin/bash
/usr/sbin/tripwire --check | mail -s "Tripwire Daily Report" root

sudo chmod +x /etc/cron.daily/tripwire-check
```

### Step 8.4: Log Analysis Setup

```bash
# Install logwatch for daily log analysis
sudo apt-get install -y logwatch

# Configure logwatch
sudo nano /etc/logwatch/conf/logwatch.conf

Output = mail
Format = html
MailTo = root
Detail = High
Range = Yesterday

# Enable daily reports
sudo cp /usr/share/logwatch/default.conf/logwatch.conf /etc/logwatch/conf/

# Manual log analysis commands
# View failed SSH attempts
sudo grep "Failed password" /var/log/auth.log | wc -l

# View sudo usage
sudo grep "sudo" /var/log/auth.log

# View Fail2ban bans
sudo fail2ban-client status sshd
```

---

## Phase 9: Layer 5 - Network Hardening

**Purpose:** SSL/TLS encryption, DNS privacy, and MAC filtering

### Step 9.1: Install SSL/TLS Certificates (Self-Signed)

```bash
# Generate self-signed certificate for Jellyfin
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/private/pi-nas.key \
  -out /etc/ssl/certs/pi-nas.crt

# When prompted:
# Country: US
# State: California
# Locality: LA
# Organization: Personal NAS
# CN: pi.local

# Set permissions
sudo chmod 600 /etc/ssl/private/pi-nas.key
sudo chmod 644 /etc/ssl/certs/pi-nas.crt

# Configure nginx for HTTPS
sudo nano /etc/nginx/sites-available/jellyfin

server {
    listen 443 ssl http2;
    server_name pi.local;
    
    ssl_certificate /etc/ssl/certs/pi-nas.crt;
    ssl_certificate_key /etc/ssl/private/pi-nas.key;
    
    # Strong SSL settings
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    
    # Redirect HTTP to HTTPS
    error_page 497 =301 https://$host:$server_port$request_uri;
    
    location / {
        proxy_pass http://jellyfin;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# Reload nginx
sudo systemctl reload nginx
```

### Step 9.2: DNS over HTTPS (DoH) Forcing

```bash
# Edit resolv.conf to use DoH provider
sudo nano /etc/resolv.conf

# Add Cloudflare DoH
nameserver 1.1.1.1
nameserver 1.0.0.1

# Make immutable (prevent ISP DNS hijacking)
sudo chattr +i /etc/resolv.conf

# Verify
sudo chattr -l /etc/resolv.conf
# Should show: immutable
```

### Step 9.3: MAC Filtering on Samba (Optional)

```bash
# Get MAC addresses of trusted devices
# From each trusted device:
ip link show | grep ether
# Example: e8:48:b8:12:34:56

# Update Samba config for MAC filtering
sudo nano /etc/samba/smb.conf

[global]
   # ... existing config ...
   
   # Allow only trusted MACs
   hosts allow = 192.168.1.0/24 e8:48:b8:12:34:56
   hosts deny = ALL

# Restart Samba
sudo systemctl restart smbd
```

---

## Phase 10: Layer 6 - System Hardening

**Purpose:** Kernel hardening, AppArmor, and automated security updates

### Step 10.1: Install & Configure AppArmor (Mandatory Access Control)

```bash
# Install AppArmor
sudo apt-get install -y apparmor apparmor-utils apparmor-profiles apparmor-profiles-extra

# Enable AppArmor
sudo systemctl enable apparmor
sudo systemctl start apparmor

# Create AppArmor profile for Jellyfin
sudo nano /etc/apparmor.d/usr.bin.jellyfin

#include <tunables/global>

/usr/bin/jellyfin {
  #include <abstractions/base>
  #include <abstractions/nameservice>
  
  /mnt/nas_storage/jellyfin/** rw,
  /var/lib/jellyfin/** rw,
  /var/cache/jellyfin/** rw,
  /proc/sys/kernel/random/uuid r,
  /sys/kernel/debug/tracepoints/syscalls/** r,
  
  deny /etc/shadow rwklx,
  deny /root/** rwklx,
}

# Load and enforce profile
sudo apparmor_parser -r /etc/apparmor.d/usr.bin.jellyfin
sudo aa-enforce /usr/bin/jellyfin

# Check status
sudo aa-status | grep jellyfin
```

### Step 10.2: Automated Security Updates

```bash
# Install unattended-upgrades
sudo apt-get install -y unattended-upgrades

# Configure automatic updates
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades

Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}-security";
    "${distro_id}ESMApps:${distro_codename}-apps-security";
};

Unattended-Upgrade::Package-Blacklist {
};

// Auto-reboot if needed
Unattended-Upgrade::Automatic-Reboot "true";
Unattended-Upgrade::Automatic-Reboot-Time "02:00";

// Send email on updates
Unattended-Upgrade::Mail "root";
Unattended-Upgrade::MailReport "on-change";

# Enable auto-updates
sudo nano /etc/apt/apt.conf.d/20auto-upgrades

APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::AutocleanInterval "7";
APT::Periodic::Unattended-Upgrade "1";

# Verify
sudo systemctl enable apt-daily.service
sudo systemctl enable apt-daily-upgrade.service
```

### Step 10.3: Immutable File Flags (Prevent Accidental Deletion)

```bash
# Mark critical config files as immutable
sudo chattr +i /etc/ssh/sshd_config
sudo chattr +i /etc/samba/smb.conf
sudo chattr +i /etc/crypttab
sudo chattr +i /etc/fstab

# Mark important directories
sudo chattr -R +i /etc/ssl/certs/
sudo chattr -R +i /etc/ssl/private/

# Verify immutable flags
sudo lsattr -l /etc/ssh/sshd_config
# Should show: immutable

# If you need to modify, first remove flag:
# sudo chattr -i /path/to/file
# [edit file]
# sudo chattr +i /path/to/file
```

### Step 10.4: Disable Unused Kernel Modules

```bash
# View loaded modules
lsmod

# Create blacklist for unused modules
sudo nano /etc/modprobe.d/blacklist-pi-nas.conf

# Disable uncommon network protocols
blacklist dccp
blacklist sctp
blacklist rds
blacklist tipc

# Disable uncommon filesystems
blacklist cramfs
blacklist freevxfs
blacklist jffs2
blacklist hfs
blacklist hfsplus

# Disable USB storage (if not needed)
# blacklist usb-storage

# Apply changes
sudo update-initramfs -u
```

### Step 10.5: Secure Boot Configuration

```bash
# Check current secure boot status
sudo mokutil --sb-state

# If supported, enable secure boot (this varies by Pi model)
# Note: Pi 5 has limited secure boot support
# Focus on other hardening measures

# Verify kernel protections
cat /proc/cmdline
# Look for: nosmep nopti

# Improve kernel command line (if needed)
sudo nano /boot/firmware/cmdline.txt

# Add security parameters:
# apparmor=1 security=apparmor slub_debug=FZP

# Reboot to apply
sudo reboot
```

---

## Phase 11: Networking & Firewall (Enhanced)

### Step 11.1: Enhanced UFW Configuration

```bash
# Enable UFW
sudo ufw enable

# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (only from NYC WireGuard IP)
sudo ufw allow from 185.220.101.50 to any port 2222 comment "SSH NYC VPN"

# Allow Jellyfin (HTTPS via nginx)
sudo ufw allow 443/tcp comment "HTTPS Jellyfin"
sudo ufw allow in on wg0 to any port 443 comment "Jellyfin via WireGuard"

# Allow Samba (only from LAN/VPN)
sudo ufw allow from 192.168.0.0/16 to any port 445 comment "Samba LAN"
sudo ufw allow in on wg0 to any port 445 comment "Samba VPN"

# Allow WireGuard
sudo ufw allow 51820/udp comment "WireGuard VPN"

# Block all other traffic
sudo ufw default deny forward

# Enable logging
sudo ufw logging on
sudo ufw logging medium

# Verify rules
sudo ufw status numbered
```

### Step 11.2: Rate Limiting

```bash
# Add rate limiting rules
sudo ufw limit 2222/tcp comment "SSH rate limit"
sudo ufw limit 443/tcp comment "HTTPS rate limit"

# Advanced rate limiting (iptables)
sudo iptables -A INPUT -p tcp --dport 2222 -m limit --limit 5/minute --limit-burst 10 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 2222 -j DROP

# Make persistent
sudo apt-get install -y iptables-persistent
sudo iptables-save | sudo tee /etc/iptables/rules.v4
```

---

## Phase 12: Monitoring & Backups (Basic)

### Step 12.1: Health Check Script

```bash
# Create comprehensive health check
nano ~/health_check.sh

#!/bin/bash
echo "=== Pi 5 NAS Security & Health Check ==="
echo ""
echo "Encryption Status:"
sudo cryptsetup status nas_encrypted
echo ""
echo "WireGuard VPN:"
sudo wg show
echo ""
echo "SSH 2FA Status:"
sudo grep "pam_google_authenticator" /etc/pam.d/sshd
echo ""
echo "AIDE Integrity:"
sudo aide --check | tail -20
echo ""
echo "Fail2ban Status:"
sudo fail2ban-client status
echo ""
echo "AppArmor Status:"
sudo aa-status | grep jellyfin
echo ""
echo "UFW Firewall:"
sudo ufw status
echo ""
echo "Disk Usage:"
df -h /mnt/nas_storage
echo ""
echo "Temperature:"
vcgencmd measure_temp
echo ""
echo "Security Logs:"
sudo tail -20 /var/log/auth-security.log

chmod +x ~/health_check.sh
```

### Step 12.2: Daily Security Summary

```bash
# Create daily security report
nano ~/daily_security_report.sh

#!/bin/bash
DATE=$(date +%Y-%m-%d_%H:%M:%S)
REPORT="/var/log/security_report_$DATE.log"

{
    echo "=== Daily Security Report ==="
    echo "Date: $DATE"
    echo ""
    echo "Failed SSH Attempts:"
    grep "Failed password" /var/log/auth.log | tail -10
    echo ""
    echo "Sudo Usage:"
    grep "sudo" /var/log/auth.log | tail -10
    echo ""
    echo "File Integrity Changes (AIDE):"
    sudo aide --check 2>&1 | grep -E "changed|added|removed" | head -10
    echo ""
    echo "Firewall Blocks:"
    sudo tail -20 /var/log/ufw.log
    echo ""
    echo "Active Connections:"
    ss -an | grep ESTABLISHED | wc -l
} > $REPORT

# Email report
mail -s "Security Report - $DATE" root < $REPORT

chmod +x ~/daily_security_report.sh

# Schedule daily at 6 AM
crontab -e
# Add: 0 6 * * * ~/daily_security_report.sh
```

### Step 12.3: Basic Backup Strategy

```bash
# Create basic backup script (see Phase 13 for enhanced version)
nano ~/backup.sh

#!/bin/bash
BACKUP_DIR="/mnt/nas_storage/backups"
DATE=$(date +%Y%m%d_%H%M%S)

# Backup critical configs
tar -czf $BACKUP_DIR/configs_$DATE.tar.gz \
  /etc/ssh/sshd_config \
  /etc/samba/smb.conf \
  /etc/nginx/sites-available/ \
  /etc/ssl/certs/ \
  /etc/audit/rules.d/

# Generate checksums
sha256sum $BACKUP_DIR/configs_$DATE.tar.gz > $BACKUP_DIR/configs_$DATE.sha256

echo "Backup completed: $DATE"

chmod +x ~/backup.sh

# Schedule daily
crontab -e
# Add: 0 3 * * * ~/backup.sh
```

---

## Phase 13 (Save for Later): Layer 2 - Enhanced Encrypted Backups

**⚠️ Phase 13 - SAVE FOR LATER IMPLEMENTATION**

This phase adds advanced backup capabilities. Implement this after your system is fully deployed and stable (Week 8+).

### Features (To Implement Later):
- [ ] Encrypted backups to external USB drive (off-site)
- [ ] Backup encryption separate from LUKS (double-encrypted)
- [ ] Automated backup verification (checksums)
- [ ] Backup integrity checks (detect corruption)
- [ ] Multi-generational backups (keep 7 daily, 4 weekly, 12 monthly)
- [ ] Automated backup restoration tests
- [ ] Cloud backup integration (optional)

### When Ready, See This Section:

```bash
# Create LUKS-encrypted USB backup volume
# Configure automated backup verification
# Setup multi-generational backup retention
# Test backup restoration

# This will be detailed in Phase 13 (future)
```

**For now:** Use Phase 12 basic backups. You can upgrade to Phase 13 later.

---

```bash
# Enable UFW
sudo ufw enable

# Default policy: deny incoming
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (your access)
sudo ufw allow 2222/tcp comment "SSH"

# Allow Jellyfin (local network only)
sudo ufw allow from 192.168.0.0/16 to any port 8096 comment "Jellyfin"

# Allow RetroPie web interface
sudo ufw allow from 192.168.0.0/16 to any port 3000 comment "RetroPie"

# Allow Samba (file sharing)
sudo ufw allow from 192.168.0.0/16 to any port 445 comment "Samba"
sudo ufw allow from 192.168.0.0/16 to any port 139 comment "Samba"

# If using Tailscale, allow its interface
sudo ufw allow in on tailscale0

# Verify rules
sudo ufw status
```

### Step 6.2: Setup Samba (Network File Sharing)

```bash
# Install Samba
sudo apt-get install -y samba samba-utils

# Create Samba config
sudo nano /etc/samba/smb.conf

# Add this at end of file:

[global]
   workgroup = WORKGROUP
   server string = Pi5 NAS
   netbios name = PINAS
   security = user
   map to guest = bad user
   usershare allow guests = no

[media]
   path = /mnt/nas_storage/media
   comment = Media Library
   browseable = yes
   writable = yes
   guest ok = no
   valid users = ubuntu
   create mask = 0755
   directory mask = 0755

[roms]
   path = /mnt/nas_storage/retropie/roms
   comment = RetroPie ROM Collection
   browseable = yes
   writable = yes
   guest ok = no
   valid users = ubuntu
   create mask = 0755
   directory mask = 0755

[jellyfin]
   path = /mnt/nas_storage/jellyfin
   comment = Jellyfin Config
   browseable = yes
   writable = no
   guest ok = no
   valid users = ubuntu

# Save (Ctrl+O, Enter, Ctrl+X)

# Set Samba password for ubuntu user
sudo smbpasswd -a ubuntu
# Enter password (can be same as linux password)

# Enable and start Samba
sudo systemctl enable smbd nmbd
sudo systemctl start smbd nmbd

# Verify
sudo systemctl status smbd
```

### Step 6.3: Access Samba from Laptop

```bash
# Windows:
# 1. File Explorer
# 2. Type: \\pi.local (or \\[your-pi-ip])
# 3. Username: ubuntu
# 4. Password: [your password]
# 5. You'll see: media, roms, jellyfin shares

# Mac/Linux:
# Finder → Go → Connect to Server
# Type: smb://pi.local
# Or: smb://ubuntu@pi.local

# Then drag-and-drop files to add content
```

### Step 6.4: Enable Fail2ban (Brute-Force Protection)

```bash
# Install Fail2ban
sudo apt-get install -y fail2ban

# Create config
sudo nano /etc/fail2ban/jail.local

# Add:
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 3
ignoreip = 127.0.0.1/8 ::1 192.168.0.0/16

[sshd]
enabled = true
port = 2222
filter = sshd
logpath = /var/log/auth.log
maxretry = 3

# Save (Ctrl+O, Enter, Ctrl+X)

# Enable and start
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

# Verify
sudo fail2ban-client status sshd
```

---

## Phase 7: Monitoring & Backups

### Step 7.1: Disk Usage Monitoring

```bash
# Check current usage
df -h

# Example output:
# Filesystem                Size  Used Avail Use% Mounted on
# /dev/nvme0n1p1 (mapper)  1.9T  100G  1.8T   5% /mnt/nas_storage

# Monitor in real-time
watch -n 5 'df -h | grep nas_storage'

# Or use a script for alerting
nano ~/check_storage.sh

#!/bin/bash
USAGE=$(df /mnt/nas_storage | awk 'NR==2 {print $5}' | sed 's/%//')
if [ $USAGE -gt 80 ]; then
  echo "WARNING: Storage at ${USAGE}%"
  # Could send email alert here
fi

chmod +x ~/check_storage.sh

# Run via cron daily:
crontab -e
# Add: 0 9 * * * ~/check_storage.sh
```

### Step 7.2: Service Status Monitoring

```bash
# Create health check script
nano ~/health_check.sh

#!/bin/bash
echo "=== Pi 5 NAS Health Check ==="
echo ""
echo "Jellyfin:"
sudo systemctl status jellyfin | grep Active
echo ""
echo "RetroPie:"
sudo systemctl status emulationstation | grep Active
echo ""
echo "Samba:"
sudo systemctl status smbd | grep Active
echo ""
echo "SSH:"
sudo systemctl status ssh | grep Active
echo ""
echo "Disk Usage:"
df -h | grep nas_storage
echo ""
echo "Temperature:"
vcgencmd measure_temp

chmod +x ~/health_check.sh

# Run daily:
crontab -e
# Add: 0 10 * * * ~/health_check.sh >> ~/health_check.log
```

### Step 7.3: Backup Strategy

```bash
# Backup important config files
nano ~/backup.sh

#!/bin/bash
BACKUP_DIR="/mnt/nas_storage/backups"
DATE=$(date +%Y%m%d_%H%M%S)

# Backup Jellyfin config
tar -czf $BACKUP_DIR/jellyfin_$DATE.tar.gz /mnt/nas_storage/jellyfin/config

# Backup RetroPie config
tar -czf $BACKUP_DIR/retropie_$DATE.tar.gz /mnt/nas_storage/retropie/config

# Backup system config
sudo tar -czf $BACKUP_DIR/etc_$DATE.tar.gz /etc/sshd /etc/samba /etc/ufw

# Clean old backups (keep last 30 days)
find $BACKUP_DIR -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed: $DATE"

chmod +x ~/backup.sh

# Run daily:
crontab -e
# Add: 0 3 * * * ~/backup.sh >> ~/backup.log
```

### Step 7.4: View System Logs

```bash
# Recent system logs
sudo journalctl -n 50

# Jellyfin logs
sudo journalctl -u jellyfin -n 20

# SSH logs
sudo journalctl -u ssh -n 20

# Samba logs
sudo journalctl -u smbd -n 20

# Follow logs in real-time
sudo journalctl -f
```

---

## Future Bridge Integration

When you deploy your Pi 5 Bridge Router, your NAS will transition from standalone to bridge-connected.

### Step 1: Connect NAS to Bridge WiFi

```bash
# When Bridge is ready with wlan0 AP active:

# On your NAS Pi, configure WiFi connection
sudo nano /etc/netplan/99-wifi.yaml

network:
  version: 2
  wifis:
    wlan0:
      dhcp4: true
      access-points:
        "Network-Primary":
          password: "YourStrongPassword123!"

# Apply config
sudo netplan apply

# Verify connection
ip addr show wlan0
# Should show IP like 10.0.0.x
```

### Step 2: Remove Standalone Remote Access

```bash
# You'll no longer need Tailscale or port forwarding
# Instead, use Bridge's Headscale VPN

# Optional: Uninstall Tailscale
sudo apt-get remove tailscale

# Keep SSH running (access via Headscale)
# SSH will work: ssh ubuntu@100.64.x.x (Headscale VPN IP)
```

### Step 3: Verify Bridge Integration

```bash
# On NAS, verify traffic is encrypted by Bridge
# Check if routed through WireGuard tunnel:

# From NAS:
curl ifconfig.me
# Should return VPN provider's IP (not ISP IP)

# DNS queries should go through Pi-hole:
nslookup google.com
# Should respond from Bridge's Pi-hole (10.0.0.1)

# Threat detection should monitor NAS traffic:
# Check Zeek IDS logs on Bridge for NAS traffic
```

### Step 4: Firewall Rules Update

```bash
# Bridge's UFW will have rules for NAS
# Your NAS's UFW stays active (defense-in-depth)

# On NAS, keep local firewall:
# - SSH: 2222/tcp (for emergencies)
# - Jellyfin: Allow from Bridge LAN (10.0.0.0/24)
# - Samba: Allow from Bridge LAN
# - RetroPie: Allow from Bridge LAN

# Update firewall rules:
sudo ufw delete allow from 192.168.0.0/16 to any port 8096
sudo ufw allow from 10.0.0.0/24 to any port 8096
```

---

## Troubleshooting

### NVMe Not Detected

```bash
# Check PCIe connection
lspci | grep -i nvme

# Check kernel messages
dmesg | tail -20
# Look for NVMe initialization messages

# Try reseating HAT and SSD:
# 1. Power off
# 2. Remove HAT
# 3. Reseat HAT firmly
# 4. Ensure SSD is properly seated
# 5. Power on
```

### LUKS Passphrase Forgotten

```bash
# ⚠️ Cannot recover without passphrase!
# Your only option:
# 1. Erase SSD: sudo cryptsetup erase /dev/nvme0n1p1
# 2. Recreate encrypted volume with new passphrase
# 3. All data is lost

# Prevention: Store passphrase securely!
```

### Jellyfin Not Starting

```bash
# Check service status
sudo systemctl status jellyfin

# View logs
sudo journalctl -u jellyfin -n 50

# Common issue: Symlink broken
ls -la /var/lib/jellyfin
# Should show: symlink → /mnt/nas_storage/jellyfin/config

# If broken, recreate:
sudo systemctl stop jellyfin
sudo rm /var/lib/jellyfin
sudo ln -s /mnt/nas_storage/jellyfin/config /var/lib/jellyfin
sudo systemctl start jellyfin
```

### Samba Not Accessible

```bash
# Check if Samba is running
sudo systemctl status smbd

# Test from localhost
smbclient -L //localhost -U ubuntu

# Check firewall
sudo ufw status | grep Samba

# Verify config syntax
sudo testparm /etc/samba/smb.conf

# Common fix: Restart Samba
sudo systemctl restart smbd
```

### RetroPie Controllers Not Working

```bash
# SSH to Pi
ssh ubuntu@pi.local

# Run RetroPie configuration
# This requires display connection

# If using headless, configure via web UI:
# http://pi.local:3000/
```

### SSH Connection Issues

```bash
# Verify SSH is running
sudo systemctl status ssh

# Check SSH port
sudo ss -tlnp | grep ssh
# Should show: 2222/tcp

# Try connecting with verbose output
ssh -vvv -p 2222 ubuntu@pi.local

# If key-based auth failing:
ls -la ~/.ssh/
# Check permissions: should be 600
chmod 600 ~/.ssh/pi5_nas
```

---

## Performance Benchmarking

### Storage Performance

```bash
# Test NVMe speed
sudo hdparm -Tt /dev/nvme0n1n1

# Write performance
dd if=/dev/zero of=/mnt/nas_storage/test bs=1M count=1024 oflag=direct
# Should see ~2000-3000 MB/s

# Read performance
dd if=/mnt/nas_storage/test of=/dev/null bs=1M iflag=direct
# Should see ~3000-4000 MB/s

# Encryption overhead (LUKS)
# Typically <5% performance hit
```

### Jellyfin Performance

```bash
# Test transcoding
# Play video in Jellyfin
# Monitor CPU:
watch -n 1 'ps aux | grep jellyfin'

# Expected:
# - Direct play: <10% CPU
# - Transcode: 30-50% CPU
```

### Network Performance

```bash
# Test Samba speed
# Copy large file to Pi via Samba
# From laptop: time cp large_file.iso /Volumes/media/

# Expected:
# - Gigabit Ethernet: 100-125 MB/s
# - WiFi: 20-50 MB/s (depending on signal)
```

---

## Security Checklist

- [ ] LUKS encryption enabled on NVMe
- [ ] SSH key-based auth working
- [ ] SSH on non-standard port (2222)
- [ ] UFW firewall enabled
- [ ] Fail2ban protecting SSH
- [ ] Jellyfin remote access: Disabled (will enable via VPN later)
- [ ] Samba access: Restricted to LAN only
- [ ] Automatic backups running daily
- [ ] Passphrase stored securely
- [ ] SSH private key stored securely with strong passphrase
- [ ] Tailscale account secured
- [ ] No default passwords remaining

---

## Quick Reference Commands

```bash
# Check encrypted volume status
sudo cryptsetup status nas_encrypted

# Manually unlock SSD (if needed)
sudo cryptsetup luksOpen /dev/nvme0n1p1 nas_encrypted

# Manually mount (if needed)
sudo mount /dev/mapper/nas_encrypted /mnt/nas_storage

# Jellyfin service control
sudo systemctl {start|stop|restart|status} jellyfin

# RetroPie service control
sudo systemctl {start|stop|restart|status} emulationstation

# Samba service control
sudo systemctl {start|stop|restart|status} smbd

# View disk usage
df -h /mnt/nas_storage

# View system logs
sudo journalctl -f

# SSH into Pi
ssh -i ~/.ssh/pi5_nas -p 2222 ubuntu@pi.local
```

---

## Summary

Your standalone Pi 5 NAS is now:

✅ **Encrypted** — All data protected by LUKS  
✅ **Private** — Jellyfin on encrypted storage  
✅ **Secure** — SSH keys, UFW firewall, Fail2ban  
✅ **Accessible** — Samba shares, Jellyfin web UI, RetroPie emulation  
✅ **Remotely Manageable** — Tailscale VPN access  
✅ **Monitored** — Health checks and backups  

When Bridge is deployed, your NAS will gain:

🚀 **ISP Privacy** — WireGuard encryption  
🚀 **DNS Filtering** — Pi-hole automatic protection  
🚀 **Threat Detection** — Zeek IDS monitoring  
🚀 **Better Remote Access** — Headscale VPN (replaces Tailscale)  
🚀 **Unified Monitoring** — Grafana dashboards  

---

**Ready to build? Start with Phase 1!**
