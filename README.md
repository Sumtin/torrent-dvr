- [Introduction](#introduction)
- [How to Use](#how-to-use)
  - [Docker Compose](#docker-compose)
  - [Portainer Stack](#portainer-stack)
  - [Manual qBittorrent Configurations](#manual-qbittorrent-configurations)
    - [Set Network Interface to VPN - MANDATORY](#set-network-interface-to-vpn---mandatory)
    - [Port Forwarding - OPTIONAL](#port-forwarding---optional)
- [Notes](#notes)
  - [Networking](#networking)
  - [Why are Radarr/Sonarr/Bazarr volumes not mounted remotely?](#why-are-radarrsonarrbazarr-volumes-not-mounted-remotely)


# Introduction
Docker Compose deployment for torrent-based digital video recorders (DVR) for movies and TV shows.

The following containers are used in this solution:

| Service | Behind VPN? | HTTP Endpoint | Docker Port | Purpose | Official Docs |
|---|:---:|---|:---:|---|---|
|Gluetun|N/A|N/A|N/A|VPN container|https://github.com/qdm12/gluetun-wiki|
|Traefik Dashboard|No|http://domain|8080|Reverse Proxy|https://doc.traefik.io/|
|WatchTower|No|N/A|N/A|Auto-updates containers|https://github.com/containrrr/watchtower| 
|qBittorrent|Yes|http://torrents.domain|5555|Bittorrent client|https://docs.linuxserver.io/images/docker-qbittorrent|
|Radarr|Yes|http://movies.domain|7878|Monitors Movies|https://docs.linuxserver.io/images/docker-radarr|
|Sonarr|Yes|http://tv.domain|8989|Monitors TV shows|https://docs.linuxserver.io/images/docker-sonarr|
|Bazarr|Yes|http://subtitles.domain|6767|Manages subtitles|https://docs.linuxserver.io/images/docker-bazarr|
|Jackett|Yes|http://jackett.domain|9117|Torrent site proxy for Sonarr/Radarr |https://docs.linuxserver.io/images/docker-jackett|
|Flaresolverr|Yes|http://flaresolverr.domain|8191|Fix for torrent sites using CloudFlare proxy|https://github.com/FlareSolverr/FlareSolverr|

# How to Use

:warning: **Default Traefik configuration only supports HTTP**

:warning: **Make sure all your mounts are properly mounted.**

## Docker Compose

1. Clone this repo
2. Create mounts! (See `MOUNT_TO_...` configs in [`.env.example`](ttps://github.com/Sumtin/torrent-dvr/blob/main/docker-compose/.env.exam) file)
3. Change working directory to `/docker-compose/`
4. Rename `.env.example` to `.env` file and modify as needed
6. Run `docker compose up -d`
7. Review [Manual qBittorrent Configurations](#manual-qbittorrent-configurations)
8. Visit each HTTP endpoint and configure the apps

## Portainer Stack

1. Create mounts! (See `MOUNT_TO_...` configs in [`.env.example`](ttps://github.com/Sumtin/torrent-dvr/blob/main/docker-compose/.env.exam) file)
2. Open `Portainer -> Stacks -> Add Stack`
3. Enter a Name (i.e. `torrent-dvr`)
4. Click `Repository`
5. Repository URL = `https://github.com/Sumtin/torrent-dvr/`
6. Repo Reference = `refs/heads/main`
7. Compose Path = `docker-compose/docker-compose.yml`
8. Under Environment Variables, select `Advanced Mode`
9. Copy entire contents of [`/docker-compose/.env.example`](https://github.com/Sumtin/torrent-dvr/blob/main/docker-compose/.env.example) into text area.
10. Update environment variables as needed
11. Create the Stack
12. Review [Manual qBittorrent Configurations](#manual-qbittorrent-configurations)
13. Visit each HTTP endpoint and configure the apps!

## Manual qBittorrent Configurations

### Set Network Interface to VPN - MANDATORY

1. From qBittorrent, `Tools -> Options -> Advanced`.
2. Change `Network interface` to `tun0`.

### Port Forwarding - OPTIONAL

If you're using VPN Port Forwarding, follow [the Gluetun wiki](https://github.com/qdm12/gluetun-wiki/blob/main/setup/advanced/vpn-port-forwarding.md) for enabling in qBitorrent.

:warning: If VPN service forwards to random port, qBittorrent must be updated any time Gluetun restarts.

# Notes

## Networking

- Default Traefik config only supports HTTP (see [Traefik docs](https://doc.traefik.io/) for HTTPS)
- All Internet-bound traffic is routed through the VPN container to ensure privacy, excluding Traefik and WatchTower.  
- Gluetun container must expose ports for child containers.
- Gluetun and Traefik share a bridge network as required by Traefik.
- DNS must be configured manually and is beyond the scope of this wiki.

## Why are Radarr/Sonarr/Bazarr volumes not mounted remotely?

Sonarr, Radarr, and Bazarr all use SQLite and it doesn't play nice when accessed over a network. 
https://access.redhat.com/solutions/120733
