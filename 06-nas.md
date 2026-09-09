# 06 – NAS: File Share (Samba + FileBrowser)

## Technology

- Samba (`dockurr/samba`) — native SMB network share; mounts as a drive on my Linux machine 
- FileBrowser Quantum — web UI for the same folder  (upload/download in a browser, generate share links).
- Tailscale + MagicDNS — reach everything by the single hostname `plex` instead of
  juggling LAN vs Tailscale IPs
- Both services run as Docker Compose stacks on the `plex` VM, pointed at the same
  `/mnt/data/share` folder owned by the user `plex`

## Implementation — SMB share (Samba)

Stack at `~/home-lab/samba/`. Single share `share` → `/mnt/data/share` (internal
`/storage`), UID/GID 1000, port 445, SMB1 disabled. Auth user `prime` (password in
`.env`).

```bash
cd ~/home-lab/samba
cp .env.example .env        # set the SMB password (never commit .env)
docker compose up -d
docker compose ps           # confirm the container is Up
```

### Verify

```bash
smbclient -L //192.168.4.163 -U prime      # list shares from the VM
```

Verified working from: `smbclient` on the VM, Parrot GUI (`smb://`), a persistent
fstab CIFS mount, and the iPhone Files app.

### Persistent mount on the Parrot desktop

Credentials file `/etc/cifs-credentials` (root-owned, `chmod 600`):

```
username=prime
password=<smb-password>
```

fstab line:

```
//192.168.4.163/share  /mnt/nas  cifs  credentials=/etc/cifs-credentials,uid=1000,gid=1000,iocharset=utf8,vers=3.0,nofail,x-systemd.automount,_netdev  0  0
```

> _With MagicDNS you can use `//plex/share` in place of the IP._

## Implementation — Web UI (FileBrowser Quantum)

Stack at `~/home-lab/compose/filebrowser/`. User `1000:1000`, port `8082:80`.

- Volumes: `./data:/home/filebrowser/data` and `/mnt/data/share:/media`
- Config `data/config.yaml`: source `share` → `/media`
- Admin password via `FILEBROWSER_ADMIN_PASSWORD` (`.env`), `TZ=America/New_York`
- Image `gtstef/filebrowser:1.5-stable` 

```bash
cd ~/home-lab/compose/filebrowser
cp .env.example .env        # set FILEBROWSER_ADMIN_PASSWORD
docker compose up -d
```

### Access

- `http://plex:8082` (over Tailscale/MagicDNS — works on the phone)
- `http://192.168.4.163:8082` (LAN)

Verified: upload + generate share link + download on another device.

## Remote access

MagicDNS is on — use **`plex` as the single hostname everywhere**
(`smb://plex/share`, `http://plex:8082`) instead of juggling LAN vs Tailscale IPs.
Off-LAN (cellular), you must use the Tailscale hostname/IP, not the LAN IP.

## Issues faced & fixes

- `mount error(13)` at mount time — wrong/placeholder credentials in
  `/etc/cifs-credentials`. Fix: write the correct user/pass, `chmod 600`.
- Samba `guest ... ACCESS_DENIED` log lines — normal anonymous probes; always
  connect as a **Registered User**, not guest.
- Off-LAN access failing — using the LAN IP on cellular; use the Tailscale
  hostname/IP.

## Limits

- FileBrowser share links only resolve for devices already on the tailnet.
  Public/external sharing would need **Tailscale Funnel** or a reverse proxy —
  deliberately deferred.

## Notes for future me

- No redundancy: everything sits on the single disk on Oracle, need to implement backups and snapshots, as well as a system to create a backup that exists externally of the system 
- The runbook that predates the Quantum switch (`nas-smb-share.md`) still references
  the archived FileBrowser image and needs updating to match this doc.
