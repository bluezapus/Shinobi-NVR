# Shinobi NVR

## Overview

Shinobi is deployed as the centralized Network Video Recorder (NVR)  
application for the CCTV infrastructure.

The application runs inside an unprivileged LXC container on Proxmox VE.  
Shinobi receives RTSP video streams from the IP cameras, processes the  
streams through FFmpeg, and stores recorded video on dedicated CCTV  
storage mounted at `/mnt/storage`.

![Keterangan](shinobi-os.png)

```text
                         Shinobi LXC
                         CT 100
                            │
                    ┌───────┴────────┐
                    │                │
                    ▼                ▼
                Shinobi            MariaDB
                NVR Application    Database
                    │
                    ▼
                  FFmpeg
                    │
                    ▼
              RTSP Camera Streams
                    │
                    ▼
               CCTV Recordings
                    │
                    ▼
               /mnt/storage
                    │
                    ▼
                4 TB HDD
```

> Production credentials, IP addresses, authentication keys, and other  
> sensitive information are excluded from this documentation.

---

# Deployment Environment

Shinobi is deployed inside an LXC container hosted by Proxmox VE.

|Parameter|Configuration|
|---|---|
|Container|`LXC 100`|
|Hostname|`shinobi`|
|Container Type|Unprivileged LXC|
|Operating System|Ubuntu 22.04.5 LTS|
|Architecture|`x86-64`|
|Virtualization|LXC|
|CPU Allocation|`4 vCPU`|
|Memory Allocation|`8 GB`|
|SWAP|`512 MB`|
|Root Disk|`62.44 GB`|
|Recording Storage|`3.6 TB`|
|Application Path|`/home/Shinobi`|

The container is isolated from the Proxmox host and provides the operating  
environment for the Shinobi NVR application.

Detailed container and virtualization configuration is documented in  
`03-proxmox.md`.

---

# Operating System

The Shinobi container runs Ubuntu 22.04.5 LTS.

```text
PRETTY_NAME="Ubuntu 22.04.5 LTS"
VERSION_ID="22.04"
VERSION_CODENAME=jammy
```

The container runs using the Proxmox host kernel:

```text
Linux 6.17.2-1-pve
```

The system architecture is:

```text
x86_64
```

The container environment is identified as:

```text
Virtualization: lxc
Chassis: container
```

---

# Shinobi Application

The Shinobi source code is installed under:

```text
/home/Shinobi
```

The application directory contains the Shinobi application source,  
configuration, libraries, web interface, plugins, SQL definitions,  
and Node.js dependencies.

```text
/home/Shinobi
├── conf.json
├── camera.js
├── cron.js
├── definitions/
├── fileBin/
├── languages/
├── libs/
├── node_modules/
├── plugins/
├── sql/
├── tools/
├── videos/
├── videos2/
└── web/
```

The `node_modules` directory contains the installed Node.js application  
dependencies.

---

# Shinobi Source Revision

The running installation is managed from a Git repository.

Current Git information:

```text
Branch:
master

Commit:
f5cb53d1

Commit Message:
Reception
```

Git description:

```text
furrykitten-3-1060-gf5cb53d1
```

The Git revision should be recorded when performing maintenance or  
upgrades so that the deployed application state can be reproduced.

> The Git revision is documented instead of assigning a semantic  
> Shinobi version number because the observed installation identifies  
> itself primarily through its Git repository state.

---

# Node.js Runtime

Shinobi uses Node.js as its application runtime.

|Component|Version|
|---|---|
|Node.js|`18.20.8`|
|npm|`10.9.8`|

---

# Database

Shinobi uses MariaDB as its database backend.

The database server is running locally inside the Shinobi LXC.

|Component|Configuration|
|---|---|
|Database Engine|MariaDB|
|Version|`10.6.23`|
|Host|`127.0.0.1`|
|Port|`3306`|
|Database|`ccio`|
|Database User|`shinobi`|

The database connection is configured in:

```text
/home/Shinobi/conf.json
```

The database password is intentionally excluded from this documentation.

The local database architecture is:

```text
┌──────────────────────┐
│    Shinobi NVR       │
│                      │
│   Node.js Runtime    │
└──────────┬───────────┘
           │
           │ TCP 3306
           │ localhost
           ▼
┌──────────────────────┐
│      MariaDB         │
│      10.6.23         │
│                      │
│      Database        │
│        ccio          │
└──────────────────────┘
```

Because MariaDB runs locally inside the same LXC container, database  
traffic does not need to traverse the production network.

---

# FFmpeg

FFmpeg is used by Shinobi for video stream processing and recording.

Installed version:

```text
FFmpeg 4.4.2
```

The installed package is built for Ubuntu 22.04 and supports a wide  
range of video, audio, and streaming libraries.

Verification:

```bash
ffmpeg -version
```

The observed build includes support for codecs and libraries including:

- H.264
    
- H.265
    
- H.264 hardware-related libraries
    
- VP8 / VP9
    
- AV1
    
- Opus
    
- AAC
    
- SRT
    
- RTSP-related functionality
    
- VA-API
    
- OpenCL
    
- OpenGL
    
- Intel Media SDK
    

The current documentation does not claim that hardware acceleration is  
actively used by Shinobi because the active monitor configuration and  
FFmpeg execution parameters were not provided.

---

# Shinobi Configuration

The main Shinobi configuration file is:

```text
/home/Shinobi/conf.json
```

The observed configuration contains the following relevant settings:

|Parameter|Configuration|
|---|---|
|Debug Logging|Disabled|
|Web Port|`8080`|
|Password Type|`sha256`|
|Additional Storage|Enabled|
|Storage Name|`hdd`|
|Storage Path|`/mnt/storage`|
|Database Host|`127.0.0.1`|
|Database Port|`3306`|
|Database|`ccio`|
|Snapshot|Enabled|
|Discord Bot|Disabled|
|FTP Server|Disabled|
|Face Manager|Disabled|
|Better P2P|Enabled|
|Wall Clock Timestamp|Enabled|

Sensitive authentication values are intentionally omitted.

---

# Web Interface

Shinobi is configured to listen on TCP port:

```text
8080
```

The logical management path is:

```text
Administrator
      │
      │ HTTP :8080
      ▼
Shinobi Web Interface
      │
      ▼
Shinobi NVR
```

The exact external access path depends on the network configuration  
described in `04-network.md`.

---

# Authentication

Shinobi is configured to use:

```text
Password Type: SHA-256
```

Authentication credentials are stored outside this public  
documentation.

The following information should never be committed to a public  
repository:

- Shinobi administrator passwords
    
- Database passwords
    
- RTSP usernames
    
- RTSP passwords
    
- API tokens
    
- Cron keys
    
- SMTP credentials
    
- Authentication secrets
    

---

# Storage Configuration

Shinobi has an additional storage location configured as:

```text
Storage Name: hdd
Storage Path: /mnt/storage
```

The storage path is backed by the dedicated CCTV HDD.

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
Shinobi Storage
name: hdd
```

The current filesystem capacity observed from inside the container is:

|Parameter|Value|
|---|--:|
|Capacity|`3.6 TB`|
|Used|`486 GB`|
|Available|`3.0 TB`|
|Usage|`14%`|
|Mount Point|`/mnt/storage`|

The dedicated storage prevents CCTV recordings from consuming the  
LXC root disk.

---

# Application Storage vs Recording Storage

The Shinobi installation contains two different types of storage paths.

### Application Directory

```text
/home/Shinobi
```

This contains:

- Shinobi source code
    
- Node.js dependencies
    
- Configuration
    
- Plugins
    
- Web interface
    
- Application libraries
    

### Recording Storage

```text
/mnt/storage
```

This is the dedicated CCTV storage location.

The configuration also contains:

```text
videosDir: /home/Shinobi/videos
```

This path is part of the Shinobi application configuration, while  
the additional storage named `hdd` points to `/mnt/storage`.

The exact recording path used by each monitor depends on its individual  
Shinobi storage configuration.

Therefore, monitor-specific recording configuration should be verified  
before treating `/mnt/storage` as the recording path for every monitor.

---

# CCTV Recording Architecture

The intended recording flow is:

```text
IP Camera
    │
    │ RTSP
    ▼
Network
    │
    ▼
Shinobi LXC
    │
    ▼
Shinobi Monitor
    │
    ▼
FFmpeg
    │
    ▼
Video Recording
    │
    ▼
hdd storage
    │
    ▼
/mnt/storage
    │
    ▼
