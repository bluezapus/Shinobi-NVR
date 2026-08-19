# RTSP CCTV

## Overview

The CCTV infrastructure consists of 16 IP cameras connected to the
production LAN and integrated with Shinobi NVR through the RTSP protocol.

Each camera provides a video stream that is consumed by a Shinobi
monitor. Shinobi uses FFmpeg to process the incoming streams and stores
recorded video on the dedicated CCTV storage.

> Production IP addresses, usernames, passwords, MAC addresses, and
> other sensitive infrastructure information are sanitized or excluded
> from this public documentation.

---

## CCTV Architecture

The RTSP video path is:

```text
IP Cameras
CAM01 - CAM16
      │
      │ RTSP
      │ TCP/UDP :554
      ▼
Network Switch
      │
      ▼
Proxmox Physical NIC
nic0
      │
      ▼
Proxmox Bridge
vmbr0
      │
      ▼
Shinobi LXC
CT 100
      │
      ▼
Shinobi NVR
      │
      ▼
FFmpeg
      │
      ▼
Recording Storage
/mnt/storage
```

The camera streams remain on the production LAN and are delivered to
the Shinobi NVR through the Proxmox network bridge.

---

## Camera Infrastructure

The current CCTV deployment consists of 16 IP cameras.

| Parameter | Configuration |
|---|---|
| Camera Count | `16` |
| Camera Model | TP-Link Tapo C220 |
| Protocol | RTSP |
| RTSP Port | `554` |
| Video Codec | H.264 |
| Target FPS | `15 FPS` |
| Network | Production LAN |
| NVR | Shinobi |
| NVR Container | LXC `100` |

Individual camera resolution, stream path, bitrate, audio configuration,
and recording settings should be verified from the actual camera and
Shinobi monitor configuration before being treated as final production
values.

---

## Network Topology

The cameras and Shinobi NVR communicate through the production LAN.

```text
                    Production LAN
                  192.168.10.0/24
                         │
                ┌────────┴────────┐
                │  Network Switch  │
                └───────┬─────────┘
                        │
                       nic0
                        │
                     vmbr0
                        │
                        ▼
                Shinobi LXC CT 100
                        │
                       eth0
                        │
                        ▼
                   Shinobi NVR
```

The production network is represented using sanitized addresses in this
document.

Detailed network configuration is documented in `04-network.md`.

---

## RTSP Protocol

RTSP (Real Time Streaming Protocol) is used by Shinobi to request and
receive video streams from the IP cameras.

The standard RTSP service port used by the cameras is:

```text
554
```

A typical connection follows this model:

```text
Shinobi
   │
   │ RTSP request
   ▼
IP Camera
   │
   │ Video stream
   ▼
Shinobi / FFmpeg
```

RTSP is responsible for stream session control, while the actual media
transport can use RTP over TCP or UDP depending on the camera and client
configuration.

---

## RTSP Connection Model

A sanitized RTSP URL generally follows this structure:

```text
rtsp://[USERNAME]:[PASSWORD]@[CAMERA-IP]:554/[STREAM-PATH]
```

Example:

```text
rtsp://[USERNAME]:[PASSWORD]@192.168.10.222:554/[STREAM02]
```

Credentials must never be committed to the public repository.

For documentation purposes, use placeholders:

```text
[USERNAME]
[PASSWORD]
[CAMERA-IP]
[STREAM-PATH]
```

---

## Camera-to-Shinobi Mapping

Each physical camera is represented by a monitor in Shinobi.

```text
Camera              Shinobi Monitor

CAM01  ───────────► Monitor CAM01
CAM02  ───────────► Monitor CAM02
CAM03  ───────────► Monitor CAM03
CAM04  ───────────► Monitor CAM04
CAM05  ───────────► Monitor CAM05
CAM06  ───────────► Monitor CAM06
CAM07  ───────────► Monitor CAM07
CAM08  ───────────► Monitor CAM08
CAM09  ───────────► Monitor CAM09
CAM10  ───────────► Monitor CAM10
CAM11  ───────────► Monitor CAM11
CAM12  ───────────► Monitor CAM12
CAM13  ───────────► Monitor CAM13
CAM14  ───────────► Monitor CAM14
CAM15  ───────────► Monitor CAM15
CAM16  ───────────► Monitor CAM16
```

---

## Camera Inventory

The following table provides the sanitized camera inventory for public
documentation.

