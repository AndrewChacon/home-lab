# Home Lab

Self-hosted media server built on Proxmox, running Jellyfin and a NAS with a web UI on a  Debian Virtual Machine with remote access over Tailscale. 
Media is ripped from physical DVDs.

## Stack at a glance

| Layer | Technology | Notes |
|---|---|---|
| Hypervisor | Proxmox VE | Bare metal on the host "Oracle" |
| Guest OS | Debian (Trixie) | VM named `plex` |
| Containers | Docker + Compose | Runs the media server |
| Media server | Jellyfin | Replaced Plex (see jellyfin doc) |
| Remote access | Tailscale | Private mesh VPN, no ports exposed |
| Ripping | MakeMKV + mkvtoolnix | On the Parrot desktop |

## Documentation index

1. [01 - Proxmox](01-proxmox.md)
2. [02 - VM storage](02-debian-vm-storage.md)
3. [03 - Jellyfin](03-jellyfin.md)
4. [04 - Tailscale](04-tailscale.md)
5. [05 - Ripping DVDs](05-ripping-dvds.md)
6. [06 - NAS](06-nas.md)

## Key addresses (fill in)

- Oracle (Proxmox) web UI: `https://<oracle-ip>:8006`
- plex VM LAN address: `192.168.4.163`
- plex VM Tailscale address: `100.73.24.65`
- Jellyfin: `http://<address>:8096`