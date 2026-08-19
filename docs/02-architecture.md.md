# System Architecture

## Overview

The CCTV infrastructure is designed as a virtualized Network Video Recorder  
(NVR) platform using Proxmox VE as the virtualization layer and Shinobi as  
the centralized NVR application.

The architecture separates the infrastructure into several logical layers:
![](images/architecture/logical-layer.png)


This separation allows the CCTV workload to be managed independently from  
the physical server while maintaining dedicated storage for recorded video.

> Production IP addresses, credentials, hostnames, and other company-specific  
> information have been sanitized for public portfolio documentation.

---

## Architecture Goals

The architecture is designed to provide:

- Centralized management of multiple IP cameras.
    
- Continuous RTSP video ingestion.
    
- Reliable long-term CCTV recording.
    
- Dedicated storage for recorded video.
    
- Isolation of the NVR application from the Proxmox host.
    
- Simple network connectivity between cameras and the NVR.
    
- Web-based NVR administration.
    
- Separation between system storage and CCTV recording storage.
    
- A maintainable and recoverable virtualization environment.
    

---

## High-Level Architecture

The overall system consists of a physical server running Proxmox VE.  
Shinobi NVR is deployed inside an LXC container.

The IP cameras are connected to the production LAN and provide RTSP streams  
to the Shinobi container.

CCTV recordings are stored on a dedicated 4 TB HDD mounted on the Proxmox  
host and exposed to the Shinobi container through a container mount.
![](images/architecture/topologi02.png)

---

## Architecture Layers

### 1. Physical Layer

The physical layer consists of the server hardware, network interface,  
network switch, IP cameras, and dedicated CCTV storage.

```text
Physical Server
├── CPU
├── RAM
├── System SSD
├── CCTV HDD
└── Physical NIC
```

The physical server provides the compute resources required by Proxmox VE  
and the virtualized NVR workload.

---

### 2. Virtualization Layer

Proxmox VE provides the virtualization layer.

```text
Physical Server
      │
      ▼
Proxmox VE
      │
      └── LXC Container 100
              │
              └── Shinobi NVR
```

The NVR application is isolated from the Proxmox host by running inside an  
LXC container.

The current Shinobi container is configured with:

|Resource|Allocation|
|---|--:|
|Container|`CT 100`|
|CPU|`4 vCPU`|
|Memory|`8 GB`|
|SWAP|`512 MB`|
|Root Disk|`62.44 GB`|
|Root Storage|`local-lvm`|
|Container Type|Unprivileged LXC|

Detailed virtualization configuration is documented in  
`03-proxmox.md`.

---

## 3. Network Layer

The network layer provides connectivity between the physical server,  
Proxmox host, Shinobi container, IP cameras, and the rest of the  
production LAN.

```text
IP Cameras
    │
    │ RTSP
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
    │
   eth0
    │
    ▼
Shinobi NVR
```

The Proxmox physical interface is attached to the `vmbr0` Linux bridge.

The bridge provides Layer 2 connectivity to the LXC container.

Detailed network configuration is documented in `04-network.md`.

---

## 4. CCTV Camera Layer

The CCTV layer consists of 16 IP cameras.

```text
                CCTV Cameras
        ┌─────────┬─────────┬─────────┐
        │         │         │         │
      CAM01     CAM02     CAM03     ...
        │         │         │
        └─────────┴─────────┴─────────┐
                                      │
                                    CAM16
                                      │
                                      ▼
                                 Network LAN
```

Each camera provides an RTSP video stream to the Shinobi NVR.

### Camera Configuration

|Parameter|Configuration|
|---|---|
|Camera Count|`16`|
|Camera Model|TP-Link Tapo C220|
|Protocol|RTSP|
|RTSP Port|`554`|
|Target FPS|`15 FPS`|
|Codec|H.264|
|Resolution|Camera-dependent|

Individual stream paths and camera-specific configuration are maintained  
separately from this architecture document.