| Camera | Sanitized IP     | Protocol |  Port | Role      |
| ------ | ---------------- | -------- | ----: | --------- |
| CAM01  | `192.168.10.222` | RTSP     | `554` | IP Camera |
| CAM02  | `192.168.10.223` | RTSP     | `554` | IP Camera |
| CAM03  | `192.168.10.224` | RTSP     | `554` | IP Camera |
| CAM04  | `192.168.10.225` | RTSP     | `554` | IP Camera |
| CAM05  | `192.168.10.226` | RTSP     | `554` | IP Camera |
| CAM06  | `192.168.10.227` | RTSP     | `554` | IP Camera |
| CAM07  | `192.168.10.228` | RTSP     | `554` | IP Camera |
| CAM08  | `192.168.10.229` | RTSP     | `554` | IP Camera |
| CAM09  | `192.168.10.230` | RTSP     | `554` | IP Camera |
| CAM10  | `192.168.10.231` | RTSP     | `554` | IP Camera |
| CAM11  | `192.168.10.232` | RTSP     | `554` | IP Camera |
| CAM12  | `192.168.10.233` | RTSP     | `554` | IP Camera |
| CAM13  | `192.168.10.234` | RTSP     | `554` | IP Camera |
| CAM14  | `192.168.10.235` | RTSP     | `554` | IP Camera |
| CAM15  | `192.168.10.236` | RTSP     | `554` | IP Camera |
| CAM16  | `192.168.10.237` | RTSP     | `554` | IP Camera |

> The IP addresses above are sanitized placeholders and do not represent
> production camera addresses.

---

## Stream Configuration

The known stream requirements are:

| Parameter | Target Configuration |
|---|---|
| Protocol | RTSP |
| Port | `554` |
| Video Codec | H.264 |
| Target FPS | `15 FPS` |
| Resolution | Camera-dependent |
| Stream Profile | To be verified |
| Bitrate | To be verified |
| Audio | To be verified |

The actual resolution and bitrate should be documented per camera after
verification.

---

## Main Stream and Sub Stream

Many IP cameras provide multiple stream profiles.

A common design is:

```text
IP Camera
│
├── Main Stream
│     └── Higher resolution / recording
│
└── Sub Stream
      └── Lower resolution / live view
```

The current documentation does not assume that all cameras use both
profiles.

The active stream profile should be verified from the Shinobi monitor
configuration.

If both profiles are used, document them separately for each camera.

---

## Shinobi Monitor Configuration

Each RTSP camera is configured as a monitor in Shinobi.

The conceptual configuration is:

```text
Shinobi Monitor
│
├── Connection Type
│
├── Camera Host
│
├── RTSP Port
│
├── Stream Path
│
├── Authentication
│
├── Video Codec
│
├── Resolution
│
├── FPS
│
├── Audio
│
└── Recording Configuration
```

Sensitive credentials should be stored only in the actual Shinobi
configuration and must not be included in public documentation.

---

## Recording Flow

The complete recording workflow is:

```text
IP Camera
    │
    │ RTSP
    ▼
Shinobi Monitor
    │
    ▼
FFmpeg
    │
    ▼
Video Processing
    │
    ▼
Recording
    │
    ▼
Shinobi Storage
    │
    ▼
/mnt/storage
    │
    ▼
Dedicated CCTV HDD
```

The dedicated recording storage is mounted inside the Shinobi LXC at:

```text
/mnt/storage
```

The Proxmox host exposes the physical CCTV storage through:

```text
/mnt/hddcctv
```

---

## Storage Relationship

The storage path is:

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

Current storage capacity observed from the Shinobi container:

| Parameter | Current State |
|---|---:|
| Capacity | `3.6 TB` |
| Used | `486 GB` |
| Available | `3.0 TB` |
| Utilization | `14%` |
| Filesystem | `ext4` |

---

## Network Requirements

For RTSP communication to work, the Shinobi container must be able to
reach each camera over the production LAN.

The minimum expected connectivity is:

```text
Shinobi
   │
   │ TCP/UDP :554
   ▼
Camera
```

Required network conditions include:

- Camera and Shinobi must have IP connectivity.
- RTSP port `554` must be reachable.
- Camera RTSP service must be enabled.
- Camera credentials must be valid.
- The configured RTSP path must be correct.
- No firewall rule may block the required RTSP traffic.

Detailed network addressing and bridge configuration are documented in
`04-network.md`.

---

## RTSP Bandwidth

Each camera generates continuous network traffic based on its configured
video bitrate.

Total CCTV bandwidth can be estimated using:

```text
Total Bandwidth ≈ Number of Cameras × Average Bitrate
```

For 16 cameras:

```text
Total Bandwidth ≈ 16 × Average Camera Bitrate
```

For example, if the average bitrate were 2 Mbps:

```text
16 × 2 Mbps = 32 Mbps
```

The actual bandwidth must be calculated from the real camera bitrate
configuration.

Recording storage requirements are similarly dependent on bitrate,
recording duration, and retention period.

---

## Storage Capacity Estimation

A simplified storage estimate can be calculated from the aggregate
bitrate.

