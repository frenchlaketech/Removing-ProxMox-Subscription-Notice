# Proxmox Arr Stack: Container Spec Plan

## 1. Host hardware

| Item | Detail |
| --- | --- |
| Machine | HP EliteDesk SFF, KBL/Q270 board (7th-gen Intel, Quick Sync iGPU) |
| RAM | 24 GB |
| Boot / fast storage | 1 TB NVMe |
| Bulk storage | 16 TB of NAS drives (assumed to be mounted into Proxmox over NFS) |
| GPU | None added. The 1070 will not fit or be powered in an SFF case; the Intel iGPU handles transcoding. |

## 2. Layout

Three guests, kept deliberately small:

1. **AdGuard Home (LXC)**: isolated so DNS stays up when the media stack is rebooted or rebuilt.
2. **Docker host (LXC or VM)**: runs Sonarr, Radarr, Prowlarr, Bazarr, Seerr, the download client, and Gluetun in one Compose project.
3. **Jellyfin (LXC)**: separate so the iGPU can be passed in cleanly and the media server can be restarted independently.

An optional fourth LXC runs a WireGuard server for remote access.

## 3. Container specs

| Guest | Type | vCPU | RAM | Root disk | Notes |
| --- | --- | --- | --- | --- | --- |
| AdGuard Home | LXC (unprivileged) | 1 | 512 MB | 4 GB | Static IP; this is the DNS server handed out by DHCP |
| Docker host | LXC (nesting on) or VM | 4 | 8 GB | 32 GB | Holds all arr configs; back up the config directories |
| Jellyfin | LXC | 4 | 4 GB | 16 GB | `/dev/dri` passed through; transcode dir on NVMe |
| WireGuard (optional) | LXC | 1 | 256 MB | 2 GB | Remote access into the LAN |

**Totals:** about 13 GB RAM allocated of 24 GB, leaving headroom for Proxmox itself and future services. vCPUs can be overcommitted since most of these sit idle.

### Inside the Docker host

| Service | Approx. RAM | Notes |
| --- | --- | --- |
| Sonarr / Radarr | 500 MB-1 GB each | Spikes during library scans |
| Prowlarr | 256-512 MB | Not routed through the VPN |
| Bazarr | \~512 MB |  |
| Seerr (Jellyseerr/Overseerr) | 512 MB-1 GB |  |
| qBittorrent / SABnzbd | 1-2 GB | Routed through Gluetun |
| Gluetun | \~100 MB | VPN client and kill switch |

## 4. Storage plan

- Mount the NAS share once on the Proxmox host, then bind-mount it into the Docker host and Jellyfin containers as `/data`.
- Use one tree so hardlinks and instant moves work: `/data/torrents`, `/data/usenet`, `/data/media/{movies,tv,music}`.
- Downloads and media must live on the same filesystem. Incomplete downloads may sit on the NVMe, but completed files and media stay on the NAS share.
- Keep app configs and databases on the NVMe.
- Use one PUID/PGID across all services, and set up the unprivileged-LXC UID/GID mapping before building anything.

## 5. Hardware transcoding (Jellyfin)

- Pass `/dev/dri/renderD128` and `card0` into the Jellyfin LXC and add the container user to the `render` group.
- Install `intel-opencl-icd` in the container for HDR tone mapping.
- Set Jellyfin hardware acceleration to VAAPI (or QSV) and enable HEVC and tone mapping.
- Verify with `vainfo` inside the container.
- Kaby Lake supports H.264/HEVC (including 10-bit) and VP9 decode. It does not support AV1.

## 6. Networking and VPN

- Give every guest a DHCP reservation or static IP.
- Route only the download client through the VPN using `network_mode: service:gluetun`. Sonarr, Radarr, Prowlarr, and Jellyfin stay off the VPN, since some indexers block VPN IPs.
- If Gluetun runs in an LXC, pass through `/dev/net/tun`.
- Point DHCP DNS at AdGuard; consider a second AdGuard instance elsewhere for redundancy.

## 7. Backups and maintenance

- Snapshot each guest before upgrades.
- Back up the config directories (Proxmox Backup Server works well). Media is replaceable; configs and databases are not.
- Review actual RAM and CPU use after a couple of weeks and trim allocations.

## 8. Suggested build order

1. Proxmox install, NAS mount, and the `/data` tree
2. AdGuard LXC and DHCP DNS change
3. Docker host with Gluetun and the download client; confirm the VPN IP and kill switch
4. Prowlarr, Sonarr, Radarr, Bazarr
5. Jellyfin LXC with iGPU passthrough
6. Seerr, optional WireGuard, then backups