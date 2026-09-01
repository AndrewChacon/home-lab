# Proxmox Host — "Oracle"

The bare-metal hypervisor. Everything else in the lab is a guest on top of this.

**Product:** Proxmox VE 9.2
**Hostname (FQDN):** oracle.home.arpa _(confirm what you set)_
**Web UI:** https://<oracle-ip>:8006 · log in as `root`
**Hardware:** see [server-overview.md](server-overview.md)

## How it was installed
1. Downloaded Proxmox VE 9.2 (amd64) ISO from proxmox.com, flashed to USB.
2. Enabled virtualization (VT-x) in BIOS and booted the installer.
3. Installed to the single 3.64 TB disk — **this wiped the previous ParrotOS install**
   (it was the only disk; nothing separate to preserve).
4. Set a static IP on the `/22` LAN, hostname `oracle`, root password.

## Post-install config
- **Repositories:** disabled the `enterprise` repo, enabled `no-subscription`
  (Datacenter → node → Updates → Repositories). Required for updates to work
  without a paid subscription.
- **Updates:** `apt update && apt dist-upgrade` (or Updates → Refresh/Upgrade in UI).
- The "No valid subscription" popup on login is expected on the free version — dismiss it.

## Managing it
- **Web UI** at https://<oracle-ip>:8006 for everything (VMs, storage, console).
- **Shell:** node → Shell in the UI, or `ssh root@<oracle-ip>`.
- Update routine: `apt update && apt dist-upgrade`, reboot if a new kernel lands.

## Storage note
Single 3.64 TB disk holds the Proxmox OS, VM disks, and (eventually) media.
There's no redundancy — it's one drive. Anything important should be backed up
off this machine.

## Gotchas
- Subnet is `/22`, not `/24` — use `192.168.4.x/22` when setting any static IP.
- Self-signed cert warning in the browser is normal; click through it.

## Links
- https://pve.proxmox.com/wiki/Downloads
- https://pve.proxmox.com/pve-docs/