```text
Daily Storage ≈
  Aggregate Bitrate × 86,400 seconds
```

For more practical calculations:

```text
Daily Storage (GB) ≈
  Bitrate (Mbps) × 10.8
```

Therefore:

```text
Daily Storage ≈
  Number of Cameras
  × Average Bitrate
  × 10.8
```

Example using 16 cameras at 2 Mbps:

```text
16 × 2 × 10.8
≈ 345.6 GB/day
```

This is an estimation. Actual disk consumption can differ because of
codec behavior, variable bitrate, audio, container overhead, recording
mode, and stream interruptions.

---

## Connectivity Testing

RTSP connectivity should be tested from the Shinobi LXC rather than
only from a workstation.

Basic network connectivity:

```bash
ping [CAMERA-IP]
```

RTSP port test:

```bash
nc -vz [CAMERA-IP] 554
```

If `nc` is not installed, another suitable TCP connectivity tool may be
used.

---

## FFprobe Testing

FFprobe can be used to inspect an RTSP stream.

Sanitized example:

```bash
ffprobe \
  -rtsp_transport tcp \
  "rtsp://[USERNAME]:[PASSWORD]@[CAMERA-IP]:554/[STREAM-PATH]"
```

The command can be used to verify:

- Stream availability
- Video codec
- Resolution
- Frame rate
- Audio stream
- Stream metadata

Credentials should not be saved in shell history or public documentation.

---

## FFmpeg Stream Testing

A short FFmpeg test can be used to confirm that the RTSP stream can be
read successfully.

Example:

```bash
ffmpeg \
  -rtsp_transport tcp \
  -i "rtsp://[USERNAME]:[PASSWORD]@[CAMERA-IP]:554/[STREAM-PATH]" \
  -t 10 \
  -f null -
```

A successful test should show that FFmpeg is able to open and process
the stream without persistent connection or decoding errors.

This test does not create a recording file.

---

## RTSP Transport

RTSP streams can use different media transport mechanisms.

TCP is generally useful for environments where predictable transport
through network infrastructure is preferred.

Example:

```bash
ffprobe \
  -rtsp_transport tcp \
  "rtsp://[USERNAME]:[PASSWORD]@[CAMERA-IP]:554/[STREAM-PATH]"
```

The actual transport mode should match the camera and Shinobi monitor
configuration.

The public documentation does not assume a single transport mode for
all production cameras unless it has been verified.

---

## Camera Verification Procedure

Each camera should be verified using the following sequence:

```text
1. Verify camera power
        │
        ▼
2. Verify camera network connectivity
        │
        ▼
3. Verify camera IP address
        │
        ▼
4. Verify RTSP service
        │
        ▼
5. Verify RTSP credentials
        │
        ▼
6. Verify RTSP stream path
        │
        ▼
7. Test RTSP with FFprobe
        │
        ▼
8. Test stream with FFmpeg
        │
        ▼
9. Verify Shinobi monitor
        │
        ▼
10. Verify recording
```

---

## Shinobi Monitor Verification

After confirming the RTSP stream independently, verify the corresponding
Shinobi monitor.

Verification should include:

- Monitor is enabled.
- Monitor status is online.
- Live view displays correctly.
- Video timestamp advances normally.
- No repeated connection errors occur.
- Recording status is active when recording is expected.
- New recordings appear in the configured storage.
- Recorded footage can be played back.

---

## Troubleshooting

### Camera Offline

Check:

```bash
ping [CAMERA-IP]
```

If the camera cannot be reached, investigate:

- Camera power.
- Network cable or Wi-Fi connectivity.
- Camera IP address.
- Network switch.
- DHCP assignment.
- Routing.
- Firewall configuration.

---

### RTSP Port Unreachable

Test:

```bash
nc -vz [CAMERA-IP] 554
```

Possible causes include:

- RTSP disabled on camera.
- Incorrect camera IP.
- Camera service unavailable.
- Firewall blocking port `554`.
- Incorrect network path.

---

### Authentication Failure

Verify:

- RTSP username.
- RTSP password.
- Camera permissions.
- RTSP authentication settings.

Do not place credentials in public logs or documentation.

---

### Invalid Stream Path

If the RTSP connection reaches the camera but the stream cannot be
opened, verify the configured stream path.

Use FFprobe:

```bash
ffprobe \
  -rtsp_transport tcp \
  "rtsp://[USERNAME]:[PASSWORD]@[CAMERA-IP]:554/[STREAM-PATH]"
```

---

### Stream Opens but Video Does Not Display

Check:

- Video codec.
- Stream profile.
- Resolution.
- FPS.
- FFmpeg compatibility.
- Camera encoding configuration.
- Shinobi monitor input configuration.

---

### Recording Stops

Check the following in order:

