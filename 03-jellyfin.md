# 03 – Plex (Abandoned) → Jellyfin

Overview: started on Plex, hit a wall, switched to Jellyfin

## Technology

- Docker + Docker Compose to run the media server as a container.
- Plex (linuxserver image) — attempted first, abandoned.
- Jellyfin (linuxserver image) — the working solution. Free, open-source, no
  paywall for remote streaming, free hardware transcoding

## Docker install (on the plex VM)

Run as the unprivileged `plex` user via the docker group (not root/sudo).

```bash
curl -fsSL https://get.docker.com | sh   # installs engine + Compose v2 plugin together
sudo usermod -aG docker plex             # add user to docker group
newgrp docker                            # load the group into the current shell (does NOT elevate to root)
docker run hello-world                   # verify
docker compose version                   # verify Compose
```

## Why Plex was abandoned

- I was not aware that Plex required a subscription for remote viewing, defeats the whole purpose of this project. 
- Jellyfin solved this issue, open source and completely free to host my media for local and remote streaming

### What carried over unchanged when switching

- Storage layout `/mnt/data/media` with `movies/` and `tv/`
- The DVD ripping workflow
- File naming conventions (`Title (Year)`, `SxxExx`)

What changed: the container image, the port (**8096** vs 32400), and a setup wizard
reached directly (no token).

## Implementation — Jellyfin

### Compose file

`compose/jellyfin/docker-compose.yml`:

```yaml
services:
  jellyfin:
    image: lscr.io/linuxserver/jellyfin:latest
    container_name: jellyfin
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=${TZ}
    volumes:
      - ./config:/config
      - ${MEDIA_ROOT}:/media
    ports:
      - 8096:8096
    restart: unless-stopped
```

`.env.example`:

```
TZ=America/New_York
MEDIA_ROOT=/mnt/data/media
```

### Launch

```bash
cd ~/home-lab/compose/jellyfin
cp .env.example .env        # REQUIRED — compose reads a file literally named .env
docker compose up -d
docker compose ps           # confirm 'jellyfin' is Up
```

Then browse to `http://192.168.4.163:8096` → setup wizard → create admin →
add libraries
Also accessible by `http://plex:8096`

### Library paths

Point libraries at the container paths, not the host paths:

- Movies library → `/media/movies`  (content type: Movies)
- TV library → `/media/tv`  (content type: Shows)

Not `/mnt/data/media/...` — Jellyfin only sees inside the container, where the disk
is mounted at `/media`.

### Force a rescan after adding files

Dashboard → Scheduled Tasks → **Scan All Libraries**.

## Streaming quality note

The client quality dropdown tops out based on the source file bitrate, not a
server/VM limit. DVD rips are ~5–8 Mbps by the format's own ceiling, so 8 Mbps is
the top option — that's full DVD quality, not throttling

## Issues faced & fixes

- Plex "Not authorized" from another machine. Unclaimed server; localhost-only
  claim. (Superseded by moving to Jellyfin.)
- `permission denied /var/run/docker.sock`. docker group only loads at login —
  `newgrp docker` or re-login. (`newgrp` loads the group, does not make you root.)
- **Library folder looked empty in the wizard.** Used host path instead of the
  container path `/media/...`.