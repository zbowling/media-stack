# Yet Another Media Stack Setup

A comprehensive Docker Compose media stack for downloading, managing, and streaming media content. This setup uses Tailscale for secure private access and supports Cloudflare DDNS for selective public access.

## 🎯 Overview

This is my personal media stack configuration that I'm sharing for others who want a similar setup. It's designed to be simple yet comprehensive, covering the full media pipeline from acquisition to consumption.

I'm using docker but in theory this should work with any container orchestrator.

This is my setup that works for me. I'm documenting it in the hope in that might help others but no warrenty or support is provided or implied flexibility is provided.

**Key Features:**
- 🔒 Designed assuming you are using Tailscale to access internal services (uses Tailscale SSL certificates to access internal services)
- 🌐 Optional public access for Jellyfin and Jellyseerr with an updated DDNS setup with Cloudflare and LetsEncrypt
- 📱 Modern web dashboard with Homepage
- 🎬 Complete *arr stack for automated media management
- 📊 Built-in monitoring and maintenance tools
- 🔧 Easy configuration management with environment files

## 🏗️ Architecture

### Core Media Components
- **[Jellyfin](https://jellyfin.org/)** - Open-source media server for streaming your content
- **[Plex](https://www.plex.tv/)** - Alternative media server (both included for flexibility)
- **[Jellyseerr](https://github.com/Fallenbagel/jellyseerr)** - Content request management (like Overseerr but for Jellyfin)

### Download & Management (*arr Stack)
- **[qBittorrent](https://www.qbittorrent.org/)** - BitTorrent client with web interface
- **[Radarr](https://radarr.video/)** - Movie collection manager
- **[Sonarr](https://sonarr.tv/)** - TV series collection manager  
- **[Lidarr](https://lidarr.audio/)** - Music collection manager
- **[Prowlarr](https://prowlarr.com/)** - Indexer manager for all *arr apps

### Infrastructure & Utilities
- **[Traefik](https://traefik.io/)** - Reverse proxy with automatic SSL/TLS
- **[Homepage](https://gethomepage.dev/)** - Modern dashboard for all services
- **[PostgreSQL](https://www.postgresql.org/)** - Database for services that need it (not currently used)
- **[pgAdmin](https://www.pgadmin.org/)** - Database administration interface

### Maintenance & Monitoring
- **[Unpackerr](https://github.com/davidnewhall/unpackerr)** - Automatic archive extraction
- **[Huntarr](https://github.com/aunefyren/huntarr)** - Media cleanup and management
- **[Cleanuperr](https://github.com/Schnouki/cleanuperr)** - Additional cleanup utilities
- **[Watchtower](https://containrrr.dev/watchtower/)** - Automatic container updates
- **[Autoheal](https://github.com/willfarrell/docker-autoheal)** - Container health monitoring
- **[Glances](https://nicolargo.github.io/glances/)** - System resource monitoring

### Network & DNS
- **[Cloudflare DDNS](https://github.com/oznu/docker-cloudflare-ddns)** - Dynamic DNS updates

### Extra Services
- **[Home Assistant](https://www.home-assistant.io/)** - Home automation platform

## 🚀 Quick Start

### 1. Clone the Repository
```bash
git clone <your-repo-url>
cd media-stack
```

### 2. Set Up Environment Files
```bash
# Copy the example secrets file
cp env/secrets.env.example env/secrets.env

# Edit the secrets file with your actual values
nano env/secrets.env
```

### 3. Generate Environment Configuration
```bash
# This will create .env from common.env + secrets.env
make
```

### 4. Start the Stack
```bash
sudo docker compose up -d
```

## ⚙️ Configuration

### Environment Files Structure

This setup uses a two-file configuration system:

- **`env/common.env`** - Default values and common settings (safe to share)
- **`env/secrets.env`** - Your personal secrets and sensitive data (DO NOT COMMIT)

The Makefile automatically combines these into a `.env` file that Docker Compose uses.

### Required Configuration

Edit `env/secrets.env` and update these critical values:

```bash
# Your domain/hostname
HOSTNAME=your-hostname.ts.net
PUBLIC_HOSTNAME=your-domain.com

# API Keys (generate these after first startup)
RADARR_API_KEY=your-radarr-api-key
SONARR_API_KEY=your-sonarr-api-key
# ... etc

# Database credentials
POSTGRES_PASSWORD=your-secure-password

# Cloudflare (if using public access)
CLOUDFLARE_EMAIL=your-email@example.com
CLOUDFLARE_DNS_API_TOKEN=your-cloudflare-token

# Plex claim token (get from plex.tv/claim)
PLEX_CLAIM=claim-your-plex-token
```

### Makefile Usage

```bash
make                # Generate .env from source files
make help          # Show all available commands  
make clean         # Remove generated .env file
make force         # Force regeneration of .env
make check-secrets # Verify secrets.env exists
```

## 🌐 Network Access

### Private Access (Tailscale)
- All services are accessible via your Tailscale network if configured correctly
- Access the dashboard at: `http://your-hostname.tailnet.ts.net/`

### Public Access (Optional)
- Jellyfin and Jellyseerr can be exposed publicly via Cloudflare
- Configure your domains in `CLOUDFLARE_DDNS_DOMAINS`
- SSL certificates are automatically managed by Traefik using LetsEncrypt

## 📁 Directory Structure

```
/mnt/media/          # Your media storage root
├── movies/          # Radarr movie library
├── tv/              # Sonarr TV library  
├── music/           # Lidarr music library
└── torrents/        # qBittorrent download directory
    ├── movies/      # Movie downloads
    ├── tv/          # TV downloads
    └── music/       # Music downloads
```

## 🔧 Customization

### Adding VPN Support
This setup doesn't include a VPN container yet, but adding [Gluetun](https://github.com/qdm12/gluetun) would be trivial:

1. Add Gluetun service to docker-compose.yml
2. Route torrent traffic through the VPN container
3. Update network dependencies

### Port Configuration
All ports are configured in `env/common.env`. The defaults are:
- Homepage: 3000
- Jellyfin: 8096  
- qBittorrent: 5080
- Traefik: 80/443

### Service Selection
You can disable services you don't need by commenting them out in the docker-compose.yml file.

## 🔍 Monitoring

- **Homepage Dashboard**: `http://your-host:3000` - Central dashboard for all services
- **Glances**: `http://your-host:61208` - System resource monitoring  
- **Traefik Dashboard**: Available via Traefik's web UI
- **pgAdmin**: `http://your-host:4080` - Database administration

## 🆘 Troubleshooting

### First Time Setup Issues
If you get an error about missing `secrets.env`:
```bash
❌ ERROR: env/secrets.env not found!
cp env/secrets.env.example env/secrets.env
# Then edit the file with your values
```

### Container Health
Use the autoheal service to automatically restart unhealthy containers:
```bash
docker logs autoheal  # Check autoheal logs
```

### API Key Generation
Most *arr applications generate API keys on first startup. Check their web interfaces under Settings → General.

## 📝 Notes

- This is my personal setup optimized for my use case
- No VPN is included by default (but easy to add)
- Tailscale provides secure remote access without opening ports
- Public access is optional and only recommended for Jellyfin/Jellyseerr
- All data persists in Docker volumes and your configured media directories

## 🤝 Contributing

Feel free to open issues or submit PRs if you find improvements or have suggestions!

## ⚖️ License

This project is provided as-is for educational and personal use. Please ensure you comply with all applicable laws and service terms when using this setup.


