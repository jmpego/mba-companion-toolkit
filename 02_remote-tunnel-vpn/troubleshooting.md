---
title: "Troubleshooting — wstunnel + WireGuard"
version: 1.0
date: 2026-05-09
---

# Troubleshooting — wstunnel + WireGuard

## Diagnóstico estruturado

Antes de mergulhar em troubleshooting específico, isolar a camada onde falha:

```
[Cliente App] ──A──▶ [WireGuard tunnel] ──B──▶ [wstunnel client] ──C──▶ [Network] ──D──▶ [Server wstunnel] ──E──▶ [Server WireGuard] ──F──▶ [Home LAN]
```

| Camada | Sintoma típico | Como testar |
|---|---|---|
| **A** App→WG | App diz "no route to host" | `ping 10.0.0.1` (server VPN IP) |
| **B** WG→wstunnel | WG tunnel não estabelece (handshake timeout) | Logs do WireGuard for Windows / `wg show` |
| **C** Network | wstunnel dá "TLS handshake failed" / "connection refused" | Logs wstunnel client com `--log-lvl=DEBUG` |
| **D** Server wstunnel | Connection chega a 443 mas server retorna 404 | `sudo journalctl -u wstunnel -f` no Pi |
| **E** Server WG | wstunnel OK, WG handshake falha | `sudo wg show` no Pi |
| **F** Routing | WG OK, mas não acedes LAN | `ip route` no Pi, IP forwarding, NAT |

Identifica a camada **primeiro**, depois aplica o fix específico.

---

## Top 6 problemas comuns

### Problema 1 — "TLS handshake failed" / "connection refused"

**Sintoma:** wstunnel client logs `error: TLS handshake failed` ou `connection refused`.

**Causas possíveis:**

1. **Firewall do cliente bloqueia outbound 443** (raro mas existe em corporate strict)
2. **Rede restritiva tem proxy HTTP forçado** (não permite TLS direct)
3. **DNS hijack / SNI inspection** intercepta `<your-domain>` e redirecciona
4. **Cert inválido** (expired, hostname mismatch)
5. **Server wstunnel down** (raríssimo se monitor activo)

**Diagnóstico:**

```bash
# 1. Confirma resolução DNS
nslookup <your-domain>
# Se retorna IP estranho (e.g. 10.x.x.x da rede local), há hijack

# 2. Confirma que TLS:443 sai
curl -v https://<your-domain>
# Se devolve cert + página, base layer OK

# 3. Confirma cert válido
openssl s_client -connect <your-domain>:443 -servername <your-domain> < /dev/null
# Verifica "Verify return code: 0 (ok)"
```

**Fix se DNS hijack confirmado:** ver Problema 4 (SNI override + IP directo).

### Problema 2 — wstunnel conecta, mas WireGuard handshake timeout

**Sintoma:** wstunnel logs OK, mas `wg show` no cliente diz "latest handshake: never" ou WG tunnel inactive.

**Causas:**

1. **Server WG não está a correr** ou config errada
2. **Path do wstunnel forwarding errado** (UDP:47111 não chega ao WG server)
3. **Public key cliente não está nos peers do server**

**Diagnóstico:**

```bash
# No servidor (Pi):
sudo systemctl status wg-quick@wg0   # active?
sudo wg show                          # peers e handshakes
sudo journalctl -u wg-quick@wg0 -n 50

# No cliente (Windows):
# WireGuard for Windows → Logs → ver mensagens
```

**Fix típico:** confirmar que `[Peer]` no `/etc/wireguard/wg0.conf` do server tem a public key correcta do cliente. Após editar, `sudo systemctl restart wg-quick@wg0`.

### Problema 3 — Tunnel sobe, mas não consigo aceder à LAN doméstica

**Sintoma:** `ping 10.0.0.1` OK, mas `ping 192.168.1.10` (LAN) timeout.

**Causas:**

