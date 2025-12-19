# n8n Workflow Automation Setup

Dette er dokumentasjon for n8n installasjon på NREC GOLD Rocky Linux 8 server.

## 📋 Oversikt

**Server detaljer:**
- Server: GOLD Rocky Linux 8 (NREC)
- IPv4: 158.37.66.204
- IPv6: 2001:700:2:8200::237e
- Domene: n8n.vuhnger.dev
- Tailscale IP: 100.124.46.65
- Lokasjon: ~/n8n/

**Teknologier:**
- n8n (workflow automation)
- Docker & Docker Compose
- Caddy (reverse proxy med automatisk SSL)
- Tailscale (privat VPN-nettverk)

## 🔐 VIKTIG: Tilgang via Tailscale

**n8n er KUN tilgjengelig via Tailscale!**

Dette betyr:
- ✅ 100% privat - ingen fra internett kan nå n8n
- ✅ Ingen porter eksponert til offentlig internett (utenom SSH)
- ✅ Industri-standard sikkerhet
- ✅ Fungerer fra laptop, mobil, hvor som helst

## 🚀 Hvordan koble til n8n

### Første gang oppsett:

#### 1. Installer Tailscale på din enhet

**Mac:**
- Last ned: https://tailscale.com/download/mac
- Eller: `brew install tailscale`

**Windows:**
- Last ned: https://tailscale.com/download/windows

**iPhone:**
- App Store: https://apps.apple.com/app/tailscale/id1470499037

**Android:**
- Play Store: https://play.google.com/store/apps/details?id=com.tailscale.ipn

**Linux:**
```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

#### 2. Logg inn på Tailscale

- Åpne Tailscale-appen
- Logg inn med **samme konto** som serveren (Google/GitHub/Microsoft/Email)
- Din konto: vuhnger@

#### 3. Koble til

Når Tailscale er koblet til (grønn indikator), åpne:

**🌐 https://n8n.vuhnger.dev**

Det er alt! Ingen Basic Auth, bare n8n sin vanlige pålogging.

---

## 🔗 Tilgangsmetoder

### Anbefalt (via custom domain):
```
https://n8n.vuhnger.dev
```

### Alternativt (via Tailscale IP):
```
http://100.124.46.65:5678
```

### Alternativt (via maskinnavn):
```
http://victor-research-1:5678
```

**Tips:** Bruk custom domain (n8n.vuhnger.dev) for best opplevelse!

---

## 📁 Filstruktur

```
~/n8n/
├── docker-compose.yml    # Docker konfigurasjon for n8n
├── n8n_data/            # Persistent data (workflows, credentials, etc.)
└── README.md            # Denne filen
```

## 🔧 Grunnleggende Kommandoer

### Start n8n
```bash
cd ~/n8n
sudo docker compose up -d
```

### Stopp n8n
```bash
cd ~/n8n
sudo docker compose down
```

### Restart n8n
```bash
cd ~/n8n
sudo docker compose restart
```

### Sjekk status
```bash
docker ps
# Eller for mer detaljer:
sudo docker compose ps
```

### Se logger
```bash
# Live logger (Ctrl+C for å avslutte)
docker logs -f n8n

# Siste 50 linjer
docker logs n8n --tail 50
```

### Oppdater n8n til nyeste versjon
```bash
cd ~/n8n
sudo docker compose pull
sudo docker compose up -d
```

## 🔐 Tailscale Administrasjon

### Sjekk Tailscale status på serveren
```bash
ssh nrec
sudo tailscale status
```

### Se Tailscale IP
```bash
sudo tailscale ip -4
# Output: 100.124.46.65
```

### Restart Tailscale
```bash
sudo systemctl restart tailscaled
```

### Koble fra Tailscale (midlertidig)
```bash
sudo tailscale down
```

### Koble til igjen
```bash
sudo tailscale up
```

### Administrer enheter via Tailscale dashboard
https://login.tailscale.com/admin/machines

Her kan du:
- Se alle tilkoblede enheter
- Fjerne enheter
- Dele tilgang med andre (familie/team)

## 🌐 HTTPS Setup med Caddy

### Caddy Status
```bash
sudo systemctl status caddy
```

### Restart Caddy
```bash
sudo systemctl restart caddy
```

### Se Caddy logger
```bash
sudo journalctl -u caddy -f
```

### Rediger Caddy konfigurasjon
```bash
sudo nano /etc/caddy/Caddyfile
# Etter endringer:
sudo systemctl reload caddy
```

**VIKTIG:** Caddy lytter KUN på Tailscale-interfacet (100.124.46.65). Dette betyr n8n er kun tilgjengelig via Tailscale!

## 🔍 Feilsøking

### "Kan ikke nå n8n.vuhnger.dev"

**Sjekk 1: Er Tailscale koblet til?**
- Mac: Se Tailscale-ikonet i menybaren (skal være grønt)
- Windows: Se Tailscale-ikonet i system tray
- Mobil: Åpne Tailscale-appen og sjekk at "Connected" er grønn

**Sjekk 2: Er du logget inn med riktig konto?**
```bash
# På din laptop:
tailscale status
# Skal vise "vuhnger@" og liste "victor-research-1"
```

**Sjekk 3: Er n8n kjører på serveren?**
```bash
ssh nrec "docker ps"
# Skal vise container "n8n" som er "Up"
```

**Sjekk 4: Er Tailscale kjører på serveren?**
```bash
ssh nrec "sudo tailscale status"
# Skal vise din laptop og andre enheter
```

### Kan ikke koble til Tailscale

**Løsning 1: Restart Tailscale**
- Mac: Quit Tailscale og start på nytt
- Windows: Restart Tailscale-appen
- Linux: `sudo systemctl restart tailscaled`

**Løsning 2: Logg inn på nytt**
```bash
tailscale logout
tailscale login
```

### n8n starter ikke

```bash
# Sjekk logger for feilmeldinger
docker logs n8n --tail 100

