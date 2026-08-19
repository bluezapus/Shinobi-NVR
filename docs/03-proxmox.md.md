# Proxmox Infrastructure

## Overview

The CCTV NVR infrastructure is hosted on a Proxmox VE hypervisor.
Shinobi NVR runs inside an unprivileged LXC container and is connected
to the production network through the Proxmox `vmbr0` bridge.

The Proxmox host provides compute, virtualization, networking, and
storage resources required by the Shinobi NVR workload.

> Production IP addresses and other company-specific information have
> been sanitized for public portfolio documentation.

---

## Proxmox Host

### Host Configuration

| Parameter | Value |
|---|---|
| Hypervisor | Proxmox VE `9.2.4` |
| Node | `pve` |
| CPU | 8 vCPU / logical CPUs |
| RAM | ~13.57 GB |
| Storage | ~3.67 TB |

The host provides the virtualization layer for the Shinobi NVR and
other LXC workloads.

![Keterangan](node-summary.png)

---

## Shinobi LXC

Shinobi NVR is deployed as an unprivileged LXC container on the
Proxmox host.

### Container Configuration

| Parameter | Value |
|---|---|
| Type | LXC Container |
| Container ID | `100` |
| Name | `Shinobi` |
| Guest OS | Ubuntu |
| Architecture | `amd64` |
| Privileged | No |
| vCPU | 4 |
| RAM | 8 GB |
| SWAP | 512 MB |
| Root Disk | 62.44 GB |
| Storage Backend | `local-lvm` |

### Container Architecture

```text
Proxmox VE
│
└── LXC Container 100
    │
    ├── Ubuntu
    ├── Shinobi NVR
    ├── 4 vCPU
    ├── 8 GB RAM
    ├── 512 MB SWAP
    └── 62.44 GB Root Disk
```

---
## Shinobi Network Configuration

The Shinobi container is connected to the Proxmox bridge `vmbr0`
through the `eth0` interface.

| Parameter | Value |
|---|---|
| Interface | `eth0` |
| Bridge | `vmbr0` |
| Firewall | Enabled |
| VLAN | Not configured |
| IPv4 | `192.168.10.221/24` |
| IPv6 | Dynamic |
| Address Assignment | DHCP |

> The production IPv4 address has been sanitized.

### Network Path

```text
Physical Network
       │
     nic0
       │
     vmbr0
       │
       ▼
LXC Container 100
       │
      eth0
       │
       ▼
  Shinobi NVR
```

---

## CCTV Storage

Dedicated storage is used for CCTV recordings.

The physical CCTV storage is mounted on the Proxmox host and exposed
to the Shinobi LXC through a container mount point.

### Storage Configuration

| Parameter | Value |
|---|---|
| Proxmox Mount | `/mnt/hddcctv` |
| Container Mount | `/mnt/storage` |
| Capacity | 3.6 TB |
| Filesystem | ext4 |
| Used | ~485 GB |
| Storage Backend | `hddcctv` |

### Storage Path

```text
Physical HDD
     │
     ▼
Proxmox Host
/mnt/hddcctv
     │
     │ Container Mount
     ▼
Shinobi LXC
/mnt/storage
     │
     ▼
Shinobi Recording Storage
```

The storage is separated from the LXC root disk so that CCTV video
recordings are stored on the dedicated high-capacity HDD rather than
the Proxmox system volume.

![Keterangan](images/screenshots/shinobi-storage.png)

---

## Resource Allocation

The Shinobi container is allocated dedicated compute and storage
resources from the Proxmox host.

| Resource | Allocation |
|---|---|
| CPU | 4 vCPU |
| Memory | 8 GB |
| SWAP | 512 MB |
| Root Disk | 62.44 GB |
| CCTV Storage | 3.6 TB |

This resource separation allows the operating system and application
environment to remain on the LXC root disk while CCTV recordings are
stored on dedicated storage.

---

## Virtualization Layout

```text
                    Proxmox VE 9.2.4
                         │
          ┌──────────────┴──────────────┐
          │                             │
      System Storage               CCTV Storage
       local-lvm                    HDD 3.6 TB
          │                             │
          ▼                             ▼
     LXC Container 100             /mnt/hddcctv
          │                             │
          │                         Container
          │                           Mount
          │                             │
          └──────────────┬──────────────┘
                         │
                    Shinobi NVR
                         │
                    /mnt/storage
```

---

## Security Configuration

The Shinobi container is configured as an **unprivileged LXC**.

| Security Parameter | Configuration |
|---|---|
| LXC Type | Unprivileged |
| Proxmox Firewall | Enabled |
| VLAN | Not configured |

Using an unprivileged container reduces the container's privileges
relative to the Proxmox host.

---

## Configuration Summary

| Component | Configuration |
|---|---|
| Hypervisor | Proxmox VE `9.2.4` |
| Node | `pve` |
| Shinobi Container | LXC `100` |
| Guest OS | Ubuntu |
| Architecture | `amd64` |
| CPU Allocation | 4 vCPU |
| Memory Allocation | 8 GB |
| SWAP | 512 MB |
| Root Disk | 62.44 GB |
| Root Storage | `local-lvm` |
| Network Bridge | `vmbr0` |
| Network Interface | `eth0` |
| VLAN | Not configured |
| Firewall | Enabled |
| CCTV Storage | 3.6 TB |
| CCTV Filesystem | ext4 |
| Host Mount | `/mnt/hddcctv` |
| Container Mount | `/mnt/storage` |

---

## Verification Commands

### Proxmox Version

```bash
pveversion
```

### CPU

```bash
lscpu
```

### Memory

```bash
free -h
```

### Storage

```bash
lsblk -o NAME,SIZE,MODEL,TYPE,FSTYPE,MOUNTPOINT
```

![Keterangan](images/screenshots/pvever-free-mount.png)
### Network

```bash
ip -br addr
```

```bash
ip route
```

### LXC Configuration

```bash
pct config 100
```

![](images/screenshots/shinobi-config.png)
### Storage Mount

```bash
df -h /mnt/hddcctv
```
![](images/screenshots/hddcctv.png)
### Container Mount

Run inside the Shinobi container:

```bash
df -h /mnt/storage
```

![](images/screenshots/shinobi-hdd-storage.png)

---

## Notes

This document describes the infrastructure configuration used to
deploy the Shinobi NVR workload on Proxmox VE.

Sensitive production information has been sanitized for public
portfolio use. The technical architecture, resource allocation,
network relationships, and storage design are preserved.