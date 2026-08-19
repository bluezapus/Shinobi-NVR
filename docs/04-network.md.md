# Network

## Overview

The CCTV infrastructure uses a Layer 2 network with a Proxmox Linux
bridge to provide network connectivity between the physical network,
the Proxmox hypervisor, LXC containers, and the Shinobi NVR.

The Shinobi NVR runs as an LXC container on Proxmox and receives IP
camera streams through the network using RTSP.

> Production IP addresses, credentials, and other sensitive information
> have been sanitized for public documentation.

---

## Network Addressing

The production network uses the `10.0.26.0/24` subnet.

For this public portfolio, production addresses are represented using
the sanitized `192.168.10.0/24` network.

| Device | Sanitized IP Address | Role |
|---|---|---|
| Gateway | `192.168.10.1` | Network Gateway |
| Proxmox | `192.168.10.156/24` | Hypervisor |
| Shinobi | `192.168.10.221/24` | NVR |
| Network | `192.168.10.0/24` | Production LAN |

### IP Assignment

Both the Proxmox host and Shinobi LXC obtain their IPv4 addresses
through DHCP.

| Component | Interface | Address Method |
|---|---|---|
| Proxmox | `vmbr0` | DHCP |
| Shinobi LXC | `eth0` | DHCP |

---

## Network Topology

The physical network interface of the Proxmox host is connected to
the `vmbr0` Linux bridge.

The bridge provides Layer 2 connectivity between the physical network,
the Proxmox host, and the LXC containers.

```text
                    Production LAN
                  192.168.10.0/24
                         │
                      Network
                       Switch
                         │
                       nic0
                         │
                      vmbr0
                         │
              ┌──────────┴──────────┐
              │                     │
       Proxmox Host           LXC Containers
       192.168.10.156                │
                                     ├── CT 100
                                     │   Shinobi NVR
                                     │      │
                                     │     eth0
                                     │
                                     ├── CT 200
                                     ├── CT 300
                                     └── CT 400
```

---

## CCTV Network Flow

The IP cameras provide video streams to the Shinobi NVR through
the production network.

```text
                 IP Cameras
              ┌────┬────┬────┐
              │    │    │    │
            CAM01 CAM02 ... CAM16
              │    │         │
              └────┴─────────┘
                       │
                     RTSP
                       │
                       ▼
                  Network Switch
                       │
                       ▼
                     nic0
                       │
                       ▼
                    vmbr0
                       │
                       ▼
                  Shinobi LXC
                    CT 100
                       │
                      eth0
                       │
                       ▼
                  Shinobi NVR
```

---

## Proxmox Network Interfaces

The Proxmox host uses `nic0` as the physical network interface.

The interface is configured as a bridge port for `vmbr0`.

| Interface | Type | Configuration | Role |
|---|---|---|---|
| `nic0` | Physical NIC | Manual | Physical network connection |
| `vmbr0` | Linux Bridge | DHCP | Host and LXC network bridge |

### Physical Interface

```text
Interface: nic0
Configuration: Manual
Role: Physical NIC
```

The physical NIC does not have an IP address assigned directly.
Instead, it is attached to the `vmbr0` Linux bridge.

---

## Proxmox Bridge

The Proxmox host uses `vmbr0` as the primary network bridge.

```text
Physical Network
       │
     nic0
       │
     vmbr0
       │
       ├── Proxmox Host
       │
       └── LXC Containers
```

### Bridge Configuration

| Parameter | Configuration |
|---|---|
| Bridge | `vmbr0` |
| Bridge Port | `nic0` |
| IP Configuration | DHCP |
| STP | Disabled |
| Forward Delay | `0` |

### Proxmox Configuration

The relevant Proxmox network configuration is:

```text
auto lo
iface lo inet loopback

iface nic0 inet manual

auto vmbr0
iface vmbr0 inet dhcp
        bridge-ports nic0
        bridge-stp off
        bridge-fd 0
```

This configuration allows `vmbr0` to act as the Layer 2 bridge
between the physical network interface and the virtualized workloads.

---

## Shinobi Network Interface

Shinobi runs inside **LXC Container 100**.

The container uses `eth0` as its network interface and connects
to the Proxmox network through `vmbr0`.

| Parameter | Value |
|---|---|
| Container ID | `100` |
| Hostname | `shinobi` |
| Interface | `eth0` |
| Proxmox Bridge | `vmbr0` |
| IPv4 | `192.168.10.221/24` |
| Address Method | DHCP |
| Gateway | `192.168.10.1` |

### Shinobi Network Path

```text
Shinobi LXC
    │
   eth0
    │
  vmbr0
    │
  nic0
    │
Physical Network
```

---

## Routing

The Shinobi container uses the network gateway for default routing.

Sanitized routing information:

```text
Destination      Gateway          Interface
------------------------------------------------
0.0.0.0/0        192.168.10.1     eth0
192.168.10.0/24  Direct           eth0
```

The production configuration also contains specific routes to remote
network endpoints through the network gateway.

Production-specific destination addresses are intentionally omitted
from the public documentation.

---

## CCTV Connectivity

The infrastructure consists of **16 IP cameras** connected to the
Shinobi NVR.

The cameras provide video streams using the RTSP protocol.

```text
IP Camera
    │
    │ RTSP
    ▼
Network
    │
    ▼
Proxmox Network Bridge
    │
    ▼
Shinobi LXC
    │
    ▼
Shinobi NVR
```

### Camera Connectivity Summary

| Parameter | Value |
|---|---|
| Camera Count | 16 |
| NVR | Shinobi |
| Streaming Protocol | RTSP |
| RTSP Port | `554` |
| Network | Production LAN |

> Individual camera IP addresses are documented separately after
> sanitization.

---

## RTSP Communication

Shinobi consumes the IP camera streams through RTSP.

The general communication model is:

```text
Camera
   │
   │ RTSP : 554
   ▼
Shinobi NVR
```

Example sanitized configuration:

```text
rtsp://[USERNAME]:[PASSWORD]@[CAMERA-IP]:554/[STREAM-PATH]
```

Production credentials are never stored in this repository.

---

## Camera Network Mapping

The project contains 16 IP cameras.

Production camera addressing is intentionally not exposed in this
public repository.

Use sanitized identifiers when documenting the cameras:

| Camera | Sanitized IP     |  Port |
| ------ | ---------------- | ----: |
| CAM01  | `192.168.10.222` | `554` |
| CAM02  | `192.168.10.223` | `554` |
| CAM03  | `192.168.10.224` | `554` |
| CAM04  | `192.168.10.225` | `554` |
| CAM05  | `192.168.10.226` | `554` |
| CAM06  | `192.168.10.227` | `554` |
| CAM07  | `192.168.10.228` | `554` |
| CAM08  | `192.168.10.229` | `554` |
| CAM09  | `192.168.10.230` | `554` |
| CAM10  | `192.168.10.231` | `554` |
| CAM11  | `192.168.10.232` | `554` |
| CAM12  | `192.168.10.233` | `554` |
| CAM13  | `192.168.10.234` | `554` |
| CAM14  | `192.168.10.235` | `554` |
| CAM15  | `192.168.10.236` | `554` |
| CAM16  | `192.168.10.237` | `554` |

> The IP addresses above are placeholders for public documentation.
> They do not represent the actual production camera addresses.

---

## Network Services

| Service | Port | Protocol | Purpose |
|---|---:|---|---|
| RTSP | `554` | TCP/UDP | IP camera video stream |
| SSH | `22` | TCP | System administration |
| Shinobi Web UI | `8080` | TCP | NVR management interface |

> Service ports should be verified against the running configuration
> before being treated as the final production configuration.

---

## Firewall

The Shinobi LXC network interface has the Proxmox firewall enabled.

| Component | Firewall |
|---|---|
| Shinobi LXC | Enabled |

Detailed firewall rules are documented separately if required.

---

## Network Security

The public repository does not expose production credentials or
sensitive infrastructure identifiers.

The following information is intentionally sanitized or excluded:

- Production IP addresses
- RTSP usernames
- RTSP passwords
- Camera credentials
- MAC addresses
- Authentication tokens
- Internal hostnames
- Company-specific network identifiers
- Sensitive routing information

Only the logical topology and technical architecture are exposed.

---

## Network Configuration Summary

| Component | Configuration |
|---|---|
| Network | `192.168.10.0/24` |
| Gateway | `192.168.10.1` |
| Proxmox IP | `192.168.10.156/24` |
| Proxmox Physical NIC | `nic0` |
| Proxmox Bridge | `vmbr0` |
| Proxmox IP Assignment | DHCP |
| Shinobi Container | CT 100 |
| Shinobi Interface | `eth0` |
| Shinobi IP | `192.168.10.221/24` |
| Shinobi IP Assignment | DHCP |
| Camera Count | 16 |
| Camera Protocol | RTSP |
| RTSP Port | `554` |
| Shinobi Web Interface | `8080/TCP` |
| LXC Firewall | Enabled |

---

## Verification Commands

### Proxmox Host

```bash
ip -br addr
```

```bash
cat /etc/network/interfaces
```

```bash
ip route
```

![Keterangan](shinobi-network.png)
### Shinobi LXC

```bash
ip -br addr
```

```bash
ip route
```

![Keterangan](shinobi-addr.png)
### LXC Configuration

```bash
pct config 100
```

![Keterangan](shinobi-config.png)