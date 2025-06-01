- [Introduction](#introduction)
- [Installation](#installation)
  - [Docker Compose](#docker-compose)
  - [Portainer Stack](#portainer-stack)
- [Configuration](#configuration)
  - [Quick Start](#quick-start)
  - [qBittorrent](#qbittorrent)
    - [Set Network Interface to VPN - MANDATORY](#set-network-interface-to-vpn---mandatory)
    - [Port Forwarding - OPTIONAL](#port-forwarding---optional)
- [Notes](#notes)
  - [Networking](#networking)
  - [Why ARR some volumes not mounted remotely?](#why-arr-some-volumes-not-mounted-remotely)


# Introduction
Docker Compose deployment for torrent-based *ARR stack behind VPN.

The following containers are used in this solution:

| Service | Behind VPN? | HTTP Endpoint | Docker Port | Purpose | Official Docs |
|---|:---:|---|:---:|---|---|
|Gluetun|N/A|N/A|N/A|VPN container|https://github.com/qdm12/gluetun-wiki|
|Traefik Dashboard|No|http://domain:8080|8080|Reverse Proxy|https://doc.traefik.io/|
|WatchTower|No|N/A|N/A|Auto-updates containers|https://github.com/containrrr/watchtower| 
|qBittorrent|Yes|http://torrents.domain|5555|Bittorrent client|https://github.com/qbittorrent/qBittorrent/wiki|
|Radarr|Yes|http://movies.domain|7878|Monitors Movies|https://wiki.servarr.com/radarr|
|Sonarr|Yes|http://tv.domain|8989|Monitors TV shows|https://wiki.servarr.com/en/sonarr|
|Bazarr|Yes|http://subtitles.domain|6767|Manages subtitles|https://www.bazarr.media/|
|Prowlarr|Yes|http://indexers.domain|9117|Torrent indexer managager |https://wiki.servarr.com/en/prowlarr|

# Installation

:warning: **Default Traefik configuration only supports HTTP**

:warning: **Make sure all your mounts are properly mounted.**

## Docker Compose

1. Clone this repo
2. Create mounts! (See `MOUNT_TO_...` configs in [`.env.example`](ttps://github.com/Sumtin/torrent-dv-arr/blob/main/docker-compose/.env.exam) file)
3. Change working directory to `/docker-compose/`
4. Rename `.env.example` to `.env` file and modify as needed
6. Run `docker compose up -d`

## Portainer Stack

1. Create mounts! (See `MOUNT_TO_...` configs in [`.env.example`](ttps://github.com/Sumtin/torrent-dv-arr/blob/main/docker-compose/.env.exam) file)
2. Open `Portainer -> Stacks -> Add Stack`
3. Enter a Name (i.e. `torrent-dv-arr`)
4. Click `Repository`
5. Repository URL = `https://github.com/Sumtin/torrent-dv-arr/`
6. Repo Reference = `refs/heads/main`
7. Compose Path = `docker-compose/docker-compose.yml`
8. Under Environment Variables, select `Advanced Mode`
9. Copy entire contents of [`/docker-compose/.env.example`](https://github.com/Sumtin/torrent-dv-arr/blob/main/docker-compose/.env.example) into text area.
10. Update environment variables as needed
11. Create the Stack

# Configuration

:warning: Containers can communicate with each other over `localhost`.

## Quick Start

Steps below will get the *ARR stack connected to each other and torrent client. Refer to each product's offical docs for further configurations.+

1. Open Sonarr and Radarr (`http://tv.domain`, `http://movies.domain`)
2. For each, `Settings -> Download Clients`, add qBittorrent at `localhost` with port `5555`
3. For each, `Settings -> General` and make note of their respective API Keys, which are needed for the Prowlarr and Bazarr configs
4. Open Prowlarr (`http://indexers.domain`)
5. `Settings -> Indexers`, then add one or more torrent indexers
6. `Settings -> Apps`, then add Radarr and Sonarr using `http://localhost:7878` and `http://localhost:8989`, respectively.
7. Open Bazarr (`http://subtitles.domain`)
8. `Settings -> Sonarr`, using `http://localhost:8989`
9.  `Settings -> Radarr`, using `http://localhost:7878`
10. Open qBittorrent (`http://torrents.domain`)
11. `Tools -> Options -> Advanced`, then change `Network interface` to `tun0`
12. `Tools -> Options -> WebUI` and select `Bypass authentication for clients on localhost` (this is for the automatic port updater script)

## qBittorrent

### Set Network Interface to VPN - MANDATORY

1. From qBittorrent, `Tools -> Options -> Advanced`.
2. Change `Network interface` to `tun0`.

### Port Forwarding - OPTIONAL

If you're using VPN Port Forwarding, follow [the Gluetun wiki](https://github.com/qdm12/gluetun-wiki/blob/main/setup/advanced/vpn-port-forwarding.md) for enabling in qBitorrent.

:warning: If VPN service forwards to random port, have a look at the VPN_PORT_FORWARDING_UP_COMMAND environment variable in the [`docker-compose.yml`](https://github.com/Sumtin/torrent-dv-arr/blob/main/docker-compose/docker-compose.yml).

# Notes

## Networking

- Default Traefik config only supports HTTP (see [Traefik docs](https://doc.traefik.io/) for HTTPS)
- All Internet-bound traffic is routed through the VPN container to ensure privacy, excluding Traefik and WatchTower.  
- Gluetun container must expose ports for child containers.
- Gluetun and Traefik share a bridge network as required by Traefik.
- DNS must be configured manually and is beyond the scope of this wiki.

## Why ARR some volumes not mounted remotely?

Sonarr, Radarr, Prowlarr, and Bazarr all use SQLite and it doesn't play nice when accessed over a network. 
https://access.redhat.com/solutions/120733
