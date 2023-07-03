# Introduction
Collection of secure deployments for torrent-based digital video recorders (DVR) for movies and TV shows.

# How to Use
## Docker Compose

1. Clone this repo
1. Change working directory to `/docker-compose/`
1. Edit `.env` file
1. :warning: **Make sure ALL your mounts are properly mounted.**
1. Run `docker compose up -d` OR install as Stack in Portainer

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