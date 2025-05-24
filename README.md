- [Introduction](#introduction)
- [How to Use](#how-to-use)
  - [Docker Compose](#docker-compose)
  - [Portainer Stack](#portainer-stack)
  - [Manual qBittorrent Configurations](#manual-qbittorrent-configurations)
    - [Mandatory: Set Network Interface to VPN](#mandatory-set-network-interface-to-vpn)
    - [Optional: Port Forwarding](#optional-port-forwarding)
- [What's Included?](#whats-included)
- [Networking](#networking)
- [Notes](#notes)
  - [Why are Radarr and Sonarr volumes not mounted remotely?](#why-are-radarr-and-sonarr-volumes-not-mounted-remotely)


# Introduction
Collection of secure deployments for torrent-based digital video recorders (DVR) for movies and TV shows.

The following containers are used in this solution:

| Service | Behind VPN? | HTTP Endpoint | Docker Port | Purpose | Official Docs |
|---|:---:|---|:---:|---|---|
|Gluetun|N/A|N/A|N/A|VPN container|https://github.com/qdm12/gluetun-wiki|
|Traefik Dashboard|No|http://your-domain|8080|Reverse Proxy|https://doc.traefik.io/|
|WatchTower|No|N/A|N/A|Auto-updates containers|https://github.com/containrrr/watchtower| 
|qBittorrent|Yes|http://torrents.your-domain|5555|Bittorrent client|https://docs.linuxserver.io/images/docker-qbittorrent|
|Radarr|Yes|http://movies.your-domain|7878|Monitors Movies|https://docs.linuxserver.io/images/docker-radarr|
|Sonarr|Yes|http://tv.your-domain|8989|Monitors TV shows|https://docs.linuxserver.io/images/docker-sonarr|
|Bazarr|Yes|http://subtitles.your-domain|6767|Manages subtitles|https://docs.linuxserver.io/images/docker-bazarr|
|Jackett|Yes|http://jackett.your-domain|9117|Torrent site proxy for Sonarr/Radarr |https://docs.linuxserver.io/images/docker-jackett|
|Flaresolverr|Yes|http://flaresolverr.your-domain|8191|Fix for torrent sites using CloudFlare proxy|https://github.com/FlareSolverr/FlareSolverr|

# How to Use

:warning: **Default Traefik configuration only supports HTTP**

:warning: **Make sure all your mounts are properly mounted.**

## Docker Compose

1. Clone this repo
1. Change working directory to `/docker-compose/`
1. Rename `.env.example` to `.env` file and modify as needed
1. Run `docker compose up -d`
1. Review [Manual qBittorrent Configurations](#manual-qbittorrent-configurations)
1. Visit each HTTP endpoint and configure the apps

## Portainer Stack

1. Open `Portainer -> Stacks -> Add Stack`
2. Enter a Name (i.e. `torrent-dvr`)
3. Click `Repository`
4. Repository URL = `https://github.com/Sumtin/torrent-dvr/`
5. Repo Reference = `refs/heads/main`
6. Compose Path = `docker-compose/docker-compose.yml`
7. Under Environment Variables, select `Advanced Mode`
8. Copy entire contents of [`/docker-compose/.env.example`](https://github.com/Sumtin/torrent-dvr/blob/main/docker-compose/.env.example) into text area.
9. Update environment variables as needed
10. Create the Stack
11. Review [Manual qBittorrent Configurations](#manual-qbittorrent-configurations)
12. Visit each HTTP endpoint and configure the apps!

## Manual qBittorrent Configurations

### Mandatory: Set Network Interface to VPN

1. From qBittorrent, `Tools -> Options -> Advanced`.
2. Change `Network interface` to `tun0`.

### Optional: Port Forwarding

If you're using VPN Port Forwarding, follow [the Gluetun wiki](https://github.com/qdm12/gluetun-wiki/blob/main/setup/advanced/vpn-port-forwarding.md) for enabling in qBitorrent.

# Networking

- Default Traefik config only supports HTTP (see [Traefik docs](https://doc.traefik.io/) for HTTPS)
- All containers except for Traefik and WatchTower are routed through the Gluetun container to ensure privacy.  
- Gluetun container must expose ports for child containers.
- Gluetun and Traefik share a bridge network as required by Traefik.
- qBittorrent requires some network configs that [must be done manually](#manual-qbittorrent-configurations).
- DNS must be configured manually and is beyond the scope of this wiki.

# Notes
## Why are Radarr and Sonarr volumes not mounted remotely?
Sonarr and Radarr both use SQLite and it doesn't play nice when accessed over a network. 
https://access.redhat.com/solutions/120733