1. **`AllowedIPs` no cliente não inclui a subnet LAN** (e.g. apenas `10.0.0.0/24`, falta `192.168.1.0/24`)
2. **IP forwarding desactivado no Pi** (`net.ipv4.ip_forward=0`)
3. **NAT/MASQUERADE não configurado** (PostUp/PostDown rules em `wg0.conf`)
4. **Firewall do Pi bloqueia FORWARD** (ufw default-deny forwarded)

**Diagnóstico:**

```bash
# No Pi:
sysctl net.ipv4.ip_forward   # deve ser 1
sudo iptables -L FORWARD -n  # deve ter ACCEPT do interface wg0
sudo iptables -t nat -L POSTROUTING -n  # deve ter MASQUERADE para eth0
```

**Fix:**

```bash
# 1. Activar forwarding (se não está):
echo 'net.ipv4.ip_forward=1' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# 2. Confirmar PostUp/PostDown em /etc/wireguard/wg0.conf:
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

# 3. Restart WG:
sudo systemctl restart wg-quick@wg0
```

E no cliente, `[Peer]` `AllowedIPs` deve incluir a tua LAN: `10.0.0.0/24, 192.168.1.0/24`.

### Problema 4 — DNS hijack / SNI inspection (rede hostil avançada)

**Sintoma:** TLS handshake falha apenas nesta rede específica, funciona em outras (4G, casa, café). DNS resolve `<your-domain>` para IP errado.

**Fix — Escalation Ladder:**

#### Nível 1 — SNI override + IP directo

```powershell
# Windows (PowerShell)
& "C:\Tools\wstunnel\wstunnel.exe" client `
  --log-lvl=DEBUG `
  --tls-sni-override <your-domain> `
  -L "udp://127.0.0.1:47111?timeout_sec=0>127.0.0.1:47111" `
  -P <PATH_SECRET> `
  wss://<HOME_PUBLIC_IP>:443
```

```bash
# macOS / Linux
wstunnel client \
  --log-lvl=DEBUG \
  --tls-sni-override <your-domain> \
  -L "udp://127.0.0.1:47111?timeout_sec=0>127.0.0.1:47111" \
  -P <PATH_SECRET> \
  wss://<HOME_PUBLIC_IP>:443
```

> **Por que funciona:** `wss://<IP>:443` bypass do DNS resolution local (DNS hijack já não interfere). `--tls-sni-override` força SNI=hostname para o cert Let's Encrypt validar (server cert é para hostname, não para IP).

#### Nível 2 — Tentar porto alternativo

Se o nível 1 falha, talvez a rede inspect TLS:443 destination IP. Tenta:
- Server: configurar wstunnel adicional em `wss://0.0.0.0:8443` (com cert + nginx adicional)
- Client: tentar 8443 em vez de 443

Algumas redes apenas inspeccionam 443.

#### Nível 3 — Domain fronting (avançado)

Algumas CDNs (Cloudflare, AWS CloudFront) permitem **domain fronting** — TLS SNI=`legitimate-cdn-host.com` mas Host header=`your-domain.com`. Server termina e routeia.

> ⚠️ Cloudflare e outras CDNs **bloquearam domain fronting** desde 2018. Funciona em CDNs menos populares (raro encontrar configurável free).

#### Nível 4 — Tor bridge + WG

Se a rede bloqueia tudo excepto Tor (improvável mas possível):
- Iniciar Tor browser → confirmar conectividade
- Configurar wireguard-go com proxy Tor SOCKS5

#### Nível 5 — Pivot 4G hotspot pessoal

Se chegaste aqui sem sucesso, a rede é hostil de mais. **Solução pragmática:**
- Activar hotspot no telemóvel (4G)
- Conectar laptop ao hotspot
- Tunnel funciona (4G data não bloqueia)
- Custo: data pessoal ~50-200MB / hora SSH light

> Esta é a "solução de último recurso". Para casos crónicos (hospital, escritório onde estás muitas horas), considera plano 4G específico (e.g. cartão SIM data dedicado, ~€10-20/mês 30GB).

### Problema 5 — VS Code Remote-SSH "spinning forever" depois de VPN reconnect

