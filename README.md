- [Introduction](#introduction)
- [Installation](#installation)
  - [Docker Compose](#docker-compose)
  - [Portainer Stack](#portainer-stack)
- [Configuration](#configuration)
  - [Quick Start](#quick-start)
    - [Sonarr \& Radarr - `http://tv.domain` & `http://movies.domain`](#sonarr--radarr---httptvdomain-httpmoviesdomain)
    - [Prowlarr - `http://indexers.domain`](#prowlarr---httpindexersdomain)
    - [Bazarr - `http://subtitles.domain`](#bazarr---httpsubtitlesdomain)
    - [qBittorrent - `http://torrents.domain`](#qbittorrent---httptorrentsdomain)
      - [Port Forwarding](#port-forwarding)
- [Notes](#notes)
  - [Networking](#networking)
  - [Why ARR most volumes not mounted remotely?](#why-arr-most-volumes-not-mounted-remotely)


# Introduction
Docker Compose deployment for torrent-based *ARR stack behind VPN and Traefik reverse proxy.

The following containers are used in this solution:

| Service | Behind VPN? | HTTP Endpoint | Docker Port | Purpose | Official Docs |
|---|:---:|---|:---:|---|---|
|Gluetun|N/A|N/A|N/A|VPN|https://github.com/qdm12/gluetun-wiki|
|Traefik Dashboard|No|http://domain:8080|8080|Reverse Proxy|https://doc.traefik.io/|
|WatchTower|No|N/A|N/A|Auto-updates containers|https://github.com/containrrr/watchtower| 
|qBittorrent|Yes|http://torrents.domain|5555|Bittorrent client|https://github.com/qbittorrent/qBittorrent/wiki|
|Radarr|Yes|http://movies.domain|7878|Monitors Movies|https://wiki.servarr.com/radarr|
|Sonarr|Yes|http://tv.domain|8989|Monitors TV|https://wiki.servarr.com/en/sonarr|
|Bazarr|Yes|http://subtitles.domain|6767|Manages subtitles|https://www.bazarr.media/|
|Prowlarr|Yes|http://indexers.domain|9117|Torrent indexer managager |https://wiki.servarr.com/en/prowlarr|

# Installation

:warning: **Default Traefik configuration only supports HTTP**

:warning: **Make sure all your mounts are properly mounted.**

:warning: **DNS must be configured manually and is beyond the scope of this wiki.**

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

Steps below will get the *ARR stack connected to each other and torrent client. Refer to each product's offical docs for further configurations.

### Sonarr & Radarr - `http://tv.domain`, `http://movies.domain`

1. `Settings -> Download Clients`, add qBittorrent at `localhost` with port `5555`
2. `Settings -> General` and scroll down til you see their API Keys. Those are needed for the Prowlarr and Bazarr configs.

### Prowlarr - `http://indexers.domain`

1. `Settings -> Indexers`, add one or more torrent indexers
2. `Settings -> Apps`, add Radarr and Sonarr using `http://localhost:7878` and `http://localhost:8989`, respectively.

### Bazarr - `http://subtitles.domain`

1. `Settings -> Sonarr`, using `http://localhost:8989`
2.  `Settings -> Radarr`, using `http://localhost:7878`
   
### qBittorrent - `http://torrents.domain`

1.  `Tools -> Options -> Advanced`, then change `Network interface` to `tun0`
2.  `Tools -> Options -> WebUI` and select `Bypass authentication for clients on localhost` (this is for the automatic port updater script, see below)

#### Port Forwarding

If you're using VPN Port Forwarding, follow [the Gluetun wiki](https://github.com/qdm12/gluetun-wiki/blob/main/setup/advanced/vpn-port-forwarding.md) for enabling in qBitorrent.

If VPN service forwards to random port, check the `VPN_PORT_FORWARDING_UP_COMMAND` environment variable in [`docker-compose.yml`](https://github.com/Sumtin/torrent-dv-arr/blob/main/docker-compose/docker-compose.yml).

# Notes

## Networking

- Default Traefik config only supports HTTP (see [Traefik docs](https://doc.traefik.io/) for HTTPS)
- All Internet-bound traffic is routed through the VPN container to ensure privacy, excluding Traefik and WatchTower.  
- Gluetun container must expose ports for child containers.
- Gluetun and Traefik share a bridge network as required by Traefik.

## Why ARR most volumes not mounted remotely?

Sonarr, Radarr, Prowlarr, and Bazarr all use SQLite and it doesn't play nice when accessed over a network. 
https://access.redhat.com/solutions/120733
