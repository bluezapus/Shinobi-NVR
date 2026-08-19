# Storage

## Overview

The CCTV infrastructure uses dedicated physical HDD storage for Shinobi
recordings. The recording storage is separated from the Proxmox system
disk and exposed to the Shinobi LXC through a container mount.

This design prevents CCTV recordings from consuming the Proxmox operating
system volume and provides a high-capacity storage target dedicated to
video data.

> Production-specific information is sanitized where applicable.
> Filesystem identifiers such as UUIDs are included only where useful for
> infrastructure administration and should be removed if this document is
> published publicly.

---

## Storage Architecture

The storage path is:

```text
Physical CCTV HDD
      │
      ▼
WDC WD40PURX-64NZ6Y0
      │
      ▼
/dev/sdc
      │
      ▼
/dev/sdc1
      │
      │ ext4
      ▼
Proxmox Host
/mnt/hddcctv
      │
      │ LXC Mount Point
      ▼
Shinobi LXC CT 100
/mnt/storage
      │
      ▼
Shinobi NVR
      │
      ▼
CCTV Recordings
```

The physical HDD is mounted by the Proxmox host and passed into the
Shinobi container as a mount point.

---

## Physical Storage Devices

The Proxmox host currently contains three primary physical storage
devices.

| Device | Capacity | Model | Filesystem | Mount Point | Purpose |
|---|---:|---|---|---|---|
| `/dev/sda` | `119.2 GB` | HONGTAI SSD SATA 2.5 H500 128GB | LVM2 | — | Proxmox system and VM/LXC storage |
| `/dev/sdb` | `1.8 TB` | TOSHIBA HDWU120 | NTFS | `/mnt/hdd-share` | General/shared storage |
| `/dev/sdc` | `3.6 TB` | WDC WD40PURX-64NZ6Y0 | ext4 | `/mnt/hddcctv` | CCTV recording storage |

The CCTV recording workload uses `/dev/sdc1`.

---

## CCTV Storage Device

The dedicated CCTV disk is:

```text
Device:       /dev/sdc
Partition:    /dev/sdc1
Model:        WDC WD40PURX-64NZ6Y0
Capacity:     3.6 TB
Filesystem:   ext4
Mount Point:  /mnt/hddcctv
```

The disk is physically separated from the SSD used for the Proxmox
system and virtualized workloads.

This provides a clear separation between:

```text
System / Virtualization Storage
            │
            ▼
         /dev/sda
            │
            ▼
        local-lvm


CCTV Recording Storage
            │
            ▼
         /dev/sdc
            │
            ▼
       /mnt/hddcctv
```

---

## Storage Capacity

Current storage utilization on the Proxmox host:

| Metric | Value |
|---|---:|
| Capacity | `3.6 TB` |
| Used | `486 GB` |
| Available | `3.0 TB` |
| Utilization | `14%` |
| Filesystem | `ext4` |

The same storage is visible inside the Shinobi container as the
`/mnt/storage` mount.

---

## Filesystem

The CCTV recording partition uses `ext4`.

```text
Device:      /dev/sdc1
Filesystem:  ext4
Mount:       /mnt/hddcctv
```

Current mount information:

```text
/mnt/hddcctv /dev/sdc1 ext4 rw,relatime
```

The filesystem is mounted read/write.

---

## Persistent Mount Configuration

The CCTV disk is configured in `/etc/fstab` using its filesystem UUID.

Sanitized configuration:

```fstab
UUID=6a9f537a-db44-4c58-aa7d-3a65eb21e31f \
/mnt/hddcctv ext4 defaults,nofail,x-systemd.device-timeout=10 0 2
```

### Mount Options

| Option | Purpose |
|---|---|
| `defaults` | Standard filesystem mount options |
| `nofail` | Allows boot to continue if the disk is unavailable |
| `x-systemd.device-timeout=10` | Limits systemd device wait time |
| `0` | Filesystem dump disabled |
| `2` | Filesystem checked after the root filesystem |