**Sintoma:** já tinhas sessão VS Code Remote-SSH a funcionar. VPN dropou (reboot tunnel, troca de rede). Após reactivar VPN, VS Code não recupera.

**Causa:** o daemon `node` do VS Code Server no servidor remoto fica em estado preso quando o socket TCP que estava em uso muda de rota.

**Fix (rápido):** `F1` → "Remote-SSH: Kill VS Code Server on Host" → reconecta normal.

**Tempo total:** ~30 segundos.

> Não é falha do servidor. É padrão conhecido de VS Code Server. Vale automatizar com tecla rápida ou snippet PowerShell que faz SSH e mata o processo node remoto.

### Problema 6 — Path secret leak (incident response)

**Sintoma:** suspeita de leak — partilhaste por engano em screenshot, chat, ou git commit acidental.

**Resposta imediata (~10 min):**

```bash
# 1. Gerar novo secret
NEW_SECRET=$(openssl rand -base64 24 | tr -d '/+=' | head -c 32)

# 2. No servidor: actualizar /etc/wstunnel.env e nginx config
sudo sed -i "s/^PATH_SECRET=.*/PATH_SECRET=$NEW_SECRET/" /etc/wstunnel.env
sudo sed -i "s/<OLD_SECRET>/$NEW_SECRET/" /etc/nginx/sites-available/<your-domain>
sudo systemctl restart wstunnel
sudo systemctl reload nginx

# 3. Em cada cliente: update start-tunnel script
# (Windows: editar C:\Tools\wstunnel\start-tunnel.ps1)
# (macOS/Linux: editar ~/start-tunnel.sh)

# 4. Verificar logs do server por tentativas de uso do secret antigo
sudo journalctl -u wstunnel --since "1 hour ago" | grep -i "old_secret_hint"

# 5. (Opcional) Audit nginx access logs
sudo grep "<OLD_SECRET>" /var/log/nginx/access.log
```

**Se vazou em git:**
- Rotaciona secret (acima)
- Verifica histórico git: `git log --all --full-history -p -- '*.md' | grep <OLD_SECRET>`
- Se confirmado em commit já pushed: **rebase + force push é possível mas não suficiente** (já estará em mirrors / forks). Mais seguro = considerar o secret comprometido e rotar.

---

## Logs úteis

### Cliente Windows
```powershell
# wstunnel logs (correr com --log-lvl=DEBUG)
# WireGuard for Windows logs
%PROGRAMDATA%\WireGuard\Logs\
```

### Cliente macOS / Linux
```bash
# wstunnel: stdout do processo
# WireGuard:
sudo wg show
sudo journalctl -u wg-quick@wg0 -n 100
```

### Servidor (Pi)
```bash
sudo journalctl -u wstunnel -f         # wstunnel server live
sudo journalctl -u wg-quick@wg0 -f     # WireGuard server live
sudo tail -f /var/log/nginx/access.log # nginx connections
sudo tail -f /var/log/nginx/error.log  # nginx errors
sudo wg show                            # WG state
ip route                                 # routing table
sudo iptables -L FORWARD -n -v          # forward stats
```

---

## Quando pedir ajuda externa

Se chegaste ao Nível 5 da escalation ladder do Problema 4 e ainda não funciona em casos não-críticos, considera:

1. **Comunidade wstunnel** — github.com/erebe/wstunnel/issues
2. **Reddit r/selfhosted, r/homelab** — comunidade activa, dispostos a ajudar
3. **Servidor Discord / Matrix** específico de WireGuard ou wstunnel

Antes de postar, ter pronto:
- Versão wstunnel cliente + servidor
- OS cliente + servidor
- Logs `--log-lvl=DEBUG` (sanitizados de path secret)
- Output `nslookup <your-domain>` desde a rede problemática
- O que já tentaste (níveis 1-5)

> Não partilhes path secret. Não partilhes IP público se preferires não. Sanitiza logs antes de colar.

---

*Fim do troubleshooting. Para setup inicial: `setup-runbook.md`. Para arquitectura: `architecture.md`.*
