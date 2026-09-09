# 04 – Remote Access with Tailscale

This is how we reach the media server from outside the home without exposing anything to the public internet

## Technology

- Tailscale — private mesh VPN. Devices join a "tailnet" and reach each other by
  stable `100.x.x.x` addresses from anywhere, as if on the same LAN.
- Chosen over port forwarding: nothing is exposed publicly, no router config, free for personal use.

## Implementation

### On the server (plex VM)

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up            # prints a URL — open it and sign in
tailscale ip -4              # get this device's 100.x address
```

### On my mobile device (Apple iPhone)

1. Install the Tailscale app, sign in with the same account as the server.
2. Install the Jellyfin app
3. Add server in Jellyfin using the **Tailscale** address, not the LAN address:

```php
http://100.x.x.x:8096
```

### Verify the tailnet

```bash
tailscale status
```

Both devices should appear under the same account

```
100.x.x.x  plex    <account>  linux
100.x.x.x  iphone  <account>  iOS
```

## Key point

Use the **`100.x` Tailscale address everywhere** (cellular and home WiFi both)
The LAN address (`192.168.x.163`) only works at home
the Tailscale address works from anywhere.

## Issues faced & fixes

- Worked on WiFi, failed on cellular. Fix: repoint the app to the
  server's Tailscale IP (`100.x.x.x`).
- Tailscale had to be installed on the server too, signed into the same account, or the two can't see each other.