The `nofail` option is particularly useful for preventing a missing
secondary storage device from blocking the entire Proxmox host during
boot.

---

## Proxmox Storage Mount

The Proxmox host exposes the physical CCTV disk at:

```text
/mnt/hddcctv
```

Verification:

```bash
findmnt /mnt/hddcctv
```

Expected structure:

```text
TARGET          SOURCE     FSTYPE
/mnt/hddcctv    /dev/sdc1  ext4
```

Storage capacity can be checked with:

```bash
df -h /mnt/hddcctv
```

---

## Shinobi LXC Storage Mount

The physical CCTV storage is passed into Shinobi LXC `100` using an LXC
mount point.

Relevant configuration:

```text
mp0: /mnt/hddcctv,mp=/mnt/storage
```

This creates the following relationship:

```text
Proxmox Host
/mnt/hddcctv
      │
      │ mp0
      ▼
Shinobi LXC
/mnt/storage
```

The Shinobi application therefore accesses the physical HDD through:

```text
/mnt/storage
```

while the Proxmox host manages the physical filesystem through:

```text
/mnt/hddcctv
```

---

## LXC Storage Configuration

Current LXC `100` configuration:

| Parameter | Configuration |
|---|---|
| Container ID | `100` |
| Hostname | `shinobi` |
| Root Storage | `local-lvm` |
| Root Disk | `64 GB` |
| CCTV Mount | `mp0` |
| Host Path | `/mnt/hddcctv` |
| Container Path | `/mnt/storage` |
| Container Type | Unprivileged LXC |
| Storage Filesystem | ext4 |

Relevant configuration:

```text
rootfs: local-lvm:vm-100-disk-0,size=64G
mp0: /mnt/hddcctv,mp=/mnt/storage
unprivileged: 1
```

---

## Root Disk vs CCTV Storage

The Shinobi LXC uses two logically separate storage areas.

```text
                 Shinobi LXC
                      │
          ┌───────────┴───────────┐
          │                       │
      Root Disk              CCTV Storage
      64 GB                   3.6 TB
          │                       │
      local-lvm               Physical HDD
          │                       │
          ▼                       ▼
       Ubuntu                 /mnt/storage
       Shinobi                     │
                                   ▼
                              Recordings
```

### Root Disk

The root disk contains:

- Ubuntu operating system
- Shinobi application
- Node.js runtime
- npm packages
- MariaDB
- Configuration files
- Application logs
- System files

The root disk is backed by the Proxmox `local-lvm` storage.

### CCTV Storage

The dedicated HDD contains:

- CCTV recordings
- Video files generated by Shinobi
- Other recording-related data configured to use the CCTV storage

The recording path is:

```text
/mnt/storage
```

---

## Why Dedicated CCTV Storage Is Used

Continuous CCTV recording generates significantly more data than a
typical application workload.

Storing recordings on the LXC root disk would create several risks:

- Root filesystem capacity could be exhausted.
- Shinobi and Ubuntu could become unstable when disk space is low.
- Proxmox system storage could be consumed by video data.
- Application and recording workloads would compete for the same storage.
- Storage expansion would be less flexible.

The current architecture separates these workloads:

```text
System Workload
      │
      ▼
SSD / local-lvm
      │
      ▼
Shinobi Root Disk


CCTV Workload
      │
      ▼
Dedicated HDD
      │
      ▼
/mnt/storage
```

---

## Storage Flow

The complete recording storage flow is:

```text
IP Camera
    │
    │ RTSP
    ▼
Shinobi
    │
    ▼
FFmpeg
    │
    ▼
Recording Process
    │
    ▼
/mnt/storage
    │
    ▼
LXC Mount
    │
    ▼
/mnt/hddcctv
    │
    ▼
/dev/sdc1
    │
    ▼
WDC 3.6 TB HDD
```

---

## Storage Utilization

Current utilization:

```text
Total:      3.6 TB
Used:       486 GB
Available:  3.0 TB
Usage:      14%
```

Visualization:

