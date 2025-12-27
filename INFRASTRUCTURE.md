# n8n Infrastructure Dokumentasjon

## Arkitektur Oversikt

**Ny konfigurasjon (2025-12-20):**
Trafikk flyt:
1. Internet (offentlig tilgjengelig)
2. NREC Server (<server-ip>)
3. Caddy Reverse Proxy (port 443, alle interfaces)
4. n8n Container (port 5678, localhost-only)

**Tidligere konfigurasjon:**
- Caddy var bound til Tailscale IP (<tailscale-ip>)
- Tilgang kun via Tailscale VPN

**Endret for å:**
- Muliggjøre Claude Desktop MCP-tilkobling
- n8n fortsatt sikret med invite-only system og Bearer token autentisering

## Filsystem Struktur

### System-nivå komponenter:

- `/usr/bin/caddy` - Caddy binær
- `/etc/caddy/Caddyfile` - Caddy konfigurasjon
- `/var/lib/caddy/` - Caddy data (SSL certs)
- `/usr/bin/tailscale` - Tailscale binær (fortsatt installert, men ikke påkrevd)
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

**Nåværende konfigurasjon:**
```
n8n.vuhnger.dev {
    reverse_proxy localhost:5678
}
```

**Merk:**
- Ingen `bind` direktiv = lytter på alle interfaces (0.0.0.0)
- Automatisk HTTPS med Let's Encrypt
- HTTP -> HTTPS redirect automatisk

**Tidligere konfigurasjon (Tailscale-only):**
```
n8n.vuhnger.dev {
    bind <tailscale-ip>
    reverse_proxy localhost:5678
}
```

### Docker Compose (~/n8n/docker-compose.yml)

Se docker-compose.yml filen.

## Sikkerhetslag

### Lag 1: n8n Invite-Only System
- INGEN offentlig registrering
- Kun admin kan legge til brukere
- Login required for all tilgang

### Lag 2: MCP Bearer Token Authentication
- MCP endpoint: `/mcp/<mcp-token>`
- Krever Authorization header med Bearer token
- Uten token = ingen tilgang

### Lag 3: HTTPS/TLS
- Let's Encrypt SSL-sertifikat
- Kryptert trafikk
- Automatisk fornyelse

### Lag 4 (Valgfritt - Fremtidig): Rate Limiting
- Kan implementeres med fail2ban
- Eller Caddy plugin (krever custom build)

## Claude Desktop MCP Integration

**Konfigurasjon:**
Fil: `~/Library/Application Support/Claude/claude_desktop_config.json` (på lokal Mac)

```json
{
  "mcpServers": {
    "n8n": {
      "command": "npx",
      "args": [
        "-y",
        "supergateway",
        "--sse",
        "https://n8n.vuhnger.dev/mcp/<mcp-token>",
        "--header",
        "Authorization: Bearer <mcp-token>"
      ]
    }
  }
}
```

**Hvordan det fungerer:**
1. Claude Desktop kjører `npx supergateway`
2. `supergateway` kobler til n8n MCP endpoint via SSE (Server-Sent Events)
3. Bearer token brukes for autentisering
4. Claude Desktop får tilgang til n8n workflows som MCP tools

## Administrasjonskommandoer

### Caddy
```bash
# Status
sudo systemctl status caddy

# Restart
sudo systemctl restart caddy

# Se konfigurasjon
cat /etc/caddy/Caddyfile

# Se logger
sudo journalctl -u caddy -f

# Test konfigurasjon
sudo caddy validate --config /etc/caddy/Caddyfile
```

### Tailscale (valgfritt - ikke lenger påkrevd)
```bash
sudo tailscale status
sudo tailscale ip -4
sudo systemctl restart tailscaled
```

### n8n
```bash
# Status
docker ps

# Start/restart
cd ~/n8n && sudo docker compose up -d

# Logs
docker logs -f n8n

# Stop
cd ~/n8n && sudo docker compose down
```

## Nettverkskonfigurasjon

### NREC Security Groups
- Port 22 (SSH): 0.0.0.0/0
- Port 80 (HTTP): 0.0.0.0/0 (for Let's Encrypt og HTTP->HTTPS redirect)
- Port 443 (HTTPS): 0.0.0.0/0 (n8n tilgang)

### Tailscale
- Server IP: <tailscale-ip>
- Network: 100.x.x.x/10
- Status: Installert men ikke påkrevd for n8n-tilgang

### DNS
- n8n.vuhnger.dev -> <server-ip> (offentlig IP)

## Installerte Pakker

- caddy-2.10.2
- docker-ce-26.1.3
- docker-compose-plugin-2.27.0
- tailscale-1.92.3

## Backup

### Hva må backes up:
- `~/n8n/n8n_data/` - Workflows, credentials, MCP tokens
- `~/n8n/docker-compose.yml` - I git
- `/etc/caddy/Caddyfile` - Dokumentert her

### Backup kommando:
```bash
cd ~/n8n
tar -czf n8n-backup-$(date +%Y%m%d).tar.gz n8n_data/
```

### Restore kommando:
```bash
cd ~/n8n
tar -xzf n8n-backup-YYYYMMDD.tar.gz
sudo docker compose down && sudo docker compose up -d
```

## Disaster Recovery

1. **Installer pakker:**
   ```bash
   sudo dnf install -y docker-ce caddy
   ```

2. **Restore Caddyfile:**
   ```bash
   sudo tee /etc/caddy/Caddyfile << 'EOF'
   n8n.vuhnger.dev {
       reverse_proxy localhost:5678
   }
   EOF
   sudo systemctl restart caddy
   ```

3. **Clone git repo og restore data:**
   ```bash
   cd ~
   git clone <repo-url> n8n
   cd n8n
   # Restore backup
   tar -xzf n8n-backup-YYYYMMDD.tar.gz
   ```

4. **Start n8n:**
   ```bash
   cd ~/n8n
   sudo docker compose up -d
   ```

## Sikkerhetstips

1. **Bruk sterkt passord** i n8n
2. **Aktiver 2FA** i n8n settings (anbefalt)
3. **Roter MCP Bearer tokens** regelmessig
4. **Overvåk logger** for uvanlig aktivitet:
   ```bash
   ssh nrec "docker logs -f n8n | grep -i 'auth\|login\|fail'"
   ```
5. **Vurder fail2ban** for automatisk IP-blokkering

## Endringslogg

### 2025-12-20
- Fjernet Tailscale-binding fra Caddy
- n8n nå offentlig tilgjengelig via HTTPS
- Lagt til Claude Desktop MCP integration
- Oppdatert sikkerhetsdokumentasjon

### 2025-12-19
- Initial setup med Tailscale-only tilgang
- HTTPS konfigurert med Let's Encrypt
- Caddy bound til Tailscale IP (<tailscale-ip>)

---

**Sist oppdatert:** 2025-12-20
