# Home Media Stack

My personal media server setup: Jellyfin for streaming, qBittorrent behind VPN with killswitch.

## What This Is

Docker Compose stack that:
- Runs Jellyfin on normal network (so streaming works locally)
- Routes qBittorrent through VPN with automatic killswitch
- Uses Gluetun to handle VPN + network isolation

If VPN drops, torrents stop. Jellyfin keeps working.

## Stack
```
┌─────────────┐
│  Jellyfin   │ → Your network (port 8096)
└─────────────┘

┌─────────────┐
│   Gluetun   │ → VPN tunnel
│      ↓      │
│ qBittorrent │ → Torrent traffic (port 8080 via VPN)
└─────────────┘
```

## Setup

**1. Clone & configure**
```bash
git clone https://github.com/diegoamndz/home-media-stack.git
cd home-media-stack
cp .env.example .env
```

**2. Edit `.env` with your VPN credentials**

**3. Start Jellyfin first (no VPN needed)**
```bash
docker-compose up -d jellyfin
```

**4. Configure VPN, then start everything**
```bash
docker-compose --profile vpn up -d
```

**Access:**
- Jellyfin: `http://localhost:8096`
- qBittorrent: `http://localhost:8080` (default: admin/adminadmin)

## Verify Killswitch Works
```bash
# Check qBittorrent IP (should be VPN)
docker exec -it gluetun wget -qO- https://api.ipify.org

# Check your real IP
curl https://api.ipify.org
```

Different IPs = working.

## File Structure
```
config/
  jellyfin/       # Jellyfin settings
  qbittorrent/    # qBittorrent settings
  gluetun/        # VPN configs

data/
  media/          # Your movies/tv/music
    movies/
    tv/
    music/
  torrents/       # Downloads
    downloads/    # Completed
    incomplete/   # In progress
```

## External Drive

If using external storage:
```bash
sudo mkdir -p /mnt/media-hdd
sudo mount /dev/sda1 /mnt/media-hdd

# Update docker-compose.yml volumes to /mnt/media-hdd
```

## Common Issues

**VPN won't connect:** Try different `SERVER_COUNTRIES` in `.env`

**Jellyfin can't read files:** 
```bash
sudo chown -R 1000:1000 data/media/
```

**qBittorrent not accessible:** Check VPN is up
```bash
docker logs gluetun
```

## Useful Commands
```bash
docker logs -f gluetun           # VPN logs
docker restart jellyfin          # Restart service
docker-compose down              # Stop all
docker-compose pull && docker-compose up -d  # Update
```

## Notes

- Using Gluetun because it supports basically every VPN provider
- Jellyfin has hardware transcoding enabled (`/dev/dri`)
- Profiles keep VPN services separate - they don't auto-start
- Works on Raspberry Pi if you have decent cooling

## Why This Setup

Tried Plex, went back to Jellyfin (open source, no account needed).  
Tried running torrents without VPN... yeah, no.  
This setup just works and I can forget about it.

---

**VPN Providers:** https://github.com/qdm12/gluetun-wiki/tree/main/setup/providers
