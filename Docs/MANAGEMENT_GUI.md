# Pi 5 NAS Management Dashboard Guide
## Cockpit + Netdata - Hardware Monitoring & System Management

---

## ⚡ TL;DR - Just Do This Now

```bash
# 1. Install both dashboards (one command)
sudo apt-get install -y cockpit cockpit-storaged netdata

# 2. Enable and start services
sudo systemctl enable cockpit.socket netdata
sudo systemctl start cockpit.socket netdata

# 3. Configure firewall (optional but recommended)
sudo ufw allow from 192.168.1.0/24 to any port 9090 comment "Cockpit - LAN"
sudo ufw allow from 192.168.1.0/24 to any port 19999 comment "Netdata - LAN"

# 4. Access dashboards in browser
# Cockpit:  https://pi.local:9090  (admin interface)
# Netdata:  http://pi.local:19999  (real-time graphs)

# 5. Login to Cockpit
# Username: your Pi username
# Password: your Pi password

# That's it! You're done. Total time: 5 minutes.
```

---

## Table of Contents

- [TL;DR](#tl-dr---just-do-this-now)
- [What You're Installing](#what-youre-installing)
- [System Requirements](#system-requirements)
- [Installation](#installation)
  - [Quick Install (5 min)](#quick-install-5-min)
  - [Detailed Installation](#detailed-installation)
  - [Verify Installation](#verify-installation)
- [Cockpit - System Administration](#cockpit---system-administration)
  - [Accessing Cockpit](#accessing-cockpit)
  - [Cockpit Features Overview](#cockpit-features-overview)
  - [Monitoring SSD Storage](#monitoring-ssd-storage)
  - [Managing Services](#managing-services)
  - [Firewall Management](#firewall-management)
  - [User Accounts](#user-accounts)
  - [Terminal Access](#terminal-access)
  - [Security in Cockpit](#security-in-cockpit)
- [Netdata - Real-time Monitoring](#netdata---real-time-monitoring)
  - [Accessing Netdata](#accessing-netdata)
  - [Netdata Dashboard Tour](#netdata-dashboard-tour)
  - [Storage Monitoring](#storage-monitoring)
  - [CPU & Memory Monitoring](#cpu--memory-monitoring)
  - [Network Monitoring](#network-monitoring)
  - [Configuring Netdata](#configuring-netdata)
  - [Email Alerts](#email-alerts)
  - [Historical Data](#historical-data)
- [Firewall & Security](#firewall--security)
  - [Access Control](#access-control)
  - [Remote Access via VPN](#remote-access-via-vpn)
  - [HTTPS Configuration](#https-configuration)
  - [Authentication](#authentication)
- [Daily Operations](#daily-operations)
  - [Daily Checks](#daily-checks)
  - [Weekly Tasks](#weekly-tasks)
  - [Monthly Maintenance](#monthly-maintenance)
  - [Storage Management](#storage-management)
- [Troubleshooting](#troubleshooting)
  - [Services Not Starting](#services-not-starting)
  - [Cannot Access Dashboards](#cannot-access-dashboards)
  - [Performance Issues](#performance-issues)
  - [Alerts Not Working](#alerts-not-working)
- [Integration with NAS Guide](#integration-with-nas-guide)
  - [Where This Fits](#where-this-fits)
  - [Dependencies](#dependencies)
  - [Next Steps](#next-steps)
- [Quick Reference](#quick-reference)
  - [Useful Commands](#useful-commands)
  - [Dashboard URLs](#dashboard-urls)
  - [Port Information](#port-information)
  - [Resource Usage](#resource-usage)

---

## What You're Installing

### Cockpit - System Administration Interface

**What it is:**
- Official Ubuntu system management tool
- Web-based system administration
- Manage services, users, firewall, storage
- Part of RHEL/Ubuntu ecosystem

**What you can do:**
```
✅ View hardware info (SSD, CPU, RAM, temp)
✅ Manage systemd services (Jellyfin, Samba, SSH, etc)
✅ Configure firewall (UFW) via GUI
✅ Manage user accounts
✅ Access terminal in browser
✅ Monitor system health
✅ View system logs
✅ Manage storage/LUKS encryption
```

### Netdata - Real-time Monitoring

**What it is:**
- Real-time system monitoring agent
- Beautiful interactive dashboards
- Instant alerts and notifications
- Minimal resource usage

**What you can do:**
```
✅ Real-time graphs (update every 1 second)
✅ Historical data (7 days by default)
✅ Storage I/O monitoring
✅ Network traffic analysis
✅ Process list (top CPU/memory consumers)
✅ Email alerts on problems
✅ Temperature monitoring
✅ Service health tracking
```

---

## System Requirements

### Hardware

```
Raspberry Pi 5 (8GB) ✓
├─ CPU: Sufficient (Cockpit <1%, Netdata <1%)
├─ RAM: 8GB with ~50MB for dashboards
└─ Storage: ~200MB for both tools
```

### Software

```
Ubuntu 24.04 ARM64 (on Pi 5)
├─ Cockpit requires: systemd, package management
├─ Netdata requires: Linux kernel, libc
└─ No conflicts with existing setup
```

### Network

```
Gigabit Ethernet (connected)
├─ Cockpit: Port 9090 (HTTPS)
├─ Netdata: Port 19999 (HTTP)
└─ Both can be accessed locally + via VPN
```

---

## Installation

### Quick Install (5 min)

```bash
# Everything in one command:
sudo apt-get install -y cockpit cockpit-storaged netdata && \
sudo systemctl enable cockpit.socket netdata && \
sudo systemctl start cockpit.socket netdata

# Verify both running:
sudo systemctl status cockpit.socket
sudo systemctl status netdata

# Access:
# Cockpit:  https://pi.local:9090
# Netdata:  http://pi.local:19999
```

### Detailed Installation

#### Step 1: Update System

```bash
sudo apt-get update
sudo apt-get upgrade -y
```

#### Step 2: Install Cockpit

```bash
# Install main Cockpit package
sudo apt-get install -y cockpit

# Install storage plugin for SSD monitoring
sudo apt-get install -y cockpit-storaged

# Optional: Additional plugins
sudo apt-get install -y cockpit-packagekit    # Software updates
sudo apt-get install -y cockpit-networkmanager # Network management
sudo apt-get install -y cockpit-pcp            # Performance analytics
```

#### Step 3: Install Netdata

```bash
# Install Netdata
sudo apt-get install -y netdata

# Verify installed
netdata -v
```

#### Step 4: Enable Services

```bash
# Enable Cockpit to start on boot
sudo systemctl enable cockpit.socket

# Enable Netdata to start on boot
sudo systemctl enable netdata
```

#### Step 5: Start Services

```bash
# Start Cockpit
sudo systemctl start cockpit.socket

# Start Netdata
sudo systemctl start netdata
```

#### Step 6: Verify Running

```bash
# Check Cockpit socket
sudo systemctl status cockpit.socket
# Should show: active (listening)

# Check Netdata service
sudo systemctl status netdata
# Should show: active (running)
```

#### Step 7: Configure Firewall

```bash
# Allow Cockpit from local network
sudo ufw allow from 192.168.1.0/24 to any port 9090 comment "Cockpit"

# Allow Netdata from local network
sudo ufw allow from 192.168.1.0/24 to any port 19999 comment "Netdata"

# For VPN access (optional)
sudo ufw allow from 100.64.0.0/10 to any port 9090 comment "Cockpit-VPN"
sudo ufw allow from 100.64.0.0/10 to any port 19999 comment "Netdata-VPN"

# Verify rules
sudo ufw status
```

### Verify Installation

```bash
# Check both services active
sudo systemctl status cockpit.socket netdata

# Check listening ports
sudo lsof -i :9090    # Cockpit
sudo lsof -i :19999   # Netdata

# Test connectivity
curl -k https://127.0.0.1:9090 > /dev/null && echo "Cockpit OK"
curl http://127.0.0.1:19999 > /dev/null && echo "Netdata OK"
```

---

## Cockpit - System Administration

### Accessing Cockpit

#### Local Network Access

```
URL: https://pi.local:9090
or:  https://192.168.1.x:9090

Login:
├─ Username: your Pi username
├─ Password: your Pi password
└─ Important: HTTPS required (not HTTP)
```

#### Remote Access via VPN

```
1. Connect to WireGuard VPN from NYC
2. Access: https://10.0.0.x:9090
   (where 10.0.0.x is your Pi's VPN IP)
3. Same login as above
```

### Cockpit Features Overview

#### Left Sidebar Navigation

```
System
├─ Overview           ← Start here
├─ Logs              ← System logs
├─ Storage           ← SSD monitoring
├─ Services          ← Start/stop services
├─ Accounts          ← User management
└─ System Hostname   ← Pi name config

Networking
├─ Interfaces        ← Ethernet status
├─ Firewall          ← UFW configuration
└─ System Hostname   ← Network config

Applications (if plugins installed)
├─ Software Updates
└─ Package Management

Settings
├─ User Settings
└─ Time & Timezone
```

### Monitoring SSD Storage

#### Accessing Storage Dashboard

```
Cockpit Dashboard → System → Storage
```

#### What You See

```
Block Devices:
├─ Kingston P34A60 (NVMe SSD)
│  ├─ Device: /dev/nvme0n1
│  ├─ Size: 2.0 TB
│  ├─ Model: Kingston P34A60
│  └─ Connection: PCIe Gen 3 x4
│
├─ Partitions:
│  ├─ /dev/nvme0n1p1 (nas_storage)
│  │  ├─ Size: 1.0 TB
│  │  ├─ Used: [BLUE BAR] X%
│  │  ├─ Free: X GB
│  │  ├─ Mount: /mnt/nas_storage
│  │  ├─ Type: LUKS Encrypted (✓)
│  │  └─ Filesystem: ext4
│  │
│  └─ /dev/nvme0n1p2 (nas_backups)
│     ├─ Size: 1.0 TB
│     ├─ Used: [BLUE BAR] X%
│     ├─ Free: X GB
│     ├─ Mount: /mnt/nas_backups
│     ├─ Type: LUKS Encrypted (✓)
│     └─ Filesystem: ext4
│
└─ Micro SD Card (/dev/mmcblk0)
   ├─ Boot partition
   └─ Usage info
```

#### SMART Information

```
Right-click on SSD → View Details

Shows:
├─ Temperature: X°C
├─ Health Status: ✓ Healthy
├─ Power-on Hours: X
├─ Error Count: 0
└─ SMART Status: Passed
```

#### Storage Actions

```
Cockpit Dashboard → System → Storage → [Device] → [Menu]

Available actions:
├─ Mount/Unmount volumes
├─ Create new partitions
├─ Resize partitions
├─ Create RAID (if available)
└─ Wipe storage
```

### Managing Services

#### Accessing Services Panel

```
Cockpit Dashboard → System → Services
```

#### What You See

```
All systemd services listed:

Service Name          Status      Enabled
─────────────────────────────────────────
jellyfin              running ✓   yes
smbd                  running ✓   yes
sshd                  running ✓   yes
ufw                   active ✓    yes
fail2ban              running ✓   yes
aide                  running ✓   yes
auditd                running ✓   yes
netdata               running ✓   yes
cockpit               running ✓   yes
...
```

#### Service Management

```
Click on any service to see options:

├─ Start (if stopped)
├─ Stop (if running)
├─ Restart (bounce service)
├─ Enable (start on boot)
├─ Disable (don't start on boot)
└─ View Logs (recent activity)

Example: Restarting Jellyfin
├─ Click "jellyfin" service
├─ Click "Restart" button
├─ Watch status change to "restarting"
└─ Automatically restarts
```

#### Service Logs

```
Click service → "Logs" tab

Shows:
├─ Last 50 log lines
├─ Real-time updates
├─ Filter by severity
└─ Export logs
```

### Firewall Management

#### Accessing Firewall

```
Cockpit Dashboard → Networking → Firewall
```

#### Current Rules Visible

```
Status: Active

Rules:
Port    Protocol  Source          Service
─────────────────────────────────────────
2222    TCP       192.168.1.0/24  SSH
8920    TCP       192.168.1.0/24  Jellyfin
445     TCP       192.168.1.0/24  Samba
139     TCP       192.168.1.0/24  Samba
9090    TCP       192.168.1.0/24  Cockpit
19999   TCP       192.168.1.0/24  Netdata
```

#### Add/Remove Rules via GUI

```
Add New Rule:
1. Click "+ Add Services" or "+ Add Ports"
2. Choose port or service
3. Select direction (incoming/outgoing)
4. Set source (any, 192.168.x.x, etc)
5. Save

Remove Rule:
1. Click rule
2. Click "Remove" button
3. Confirm
```

### User Accounts

#### Accessing User Management

```
Cockpit Dashboard → System → Accounts
```

#### What You Can Do

```
Create new user:
├─ Username
├─ Full name
├─ Password
├─ Shell (bash, sh, etc)
└─ Groups (sudo, etc)

Modify user:
├─ Change password
├─ Lock/unlock account
├─ Add to groups
└─ Delete user

View current users:
└─ See all system accounts
```

### Terminal Access

#### SSH Terminal in Browser

```
Cockpit Dashboard → Top Right → Terminal

Benefits:
├─ No separate SSH client needed
├─ Direct command execution
├─ Copy/paste support
├─ Full terminal functionality
└─ Access anywhere via HTTPS
```

#### Using Terminal

```
Type commands as normal:
$ journalctl -u jellyfin -f     # Follow Jellyfin logs
$ df -h                          # Disk space
$ top                            # Running processes
$ sudo systemctl restart jellyfin # Restart service
```

### Security in Cockpit

#### Authentication

```
Cockpit uses Linux system authentication:
├─ Username: your Pi login
├─ Password: your Pi password
└─ No separate Cockpit password needed
```

#### HTTPS Only

```
Cockpit enforces HTTPS:
├─ HTTP requests redirect to HTTPS
├─ Self-signed certificate
├─ Browser may warn "not trusted" (OK)
└─ Data encrypted in transit
```

#### Authorization

```
Regular users see:
├─ System info (read-only)
├─ Logs
├─ Terminal access

Admin (sudo) users can:
├─ Restart services
├─ Modify firewall
├─ Manage users
├─ Change configuration
```

---

## Netdata - Real-time Monitoring

### Accessing Netdata

#### Local Network

```
URL: http://pi.local:19999
or:  http://192.168.1.x:19999
```

#### Remote Access via VPN

```
1. Connect WireGuard from NYC
2. Access: http://10.0.0.x:19999
3. No login required (firewall protected)
```

#### Browser Compatibility

```
Works on:
├─ Chrome/Chromium
├─ Firefox
├─ Safari
├─ Edge
└─ Mobile browsers (touch-friendly)
```

### Netdata Dashboard Tour

#### Main Sections

```
http://pi.local:19999

├─ Overview (Home)
│  └─ Quick view of all systems
│
├─ System
│  ├─ CPU usage (all cores)
│  ├─ Memory usage (RAM)
│  ├─ Swap usage
│  ├─ System load
│  └─ Interrupts
│
├─ Disk
│  ├─ Disk space (all partitions)
│  ├─ Disk I/O read/write
│  ├─ IOPS (operations per second)
│  └─ Latency
│
├─ Memory
│  ├─ RAM usage
│  ├─ Page faults
│  └─ Memory pressure
│
├─ Networking
│  ├─ Network interfaces
│  ├─ In/Out bandwidth
│  ├─ Packets in/out
│  └─ Errors/dropped
│
├─ Services
│  ├─ Service list
│  ├─ Resource usage
│  └─ Status
│
└─ Custom
   └─ Create your own dashboard
```

### Storage Monitoring

#### SSD Metrics in Netdata

```
Netdata Dashboard → Disk section

Kingston P34A60 Storage:
├─ /dev/nvme0n1p1 (nas_storage)
│  ├─ Space used: [GRAPH]
│  ├─ Space available: [GRAPH]
│  ├─ Read speed: MB/s [GRAPH]
│  ├─ Write speed: MB/s [GRAPH]
│  ├─ IOPS read: [GRAPH]
│  ├─ IOPS write: [GRAPH]
│  └─ Latency: ms [GRAPH]
│
└─ /dev/nvme0n1p2 (nas_backups)
   ├─ Space used: [GRAPH]
   ├─ Space available: [GRAPH]
   ├─ Activity graphs
   └─ Historical 7 days
```

#### Real-time vs Historical

```
Click any chart:

Real-time view:
├─ Last 1 hour in detail
├─ Updates every 1 second
└─ See live changes

Historical view:
├─ Last 7 days available
├─ Zoom in/out timeline
├─ Analyze trends
└─ Click date/time picker
```

#### Storage Alerts

```
Netdata watches for:
├─ Disk usage > 80%
├─ Disk usage > 90%
├─ Disk I/O wait > 20%
└─ Any SMART errors

Alert triggers:
└─ Email notification (configured below)
```

### CPU & Memory Monitoring

#### CPU Metrics

```
Netdata Dashboard → System → CPU usage

Shows per-core:
├─ user
├─ system  
├─ softirq
├─ irq
├─ steal
├─ guest
└─ idle

Graph:
├─ Stacked area chart
├─ Colors for each type
├─ Real-time updates
└─ 7-day history
```

#### Memory Metrics

```
Netdata Dashboard → Memory section

Shows:
├─ RAM used
├─ RAM free
├─ RAM buffers
├─ RAM cached
├─ Swap used
└─ Swap free

Graphs:
├─ Real-time usage
├─ Page faults
├─ Memory pressure
└─ 7-day trends
```

#### Temperature Monitoring

```
Netdata Dashboard → Sensors section

Shows:
├─ CPU temperature (Pi 5)
├─ SSD temperature (if SMART available)
├─ Other sensors (if present)
└─ Trend graphs

Temperature alerts:
├─ High temp warning (60°C)
├─ Critical temp alert (75°C)
└─ Email on trigger
```

### Network Monitoring

#### Network Interface Stats

```
Netdata Dashboard → Network section

Shows:
├─ eth0 (Gigabit Ethernet)
│  ├─ Received: X Mbps [GRAPH]
│  ├─ Sent: X Mbps [GRAPH]
│  ├─ Packets in: [GRAPH]
│  ├─ Packets out: [GRAPH]
│  ├─ Errors: [GRAPH]
│  └─ Dropped: [GRAPH]
│
├─ lo (loopback)
│  └─ Local system traffic
│
└─ wg0 (WireGuard VPN, if active)
   └─ VPN traffic stats
```

#### Real-time Throughput

```
Graph shows:
├─ Inbound traffic (blue)
├─ Outbound traffic (red)
├─ Updates every 1 second
└─ Peaks/valleys clearly visible

Use case:
├─ See if Jellyfin is streaming
├─ Check backup progress
├─ Monitor WireGuard VPN usage
└─ Troubleshoot network issues
```

### Configuring Netdata

#### Main Configuration File

```bash
sudo nano /etc/netdata/netdata.conf

Key settings:

[global]
  bind to = 0.0.0.0:19999        # Listen on all IPs
  update every = 1               # Update frequency (seconds)
  history = 10080                # 7 days of history
  memory mode = dbengine         # Store in database
  
[plugins:proc]
  enable = yes                   # Enable process monitoring
  
[plugins:diskspace]
  diskspace = yes                # Enable disk monitoring
  
[health]
  enabled = yes                  # Enable alerts
```

#### Enable/Disable Features

```bash
# Enable specific monitoring modules
sudo nano /etc/netdata/netdata.conf

[plugins:apps]
  enable = yes          # Process monitoring

[plugins:tc]
  tc plugin = yes       # Network traffic control

[plugins:sensors]
  enable = yes          # Temperature sensors

# Restart Netdata
sudo systemctl restart netdata
```

#### Performance Tuning

```bash
# If using too much CPU/RAM, reduce history:
sudo nano /etc/netdata/netdata.conf

[global]
  history = 1440        # Reduce from 10080 to 1 day

# Or reduce update frequency:
[global]
  update every = 2      # Update every 2 seconds instead of 1

# Restart
sudo systemctl restart netdata
```

### Email Alerts

#### Enable Email Notifications

```bash
# Configure Netdata alerts
sudo nano /etc/netdata/health_alarm_notify.conf

# Find SENDMAIL section and uncomment:
SEND_EMAIL="YES"
EMAIL_SENDER="netdata@pi.local"
RECIPIENTS_EMAIL="your-email@example.com"
```

#### Test Email

```bash
# Send test alert to verify email works:
sudo /usr/libexec/netdata/plugins.d/alarm-notify.sh test

# You should receive email with test alert
```

#### Alert Rules

```bash
# Alert configurations
sudo nano /etc/netdata/health.d/storage.conf

Example alert (disk space):
alarm: disk_full
  on: disk.space
  lookup: average -5m
  condition: $used > 0.85
  alarm: DISK SPACE WARNING
  info: SSD is 85% full
  to: email

# Restart Netdata
sudo systemctl restart netdata
```

### Historical Data

#### Data Retention

```
Netdata stores:
├─ Real-time: Last 1 hour (1-second resolution)
├─ History: Last 7 days by default
└─ All metrics available for analysis
```

#### Viewing Historical Data

```
1. Open any chart in Netdata
2. Click "Zoom Out" or timeline at bottom
3. Select date/time range to analyze
4. View historical graph
5. Download as image if needed
```

#### Exporting Data

```bash
# Export metrics as CSV/JSON
# Via API endpoint:
curl http://pi.local:19999/api/v1/data?chart=system.cpu

# Or export entire database:
sudo tar czf netdata-backup.tar.gz /var/lib/netdata/
```

---

## Firewall & Security

### Access Control

#### Who Can Access Dashboards

```
Local Network (192.168.1.0/24):
├─ Cockpit:  ✓ Allowed
├─ Netdata:  ✓ Allowed
└─ No VPN needed

VPN (WireGuard):
├─ Cockpit:  ✓ Allowed (from 100.64.0.0/10)
├─ Netdata:  ✓ Allowed
└─ Firewall rules permit VPN IPs

Public Internet:
├─ Cockpit:  ✗ Blocked
├─ Netdata:  ✗ Blocked
└─ Firewall denies all other sources
```

#### UFW Rules

```bash
# View current rules
sudo ufw status

# Should show:
9090                       ALLOW       192.168.1.0/24
19999                      ALLOW       192.168.1.0/24
9090                       ALLOW       100.64.0.0/10
19999                      ALLOW       100.64.0.0/10
```

### Remote Access via VPN

#### Setup for Remote Access

```bash
1. Connect WireGuard VPN from NYC
2. Your Pi gets IP: 10.0.0.x (or 100.64.0.x)
3. Access Cockpit: https://10.0.0.x:9090
4. Access Netdata: http://10.0.0.x:19999
```

#### VPN + Firewall Security

```
Flow:
NYC Laptop
    ↓ (WireGuard VPN)
    ↓ (Encrypted tunnel)
    ↓
Pi 5 NAS (Port 9090/19999)
    ↓ (Firewall checks)
    ↓ (VPN range allowed)
    ↓
Dashboard access granted
```

### HTTPS Configuration

#### Cockpit (Already HTTPS)

```
Cockpit enforces HTTPS automatically:
├─ Generates self-signed certificate
├─ Browser may warn "not secure" (OK)
├─ Data still encrypted
├─ Click "Advanced" → "Proceed" on first visit
```

#### Netdata (Optional HTTPS)

```bash
# By default: HTTP only
# To enable HTTPS:

sudo nano /etc/netdata/netdata.conf

[web]
  ssl certificate = /etc/ssl/certs/jellyfin-cert.pem
  ssl key = /etc/ssl/private/jellyfin-key.pem
  
sudo systemctl restart netdata

# Then access: https://pi.local:19999 instead of http
```

### Authentication

#### Cockpit Authentication

```
Required:
├─ Username: your Pi login (e.g., ubuntu)
├─ Password: your Pi password
└─ No separate auth needed

Change Pi password:
$ passwd
```

#### Netdata Authentication

```
By default: None (relies on firewall)

Optional: Add HTTP Basic Auth
sudo nano /etc/netdata/netdata.conf

[web]
  enable auth = yes
  auth realm = Netdata
  
sudo htpasswd -c /etc/netdata/.htpasswd user1
sudo systemctl restart netdata
```

---

## Daily Operations

### Daily Checks

#### Morning Checklist (2 minutes)

```
Every morning, check:

1. Cockpit Dashboard
   └─ System → Overview
   ├─ CPU usage: Normal?
   ├─ Memory: No low-memory warnings?
   └─ Disk: Still have free space?

2. Netdata Dashboard
   ├─ Storage: No alerts?
   ├─ Network: Traffic normal?
   └─ Services: All green?

3. Services Status
   └─ Cockpit → System → Services
   ├─ Jellyfin: Running?
   ├─ Samba: Running?
   └─ SSH: Running?

If all green → Good to go!
```

#### Quick Health Script

```bash
# Create daily health check
nano ~/daily-check.sh

#!/bin/bash
echo "=== Daily NAS Health Check ==="
echo ""
echo "Disk Space:"
df -h | grep nvme0n1
echo ""
echo "Memory:"
free -h | tail -2
echo ""
echo "CPU Temperature:"
sensors 2>/dev/null | grep -i temp || echo "N/A"
echo ""
echo "Critical Services:"
for service in jellyfin smbd sshd ufw fail2ban; do
  status=$(systemctl is-active $service)
  echo "$service: $status"
done

chmod +x ~/daily-check.sh

# Run daily
# Or check via Cockpit dashboard instead
```

### Weekly Tasks

#### Storage Review

```
Every Monday (or weekly):

1. Check Storage Capacity
   └─ Cockpit → System → Storage
   ├─ How full is NAS? (target: <80%)
   ├─ Jellyfin folder size
   ├─ RetroPie folder size
   └─ Backup folder size

2. Verify Backups Ran
   └─ Check backup logs
   ├─ Last backup successful?
   ├─ Backup size reasonable?
   └─ Integrity checks passed?

3. Review Netdata Trends
   └─ Check 7-day graphs
   ├─ Any unusual spikes?
   ├─ Storage filling slowly?
   └─ Performance issues?

4. Check Logs
   └─ Cockpit → System → Logs
   ├─ Any errors last 7 days?
   ├─ Failed login attempts?
   └─ Service crashes?
```

### Monthly Maintenance

#### First Friday of Month

```
1. Security Updates
   └─ Cockpit → Applications → Software Updates
   ├─ Any updates available?
   ├─ Click "Install Updates"
   └─ Reboot if kernel updates

2. Temperature & Health
   └─ Netdata → Sensors section
   ├─ CPU temp: Normal? (<60°C)
   ├─ SSD temp: OK? (<50°C)
   ├─ Any SMART warnings?
   └─ Clean cooling vents if needed

3. Audit Logs
   └─ Check security events
   ├─ Failed login attempts?
   ├─ Unauthorized access?
   └─ Configuration changes?

4. Test Recovery
   └─ Verify backups are restorable
   ├─ Extract test backup
   ├─ Verify files readable
   └─ Check integrity

5. Firewall Review
   └─ Cockpit → Networking → Firewall
   ├─ Rules still correct?
   ├─ Any orphaned rules?
   └─ Remove obsolete entries
```

### Storage Management

#### When Storage Gets Full

```
If disk > 80% full:

1. Check what's consuming space
   └─ du -sh /mnt/nas_storage/*
   ├─ Jellyfin folder size
   ├─ RetroPie ROMs size
   └─ Find largest files

2. Clean old backups
   └─ Remove backups older than 90 days
   ├─ ls -lh /mnt/nas_backups/backups/
   └─ rm old_backup.tar.gz

3. Archive old Jellyfin media
   └─ Move unwatched shows to USB drive
   ├─ Delete after confirming copy
   └─ Verify backup before delete

4. Increase storage
   └─ Add external USB backup drive
   ├─ Or upgrade to larger SSD
   └─ See enhanced NAS guide Phase 9

5. Monitor going forward
   └─ Set Netdata alert for 85% full
   ├─ Get email warning early
   └─ Proactively manage space
```

---

## Troubleshooting

### Services Not Starting

#### Cockpit Won't Start

```bash
# Check status
sudo systemctl status cockpit.socket
sudo systemctl status cockpit.service

# View error logs
sudo journalctl -u cockpit.socket -n 50

# Restart
sudo systemctl restart cockpit.socket

# If still fails:
sudo systemctl daemon-reload
sudo systemctl restart cockpit.socket
```

#### Netdata Won't Start

```bash
# Check status
sudo systemctl status netdata

# View logs
sudo journalctl -u netdata -n 50

# Restart
sudo systemctl restart netdata

# Check configuration
sudo netdata -D -c /etc/netdata/netdata.conf

# If config error, fix and restart
```

### Cannot Access Dashboards

#### Cockpit Not Accessible

```bash
# Check if listening
sudo lsof -i :9090
# Should show: cockpit (LISTEN)

# If not listening:
sudo systemctl restart cockpit.socket

# Check firewall
sudo ufw status | grep 9090
# Should show: ALLOW 192.168.1.0/24

# Test from another device
ping pi.local
curl -k https://pi.local:9090

# If DNS fails
# Try IP: https://192.168.1.x:9090
```

#### Netdata Not Accessible

```bash
# Check if listening
sudo lsof -i :19999
# Should show: netdata (LISTEN)

# If not listening:
sudo systemctl restart netdata

# Check firewall
sudo ufw status | grep 19999
# Should show: ALLOW 192.168.1.0/24

# Test from another device
curl http://pi.local:19999

# If DNS fails
# Try IP: http://192.168.1.x:19999
```

### Performance Issues

#### High CPU Usage

```bash
# Check what's consuming CPU
# Via Netdata → System → CPU

# See process list
# Via Netdata → Processes

# If Netdata using too much:
# Reduce update frequency
sudo nano /etc/netdata/netdata.conf
# Change: update every = 2 (from 1)
sudo systemctl restart netdata

# If Cockpit using too much:
sudo systemctl restart cockpit.socket
```

#### High Memory Usage

```bash
# Check memory via Netdata
# Netdata → Memory section

# See largest processes
# Via Netdata → Processes → Sorted by memory

# If needed, reduce Netdata history:
sudo nano /etc/netdata/netdata.conf
# Change: history = 1440 (from 10080)
sudo systemctl restart netdata

# Free up RAM
sudo sync
sudo echo 3 > /proc/sys/vm/drop_caches
```

### Alerts Not Working

#### Email Alerts Not Sending

```bash
# Check email configuration
sudo nano /etc/netdata/health_alarm_notify.conf
# Verify SEND_EMAIL="YES"
# Verify RECIPIENTS_EMAIL is set

# Test email
sudo /usr/libexec/netdata/plugins.d/alarm-notify.sh test

# If no email received:
# 1. Check mail service running
sudo systemctl status postfix

# 2. Check mail logs
sudo tail -f /var/log/mail.log

# 3. Verify SMTP settings
# For Gmail: Use app-specific password, not Gmail password

# 4. Restart Netdata
sudo systemctl restart netdata
```

#### Alerts Not Triggering

```bash
# Check health status
sudo netdata -D | grep -i health

# View alert logs
sudo tail -f /var/log/netdata/health-alarm-notify.log

# Check alert rules
cat /etc/netdata/health.d/*

# Manual test
# Trigger artificially high load
# stress-test CPU
# Watch Netdata for alert
```

---

## Integration with NAS Guide

### Where This Fits

#### In Overall Deployment

```
Week 1-4: Complete Phases 1-9 of Enhanced NAS guide
          (OS, encryption, services, monitoring scripts)

Week 5: ← YOU ARE HERE
├─ Install Cockpit (5 min)
├─ Install Netdata (5 min)
└─ Configure firewall (5 min)

Result:
├─ Existing security layers: Still intact
├─ New monitoring: Visual dashboards
├─ New management: GUI-based administration
└─ No conflicts with Phase 7-9 monitoring
```

#### How Cockpit + Netdata Enhance Current Setup

```
Current Phase 7-9 provides:
├─ Command-line monitoring scripts
├─ Automated security checks
├─ Email alerts
└─ Log analysis

Cockpit + Netdata add:
├─ Visual hardware dashboard
├─ Real-time graphs (updated every 1 sec)
├─ Service management GUI
├─ Storage monitoring GUI
├─ Firewall management GUI
├─ Historical trends (7 days)
└─ No conflicts - only enhancements
```

### Dependencies

```
Cockpit + Netdata require:

From NAS Setup:
✓ Ubuntu 24.04 ARM64 (already installed)
✓ LUKS encryption (already running)
✓ UFW firewall (already configured)
✓ SSH access (already configured)
✓ WireGuard VPN (optional, for remote access)

New requirements:
✓ 50MB additional RAM (out of 8GB = no issue)
✓ 200MB disk space (out of 2TB = no issue)
✓ Ports 9090, 19999 (open via firewall)
└─ No other dependencies

Compatible with all existing services:
✓ Jellyfin
✓ Samba/SMB
✓ RetroPie
✓ AIDE/auditd/Tripwire
✓ Fail2ban
└─ No conflicts or interference
```

### Next Steps

#### Immediate (After Installing)

```
1. Login to Cockpit
   └─ https://pi.local:9090
   ├─ Explore System → Overview
   ├─ Check System → Storage
   └─ View System → Services

2. Login to Netdata
   └─ http://pi.local:19999
   ├─ Review Overview
   ├─ Check historical data
   └─ Familiarize with graphs

3. Configure firewall
   └─ Set UFW rules (from Step 7 above)
   ├─ Restrict to LAN
   ├─ Add VPN access if desired
   └─ Verify rules applied
```

#### Short Term (This Week)

```
1. Setup email alerts in Netdata
   └─ Configure storage > 85% warning
   ├─ Test alert delivery
   └─ Adjust thresholds as needed

2. Create bookmarks
   ├─ Cockpit: https://pi.local:9090
   ├─ Netdata: http://pi.local:19999
   └─ Quick access to dashboards

3. Daily monitoring routine
   └─ Check Cockpit Overview (2 min)
   └─ Scan Netdata alerts (1 min)
   └─ Total: 3 minutes daily
```

#### Long Term (Integration)

```
1. Combine with existing Phase 7-9 monitoring
   └─ CLI scripts run automatically
   └─ Dashboards for visual review
   └─ Email alerts for critical issues

2. Monitor trends
   └─ Weekly storage review
   └─ Monthly health check
   └─ Plan upgrades based on trends

3. Maintain dashboards
   └─ Monthly firewall review
   └─ Security updates
   └─ Optimize performance

Result:
├─ Comprehensive monitoring (automated + visual)
├─ Multiple access methods (CLI + web)
├─ Enterprise-grade visibility
└─ Professional NAS operation
```

---

## Quick Reference

### Useful Commands

#### Status Commands

```bash
# Check both services
sudo systemctl status cockpit.socket netdata

# Check listening ports
sudo lsof -i :9090
sudo lsof -i :19999

# Check firewall rules
sudo ufw status

# Check system logs
sudo journalctl -xe

# Check Netdata status
netdata -v
```

#### Management Commands

```bash
# Restart services
sudo systemctl restart cockpit.socket
sudo systemctl restart netdata

# Enable on boot
sudo systemctl enable cockpit.socket netdata

# Disable (don't start on boot)
sudo systemctl disable cockpit.socket netdata

# Stop services
sudo systemctl stop cockpit.socket
sudo systemctl stop netdata

# Start services
sudo systemctl start cockpit.socket
sudo systemctl start netdata
```

#### Firewall Commands

```bash
# Allow from LAN
sudo ufw allow from 192.168.1.0/24 to any port 9090
sudo ufw allow from 192.168.1.0/24 to any port 19999

# Allow from VPN
sudo ufw allow from 100.64.0.0/10 to any port 9090
sudo ufw allow from 100.64.0.0/10 to any port 19999

# View rules
sudo ufw status

# Remove rule
sudo ufw delete allow from 192.168.1.0/24 to any port 9090
```

#### Troubleshooting Commands

```bash
# View Cockpit logs
sudo journalctl -u cockpit.socket -n 50
sudo journalctl -u cockpit.service -n 50

# View Netdata logs
sudo journalctl -u netdata -n 50

# View system logs
sudo tail -f /var/log/syslog

# Check configuration
sudo cat /etc/netdata/netdata.conf | grep -v '^#' | grep -v '^$'

# Test Cockpit
curl -k https://127.0.0.1:9090 > /dev/null && echo "OK"

# Test Netdata
curl http://127.0.0.1:19999 > /dev/null && echo "OK"
```

### Dashboard URLs

#### Local Network

```
Cockpit:
https://pi.local:9090

Netdata:
http://pi.local:19999

Or use IP:
Cockpit:
https://192.168.1.x:9090

Netdata:
http://192.168.1.x:19999
```

#### Remote (VPN)

```
After connecting WireGuard from NYC:

Cockpit:
https://10.0.0.x:9090  (or 100.64.0.x)

Netdata:
http://10.0.0.x:19999  (or 100.64.0.x)
```

#### Login Credentials

```
Cockpit:
├─ Username: your Pi username
├─ Password: your Pi password
└─ Saved in browser (optional)

Netdata:
└─ No login required (firewall protected)
```

### Port Information

```
Cockpit:
├─ Port: 9090
├─ Protocol: HTTPS (enforced)
├─ Access: LAN + VPN only
└─ Process: cockpit

Netdata:
├─ Port: 19999
├─ Protocol: HTTP (or HTTPS optional)
├─ Access: LAN + VPN only
└─ Process: netdata
```

### Resource Usage

```
RAM Usage:
├─ Cockpit: ~20MB
├─ Netdata: ~30MB
├─ Total: ~50MB
└─ Your 8GB: Still 7.95GB free

CPU Usage:
├─ Cockpit: <1% when idle
├─ Netdata: <1% normal, 2-3% on graph updates
└─ Negligible impact

Disk Usage:
├─ Cockpit: ~150MB
├─ Netdata: ~100MB + 7-day history (~500MB)
└─ Total: ~750MB (out of 2TB = 0.03%)
```

---

## Summary

### What You Have Now

```
✅ Cockpit for system administration
   ├─ Storage monitoring
   ├─ Service management
   ├─ Firewall GUI
   ├─ User management
   └─ Terminal access

✅ Netdata for real-time monitoring
   ├─ CPU, RAM, disk graphs
   ├─ Network monitoring
   ├─ Service health
   ├─ Email alerts
   └─ 7-day historical data

✅ Both integrated with NAS security
   ├─ Firewall protected
   ├─ Authentication required (Cockpit)
   ├─ VPN accessible
   └─ Production-ready
```

### Time Investment

```
Installation: 15 minutes
├─ 5 minutes: Install packages
├─ 5 minutes: Configure firewall
└─ 5 minutes: Verify and test

Daily use: 3-5 minutes
├─ Morning check (2 min)
├─ Evening scan (2 min)
└─ Alert review (1 min)

Weekly tasks: 10-15 minutes
└─ Storage review
└─ Service health
└─ Log analysis

Monthly maintenance: 30 minutes
└─ Updates
└─ Health checks
└─ Recovery testing
```

### Recommendation

**Start today with the quick install commands (5 min).** Explore the dashboards while waiting for Phase 1-9 of the NAS setup to complete. By Week 5, you'll be a Cockpit + Netdata expert!

---

**Status:** ✅ Production-Ready  
**Time to Deploy:** 15 minutes  
**Difficulty:** Easy  
**Resource Impact:** Minimal  
**Value:** Excellent - Professional NAS monitoring

Ready to get started? Run the TL;DR commands above! 🚀