```text
Camera
  │
  ▼
RTSP Connectivity
  │
  ▼
Shinobi Monitor
  │
  ▼
FFmpeg
  │
  ▼
Storage
```

Storage availability:

```bash
df -h /mnt/storage
```

If storage utilization is high, investigate recording retention and
storage management.

---

## Security Considerations

RTSP credentials provide access to camera video streams and must be
treated as sensitive information.

The following must not be committed to a public repository:

- RTSP username
- RTSP password
- Full RTSP URLs containing credentials
- Production camera IP addresses
- Camera MAC addresses
- Camera authentication tokens
- Shinobi API credentials

Use sanitized examples:

```text
rtsp://[USERNAME]:[PASSWORD]@[CAMERA-IP]:554/[STREAM-PATH]
```

Production configuration should remain in the actual Shinobi
environment or a secure secrets-management mechanism.

---

## Camera Configuration Template

The following template can be used for documenting each camera after
production verification.

| Parameter | Value |
|---|---|
| Camera ID | `CAM01` |
| Camera Name | `[LOCATION]` |
| Model | `TP-Link Tapo C220` |
| IP Address | `[SANITIZED-IP]` |
| Protocol | `RTSP` |
| Port | `554` |
| Stream Path | `[STREAM-PATH]` |
| Stream Profile | `[MAIN/SUB]` |
| Codec | `H.264` |
| Resolution | `[RESOLUTION]` |
| FPS | `15 FPS` |
| Bitrate | `[BITRATE]` |
| Audio | `[ENABLED/DISABLED]` |
| Shinobi Monitor | `CAM01` |
| Recording Mode | `[MODE]` |
| Storage | `hdd` |

---

## Camera Documentation Matrix

The following matrix is intended to be completed after verifying the
actual Shinobi monitor configurations.

| Camera | IP | RTSP | Codec | Resolution | FPS | Bitrate | Monitor |
|---|---|---:|---|---|---:|---:|---|
| CAM01 | `[SANITIZED]` | 554 | H.264 | To verify | 15 | To verify | CAM01 |
| CAM02 | `[SANITIZED]` | 554 | H.264 | To verify | 15 | To verify | CAM02 |
| CAM03 | `[SANITIZED]` | 554 | H.264 | To verify | 15 | To verify | CAM03 |
| CAM04 | `[SANITIZED]` | 554 | H.264 | To verify | 15 | To verify | CAM04 |
| CAM05 | `[SANITIZED]` | 554 | H.264 | To verify | 15 | To verify | CAM05 |
| CAM06 | `[SANITIZED]` | 554 | H.264 | To verify | 15 | To verify | CAM06 |
| CAM07 | `[SANITIZED]` | 554 | H.264 | To verify | 15 | To verify | CAM07 |
| CAM08 | `[SANITIZED]` | 554 | H.264 | To verify | 15 | To verify | CAM08 |
| CAM09 | `[SANITIZED]` | 554 | H.264 | To verify | 15 | To verify | CAM09 |
| CAM10 | `[SANITIZED]` | 554 | H.264 | To verify | 15 | To verify | CAM10 |
| CAM11 | `[SANITIZED]` | 554 | H.264 | To verify | 15 | To verify | CAM11 |
| CAM12 | `[SANITIZED]` | 554 | H.264 | To verify | 15 | To verify | CAM12 |
| CAM13 | `[SANITIZED]` | 554 | H.264 | To verify | 15 | To verify | CAM13 |
| CAM14 | `[SANITIZED]` | 554 | H.264 | To verify | 15 | To verify | CAM14 |
| CAM15 | `[SANITIZED]` | 554 | H.264 | To verify | 15 | To verify | CAM15 |
| CAM16 | `[SANITIZED]` | 554 | H.264 | To verify | 15 | To verify | CAM16 |

---
## Configuration Summary

| Component | Configuration |
|---|---|
| Camera Count | `16` |
| Camera Model | TP-Link Tapo C220 |
| Protocol | RTSP |
| RTSP Port | `554` |
| Video Codec | H.264 |
| Target FPS | `15 FPS` |
| NVR | Shinobi |
| NVR Container | LXC `100` |
| Network Bridge | `vmbr0` |
| Shinobi Interface | `eth0` |
| Recording Storage | `hdd` |
| Container Storage Path | `/mnt/storage` |
| Proxmox Storage Path | `/mnt/hddcctv` |
| Storage Filesystem | `ext4` |
| Storage Capacity | `3.6 TB` |

---

## Notes

This document describes the RTSP-based CCTV integration architecture
and the known camera requirements.

Camera-specific values such as resolution, bitrate, stream path,
audio configuration, and exact production IP addresses should be
updated after verification from the actual camera and Shinobi monitor
configuration.

Production credentials and sensitive infrastructure information must
remain outside the public repository.