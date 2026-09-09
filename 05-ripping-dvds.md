# 05 – Ripping DVDs

## Technology

- `MakeMKV` — reads/decrypts the disc, outputs as an `.mkv` file
- `mkvtoolnix` — inspect and split multi-episode titles
- `scp` — copy finished files from the Parrot desktop to the VM

## MakeMKV install (Parrot is Debian-based)

```bash
sudo apt install -y flatpak
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
flatpak install -y flathub com.makemkv.MakeMKV
flatpak run com.makemkv.MakeMKV
# if it can't see the drive:
flatpak override --user --device=all com.makemkv.MakeMKV   # silent on success
```

- Free while in beta; **beta key rotates ~monthly** — grab the current one from the
  official forum thread and enter via Help → Register.

## Ripping workflow

1. Insert disc → click the disc icon → MakeMKV scans and lists titles
2. Select titles:
   - Movie: one long title
   - TV: several similar-length titles = episodes
   - Each checked title becomes its own file
3. Set output folder → Make MKV

## Naming for Jellyfin

Movies — folder + file both `Title (Year)`:

```
/mnt/data/media/movies/La La Land (2016)/La La Land (2016).mkv
```

TV — show / season / episode, with the `SxxExx` code:

```
/mnt/data/media/tv/Trigun (1998)/Season 01/Trigun (1998) - S01E01.mkv
```

The year (movies) and the `SxxExx` code (TV) are the parts that make matching work.

## Splitting a multi-episode title (when a disc authors episodes as ONE title)

Some discs (e.g. Trigun) put several episodes in a single title. Inspect and split
by chapters:

```bash
sudo apt install -y mkvtoolnix
mkvmerge -i "B1_t00.mkv"                                   # track list + chapter count
mkvextract "B1_t00.mkv" chapters - 2>/dev/null | grep ChapterTimeStart   # chapter times
```

Read the chapter timestamps to find episode boundaries (they repeat on a regular
pattern — e.g. Trigun disc 1 = 35 chapters, 5 per episode = 7 episodes, boundaries at
chapters 6, 11, 16, 21, 26, 31). Then split on those chapter numbers:

```bash
mkvmerge -o "Trigun.mkv" --split chapters:6,11,16,21,26,31 "B1_t00.mkv"
# -> Trigun-001.mkv ... Trigun-007.mkv
```

Rename in order (episode count **continues** across discs — disc 2 starts at E08):

```bash
mv Trigun-001.mkv "Trigun (1998) - S01E01.mkv"
# ...etc
```

Spot-check episode 2 plays from its real start before trusting all cuts.

## Copying files to the server

`scp` runs on the laptop (not inside the SSH session). Make the destination folder
on the server first:

```bash
# on the server (SSH):
mkdir -p "/mnt/data/media/movies/La La Land (2016)"

# on the laptop:
scp "/path/on/laptop/file.mkv" \
  "plex@192.168.4.163:/mnt/data/media/movies/La La Land (2016)/La La Land (2016).mkv"

# verify on the server:
ls -lah "/mnt/data/media/movies/La La Land (2016)/"
```

Copy-and-rename happens in one step: source is the ugly ripped name, destination is the
proper Jellyfin name.

## Issues faced & fixes

- "Failed to open disc" (The Matrix). Dual-layer disc (`Number of layers: 2 (OTP)`)
  that the slim combo drive (`hp CDDVDW TS-U633J`) couldn't read past ~20 min (the layer
  break). The disc itself was fine (played on PS5) — the **slim drive was the bottleneck**.
- **One giant file instead of episodes.** Selected the "Play All" title, or the disc
  authored episodes as a single title (then split by chapters, above).