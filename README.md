# Home Lab

Self-hosted services running on a single machine, virtualized with Proxmox.
Built incrementally and documented as I go, so any part can be rebuilt from this repo.

## The stack at a glance
- **Oracle** — the physical machine, running Proxmox VE (the hypervisor).
- **plex** — a Debian VM on Oracle that runs Docker and will host my services.
- Services run as Docker containers inside the `plex` VM.

## Docs
| File | What's in it |
|---|---|
| [docs/server-overview.md](docs/server-overview.md) | Full specs of the host + inventory of every VM and service (start here) |
| [docs/proxmox-oracle.md](docs/proxmox-oracle.md) | The Proxmox host "Oracle" — install, config, how to manage it |
| [docs/plex-vm.md](docs/plex-vm.md) | The Debian VM "plex" — setup, Docker, how it's configured |
| [docs/nas-fileshare.md](docs/nas-fileshare.md) | Planned: NAS / file share over the LAN (Samba) |

## Compose files
- [compose/plex/](compose/plex/) — Plex media server
- [compose/samba/](compose/samba/) — Samba file share (planned)

Each service folder has a `docker-compose.yml` and a `.env.example`. Copy the
example to `.env`, fill in real values, then `docker compose up -d`.

## ⚠️ Secrets
Never commit real passwords, tokens, or API keys. Those live in `.env` files,
which are git-ignored. Only the `.env.example` templates get committed.
