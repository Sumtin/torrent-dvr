# Introduction
Collection of secure deployments for torrent-based digital video recorders (DVR) for movies and TV shows.

# Networking

- Bittorrent and Gatus are routed through the ExpressVPN container to ensure privacy.  
- There are two instances of OpenSpeedTest.  One behind VPN, the other not, so you can test performance.
![image](https://github.com/Sumtin/torrent-dvr/assets/6676557/06efc94e-dedb-4ca3-90b4-585fa202c308)



Currently uses default Docker network.

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
9. Update as environment variables as needed

# What's Included?

The following continers are used in this solution:

- [ExpressVPN](https://github.com/polkaned/dockerfiles/tree/master/expressvpn)
- [Watchtower](https://github.com/containrrr/watchtower)
- [qBittorrent](https://docs.linuxserver.io/images/docker-qbittorrent)
- [Gatus](https://github.com/TwiN/gatus)
- [Sonarr](https://docs.linuxserver.io/images/docker-sonarr)
- [Radarr](https://docs.linuxserver.io/images/docker-radarr)
- [Bazarr](https://docs.linuxserver.io/images/docker-bazarr)
- [Jackett](https://docs.linuxserver.io/images/docker-jackett)
- [Flaresolverr](https://github.com/FlareSolverr/FlareSolverr)

# Notes
## Why are Radarr and Sonarr volumes not mounted remotely?
Sonarr and Radarr both use SQLite and it doesn't play nice when accessed over a network. 
https://access.redhat.com/solutions/120733
