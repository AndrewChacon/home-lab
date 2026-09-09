# 01 – Proxmox Host (Oracle)

## Technology

- Proxmox VE (Virtual Environment) - an open source hypervisor, an OS whose job is to run other systems on top of it. Lets us carve up the physical hardware into isolated VMs and containers, each behave like its own independent computer. Everything is managed through a web interface we can access in the browser. 
- Advantages: 
	- consolidation
	- backups
	- isolation
	- resource control
	- free 
- Trade off - virtualization adds more abstraction 
- Host name -  `Oracle`

## Hardware

| Component   | Spec                                                            |
| ----------- | --------------------------------------------------------------- |
| Motherboard | Gigabyte Z270X-Ultra Gaming                                     |
| CPU         | Intel Core i7-7700K (4c/8t, has Quick Sync via HD Graphics 630) |
| RAM         | 32 GB                                                           |
| Storage     | single 3.64 TiB disk                                            |
| Network     | `192.168.x.x/22`                                                |

## Implementation

1. Download the ISO file for Proxmox VE 
2. Create a flashed USB installer with `balenaEtcher`
3. Install to the single disk (will wipe the entire disk)
4. Set the static IP during the installation process
5. Access the web UI at `https://<oracle-ip>:8006`, logged in as `root`