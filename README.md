- [Introduction](#introduction)
- [Installation](#installation)
  - [Docker Compose](#docker-compose)
  - [Portainer Stack](#portainer-stack)
- [Configuration](#configuration)
  - [General](#general)
  - [qBittorrent](#qbittorrent)
    - [Set Network Interface to VPN - MANDATORY](#set-network-interface-to-vpn---mandatory)
    - [Port Forwarding - OPTIONAL](#port-forwarding---optional)
- [Notes](#notes)
  - [Networking](#networking)
  - [Why are \*ARR volumes not mounted remotely?](#why-are-arr-volumes-not-mounted-remotely)


# Introduction
Docker Compose deployment for torrent-based *ARR stack behind VPN.

The following containers are used in this solution:

| Service | Behind VPN? | HTTP Endpoint | Docker Port | Purpose | Official Docs |
|---|:---:|---|:---:|---|---|
|Gluetun|N/A|N/A|N/A|VPN container|https://github.com/qdm12/gluetun-wiki|
|Traefik Dashboard|No|http://domain|8080|Reverse Proxy|https://doc.traefik.io/|
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
2. Create mounts! (See `MOUNT_TO_...` configs in [`.env.example`](ttps://github.com/Sumtin/torrent-dvaar/blob/main/docker-compose/.env.exam) file)
3. Change working directory to `/docker-compose/`
4. Rename `.env.example` to `.env` file and modify as needed
6. Run `docker compose up -d`
7. Visit each HTTP endpoint and configure the apps. Recommend starting with Prowlarr.
8. Review [Manual qBittorrent Configurations](#manual-qbittorrent-configurations)

## Portainer Stack

1. Create mounts! (See `MOUNT_TO_...` configs in [`.env.example`](ttps://github.com/Sumtin/torrent-dvaar/blob/main/docker-compose/.env.exam) file)
2. Open `Portainer -> Stacks -> Add Stack`
3. Enter a Name (i.e. `torrent-dvaar`)
4. Click `Repository`
5. Repository URL = `https://github.com/Sumtin/torrent-dvaar/`
6. Repo Reference = `refs/heads/main`
7. Compose Path = `docker-compose/docker-compose.yml`
8. Under Environment Variables, select `Advanced Mode`
9. Copy entire contents of [`/docker-compose/.env.example`](https://github.com/Sumtin/torrent-dvaar/blob/main/docker-compose/.env.example) into text area.
10. Update environment variables as needed
11. Create the Stack
12. Visit each HTTP endpoint and configure the apps. Recommend starting with Prowlarr.
13. Review [Manual qBittorrent Configurations](#manual-qbittorrent-configurations)


# Configuration

## General

:warning: All containers can be accessed, and can communicate with each other, over `localhost`.

I recommend starting with configuring Prowlarr (`http://indexers.domain`) with Radarr and Sonarr using `http://localhost:7878` and `http://localhost:8989`, respectively.

## qBittorrent

### Set Network Interface to VPN - MANDATORY

1. From qBittorrent, `Tools -> Options -> Advanced`.
2. Change `Network interface` to `tun0`.

### Port Forwarding - OPTIONAL

If you're using VPN Port Forwarding, follow [the Gluetun wiki](https://github.com/qdm12/gluetun-wiki/blob/main/setup/advanced/vpn-port-forwarding.md) for enabling in qBitorrent.

:warning: If VPN service forwards to random port, have a look at the VPN_PORT_FORWARDING_UP_COMMAND environment variable in the [`docker-compose.yml`](https://github.com/Sumtin/torrent-dvaar/blob/main/docker-compose/docker-compose.yml).

# Notes

## Networking

- Default Traefik config only supports HTTP (see [Traefik docs](https://doc.traefik.io/) for HTTPS)
- All Internet-bound traffic is routed through the VPN container to ensure privacy, excluding Traefik and WatchTower.  
- Gluetun container must expose ports for child containers.
- Gluetun and Traefik share a bridge network as required by Traefik.
- DNS must be configured manually and is beyond the scope of this wiki.

## Why are *ARR volumes not mounted remotely?

Sonarr, Radarr, Prowlarr, and Bazarr all use SQLite and it doesn't play nice when accessed over a network. 
https://access.redhat.com/solutions/120733
