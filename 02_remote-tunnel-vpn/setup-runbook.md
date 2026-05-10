---
title: "Setup Runbook — wstunnel server + clients"
version: 1.0
date: 2026-05-09
audience: Operator a fazer setup pela primeira vez
estimated_time: 4-6h primeira vez (com curva de aprendizagem)
---

# Setup Runbook — wstunnel Server + Clients

> **Convenções nos comandos:**
> - `<your-domain>` = o teu hostname DDNS (e.g. `myhome.duckdns.org`)
> - `<HOME_PUBLIC_IP>` = IP público do teu router (`curl ifconfig.me`)
> - `<PATH_SECRET>` = string aleatória de 32 caracteres (gera com `openssl rand -base64 24`)
> - `<RPI_LAN_IP>` = IP local do teu mini-server (e.g. `192.168.1.10`)
> - `<USER>` = teu username (no Pi e no cliente)

---

## Parte A — Servidor (Raspberry Pi / Linux)

### A.1 Preparar o Pi

Assumindo Raspbian/Debian recém-instalado, com SSH acessível e user `<USER>` com sudo.

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget unzip ufw fail2ban nginx certbot python3-certbot-nginx
```

Configurar firewall básico (deixar SSH e HTTPS apenas):

```bash
sudo ufw allow 22/tcp comment 'SSH'
sudo ufw allow 443/tcp comment 'HTTPS / wstunnel'
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw enable
```

### A.2 DDNS (DuckDNS exemplo)

1. Cria conta gratuita em https://www.duckdns.org
2. Reserva subdomain (e.g. `myhome.duckdns.org`)
3. Copia o token

No Pi:

```bash
mkdir -p ~/duckdns
cat > ~/duckdns/duck.sh <<'EOF'
echo url="https://www.duckdns.org/update?domains=YOUR_SUBDOMAIN&token=YOUR_TOKEN&ip=" | curl -k -o ~/duckdns/duck.log -K -
EOF
chmod +x ~/duckdns/duck.sh
~/duckdns/duck.sh && cat ~/duckdns/duck.log  # deve dizer "OK"
```

Adicionar ao crontab (update a cada 5 min):

```bash
crontab -e
# adicionar linha:
*/5 * * * * /home/<USER>/duckdns/duck.sh >/dev/null 2>&1
```

### A.3 Router — DMZ ou Port Forward

Login na admin page do router (típico `192.168.1.1` ou `192.168.0.1`).

**Opção mais simples:**
- DMZ → `<RPI_LAN_IP>` (o mini-server fica directamente exposto a 443)

**Opção mais segura:**
- Port forward `443/tcp` → `<RPI_LAN_IP>:443`

Testar de fora da rede de casa:
```bash
# de outro device com 4G/dados móveis (não wifi de casa)
curl -v https://<your-domain>:443
# deve responder algo (nginx default page ou cert error — cert vamos resolver a seguir)
```

### A.4 TLS cert via Let's Encrypt

```bash
sudo certbot --nginx -d <your-domain>
# segue prompts: email, ToS yes, redirect HTTP→HTTPS yes
```

Verifica auto-renew:
```bash
sudo certbot renew --dry-run
# deve dizer "Congratulations, all renewals succeeded"
```

### A.5 Instalar wstunnel server

```bash
WSTUNNEL_VERSION="10.1.4"  # conferir release latest em github.com/erebe/wstunnel
ARCH=$(dpkg --print-architecture)  # arm64 para Raspberry Pi 5 64-bit
cd /tmp
wget "https://github.com/erebe/wstunnel/releases/download/v${WSTUNNEL_VERSION}/wstunnel_${WSTUNNEL_VERSION}_linux_${ARCH}.tar.gz"
tar xzf wstunnel_${WSTUNNEL_VERSION}_linux_${ARCH}.tar.gz
sudo mv wstunnel /usr/local/bin/
wstunnel --version  # confirma 10.1.4
```

### A.6 Gerar path secret + guardar em `/etc/wstunnel.env`

```bash
SECRET=$(openssl rand -base64 24 | tr -d '/+=' | head -c 32)
echo "PATH_SECRET=$SECRET" | sudo tee /etc/wstunnel.env
sudo chmod 600 /etc/wstunnel.env
sudo cat /etc/wstunnel.env  # guarda este valor em sítio seguro — vais precisar para os clientes
```

> ⚠️ **Não comites este valor em git.** Não o cole em chats. Apenas em `/etc/wstunnel.env` (root-only) e nos clientes (em ambiente local protegido).

### A.7 Systemd service para wstunnel server

```bash
sudo tee /etc/systemd/system/wstunnel.service > /dev/null <<'EOF'
[Unit]
Description=wstunnel server
After=network.target