# Sjekk at containeren kjører
docker ps -a

# Restart
cd ~/n8n
sudo docker compose down
sudo docker compose up -d
```

### SSL-sertifikat problemer

```bash
# Se Caddy logger
sudo journalctl -u caddy -xe

# Verifiser DNS
dig n8n.vuhnger.dev +short
# Skal returnere: 158.37.66.204

# Test SSL (via Tailscale)
curl -I https://n8n.vuhnger.dev
```

### Tillatelse problemer med Docker

```bash
# Hvis du får "permission denied" når du kjører docker kommandoer:
# Logg ut og inn igjen, eller bruk:
newgrp docker

# Alternativt bruk sudo:
sudo docker compose up -d
```

## 📊 Docker Compose Konfigurasjon

```yaml
version: '3.8'

services:
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    ports:
      - '5678:5678'
    environment:
      - N8N_HOST=n8n.vuhnger.dev
      - N8N_PORT=5678
      - N8N_PROTOCOL=https
      - WEBHOOK_URL=https://n8n.vuhnger.dev/
      - N8N_SECURE_COOKIE=true
    volumes:
      - ./n8n_data:/home/node/.n8n
    networks:
      - n8n-network

networks:
  n8n-network:
    driver: bridge
```

## 🔒 Sikkerhet

**Nåværende sikkerhetskonfigurasjon:**
- ✅ HTTPS aktivert via Caddy med Let's Encrypt SSL
- ✅ Kun tilgjengelig via Tailscale (privat VPN)
- ✅ Caddy lytter KUN på Tailscale-interface (100.124.46.65)
- ✅ Ingen offentlige porter eksponert (utenom SSH port 22)
- ✅ n8n innebygd autentisering
- ✅ Secure cookies aktivert

**NREC Security Group regler:**
- Port 22 (SSH): CIDR 0.0.0.0/0
- Port 80 (HTTP): CIDR 0.0.0.0/0 (for SSL challenge - ikke i bruk etter Tailscale)
- Port 443 (HTTPS): CIDR 0.0.0.0/0 (ikke i bruk etter Tailscale)

**Tips:** Du kan fjerne port 80 og 443 fra security group for ekstra sikkerhet siden all tilgang nå går via Tailscale!

## 📦 Backup

### Backup n8n data
```bash
cd ~/n8n
tar -czf n8n-backup-$(date +%Y%m%d).tar.gz n8n_data/
```

### Restore fra backup
```bash
cd ~/n8n
sudo docker compose down
tar -xzf n8n-backup-YYYYMMDD.tar.gz
sudo docker compose up -d
```

## 🔄 Daglig Bruk Workflow

### Fra laptop:
1. Start Tailscale (hvis ikke allerede koblet til)
2. Åpne https://n8n.vuhnger.dev
3. Logg inn på n8n (huskes av nettleser)
4. Jobb med workflows!

### Fra mobil:
1. Åpne Tailscale-appen og koble til
2. Åpne Safari/Chrome
3. Gå til https://n8n.vuhnger.dev
4. Logg inn på n8n

**Du logger inn i Tailscale én gang per enhet, deretter er det alltid tilgjengelig!**

## 🌐 Dele tilgang med andre (valgfritt)

Hvis du vil gi familie/kolleger tilgang:

1. Gå til https://login.tailscale.com/admin/machines
2. Finn "victor-research-1" (serveren)
3. Klikk "Share..."
4. Legg til deres Tailscale-konto

De får da tilgang til n8n via samme URL: https://n8n.vuhnger.dev

## 📚 Ressurser

- n8n Dokumentasjon: https://docs.n8n.io
- Caddy Dokumentasjon: https://caddyserver.com/docs/
- Tailscale Dokumentasjon: https://tailscale.com/kb/
- NREC Dokumentasjon: https://docs.nrec.no

## 🆘 Support

**n8n Community:**
- Forum: https://community.n8n.io
- GitHub: https://github.com/n8n-io/n8n

**Tailscale:**
- Support: https://tailscale.com/contact/support
- Forum: https://forum.tailscale.com

**Server info:**
- SSH: `ssh rocky@158.37.66.204` eller `ssh nrec`
- OS: Rocky Linux 8
- Docker version: 26.1.3
- Docker Compose version: v2.27.0
- Tailscale version: 1.92.3

## 🎯 Quick Reference

| Hva | Kommando/URL |
|-----|--------------|
| Tilgang til n8n | https://n8n.vuhnger.dev |
| Sjekk Tailscale status | `tailscale status` |
| Start n8n | `cd ~/n8n && sudo docker compose up -d` |
| Se n8n logger | `docker logs -f n8n` |
| Restart alt | `ssh nrec "cd ~/n8n && sudo docker compose restart && sudo systemctl restart caddy"` |
| Tailscale admin | https://login.tailscale.com/admin/machines |

---

**Sist oppdatert:** 2025-12-19  
**Versjon:** 2.0 (Tailscale-enabled)
