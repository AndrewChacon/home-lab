# Debian VM — "plex"

The workhorse VM that runs Docker. All service containers live in here.

**OS:** Debian 13 (Trixie)
**Runs on:** Oracle (Proxmox) · bridged on `vmbr0` (real LAN)
**Resources:** 4 vCPU · 8 GB RAM · 32 GB OS disk
**CPU type:** `host` (exposes i7-7700K Quick Sync for Plex transcoding)
**Access:** `ssh plex@<vm-ip>`
**Login user:** `plex` (in `sudo` and `docker` groups)

## How it was set up
1. Created the VM in Proxmox, booted the Debian 13 netinst ISO.
2. Minimal install: **no desktop environment**, **SSH server** enabled,
   standard system utilities.
3. Added `plex` to sudo: `su -`, then `usermod -aG sudo plex`, then re-login.
4. Enabled SSH: `systemctl enable --now ssh` (service is `ssh`, not `sshd` on Debian).
5. Installed prerequisites: `apt install -y curl git nano`
   (minimal Debian ships without curl).

## Docker install
Installed via the official convenience script — pulls the engine **and** the
Compose v2 plugin together:
```bash
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker plex     # run docker without sudo
# log out and back in for the group to take effect
```
Verify:
```bash
docker run hello-world
docker compose version           # v2 syntax: `docker compose`, not `docker-compose`
```

## Gotchas (things that actually bit me)
- **`plex` not in sudoers:** the plain Debian installer doesn't auto-add your user
  to sudo. Fix via `su -` then `usermod -aG sudo plex`.
- **docker.sock permission denied:** group membership only loads on a *fresh* login.
  `newgrp docker` pulls it into the current shell without a reconnect.
  (You stay the `plex` user — this is NOT running Docker as root.)
- **docker group ≈ root:** anyone in the docker group can escalate to root via Docker.
  Fine for a single-admin LAN box; use rootless Docker if that ever matters.
- Change the `plex`/`plex` password before this box is ever exposed remotely (`passwd`).

## What runs here
See [server-overview.md](server-overview.md) for the live service list. Compose
files live in [../compose/](../compose/); deploy with `docker compose up -d` from
each service folder.
