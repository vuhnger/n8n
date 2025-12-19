# n8n Infrastructure Dokumentasjon

## Arkitektur Oversikt

Trafikk flyt:
1. Din laptop/mobil
2. Tailscale VPN (kryptert tunnel)
3. NREC Server (158.37.66.204)
4. Caddy Reverse Proxy (port 443, Tailscale-only)
5. n8n Container (port 5678, localhost-only)

## Filsystem Struktur

### System-nivå komponenter:

- `/usr/bin/caddy` - Caddy binær
- `/etc/caddy/Caddyfile` - Caddy konfigurasjon
- `/var/lib/caddy/` - Caddy data (SSL certs)
- `/usr/bin/tailscale` - Tailscale binær
- `/var/lib/tailscale/` - Tailscale data
- `/usr/bin/docker` - Docker binær

### Applikasjon (~/n8n/):

- `docker-compose.yml` - n8n container konfigurasjon
- `README.md` - Bruker dokumentasjon
- `INFRASTRUCTURE.md` - Denne filen
- `.gitignore` - Git ignore regler
- `n8n_data/` - n8n persistent data (IKKE i git!)

## Konfigurasjonsfiler

### Caddy (/etc/caddy/Caddyfile)

```
n8n.vuhnger.dev {
    bind 100.124.46.65
    reverse_proxy localhost:5678
}
```

### Docker Compose (~/n8n/docker-compose.yml)

Se docker-compose.yml filen.

## Sikkerhetslag

1. **Tailscale VPN** - Kryptert tunnel, Zero Trust
2. **Caddy** - HTTPS + SSL, kun Tailscale-interface
3. **n8n** - Innebygd autentisering

## Administrasjonskommandoer

### Caddy
```bash
sudo systemctl status caddy
sudo systemctl restart caddy
cat /etc/caddy/Caddyfile
sudo journalctl -u caddy -f
```

### Tailscale
```bash
sudo tailscale status
sudo tailscale ip -4
sudo systemctl restart tailscaled
```

### n8n
```bash
docker ps
cd ~/n8n && sudo docker compose up -d
docker logs -f n8n
```

## Nettverkskonfigurasjon

### NREC Security Groups
- Port 22 (SSH): 0.0.0.0/0
- Port 80 (HTTP): 0.0.0.0/0 (kan fjernes)
- Port 443 (HTTPS): 0.0.0.0/0 (kan fjernes)

### Tailscale
- Server IP: 100.124.46.65
- Network: 100.x.x.x/10

## Installerte Pakker

- caddy-2.10.2
- docker-ce-26.1.3
- docker-compose-plugin-2.27.0
- tailscale-1.92.3

## Backup

### Hva må backes up:
- `~/n8n/n8n_data/` - Workflows, credentials
- `~/n8n/docker-compose.yml` - I git
- `/etc/caddy/Caddyfile` - Dokumentert her

### Backup kommando:
```bash
cd ~/n8n
tar -czf n8n-backup-$(date +%Y%m%d).tar.gz n8n_data/
```

## Disaster Recovery

1. Installer Docker, Caddy, Tailscale
2. Restore Caddyfile
3. Clone git repo og restore data
4. Start services

Se README.md for detaljerte instruksjoner.

---

**Sist oppdatert:** 2025-12-19
