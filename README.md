# n8n Workflow Automation Setup

Dette er dokumentasjon for n8n installasjon på NREC GOLD Rocky Linux 8 server.

## 📋 Oversikt

**Server detaljer:**
- Server: GOLD Rocky Linux 8 (NREC)
- IPv4: 158.37.66.204
- IPv6: 2001:700:2:8200::237e
- Domene: n8n.vuhnger.dev
- Tailscale IP: 100.124.46.65 (valgfritt)
- Lokasjon: ~/n8n/

**Teknologier:**
- n8n (workflow automation)
- Docker & Docker Compose
- Caddy (reverse proxy med automatisk SSL)
- Tailscale (valgfritt privat VPN-nettverk)

## 🌍 Tilgang til n8n

**n8n er nå offentlig tilgjengelig via HTTPS!**

Endret 2025-12-20 for å muliggjøre Claude Desktop MCP-integrasjon.

### Tilgangsmåter:

#### Anbefalt (via custom domain):
```
https://n8n.vuhnger.dev
```

#### Alternativt (via Tailscale, hvis aktivert):
```
http://100.124.46.65:5678
```

## 🔐 Sikkerhet

**n8n er trygt eksponert offentlig fordi:**

1. **Invite-only system**
   - INGEN offentlig registrering
   - Kun admin kan legge til brukere
   - Login required for all tilgang

2. **MCP Bearer Token Authentication**
   - MCP endpoint krever Authorization header
   - Uten token = ingen tilgang

3. **HTTPS/TLS**
   - Let's Encrypt SSL-sertifikat
   - Kryptert trafikk
   - Automatisk fornyelse

4. **Sterkt passord** (påkrevd ved første oppsett)

**Fremtidig forbedring:**
- Rate limiting (via fail2ban eller Caddy plugin)
- 2FA (to-faktor autentisering)

## 🤖 Claude Desktop MCP Integration

n8n kan nå brukes som MCP server for Claude Desktop, som gir Claude tilgang til workflows og tools.

### Konfigurering på Mac:

1. **Opprett/rediger Claude Desktop config:**
   ```bash
   nano ~/Library/Application\ Support/Claude/claude_desktop_config.json
   ```

2. **Legg til følgende:**
   ```json
   {
     "mcpServers": {
       "n8n": {
         "command": "npx",
         "args": [
           "-y",
           "supergateway",
           "--sse",
           "https://n8n.vuhnger.dev/mcp/628c7c7f-2ddb-420d-a50b-e84598ce1797",
           "--header",
           "Authorization: Bearer 628c7c7f-2ddb-420d-a50b-e84598ce1797"
         ]
       }
     }
   }
   ```

3. **Restart Claude Desktop** (Cmd+Q og åpne på nytt)

4. **Verifiser tilkobling:**
   - Claude Desktop skal vise "n8n: Connected"
   - Claude kan nå bruke n8n workflows som tools

**MCP Endpoint:**
- URL: `https://n8n.vuhnger.dev/mcp/628c7c7f-2ddb-420d-a50b-e84598ce1797`
- Auth: Bearer token `628c7c7f-2ddb-420d-a50b-e84598ce1797`

## 📁 Filstruktur

```
~/n8n/
├── docker-compose.yml       # Docker konfigurasjon for n8n
├── n8n_data/               # Persistent data (workflows, credentials, MCP tokens)
├── README.md               # Denne filen
├── INFRASTRUCTURE.md       # Teknisk infrastruktur dokumentasjon
└── .gitignore             # Git ignore regler
```

**VIKTIG:** `n8n_data/` inneholder sensitiv data og skal IKKE committes til git!

## 🛠️ Administrasjonskommandoer

### Start n8n:
```bash
cd ~/n8n
sudo docker compose up -d
```

### Stop n8n:
```bash
cd ~/n8n
sudo docker compose down
```

### Restart n8n:
```bash
cd ~/n8n
sudo docker compose restart
```

### Se n8n logger:
```bash
docker logs -f n8n
```

### Se n8n status:
```bash
docker ps
```

### Backup n8n data:
```bash
cd ~/n8n
tar -czf n8n-backup-$(date +%Y%m%d).tar.gz n8n_data/
```

### Restore n8n data:
```bash
cd ~/n8n
tar -xzf n8n-backup-YYYYMMDD.tar.gz
sudo docker compose down && sudo docker compose up -d
```

## 🔧 Troubleshooting

### Problem: n8n ikke tilgjengelig

**Sjekk at n8n kjører:**
```bash
docker ps
```

**Se n8n logger:**
```bash
docker logs -f n8n
```

**Restart n8n:**
```bash
cd ~/n8n && sudo docker compose restart
```

### Problem: Caddy ikke fungerer

**Sjekk Caddy status:**
```bash
ssh nrec "sudo systemctl status caddy"
```

**Se Caddy logger:**
```bash
ssh nrec "sudo journalctl -u caddy -f"
```

**Restart Caddy:**
```bash
ssh nrec "sudo systemctl restart caddy"
```

### Problem: SSL-sertifikat feil

Let's Encrypt sertifikater fornyes automatisk av Caddy. Hvis det oppstår problemer:

```bash
ssh nrec "sudo journalctl -u caddy -xe | grep -i cert"
```

### Problem: Claude Desktop MCP connection failed

**Verifiser at MCP endpoint er tilgjengelig:**
```bash
curl -I https://n8n.vuhnger.dev/mcp/628c7c7f-2ddb-420d-a50b-e84598ce1797 \
  -H "Authorization: Bearer 628c7c7f-2ddb-420d-a50b-e84598ce1797"
```

**Sjekk Claude Desktop logs:**
```bash
tail -f ~/Library/Logs/Claude/mcp-server-n8n.log
```

**Restart Claude Desktop:**
- Cmd+Q og åpne på nytt

## 🏥 Helsesjekk og Overvåking

n8n-instansen overvåkes av en egen helsesjekk-tjeneste i [backend-repoet](https://github.com/vuhnger/backend).

- **Helsesjekk API:** `https://api.vuhnger.dev/n8n/health`
- **Status Dashboard:** `https://api.vuhnger.dev`

Denne tjenesten verifiserer at n8n er tilgjengelig og svarer korrekt.

## 📚 Lenker

- **n8n Dokumentasjon:** https://docs.n8n.io/
- **n8n Community:** https://community.n8n.io/
- **Caddy Dokumentasjon:** https://caddyserver.com/docs/
- **Tailscale Dokumentasjon:** https://tailscale.com/kb/
- **MCP Dokumentasjon:** https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-langchain.mcptrigger/

## 📝 Endringslogg

### 2025-12-20
- ✅ Fjernet Tailscale-binding fra Caddy
- ✅ n8n nå offentlig tilgjengelig via HTTPS
- ✅ Lagt til Claude Desktop MCP integration
- ✅ Oppdatert sikkerhetsdokumentasjon

### 2025-12-19
- ✅ Initial setup med Tailscale-only tilgang
- ✅ HTTPS konfigurert med Let's Encrypt
- ✅ Caddy reverse proxy konfigurert

---

**Sist oppdatert:** 2025-12-20