[Service]
Type=simple
EnvironmentFile=/etc/wstunnel.env
ExecStart=/usr/local/bin/wstunnel server \
  --restrict-http-upgrade-path-prefix=${PATH_SECRET} \
  wss://0.0.0.0:8443
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now wstunnel
sudo systemctl status wstunnel  # deve dizer "active (running)"
```

> Nota: usei `8443` interno (não 443) porque nginx vai escutar em 443 e fazer reverse-proxy para 8443. Mantém TLS termination centralizado.

### A.8 Configurar nginx reverse proxy

Editar `/etc/nginx/sites-available/<your-domain>` (criado pelo certbot em A.4):

```nginx
server {
    listen 443 ssl http2;
    server_name <your-domain>;

    ssl_certificate /etc/letsencrypt/live/<your-domain>/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/<your-domain>/privkey.pem;

    # WebSocket upgrade para wstunnel
    location / {
        # Default: serve uma página fake (e.g., personal landing page)
        return 200 'Hello world!';
        add_header Content-Type text/plain;
    }

    # Match path secret -- wstunnel handler
    location ~ ^/<PATH_SECRET> {
        proxy_pass http://127.0.0.1:8443;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_read_timeout 86400;
    }
}
```

> Substitui `<PATH_SECRET>` pelo valor real (escapando caracteres especiais se houver). Recarrega nginx:

```bash
sudo nginx -t  # syntax check
sudo systemctl reload nginx
```

### A.9 WireGuard server

```bash
sudo apt install -y wireguard
cd /etc/wireguard
sudo wg genkey | sudo tee server.key | sudo wg pubkey | sudo tee server.pub
sudo chmod 600 server.key
```

Cria `/etc/wireguard/wg0.conf`:

```ini
[Interface]
Address = 10.0.0.1/24
ListenPort = 47111
PrivateKey = <conteúdo de /etc/wireguard/server.key>
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
# Cliente Surface / Mac (preencher com publickey do cliente após gerar)
PublicKey = <CLIENT_PUBKEY_PLACEHOLDER>
AllowedIPs = 10.0.0.2/32
```

Activar IP forwarding:

```bash
echo 'net.ipv4.ip_forward=1' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

Activar serviço (depois de configurar pelo menos 1 peer):

```bash
sudo systemctl enable --now wg-quick@wg0
sudo systemctl status wg-quick@wg0
```

---

## Parte B — Cliente Windows (PowerShell)

### B.1 Pré-requisitos

