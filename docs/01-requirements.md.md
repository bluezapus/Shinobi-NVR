# Requirements

## Hardware

| Component      | Specification                             |
| -------------- | ----------------------------------------- |
| Server         | `O.E.M. B450M-HDV R4.0`                   |
| CPU            | `AMD Ryzen 5 3400G` — 4 cores / 8 threads |
| RAM            | `~13.57 GB`                               |
| System Storage | `128 GB SATA SSD`                         |
| CCTV Storage   | `4 TB HDD`                                |
| Physical NIC   | `nic0`                                    |

### Storage Layout

- `128 GB SSD` — Proxmox VE system and virtualized workload storage
- `4 TB HDD` — CCTV recording storage
- CCTV mount point: `/mnt/hddcctv`

---

## Software

| Component | Version / Configuration |
|---|---|
| Hypervisor | Proxmox VE `9.2.4` |
| Shinobi | `2.0.0` |
| Guest OS | Debian GNU/Linux 13 (Trixie) |
| Architecture | `amd64` |

> Software versions should be verified against the final running
> configuration before publication.

---

## CCTV

The system is designed to manage **16 IP cameras** through the
Shinobi NVR.

| Requirement       | Specification                     |
| ----------------- | --------------------------------- |
| Camera Models     | `TP-Link Tapo C220`               |
| Number of Cameras | `16`                              |
| Protocol          | `RTSP`                            |
| RTSP Port         | `554`                             |
| Video Codec       | `H.264`                           |
| Target FPS        | `15 FPS`                          |
| Resolution        | Camera-dependent / to be verified |


> Individual camera stream configurations may differ. Resolution,
> codec, FPS, RTSP path, and stream profile should be documented
> per camera after verification.

---

## Network

The CCTV infrastructure operates on a production LAN.

For public documentation, production IP addresses are sanitized.

| Component | Sanitized Configuration |
|---|---|
| Network | `192.168.10.0/24` |
| Gateway | `192.168.10.1` |
| Proxmox | `192.168.10.156/24` |
| Shinobi | `192.168.10.221/24` |
| Physical NIC | `nic0` |
| Proxmox Bridge | `vmbr0` |
| Network Device | Network Switch |

> Production IP addresses have been sanitized for public portfolio
> documentation.

---

## Functional Requirements

The infrastructure must provide:

- Centralized NVR management for 16 IP cameras.
- RTSP-based video ingestion.
- Continuous video recording.
- Dedicated storage for CCTV recordings.
- Network connectivity between IP cameras and Shinobi.
- Virtualized NVR infrastructure using Proxmox VE.
- Web-based Shinobi NVR management.
- Persistent storage for recorded video.
- Network and system administration access.

---

## Storage Requirements

CCTV recordings require dedicated high-capacity storage.

| Requirement | Configuration |
|---|---|
| Storage Type | HDD |
| Capacity | `4 TB` |
| Filesystem | `ext4` |
| Proxmox Mount | `/mnt/hddcctv` |
| Container Mount | `/mnt/storage` |
| Purpose | CCTV video recordings |

The CCTV recording storage is separated from the Proxmox system
storage to prevent video recordings from consuming the hypervisor's
system volume.

---

## Deployment Requirements

The target deployment consists of:

![](images/architecture/topologi01.png)
### Virtualization

- Proxmox VE is used as the hypervisor.
- Shinobi is deployed inside an LXC container.
- Shinobi uses a dedicated virtual CPU and memory allocation.
- CCTV recordings are stored on dedicated HDD storage.

### Networking

- Physical NIC connected to the production network.
- Proxmox uses `vmbr0` as the Linux network bridge.
- Shinobi connects through `eth0`.
- IP cameras communicate with Shinobi using RTSP.

---

## Requirements Summary

| Category           | Requirement        |
| ------------------ | ------------------ |
| Hypervisor         | Proxmox VE `9.2.4` |
| NVR                | Shinobi `2.0.0`    |
| Camera Model       | TP-Link Tapo C220  |
| Cameras            | 16 IP cameras      |
| Streaming          | RTSP               |
| RTSP Port          | `554`              |
| Target FPS         | `15 FPS`           |
| Recording Storage  | `4 TB HDD`         |
| Storage Filesystem | `ext4`             |
| Network            | `192.168.10.0/24`  |
| Network Bridge     | `vmbr0`            |
| Physical NIC       | `nic0`             |
| NVR Deployment     | LXC Container      |

