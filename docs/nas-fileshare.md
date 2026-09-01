# NAS / File Share (planned)

Goal: share a folder over the home network so any device — laptop, phone, another
PC — can read and write files without cables or USB sticks. Runs as a Samba
container inside the `plex` VM, pointed at storage on Oracle's big pool.

**Status:** 📋 planned
**Where:** Docker container in the `plex` VM
**Protocol:** SMB (ports 139, 445)

## Prerequisite
Same as Plex: the `plex` VM needs real storage from Oracle's 3.64 TB pool mounted
inside it. Once that mount exists, point both Plex (`MEDIA_ROOT`) and Samba
(`SHARE_ROOT`) at folders on it.

## Plan
1. Add / reuse the large virtual disk on the `plex` VM, mount it (e.g. `/mnt/data`).
2. `cp compose/samba/.env.example .env`, set `SHARE_ROOT`, user, and password.
3. `docker compose up -d` in `compose/samba/`.
4. Connect from other devices:
   - **Windows:** `\\<vm-ip>\share`
   - **macOS:** Finder → Cmd+K → `smb://<vm-ip>/share`
   - **Linux / phone:** `smb://<vm-ip>/share`

## Alternative
For a single share, native Samba (`apt install samba` + edit `/etc/samba/smb.conf`)
is simpler than a container. The container keeps everything reproducible in this
repo — pick whichever you prefer when you get here.

## Notes to fill in later
- Which folder(s) to share, and read-only vs read-write.
- Whether media and file-share point at the same disk or separate folders.

## Links
- https://github.com/dperson/samba
