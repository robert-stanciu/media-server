# Media Server Stack

A fully automated movie/TV downloading and streaming setup using Docker Compose.

## Services

| Service        | URL                        | Purpose                  |
|----------------|----------------------------|--------------------------|
| Jellyfin       | http://localhost:8096      | Media streaming          |
| qBittorrent    | http://localhost:8080      | Torrent client           |
| Prowlarr       | http://localhost:9696      | Indexer management       |
| Radarr         | http://localhost:7878      | Movie automation         |
| Sonarr         | http://localhost:8989      | TV show automation       |
| Bazarr         | http://localhost:6767      | Subtitle automation      |

## Prerequisites

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed (works on Windows, Mac, Linux)
- On Windows: enable WSL2 backend in Docker Desktop settings

## Folder Structure

```
media-server/
├── docker-compose.yml
├── config/          (auto-created, stores app configs)
│   ├── jellyfin/
│   ├── qbittorrent/
│   ├── radarr/
│   ├── sonarr/
│   ├── prowlarr/
│   └── bazarr/
├── media/           (your library — point Jellyfin here)
│   ├── movies/
│   └── tv/
└── downloads/       (qBittorrent downloads here, Radarr/Sonarr move completed files to media/)
```

## Quick Start

```bash
# Start everything
docker compose up -d

# Stop everything
docker compose down

# View logs
docker compose logs -f

# Update all images
docker compose pull && docker compose up -d

# Stop and remove all data (nuclear option)
docker compose down -v
```

## Setup Order (first time only)

### 1. qBittorrent
- Go to http://localhost:8080
- Default login: `admin` / check logs with `docker compose logs qbittorrent` for temp password
- Change password in Settings → Web UI
- Set default download path to `/downloads`

### 2. Prowlarr
- Go to http://localhost:9696
- Add torrent indexers (1337x, TorrentGalaxy, YTS, etc.)
- Go to Settings → Apps → Add Radarr and Sonarr (use container names as hostnames):
  - Radarr: `http://radarr:7878`
  - Sonarr: `http://sonarr:8989`
  - You'll need their API keys (found in each app under Settings → General)

### 3. Radarr
- Go to http://localhost:7878
- Settings → Media Management → Add Root Folder → `/movies`
- Settings → Download Clients → Add qBittorrent:
  - Host: `qbittorrent`
  - Port: `8080`
  - Username/password from step 1
- Set quality profile (e.g., prefer 1080p Bluray)

### 4. Sonarr
- Go to http://localhost:8989
- Same as Radarr but root folder is `/tv`

### 5. Bazarr
- Go to http://localhost:6767
- Settings → Subtitles → add providers (OpenSubtitles.com, Subscene, etc.)
- Settings → Languages → add Romanian + English
- Connect to Radarr and Sonarr (use container names as hosts)

### 6. Jellyfin
- Go to http://localhost:8096
- Create admin account
- Add libraries:
  - Movies → `/data/movies`
  - TV Shows → `/data/tv`

## Accessing from Other Devices

Find your machine's local IP (`ipconfig` on Windows, `ifconfig` on Mac) and use:
```
http://YOUR_LOCAL_IP:8096
```

## Migrating to Another Machine

1. Copy the entire `media-server/` folder to the new machine
2. Install Docker Desktop
3. Run `docker compose up -d`
4. All configs and media are preserved

## Notes

- All containers use `restart: unless-stopped` — they start automatically with Docker
- The `config/` folder contains all app settings — back this up
- The `media/` folder is your library — this is what takes up disk space
- PUID/PGID are set to 1000 (default first user on Linux). On Windows/Mac Docker Desktop this is handled automatically
- Timezone is set to `Europe/Bucharest` — change in docker-compose.yml if needed