```text
CCTV HDD
┌──────────────────────────────────────────────┐
│██████                                        │
│ Used ~14%                                     │
│                                               │
│ Available ~86%                                │
└──────────────────────────────────────────────┘
```

At the documented point in time, approximately 86% of the CCTV storage
remains available.

Storage utilization should be monitored because continuous recording
causes disk consumption to increase over time.

---

## Recording Capacity

Actual recording consumption depends primarily on:

- Number of cameras
- Video bitrate
- Resolution
- Frame rate
- Codec
- Audio
- Recording mode
- Motion/event activity
- Retention policy

A simplified estimate is:

```text
Daily Storage (GB)
≈ Number of Cameras
× Average Bitrate (Mbps)
× 10.8
```

For example, using a hypothetical average bitrate of 2 Mbps:

```text
16 × 2 × 10.8
≈ 345.6 GB/day
```

This is an estimation only. Actual consumption must be calculated from
the real Shinobi/camera configuration.

---

## Retention Planning

The available storage should be evaluated against the desired CCTV
retention period.

A simplified retention estimate is:

```text
Retention Days
≈ Available Storage
  / Daily Recording Consumption
```

For accurate planning, calculate the actual bitrate for each camera
rather than assuming that all cameras use the same bitrate.

The current project documentation does not specify a final retention
period. This value should be added after the Shinobi retention policy
has been verified.

---

## Storage Monitoring

The primary storage monitoring command is:

```bash
df -h /mnt/hddcctv
```

Inside the Shinobi LXC:

```bash
df -h /mnt/storage
```

Expected relationship:

```text
Proxmox:
    /mnt/hddcctv
        │
        ▼
    Same physical storage
        │
        ▼
Shinobi:
    /mnt/storage
```

Both paths should report approximately the same total capacity and
utilization.

---

## Disk Identification

The physical storage can be inspected using:

```bash
lsblk -o NAME,SIZE,MODEL,TYPE,FSTYPE,MOUNTPOINT
```

Current CCTV storage:

```text
sdc
└─sdc1
   ├─ Size:       3.6T
   ├─ Model:      WDC WD40PURX-64NZ6Y0
   ├─ Type:       part
   ├─ Filesystem: ext4
   └─ Mount:      /mnt/hddcctv
```

---

## Mount Verification

Verify the active mount:

```bash
findmnt /mnt/hddcctv
```

Verify the filesystem:

```bash
df -Th /mnt/hddcctv
```

Verify the LXC mount:

```bash
pct config 100
```

Expected LXC mount configuration:

```text
mp0: /mnt/hddcctv,mp=/mnt/storage
```

---

## Storage Health

The current documentation confirms filesystem and capacity information,
but it does not include a SMART health report.

For production maintenance, the physical HDD should be monitored using
SMART data where supported.

Example:

```bash
smartctl -a /dev/sdc
```

Important values to monitor include:

- Overall SMART health
- Reallocated sectors
- Pending sectors
- Uncorrectable sectors
- Temperature
- Power-on hours
- SMART error log

> SMART monitoring is not currently documented as an active monitoring
> service in this project. The command above is provided as a maintenance
> verification procedure.

---

## Storage Failure Considerations

The CCTV recording disk is a dedicated single HDD.

If `/dev/sdc` or `/dev/sdc1` becomes unavailable:

```text
Physical HDD Failure
        │
        ▼
/mnt/hddcctv unavailable
        │
        ▼
/mnt/storage unavailable
        │
        ▼
Shinobi cannot write recordings
```

The `nofail` mount option prevents a missing disk from unnecessarily
blocking the Proxmox host boot process, but it does not provide data
redundancy.

The current architecture therefore does not provide RAID redundancy for
the CCTV recording disk.

---

## Backup Considerations

CCTV recordings are generally large and continuously generated.

A backup strategy should distinguish between:

### Application Configuration

Should be backed up:

- Shinobi configuration
- Shinobi database
- Monitor configuration
- Camera configuration
- User configuration
- System configuration
- Proxmox LXC configuration

