# NVMe Partition & OS Migration Guide
**Pi 5 — Ubuntu Server 24.04**

---

## Target Layout for 238.5GB NVMe

| Partition | Size | Format | Mount | Purpose |
|---|---|---|---|---|
| nvme0n1p1 | 512MB | FAT32 | `/boot/firmware` | Bootloader, kernel |
| nvme0n1p2 | 50GB | ext4 | `/` | OS, Docker, apps |
| nvme0n1p3 | ~188GB | ext4 | `/mnt/data` | Media, NAS, backups |

---

## Step 1: Wipe and Partition NVMe

```bash
sudo umount /mnt/nvme 2>/dev/null
sudo wipefs -a /dev/nvme0n1
sudo parted /dev/nvme0n1 mklabel gpt
sudo parted /dev/nvme0n1 mkpart primary fat32 1MiB 513MiB
sudo parted /dev/nvme0n1 set 1 boot on
sudo parted /dev/nvme0n1 mkpart primary ext4 513MiB 51713MiB
sudo parted /dev/nvme0n1 mkpart primary ext4 51713MiB 100%
lsblk /dev/nvme0n1
```

---

## Step 2: Format All Partitions

```bash
sudo mkfs.vfat -F 32 /dev/nvme0n1p1
sudo mkfs.ext4 -L os-root /dev/nvme0n1p2
sudo mkfs.ext4 -L data /dev/nvme0n1p3
```

---

## Step 3: Mount NVMe Partitions

```bash
sudo mkdir -p /mnt/nvme-boot /mnt/nvme-root /mnt/nvme-data
sudo mount /dev/nvme0n1p1 /mnt/nvme-boot
sudo mount /dev/nvme0n1p2 /mnt/nvme-root
sudo mount /dev/nvme0n1p3 /mnt/nvme-data
```

---

## Step 4: Clone microSD → NVMe

```bash
sudo rsync -axHAXv --progress / /mnt/nvme-root/ \
  --exclude=/mnt \
  --exclude=/proc \
  --exclude=/sys \
  --exclude=/dev \
  --exclude=/tmp \
  --exclude=/run \
  --exclude=/media \
  --exclude=/lost+found

sudo rsync -axHAXv --progress /boot/firmware/ /mnt/nvme-boot/
```

---

## Step 5: Get NVMe UUIDs

```bash
sudo blkid /dev/nvme0n1p1
sudo blkid /dev/nvme0n1p2
sudo blkid /dev/nvme0n1p3
```

Copy all three UUIDs — needed for Steps 6 and 7.

---

## Step 6: Update fstab on NVMe Root

```bash
sudo nano /mnt/nvme-root/etc/fstab
```

Replace entire contents with:
```
UUID=<nvme0n1p1-uuid>  /boot/firmware  vfat  defaults          0  2
UUID=<nvme0n1p2-uuid>  /               ext4  defaults,relatime  0  1
UUID=<nvme0n1p3-uuid>  /mnt/data       ext4  defaults,noatime   0  2
```

---

## Step 7: Update cmdline.txt on NVMe Boot

```bash
sudo nano /mnt/nvme-boot/cmdline.txt
```

Replace `root=` value with nvme0n1p2 UUID — keep as a **single line**:
```
console=serial0,115200 console=tty1 root=UUID=<nvme0n1p2-uuid> rootfstype=ext4 fsck.repair=yes rootwait quiet
```

---

## Step 8: Create Data Directory Structure

```bash
sudo mkdir -p /mnt/nvme-data/{media,backups,downloads,docker-volumes}
sudo mkdir -p /mnt/nvme-data/media/{movies,tvshows,music,photos}
sudo chown -R pi:pi /mnt/nvme-data
```

---

## Step 9: Set NVMe Boot Order in EEPROM

```bash
sudo rpi-eeprom-update -a    # update if needed, reboot and return
sudo -E rpi-eeprom-config --edit
```

Set:
```
BOOT_ORDER=0x16
```
`6` = NVMe first, `1` = microSD fallback. Save and exit.

```bash
sudo reboot
```

---

## Step 10: Verify After Reboot

```bash
findmnt /          # should show /dev/nvme0n1p2
lsblk              # confirm all partitions mounted
df -h              # confirm sizes
```

Expected:
```
/dev/nvme0n1p2   50G   ~4G   46G   /
/dev/nvme0n1p1  512M  ~50M  462M   /boot/firmware
/dev/nvme0n1p3  188G    0G  188G   /mnt/data
```

---

## Step 11: Remove microSD

Once stable across several reboots:
```bash
sudo poweroff
```
Remove microSD, power on — Pi boots cleanly from NVMe.

---

## Troubleshooting

Reinsert microSD if Pi fails to boot — `BOOT_ORDER=0x16` falls back automatically.

| Symptom | Fix |
|---|---|
| Boots to microSD instead of NVMe | Recheck EEPROM `BOOT_ORDER` (Step 9) |
| Kernel panic / wrong root | Wrong UUID in `cmdline.txt` (Step 7) |
| Mount errors on boot | Wrong UUID in `fstab` (Step 6) |
| `/mnt/data` missing after boot | fstab entry missing nvme0n1p3 (Step 6) |
| rsync errors | Rerun Step 4, check disk space with `df -h` |
