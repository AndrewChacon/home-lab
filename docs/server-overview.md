# Server Overview

The single source of truth for the whole lab: what the hardware is, and everything running on it.

## Physical host — "Oracle"
| Component | Detail |
|---|---|
| Role | Proxmox VE hypervisor (hosts all VMs) |
| Motherboard | Gigabyte Z270X-Ultra Gaming |
| CPU | Intel Core i7-7700K — 4 cores / 8 threads @ 4.5 GHz |
| iGPU | Intel HD Graphics 630 (Quick Sync — usable for Plex HW transcoding) |
| RAM | 32 GB |
| Storage | 1 × 3.64 TB disk (single pool — OS + VM storage + media all share it) |
| Discrete GPU | None |
| Hypervisor | Proxmox VE 9.2 |
| Filesystem @ install | _TBD — fill in what you picked (ext4/LVM-thin default, or ZFS)_ |

## Network
Subnet is a **/22** (note: not the usual /24), gateway most likely `192.168.4.1`.

| Host | Role | IP | Access |
|---|---|---|---|
| Oracle | Proxmox hypervisor | _192.168.4.x — fill in_ | https://<ip>:8006 |
| plex | Debian VM, Docker host | _192.168.4.x — fill in_ | `ssh plex@<ip>` |

## Guests (VMs) on Oracle
| Name | OS | vCPU | RAM | Disk | Purpose | Status |
|---|---|---|---|---|---|---|
| plex | Debian 13 (Trixie) | 4 | 8 GB | 32 GB | Runs Docker; hosts services | ✅ running |

_CPU type is set to `host` on the plex VM so the i7's Quick Sync is exposed for later Plex transcoding._

## Services (containers inside the plex VM)
| Service | Container host | Status | Doc |
|---|---|---|---|
| Docker + Compose | plex VM | ✅ installed | [plex-vm.md](plex-vm.md) |
| Plex | plex VM | 📋 planned | [../compose/plex/](../compose/plex/) |
| Samba (NAS) | plex VM | 📋 planned | [nas-fileshare.md](nas-fileshare.md) |

Legend: ✅ done · 🚧 in progress · 📋 planned

## Topology
```
                Home LAN (192.168.4.0/22, gw 192.168.4.1)
                              │
                     ┌────────┴────────┐
                     │  Oracle (host)  │   Proxmox VE 9.2
                     │  i7-7700K·32GB  │   https://<ip>:8006
                     └────────┬────────┘
                              │ vmbr0 (bridge)
                     ┌────────┴────────┐
                     │   plex (VM)     │   Debian 13 + Docker
                     │   Debian·Docker │   ssh plex@<ip>
                     └────────┬────────┘
                              │  docker containers
                   ┌──────────┼──────────┐
                 [Plex]     [Samba]     (future services)
```

## Known gaps / next steps
- [ ] Fill in the two static IPs above and the install filesystem.
- [ ] **Media storage:** the plex VM only has a 32 GB OS disk. Media must come from
      Oracle's 3.64 TB pool — add a large virtual disk to the VM and mount it as the
      Plex `MEDIA_ROOT`. This is the prerequisite before Plex can serve anything.
- [ ] Bring up Plex (compose ready).
- [ ] Set up Samba file share.
