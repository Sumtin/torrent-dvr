# Introduction
Collection of secure deployments for torrent-based digital video recorders (DVR) for movies and TV shows.

# How to Use
## Docker Compose

1. Clone this repo
1. Change working directory to `/docker-compose/`
1. Edit `.env` file as needed
1. :warning: **Make sure ALL your mounts are properly mounted.**
1. Run `docker compose up -d`

## Portainer Stack

1. Open `Portainer -> Stacks -> Add Stack`
2. Enter a Name
3. Click `Repository`
4. Repository URL = `https://github.com/Sumtin/torrent-dvr/`
5. Repo Reference = `refs/heads/main`
6. Compose Path = `docker-compose/docker-compose.yml`
7. Under Environment Variables, select `Advanced Mode`
8. Copy entire contents of [`/docker-compose/.env`](https://github.com/Sumtin/torrent-dvr/blob/main/docker-compose/.env) into text area.
9. Update environment variables as needed

# What's Included?

The following continers are used in this solution:

| Service | Behind VPN? | Port | Official Docs |
|--|--|--|--|
| ExpressVPN | N/A | N/A | https://github.com/polkaned/dockerfiles/tree/master/expressvpn |
|WatchTower|No|N/A|https://github.com/containrrr/watchtower| 
|qBittorrent|Yes|5555|https://docs.linuxserver.io/images/docker-qbittorrent|
|Gatus|Yes|8080|https://github.com/TwiN/gatus|
|Sonarr|No|8989|https://docs.linuxserver.io/images/docker-sonarr|
|Radarr|No|7878|https://docs.linuxserver.io/images/docker-radarr|
|Bazarr|No|6767|https://docs.linuxserver.io/images/docker-bazarr|
|Jackett|No|9117|https://docs.linuxserver.io/images/docker-jackett|
|Flaresolverr|No|8191|https://github.com/FlareSolverr/FlareSolverr|
|OpenSpeedTest|Yes|3000|https://openspeedtest.com/|
|OpenSpeedTest|o|3030|https://openspeedtest.com/|

# Networking

- Bittorrent and Gatus are routed through the ExpressVPN container to ensure privacy.  
- There are two instances of OpenSpeedTest.  One behind VPN, the other not, so you can test performance.
- Currently uses default Docker network.
![image](https://github.com/Sumtin/torrent-dvr/assets/6676557/06efc94e-dedb-4ca3-90b4-585fa202c308)


# Notes
## Why are Radarr and Sonarr volumes not mounted remotely?
Sonarr and Radarr both use SQLite and it doesn't play nice when accessed over a network. 
https://access.redhat.com/solutions/120733
