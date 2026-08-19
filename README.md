# CCTV NVR Infrastructure

Infrastructure documentation for a centralized CCTV Network Video Recorder
(NVR) system built with Proxmox VE, Shinobi NVR, IP cameras, and dedicated
HDD storage.

The system is designed to manage **16 IP cameras** using RTSP streams and
store CCTV recordings on dedicated high-capacity storage.

---

## Overview

This project implements a virtualized CCTV NVR infrastructure using:

- Proxmox VE as the hypervisor
- Shinobi NVR running inside an LXC container
- 16 TP-Link Tapo C220 IP cameras
- RTSP for video streaming
- Dedicated 3.6 TB HDD storage for CCTV recordings
- Linux bridge networking using `vmbr0`

The infrastructure separates the NVR operating system and application
storage from the CCTV recording storage.

This improves storage isolation and prevents continuous CCTV recording data
from consuming the Proxmox system volume.

---

## Architecture

![Keterangan](architecture.png)

---

## Key Components

| Component             | Configuration      |
| --------------------- | ------------------ |
| Hypervisor            | Proxmox VE `9.2.4` |
| NVR                   | Shinobi            |
| Shinobi Deployment    | LXC Container      |
| Container ID          | `100`              |
| Guest OS              | Ubuntu 22.04.5 LTS |
| Architecture          | `amd64`            |
| CPU Allocation        | 4 vCPU             |
| Memory                | 8 GB               |
| SWAP                  | 512 MB             |
| Root Disk             | 64 GB              |
| Camera Count          | 16                 |
| Camera Model          | TP-Link Tapo C220  |
| Streaming Protocol    | RTSP               |
| RTSP Port             | `554`              |
| CCTV Storage          | 3.6 TB HDD         |
| Filesystem            | `ext4`             |
| Proxmox CCTV Mount    | `/mnt/hddcctv`     |
| Shinobi Storage Mount | `/mnt/storage`     |
| Network Bridge        | `vmbr0`            |
| Physical NIC          | `nic0`             |

---

## Technology Stack

### Virtualization

**Proxmox VE**

Provides the virtualization layer and manages the Shinobi LXC container.

### NVR

**Shinobi NVR**

Provides centralized CCTV monitoring, camera management, RTSP stream
ingestion, recording, and web-based NVR management.

### Operating System

**Ubuntu 22.04.5 LTS**

Runs inside the Shinobi LXC container.

### Video Processing

**FFmpeg**

Used by the Shinobi environment for video stream processing and recording.

### Database

**MariaDB 10.6**

Used by the Shinobi installation for application data.

### Networking

**Linux Bridge (`vmbr0`)**

Provides Layer 2 connectivity between the physical network interface,
Proxmox host, and LXC containers.

---

## Storage Architecture

The system uses separate storage for the operating system and CCTV
recordings.

![Keterangan](architecture-storage.png)


The dedicated CCTV HDD has a capacity of approximately `3.6 TB`.

Current documented utilization:

- Used: `486 GB`
- Available: `3.0 TB`
- Utilization: `14%`

---

## Network Architecture

The CCTV infrastructure operates on a production LAN.

For public documentation, production IP addresses are sanitized.

![Keterangan](architecture-network.png)

Sanitized addressing:

| Component | Address |
|---|---|
| Network | `192.168.10.0/24` |
| Gateway | `192.168.10.1` |
| Proxmox | `192.168.10.156/24` |
| Shinobi | `192.168.10.221/24` |
| Bridge | `vmbr0` |
| Physical NIC | `nic0` |

> Production IP addresses are sanitized for public documentation.

---

## CCTV Architecture

The NVR manages 16 IP cameras.

Each camera provides an RTSP stream to Shinobi.
![Keterangan](architecture-cctv.png)

The documented camera configuration uses:

- Camera count: 16
- Camera model: TP-Link Tapo C220
- Protocol: RTSP
- RTSP port: `554`
- Target frame rate: `15 FPS`
- Video codec: H.264

Individual camera stream parameters may differ and should be verified
against the final production configuration.

---

## Shinobi Environment

The Shinobi NVR runs inside an unprivileged LXC container.

Current environment:
![Keterangan](environment.png)

The Shinobi source is located at:

```text
/home/Shinobi
```

The configured Shinobi web port is:

```text
8080/TCP
```

The application recording storage is configured through:

```text
/mnt/storage
```

The currently identified Shinobi Git revision is:

```text
f5cb53d1
```

---

## Security

The Shinobi workload runs as an **unprivileged LXC container**.

Security-related configuration includes:

- Unprivileged LXC
- Proxmox firewall enabled on the Shinobi network interface
- Production credentials excluded from documentation
- Production IP addresses sanitized
- RTSP credentials excluded from the repository
- Sensitive routing information excluded