---

## 5. NVR Application Layer

Shinobi acts as the centralized CCTV management and recording platform.

Its primary responsibilities are:

- Connecting to IP camera RTSP streams.
    
- Processing incoming video streams.
    
- Recording CCTV footage.
    
- Managing camera configurations.
    
- Providing a web-based management interface.
    
- Managing recording retention.
    
- Providing access to recorded footage.
    

The application runs inside LXC Container 100.

```text
                 Shinobi NVR
                     │
        ┌────────────┼────────────┐
        │            │            │
        ▼            ▼            ▼
   RTSP Input    Recording    Web Interface
        │            │            │
        ▼            ▼            ▼
   IP Cameras    /mnt/storage   Administrators
```

---

## 6. Storage Layer

CCTV recordings are stored separately from the operating system and  
application environment.

The storage architecture consists of:

```text
                    Physical HDD
                         │
                         ▼
                Proxmox Host Storage
                  /mnt/hddcctv
                         │
                  Container Mount
                         │
                         ▼
                    /mnt/storage
                         │
                         ▼
                  Shinobi Recordings
```

The separation provides a clear distinction between:

|Storage|Purpose|
|---|---|
|`local-lvm`|Proxmox/LXC system storage|
|`62.44 GB` LXC disk|Shinobi operating system and application|
|`4 TB HDD`|CCTV recording storage|
|`/mnt/hddcctv`|Proxmox host mount|
|`/mnt/storage`|Shinobi container recording path|

This design prevents CCTV recordings from consuming the LXC root disk.

---

# Data Flow

## CCTV Video Flow

The primary data flow is the movement of video from the IP cameras to  
Shinobi and then to the dedicated recording storage.

```text
┌─────────────┐
│ IP Camera   │
│ CAM01-CAM16 │
└──────┬──────┘
       │
       │ RTSP
       │ TCP/UDP :554
       ▼
┌───────────────┐
│ Network       │
│ Infrastructure│
└──────┬────────┘
       │
       ▼
┌───────────────┐
│ Proxmox vmbr0 │
└──────┬────────┘
       │
       ▼
┌────────────────┐
│ Shinobi LXC    │
│ Container 100  │
└───────┬────────┘
        │
        ▼
┌────────────────┐
│ Shinobi NVR    │
└───────┬────────┘
        │
        │ Recording
        ▼
┌────────────────┐
│ /mnt/storage   │
│ CCTV Storage   │
└───────┬────────┘
        │
        ▼
┌────────────────┐
│ 4 TB HDD       │
│ ext4           │
└────────────────┘
```

---

## Management Flow

System administration and NVR management are performed through the  
network.

```text
Administrator
      │
      │ HTTP / HTTPS
      ▼
Shinobi Web Interface
      │
      ▼
Shinobi NVR
      │
      ├── Camera Management
      ├── Live View
      ├── Recording Management
      └── System Monitoring
```

Proxmox administration is performed separately through the Proxmox  
management interface.

```text
Administrator
      │
      ▼
Proxmox Management
      │
      ▼
Proxmox VE
      │
      └── LXC Container 100
              │
              └── Shinobi NVR
```

---

# Component Relationships

The major system components have the following relationships:

|Component|Connects To|Purpose|
|---|---|---|
|IP Cameras|Network Switch|Provide CCTV streams|
|Network Switch|`nic0`|Physical network connectivity|
|`nic0`|`vmbr0`|Bridge physical network|
|`vmbr0`|Shinobi LXC|Container network connectivity|
|Shinobi LXC|IP Cameras|Receive RTSP streams|
|Shinobi NVR|`/mnt/storage`|Store recordings|
|`/mnt/storage`|CCTV HDD|Persistent recording storage|
|Administrator|Shinobi|NVR management|
|Administrator|Proxmox|Infrastructure management|

---

# Storage Architecture

The storage design intentionally separates system storage from recording  
storage.