Dedicated CCTV HDD
```

The NVR receives the camera stream through RTSP and uses FFmpeg as the  
video processing layer.

---

# Camera Integration

The CCTV infrastructure contains 16 IP cameras.

```text
CAM01 ──┐
CAM02 ──┤
CAM03 ──┤
CAM04 ──┤
CAM05 ──┤
CAM06 ──┤
CAM07 ──┤
CAM08 ──┤
CAM09 ──┤
CAM10 ──┼──► Shinobi NVR
CAM11 ──┤
CAM12 ──┤
CAM13 ──┤
CAM14 ──┤
CAM15 ──┤
CAM16 ──┘
```

Camera connectivity uses RTSP.

```text
RTSP
Port: 554
```

The camera network configuration and sanitized camera mapping are  
documented in `04-network.md`.

---

# Monitor Architecture

Each IP camera is represented as a monitor within Shinobi.

The conceptual structure is:

```text
Shinobi NVR
│
├── Monitor CAM01
├── Monitor CAM02
├── Monitor CAM03
├── Monitor CAM04
├── ...
└── Monitor CAM16
```

Each monitor may define its own:

- RTSP URL
    
- Stream profile
    
- Resolution
    
- FPS
    
- Codec
    
- Recording mode
    
- Storage location
    
- Motion detection configuration
    
- Audio configuration
    
- Retention policy
    

Monitor-specific values are intentionally not included here because the  
actual monitor configuration was not provided.

---

# Snapshot Configuration

The observed Shinobi configuration contains:

```text
doSnapshot: true
```

Snapshot functionality is therefore enabled at the application level.

Actual snapshot behavior depends on individual monitor configuration.

---

# Additional Features

The following Shinobi features were observed in the configuration:

|Feature|Status|
|---|---|
|Snapshot|Enabled|
|Discord Bot|Disabled|
|Drop-in Event Server|Disabled|
|FTP Server|Disabled|
|Face Manager|Disabled|
|Better P2P|Enabled|
|Wall Clock Timestamp|Enabled|

These settings should be reviewed after Shinobi upgrades because  
application behavior and configuration options may change between  
revisions.

---

# Service Management

No dedicated `shinobi.service` systemd unit was found in the current  
installation.

The following checks returned no Shinobi systemd service:

```bash
systemctl status shinobi
```

```bash
systemctl list-units --type=service | grep -i shinobi
```

Therefore, this documentation does not assume that Shinobi is managed  
through a systemd service named `shinobi`.

The exact process manager or startup mechanism should be documented  
separately if required.

MariaDB, however, is managed by systemd:

```text
mariadb.service
```

and is currently reported as:

```text
loaded active running
```

---

# Database Service Management

MariaDB is running as a system service:

```text
mariadb.service
```

Verification:

```bash
systemctl status mariadb
```

Basic service operations:

```bash
systemctl start mariadb
```

```bash
systemctl stop mariadb
```

```bash
systemctl restart mariadb
```

```bash
systemctl status mariadb
```

Database availability should be verified before troubleshooting  
Shinobi database-related failures.

---

# Shinobi Directory Structure

The main application directory contains the following components:

```text
/home/Shinobi
│
├── conf.json
├── package.json
├── package-lock.json
│
├── camera.js
├── cron.js
│
├── definitions/
├── fileBin/
├── languages/
├── libs/
├── node_modules/
├── plugins/
├── sql/
├── tools/
│
├── videos/
├── videos2/
│
└── web/
```

Important components include:

|Directory/File|Purpose|
|---|---|
|`conf.json`|Shinobi configuration|
|`package.json`|Node.js package definition|
|`node_modules/`|Installed Node.js dependencies|
|`libs/`|Shinobi application libraries|
|`plugins/`|Shinobi plugins|
|`sql/`|Database-related files|
|`web/`|Web interface|
|`videos/`|Local application video directory|
|`videos2/`|Additional local video directory|
|`camera.js`|Camera-related application component|
|`cron.js`|Scheduled task component|

---

# Resource Usage

The current LXC root filesystem is lightly utilized.

```text
Root Disk
63 GB total
2.6 GB used
57 GB available
5% utilization
```

The CCTV storage is currently:

```text
3.6 TB total
486 GB used
3.0 TB available
14% utilization
```

This indicates that the current recording storage has substantial  
remaining capacity.

Storage utilization should be monitored because continuous CCTV  
recording will gradually consume available disk space.

---

# Storage Monitoring

The primary storage verification command is:

```bash
df -h /mnt/storage
```

Example current state:

```text
Filesystem   Size  Used  Avail  Use%
/dev/sdc1    3.6T  486G  3.0T   14%
```

The root filesystem can be checked using:

```bash
df -h /
```

Both should be monitored independently.

---

# Backup Considerations

Shinobi contains two major categories of data:

### Application and Configuration

Important files include:

```text
/home/Shinobi/conf.json
/home/Shinobi/package.json
/home/Shinobi/package-lock.json
```

The MariaDB database also contains Shinobi configuration and application  
data.

### CCTV Recordings

CCTV recordings are stored on the dedicated recording storage.

```text
/mnt/storage
```

A backup strategy should therefore consider:

```text
Shinobi Configuration
        │
        ├── conf.json
        └── Application configuration
                 │
                 ▼
              Backup