### CCTV Recordings

The recording dataset is significantly larger and may require a
separate retention and backup strategy.

The current documentation does not claim that `/mnt/hddcctv` is backed
up to another storage device.

---

## Other Storage

The Proxmox host also contains a separate Toshiba HDD mounted at:

```text
/mnt/hdd-share
```

Current state:

| Parameter | Value |
|---|---:|
| Device | `/dev/sdb1` |
| Model | `TOSHIBA HDWU120` |
| Filesystem | `ntfs` |
| Capacity | `1.9 TB` |
| Used | `1.8 TB` |
| Available | `108 GB` |
| Utilization | `95%` |
| Mount | `/mnt/hdd-share` |

This storage is **not used as the primary Shinobi CCTV recording
storage**.

Its high utilization should be monitored independently.

---

## Proxmox System Storage

The Proxmox system uses the SSD:

```text
/dev/sda
```

The SSD contains the Proxmox LVM layout, including:

```text
pve-root
pve-swap
pve-data
vm-100-disk-0
vm-200-disk-0
vm-300-disk-0
vm-400-disk-0
```

The Shinobi LXC root disk is:

```text
local-lvm:vm-100-disk-0
```

with a configured size of:

```text
64 GB
```

This disk is separate from the physical CCTV recording HDD.

---

## Current Storage Layout

```text
                         Proxmox VE
                             │
             ┌───────────────┴────────────────┐
             │                                │
          SSD /dev/sda                    HDD /dev/sdc
             │                                │
             │                                │
         local-lvm                        /dev/sdc1
             │                                │
             │                            ext4 3.6 TB
             │                                │
             ▼                                ▼
       LXC Root Disk                    /mnt/hddcctv
          64 GB                                │
             │                                │
             ▼                                │
        Shinobi LXC                            │
             │                                 │
             └──────────────┐                  │
                            │                  │
                            ▼                  │
                       /mnt/storage ◄──────────┘
                            │
                            ▼
                     CCTV Recordings
```

---

## Verification Commands

### Identify Disks

```bash
lsblk -o NAME,SIZE,MODEL,TYPE,FSTYPE,MOUNTPOINT
```

### Check Filesystem Usage

```bash
df -h
```

### Check Persistent Mount Configuration

```bash
cat /etc/fstab
```

### Check Shinobi LXC Configuration

```bash
pct config 100
```

![](images/screenshots/storage-pve.png)
### Check Storage Inside Shinobi

Run inside CT `100`:

```bash
df -h /mnt/storage
```
![Keterangan](storage-pct100.png)
### Check Disk Health

If `smartmontools` is installed:

```bash
smartctl -a /dev/sdc1
```

---

## Configuration Summary

| Component | Configuration |
|---|---|
| CCTV Disk | `/dev/sdc` |
| CCTV Partition | `/dev/sdc1` |
| Disk Model | `WDC WD40PURX-64NZ6Y0` |
| Capacity | `3.6 TB` |
| Filesystem | `ext4` |
| Proxmox Mount | `/mnt/hddcctv` |
| Shinobi Mount | `/mnt/storage` |
| LXC | `CT 100` |
| LXC Root Disk | `64 GB` |
| Root Storage | `local-lvm` |
| Current Used | `486 GB` |
| Current Available | `3.0 TB` |
| Current Usage | `14%` |
| Mount Option | `defaults,nofail,x-systemd.device-timeout=10` |
| RAID | Not configured |
| CCTV Backup | Not documented |
| Retention Policy | To be verified |

---

## Notes

The CCTV recording storage is intentionally separated from the Shinobi
LXC root disk.

The current design provides approximately 3.6 TB of dedicated storage
for CCTV recordings and exposes it to Shinobi through an LXC mount.

The storage configuration is persistent through `/etc/fstab` on the
Proxmox host and the LXC mount definition in container `100`.

Retention duration, actual per-camera bitrate, SMART monitoring, and
backup policy should be documented separately after verification of the
running production configuration.