The public documentation intentionally does not expose:

- Camera passwords
- RTSP usernames
- Authentication tokens
- Production credentials
- MAC addresses
- Sensitive internal hostnames
- Company-specific network identifiers

---

## Documentation

The project documentation is organized into seven stages.

| Document                                   | Description                                                             |
| ------------------------------------------ | ----------------------------------------------------------------------- |
| [`01-requirement.md`](01-requirement.md)   | Hardware, software, CCTV, network, storage, and deployment requirements |
| [`02-architecture.md`](02-architecture.md) | Overall infrastructure architecture and design                          |
| [`03-proxmox.md`](03-proxmox.md)           | Proxmox VE host and Shinobi LXC configuration                           |
| [`04-network.md`](04-network.md)           | Network topology, bridge, addressing, and RTSP connectivity             |
| [`05-shinobi.md`](05-shinobi.md)           | Shinobi NVR software environment and configuration                      |
| [`06-rtsp-cctv.md`](06-rtsp-cctv.md)       | RTSP camera integration and CCTV stream architecture                    |
| [`07-storage.md`](07-storage.md)           | CCTV storage architecture, mount points, filesystem, and capacity       |

---

## Resource Allocation

The Shinobi NVR is allocated:

```text
CPU       : 4 vCPU
Memory    : 8 GB
SWAP      : 512 MB
Root Disk : 64 GB
CCTV Disk : 3.6 TB
```

The separation between root storage and recording storage is intentional.
![Keterangan](allocation.png)

---

## Storage Design

The CCTV storage is mounted persistently on the Proxmox host.
![Keterangan](architecture-storage02.png)

The Proxmox host uses `/etc/fstab` to mount the filesystem.

The Shinobi LXC uses the following mount configuration:

```text
mp0: /mnt/hddcctv,mp=/mnt/storage
```

---

## Monitoring and Verification

Useful verification commands include:

### Proxmox

```bash
pveversion
```

```bash
lscpu
```

```bash
free -h
```

```bash
lsblk -o NAME,SIZE,MODEL,TYPE,FSTYPE,MOUNTPOINT
```

```bash
ip -br addr
```

### Shinobi LXC

```bash
hostnamectl
```

```bash
node -v
```

```bash
npm -v
```

```bash
ffmpeg -version
```

### Storage

```bash
df -h /mnt/hddcctv
```

```bash
df -h /mnt/storage
```

```bash
findmnt /mnt/hddcctv
```

### LXC

```bash
pct config 100
```

---

## Current Status

| Area | Status |
|---|---|
| Proxmox VE | Configured |
| Shinobi LXC | Configured |
| Network Bridge | Configured |
| Shinobi Network | Configured |
| CCTV Storage | Configured |
| RTSP Integration | Configured |
| IP Cameras | 16 cameras |
| Dedicated HDD | Configured |
| Storage Mount | Configured |
| LXC Firewall | Enabled |
| Retention Policy | To be verified |
| SMART Monitoring | To be documented |
| CCTV Backup Strategy | To be documented |

---

## Design Considerations

### 1. Virtualization

Shinobi is isolated inside an LXC container instead of running directly
on the Proxmox host.

### 2. Storage Separation

CCTV recordings are stored on a dedicated HDD rather than the Proxmox
system disk.

### 3. Network Bridging

The `vmbr0` Linux bridge provides direct Layer 2 connectivity between
the physical network and virtualized workloads.

### 4. Resource Isolation

The Shinobi workload has dedicated CPU and memory allocation.

### 5. Security Isolation

The Shinobi container operates as an unprivileged LXC.

### 6. Documentation and Sanitization

Production-specific credentials and sensitive infrastructure identifiers
are excluded from the public documentation.

---

## Limitations

The current documentation does not claim the following capabilities
unless explicitly verified:

- RAID redundancy
- CCTV recording backup
- Off-site backup
- Automatic cloud backup
- Defined CCTV retention period
- SMART-based alerting
- High-availability Proxmox
- Redundant network connectivity

These items can be added if implemented in the production environment.

---

## Project Summary

This project demonstrates the deployment of a centralized, virtualized
CCTV NVR infrastructure using Proxmox VE and Shinobi.

The final architecture provides:

- 16-camera centralized NVR management
- RTSP-based video ingestion
- Continuous CCTV recording
- Dedicated 3.6 TB recording storage
- LXC-based application isolation
- Linux bridge networking
- Separated system and recording storage
- Web-based NVR management
- Production-oriented infrastructure documentation

---

## Disclaimer

This repository is a sanitized technical documentation of a CCTV
infrastructure deployment.

Production IP addresses, credentials, authentication information,
camera-specific secrets, and other sensitive infrastructure information
have been removed or replaced with representative values.

The documentation is intended for infrastructure documentation,
technical portfolio, and educational purposes.