MariaDB
   │
   ▼
Database Backup

CCTV Recordings
   │
   ▼
/mnt/storage
   │
   ▼
Optional Long-Term Archive
```

The recording data should not be treated as a substitute for a backup  
of the Shinobi database and configuration.

---

# Security Considerations

The following sensitive information must not be committed to the public  
repository:

- Database password
    
- Shinobi administrator credentials
    
- RTSP credentials
    
- Cron key
    
- SMTP credentials
    
- API keys
    
- Authentication tokens
    
- Production IP addresses
    
- Internal hostnames
    
- Camera identifiers containing sensitive information
    

Configuration files should be sanitized before being added to the  
documentation repository.

For example:

```text
db:
  host: 127.0.0.1
  user: shinobi
  password: [REDACTED]
  database: ccio
  port: 3306
```

---

# Configuration Verification

The following commands can be used to verify the Shinobi environment.

### Operating System

```bash
cat /etc/os-release
```

### Kernel

```bash
uname -a
```

### Node.js

```bash
node -v
```

### npm

```bash
npm -v
```

### FFmpeg

```bash
ffmpeg -version
```

![Keterangan](shinobi-ver.png)

---
# Operational Verification

A basic Shinobi verification procedure should confirm:

```text
1. LXC container is running
        │
        ▼
2. Ubuntu filesystem is accessible
        │
        ▼
3. MariaDB is running
        │
        ▼
4. Shinobi application is running
        │
        ▼
5. Shinobi Web UI is accessible
        │
        ▼
6. Camera monitors are online
        │
        ▼
7. RTSP streams are received
        │
        ▼
8. FFmpeg processes the streams
        │
        ▼
9. Recordings are generated
        │
        ▼
10. Recordings are stored correctly
```

---

# Troubleshooting Areas

Shinobi problems can generally be categorized into several layers.

|Layer|Typical Problem|
|---|---|
|LXC|Container unavailable|
|OS|Resource or filesystem problem|
|Node.js|Runtime or dependency failure|
|Shinobi|Application failure|
|MariaDB|Database unavailable|
|FFmpeg|Stream processing failure|
|Network|RTSP connectivity failure|
|Camera|Camera or stream unavailable|
|Storage|Recording disk unavailable/full|
|Authentication|Login or permission issue|

A systematic troubleshooting process should identify the affected layer  
before changing the configuration.

Detailed troubleshooting procedures are documented in  
`08-troubleshooting.md`.

---

# Current Environment Summary

|Component|Current Configuration|
|---|---|
|Container|LXC `100`|
|Hostname|`shinobi`|
|OS|Ubuntu `22.04.5 LTS`|
|Architecture|`x86_64`|
|Node.js|`18.20.8`|
|npm|`10.9.8`|
|Shinobi Path|`/home/Shinobi`|
|Git Branch|`master`|
|Git Commit|`f5cb53d1`|
|Database|MariaDB `10.6.23`|
|Database Name|`ccio`|
|Database Port|`3306`|
|FFmpeg|`4.4.2`|
|Shinobi Web Port|`8080`|
|Password Hash|SHA-256|
|Additional Storage|`hdd`|
|Storage Path|`/mnt/storage`|
|Storage Capacity|`3.6 TB`|
|Storage Used|`486 GB`|
|Storage Available|`3.0 TB`|
|Storage Usage|`14%`|
|Cameras|`16`|
|Camera Protocol|RTSP|
|RTSP Port|`554`|
|Shinobi systemd service|Not present|
|MariaDB systemd service|Active|

---

# Architecture Summary

The deployed Shinobi environment consists of a Node.js-based NVR  
application running inside an Ubuntu LXC container.

The application uses MariaDB for persistent application data and FFmpeg  
for video stream processing.

CCTV recordings are separated from the application root filesystem and  
stored on a dedicated high-capacity HDD mounted at `/mnt/storage`.

The resulting architecture is:

```text
                    Proxmox VE
                         │
                    LXC CT 100
                         │
                  Ubuntu 22.04.5
                         │
             ┌───────────┴───────────┐
             │                       │
             ▼                       ▼
        Shinobi NVR              MariaDB
        Node.js 18              10.6.23
             │
             ▼
          FFmpeg
          4.4.2
             │
             │ RTSP
             ▼
       16 IP Cameras
             │
             ▼
        Video Recording
             │
             ▼
        hdd storage
             │
             ▼
       /mnt/storage
             │
             ▼
          4 TB HDD
```

This architecture provides a centralized NVR platform while maintaining  
separation between the application environment, database, network  
connectivity, and CCTV recording storage.