- Windows 10/11
- WireGuard for Windows instalado (https://www.wireguard.com/install/)
- PowerShell 5.1+

### B.2 Download wstunnel client

Em https://github.com/erebe/wstunnel/releases, baixar `wstunnel_X.Y.Z_windows_amd64.tar.gz`. Extrair `wstunnel.exe` para `C:\Tools\wstunnel\`.

### B.3 Gerar par de chaves WireGuard

Abrir WireGuard for Windows → "Add Tunnel" → "Add empty tunnel".
Copiar a public key gerada e enviar para o admin do server (`<CLIENT_PUBKEY_PLACEHOLDER>` em A.9).

Configurar tunnel:

```ini
[Interface]
PrivateKey = <gerada pelo WireGuard for Windows>
Address = 10.0.0.2/24
DNS = 1.1.1.1

[Peer]
PublicKey = <server pubkey, /etc/wireguard/server.pub>
Endpoint = 127.0.0.1:47111
AllowedIPs = 10.0.0.0/24, <HOME_LAN_SUBNET>/24
PersistentKeepalive = 25
```

> **`Endpoint = 127.0.0.1:47111`** — não é o servidor real! É o loopback onde o wstunnel client vai escutar. wstunnel vai redireccionar para o server real.

> **`AllowedIPs`** — define o que vai pelo tunnel. `10.0.0.0/24` é a rede interna VPN. `<HOME_LAN_SUBNET>/24` é a tua LAN doméstica (e.g. `192.168.1.0/24`). **Não uses `0.0.0.0/0`** — vai mandar TODO o tráfego pelo tunnel, defeating the plausibility argument.

### B.4 Script PowerShell para iniciar tunnel

`C:\Tools\wstunnel\start-tunnel.ps1`:

```powershell
$Server = "wss://<your-domain>:443"
$PathSecret = "<PATH_SECRET>"
$LocalBind = "udp://127.0.0.1:47111"
$RemoteFwd = "127.0.0.1:47111"

& "C:\Tools\wstunnel\wstunnel.exe" client `
  --log-lvl=INFO `
  -L "$LocalBind`?timeout_sec=0>$RemoteFwd" `
  -P $PathSecret `
  $Server
```

> ⚠️ ACL: restringir leitura deste ficheiro (contém path secret):
> ```powershell
> icacls C:\Tools\wstunnel\start-tunnel.ps1 /inheritance:r /grant:r "$env:USERNAME:R"
> ```

### B.5 Decision tree de uso

```
Estás numa rede:
  ├── A) Casa (LAN) → não precisas de tunnel; conecta directo a <RPI_LAN_IP>
  ├── B) 4G / hotspot pessoal → wireguard direct para <your-domain>:47111 funciona (sem wstunnel)
  └── C) Rede restritiva (hospital, hotel, eduroam, corporate)
        ├── 1. Iniciar `start-tunnel.ps1` (PowerShell)
        ├── 2. Esperar ~5s (TLS handshake + WS upgrade)
        ├── 3. Activar tunnel WireGuard for Windows
        ├── 4. Testar: `ping 10.0.0.1` deve responder
        └── 5. Aceder ao home server: SSH, browser para dashboards, etc.
```

---

## Parte C — Cliente macOS (Terminal)

### C.1 Instalar dependências

```bash
brew install wireguard-tools
brew install --cask wireguard-tools
# wstunnel via cargo OR baixar binário Mach-O do github releases
```

Para wstunnel, descarrega binário darwin_arm64 (Apple Silicon) ou darwin_amd64 (Intel) de github releases. Move para `/usr/local/bin/wstunnel`.

### C.2 Config WireGuard

Cria `~/wg0.conf` (similar ao Windows, secção B.3):

```ini
[Interface]
PrivateKey = <gerada via 'wg genkey'>
Address = 10.0.0.3/24
DNS = 1.1.1.1

[Peer]
PublicKey = <server pubkey>
Endpoint = 127.0.0.1:47111
AllowedIPs = 10.0.0.0/24, <HOME_LAN_SUBNET>/24
PersistentKeepalive = 25
```

### C.3 Script para iniciar tunnel

`~/start-tunnel.sh`:

```bash
#!/bin/bash
set -e

SERVER="wss://<your-domain>:443"
PATH_SECRET="<PATH_SECRET>"  # ou source de ~/.wstunnel.env protegido

wstunnel client \
  --log-lvl=INFO \
  -L "udp://127.0.0.1:47111?timeout_sec=0>127.0.0.1:47111" \
  -P "$PATH_SECRET" \
  "$SERVER"
```

```bash
chmod 700 ~/start-tunnel.sh
chmod 600 ~/.wstunnel.env  # se separares secret
```

Iniciar tunnel:
```bash
# terminal 1: wstunnel
~/start-tunnel.sh

# terminal 2: wireguard
sudo wg-quick up ~/wg0.conf
```

---

## Parte D — Cliente iOS (workaround)

iOS não tem cliente wstunnel nativo. **Workaround:** usar app **Amnezia VPN** (free, open source). Configuração compatível com WireGuard mas com obfuscation built-in.

Alternativa: **WireGuard sem wstunnel** se a rede do iOS device permitir UDP nativo (4G data, casa). Em redes restritivas, Amnezia é o mais próximo.

---

## Parte E — Path secret rotation (a cada 90 dias)

```bash
# No servidor:
NEW_SECRET=$(openssl rand -base64 24 | tr -d '/+=' | head -c 32)
sudo sed -i "s/^PATH_SECRET=.*/PATH_SECRET=$NEW_SECRET/" /etc/wstunnel.env
sudo systemctl restart wstunnel
echo "New secret: $NEW_SECRET"
# Update nginx config (location ~ ^/<NEW_SECRET>) e reload nginx

# Em cada cliente:
# - Update start-tunnel script com $NEW_SECRET
# - Test handshake
```

---

## Smoke test final

Em rede restritiva (hospital, café, hotel):

1. ✅ Wifi conectado, browser funciona em sites HTTPS
2. ✅ Iniciar wstunnel client (PowerShell ou shell script)
3. ✅ wstunnel logs mostram "TLS handshake successful"
4. ✅ Activar WireGuard tunnel
5. ✅ `ping 10.0.0.1` responde
6. ✅ `ssh user@<RPI_LAN_IP>` connects
7. ✅ Browser to `http://<RPI_LAN_IP>` dashboard funciona

Se algum destes 7 falha, ver `troubleshooting.md`.

---

*Próximo: `troubleshooting.md` — top 6 problemas + escalation ladder.*
