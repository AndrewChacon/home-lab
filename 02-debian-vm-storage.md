# 02 – Debian VM & Media Storage

> _Overview: the guest that runs the media server, and how its media disk was set up._

## Technology

- Debian running on proxmox
- VM name: `plex`, user: `plex`.
- A second virtual disk carved from Oracle's pool, mounted at `/mnt/data`

## Implementation — creating the VM

Created via the Proxmox VM wizard:

- 4 vCPU, 4 GB RAM, 32 GB OS disk, bridged `vmbr0`
- CPU type = `host`
- Minimal Debian install: SSH server enabled

### Base packages

```bash
sudo apt install -y curl git nano openssh-server
sudo systemctl enable --now ssh    # service is 'ssh' on Debian, NOT 'sshd'
```

## Implementation — adding & mounting the media disk

### Part 1 — add the disk (Proxmox UI)

- plex VM → Hardware → Add → Hard Disk
- Size allocated: 500 GB

### Part 2 — format, mount, persist (inside the VM)

```bash
lsblk                              # identify the new empty disk (was /dev/sdb) — VERIFY BY SIZE
sudo mkfs.ext4 /dev/sdb            # format ext4
sudo mkdir -p /mnt/data
sudo mount /dev/sdb /mnt/data      # temporary mount
sudo blkid /dev/sdb               # get the UUID for fstab
sudo nano /etc/fstab              # add the line below (use the real UUID)
```

fstab line (mount by UUID, not device name, so a rename can't break boot):

```
UUID=<uuid>  /mnt/data  ext4  defaults,nofail  0  2
```

Test the fstab entry before trusting it:

```bash
sudo systemctl daemon-reload      # clear the "fstab modified" systemd hint
sudo umount /mnt/data
sudo mount -a
df -h /mnt/data                   # should show the disk mounted
```

### Part 3 — ownership & folder layout

```bash
sudo chown -R plex:plex /mnt/data
mkdir -p /mnt/data/media/{movies,tv}
mkdir -p /mnt/data/share          # staged for the future Samba share
```

## Issues faced & fixes
  
- `openssh-server` wasn't installed, enabled it with `systemctl enable --now ssh`
- `curl: command not found`. Minimal install has no curl — installed it manually 
- systemd "fstab modified" hint. Harmless; cleared with `daemon-reload`