```text
                       Proxmox VE
                           │
             ┌─────────────┴─────────────┐
             │                           │
        System Storage              CCTV Storage
          local-lvm                   4 TB HDD
             │                           │
             ▼                           ▼
      LXC Root Disk                 /mnt/hddcctv
       62.44 GB                         │
             │                           │
             ▼                           ▼
       Ubuntu / Shinobi             Container Mount
                                         │
                                         ▼
                                    /mnt/storage
                                         │
                                         ▼
                                  CCTV Recordings
```

This separation provides the following benefits:

- CCTV recordings do not consume the LXC root disk.
    
- The recording storage can be monitored independently.
    
- The NVR application remains separated from recording data.
    
- Storage capacity can be managed independently from the operating  
    system.
    
- Backup or migration strategies can be developed independently for  
    application configuration and video recordings.
    

---

# Network and Storage Boundaries

The system has clear boundaries between compute, network, and storage.

```text
                  ┌──────────────────────┐
                  │   Physical Server    │
                  │                      │
                  │      Proxmox VE      │
                  └──────────┬───────────┘
                             │
              ┌──────────────┼──────────────┐
              │              │              │
              ▼              ▼              ▼
          Compute         Network        Storage
              │              │              │
              │              │              │
              ▼              ▼              ▼
          LXC 100          vmbr0       CCTV HDD
              │              │              │
              ▼              │              │
          Shinobi ◄──────────┘              │
              │                             │
              └─────────────────────────────┘
```

---

# Security Architecture

The architecture applies several basic isolation mechanisms.

### LXC Isolation

Shinobi runs inside an unprivileged LXC container.

This limits the privileges of the NVR workload relative to the Proxmox  
host.

### Network Isolation

The NVR is accessed through the production network and communicates with  
IP cameras using their RTSP services.

### Credential Protection

Camera credentials and other authentication information are not stored  
in the public documentation.

Example RTSP URLs use placeholders:

```text
rtsp://[USERNAME]:[PASSWORD]@[CAMERA-IP]:554/[STREAM-PATH]
```

### Firewall

The Proxmox firewall is enabled for the Shinobi LXC.

Detailed firewall rules should be documented separately if they are  
required for deployment or auditing.

---

# Failure Domains

The architecture separates several major failure domains.

|Component|Potential Failure|Impact|
|---|---|---|
|IP Camera|Camera unavailable|Recording unavailable for affected camera|
|Network Switch|Network failure|Multiple cameras/NVR connectivity affected|
|Physical NIC|NIC failure|Proxmox network connectivity lost|
|Proxmox Host|Host failure|Shinobi NVR unavailable|
|LXC Container|Container failure|Shinobi service unavailable|
|Shinobi Application|Application failure|Recording/management unavailable|
|CCTV HDD|Storage failure|Recording storage unavailable|
|System SSD|System storage failure|Proxmox/LXC system unavailable|

The architecture therefore treats the physical server, network, NVR  
application, and recording storage as separate operational components.

---

# Scalability Considerations

The current architecture is designed for 16 IP cameras.

Additional cameras may require evaluation of:

- CPU utilization.
    
- Memory utilization.
    
- Network bandwidth.
    
- RTSP stream bitrate.
    
- Recording storage capacity.
    
- Disk write throughput.
    
- Shinobi monitor limits.
    
- Retention period.
    

Increasing the camera count should not be performed solely by increasing  
the number of Shinobi monitors. The complete resource chain should be  
evaluated.

```text
Additional Cameras
       │
       ├── Network Bandwidth
       ├── CPU Utilization
       ├── Memory Utilization
       ├── Disk Write Throughput
       └── Storage Capacity
```

---

# Operational Boundaries

The responsibilities of each layer are separated as follows:

|Layer|Responsibility|
|---|---|
|Physical Server|Compute and physical storage|
|Proxmox VE|Virtualization and host management|
|`vmbr0`|Network bridging|
|LXC|Application isolation|
|Shinobi|NVR management and recording|
|Network|Camera/NVR connectivity|
|CCTV HDD|Video recording storage|
|Administrator|Infrastructure and NVR management|

This separation makes troubleshooting easier because problems can be  
classified according to the affected layer.

---

# Architecture Summary

The final architecture can be summarized as:

```text
                         ┌───────────────────┐
                         │   Administrators  │
                         └─────────┬─────────┘
                                   │
                          Management Network
                                   │
                                   ▼
┌──────────────┐          ┌───────────────────┐
│ IP Cameras   │──RTSP────►     Network       │
│ CAM01-CAM16  │          │      Switch       │
└──────────────┘          └─────────┬─────────┘
                                    │
                                   nic0
                                    │
                                    ▼
                           ┌─────────────────┐
                           │    vmbr0        │
                           │ Proxmox Bridge  │
                           └────────┬────────┘
                                    │
                    ┌───────────────┴──────────────┐
                    │                              │
                    ▼                              ▼
             ┌──────────────┐              ┌──────────────┐
             │ Proxmox Host │              │  LXC CT 100  │
             │              │              │              │
             │ Proxmox VE   │─────────────►│ Shinobi NVR  │
             └──────────────┘              └──────┬───────┘
                                                  │
                                                  │
                                            /mnt/storage
                                                  │
                                                  ▼
                                           ┌─────────────┐
                                           │ 4 TB HDD    │
                                           │ ext4        │
                                           │ CCTV Data   │
                                           └─────────────┘
```

---

# Related Documentation

The architecture is supported by the following documentation:

|Document|Purpose|
|---|---|
|`01-requirements.md`|Hardware, software, CCTV, network, and functional requirements|
|`02-architecture.md`|Overall system architecture and component relationships|
|`03-proxmox.md`|Proxmox host, LXC, resource, and storage configuration|
|`04-network.md`|Network topology, addressing, bridge, and RTSP connectivity|
|`05-shinobi.md`|Shinobi NVR installation and application configuration|
|`06-rtsp-cctv.md`|Camera and RTSP stream configuration|
|`07-storage.md`|CCTV storage and recording retention|
|`08-troubleshooting.md`|Troubleshooting and incident resolution|
|`09-testing.md`|Infrastructure and application verification|

---

# Design Principles

The architecture follows these principles:

1. **Separation of concerns**  
    Compute, networking, NVR application, and recording storage are  
    treated as separate infrastructure components.
    
2. **Dedicated recording storage**  
    CCTV recordings are stored on dedicated HDD storage rather than the  
    LXC root disk.
    
3. **Application isolation**  
    Shinobi is deployed inside an unprivileged LXC container instead of  
    running directly on the Proxmox host.
    
4. **Centralized NVR management**  
    All cameras are managed through a centralized Shinobi NVR instance.
    
5. **Documented network boundaries**  
    Communication between cameras, network infrastructure, Proxmox,  
    and Shinobi is explicitly defined.
    
6. **Sanitized public documentation**  
    Production credentials, IP addresses, MAC addresses, and  
    company-specific infrastructure information are excluded from the  
    public repository.
    

---

## Final Architecture

The system implements a virtualized CCTV NVR architecture where:

- **Proxmox VE** provides the virtualization platform.
    
- **LXC Container 100** provides application isolation.
    
- **Shinobi** provides centralized NVR functionality.
    
- **`vmbr0`** provides virtual-to-physical network connectivity.
    
- **16 IP cameras** provide RTSP video streams.
    
- **A dedicated 4 TB HDD** provides persistent CCTV recording storage.
    
- **Administrators** manage the infrastructure through Proxmox and  
    Shinobi web interfaces.
    

This architecture provides a clear separation between infrastructure,  
networking, application services, and CCTV data while maintaining a  
relatively simple deployment model suitable for a small-to-medium CCTV  
environment.