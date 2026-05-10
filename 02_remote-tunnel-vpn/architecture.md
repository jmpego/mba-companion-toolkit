---
title: "Architecture — Remote Tunnel via wstunnel"
version: 1.0
date: 2026-05-09
---

# Arquitectura — Remote Tunnel via wstunnel

## Topologia geral

```
┌─────────────────────────────┐         ┌──────────────────────────────────┐
│  Rede restritiva (hostil)   │         │   Tua casa                       │
│                             │         │                                  │
│  ┌─────────────────────┐    │         │   ┌──────────────────────┐       │
│  │ Cliente             │    │  HTTPS  │   │ Router doméstico     │       │
│  │ (Surface, Mac, iOS) │────┼─────────┼──▶│ + DMZ ou port-fwd 443│       │
│  │                     │    │  :443   │   └──────────┬───────────┘       │
│  │  - WireGuard client │    │  TLS    │              │                   │
│  │  - wstunnel client  │    │  (SNI=  │              ▼                   │
│  │                     │    │  yours) │   ┌──────────────────────┐       │
│  └─────────────────────┘    │         │   │ Mini-server          │       │
│                             │         │   │ (Raspberry Pi 5,     │       │
│  Apenas TLS:443 sai limpo   │         │   │  NUC, or similar)    │       │
│  Resto bloqueado.           │         │   │                      │       │
│                             │         │   │ - nginx (TLS term.)  │       │
└─────────────────────────────┘         │   │ - wstunnel server    │       │
                                        │   │ - WireGuard server   │       │
                                        │   │ - DDNS client        │       │
                                        │   │ - Pi-hole (optional) │       │
                                        │   └──────────┬───────────┘       │
                                        │              │                   │
                                        │              ▼                   │
                                        │   ┌──────────────────────┐       │
                                        │   │ LAN doméstica        │       │
                                        │   │ - NAS                │       │
                                        │   │ - desktop / iMac     │       │
                                        │   │ - dashboards         │       │
                                        │   │ - LLM self-hosted    │       │
                                        │   └──────────────────────┘       │
                                        └──────────────────────────────────┘
```

## Componentes

### 1. Cliente (na rede restritiva)

| Componente | Função | Notas |
|---|---|---|
| **WireGuard client** | Termina o tunnel, encripta tráfego camada IP | Endereço local virtual `10.0.0.X`, peer = `127.0.0.1:47111` (loopback) |
| **wstunnel client** | Encapsula UDP/47111 em WebSocket sobre TLS:443 | Listen `udp://127.0.0.1:47111`, forward to `wss://<your-domain>:443` |
| **Path secret** | "Password" para que server reconheça cliente legítimo | Random string ~32 chars, em ambiente local protegido |

**Fluxo do tráfego (client → home):**
1. App tenta aceder a `192.168.X.Y` (rede de casa)
2. WireGuard captura, encripta como pacote WireGuard, envia para `127.0.0.1:47111`
3. wstunnel client lê UDP:47111, encapsula em WebSocket frame, envia HTTPS:443 para servidor de casa
4. Tráfego sai da rede restritiva como HTTPS normal — indistinguível de browsing

### 2. Servidor (em casa)

| Componente | Função | Notas |
|---|---|---|
| **DDNS client** | Mapeia hostname → IP público dinâmico do router | DuckDNS, no-ip free, ou hostname do teu provider |
| **Router DMZ / port-fwd** | Encaminha 443 → mini-server | DMZ é mais simples, port-fwd é mais granular |
| **wstunnel server** | Termina WebSocket, devolve UDP a WireGuard server | Listen `wss://0.0.0.0:443`, com path matching |
| **WireGuard server** | Termina tunnel WireGuard, roteia para LAN | Peers configurados manualmente |
| **TLS cert (Let's Encrypt)** | Cert válido para hostname → TLS handshake bem-sucedido | Certbot auto-renew |
| **(Opcional) nginx reverse proxy** | TLS termination + SNI plausibility | Útil se quiseres servir página fake em `/` para casual scanning |

**Fluxo do tráfego (home → client):**
- Mesmo caminho inverso. Tudo simétrico.

### 3. Path Secret — único factor de discovery defense

```
URL real: wss://your-domain.com/abc123XYZ-random-32-chars
                              ↑
                              path secret
```

- Sem path secret correcto, o server retorna **HTTP 404** ou serve uma página fake
- Casual port scanner que tenta `wss://your-domain:443/` recebe 404 — não sabe que há tunnel
- **Apenas quem souber o path secret** consegue handshake

## Decisões de design

### D1 — Porquê WebSocket e não HTTP/2?

WebSocket é **bidirecional, low-latency, persistente**. HTTP/2 funciona mas requer mais setup. WebSocket sobre TLS:443 é o padrão dominante para tunneling em redes restritivas (Trojan, V2Ray, etc.).

### D2 — Porquê WireGuard inside e não OpenVPN?

WireGuard é **modern, fast, simple** (~4000 lines of code). OpenVPN funciona mas é overkill para este caso. Single-config-file model do WireGuard simplifica muito.

### D3 — Porquê não Tailscale?

Tailscale é excelente. **Mas:**
- Dados passam por relays Tailscale (DERP) quando direct connection não é possível
- Requer conta Tailscale (terceiro)
- Não funciona em redes que bloqueiam DERP IP ranges (raro mas acontece)

Self-hosted = full control + zero terceiros + auditável.

### D4 — Porquê Raspberry Pi 5 (8GB)?

| Factor | Razão |
|---|---|
| **Power consumption** | ~5-8W idle. €2-4/mês de electricidade |
| **Always-on** | Sem fan ruidoso, sem barulho |
| **8GB RAM** | Espaço para Pi-hole + wstunnel + WireGuard + monitoring + outros containers |
| **GPIO** | Possibilidade de extras (UPS HAT, sensors) |
| **Software stack** | Raspbian/Debian stable, packages standard |

Alternativas válidas: Mac mini (mais power but mais $), NUC (mais power, mais $), Synology DS923+ (dedicated NAS but locked-down OS).

### D5 — Porquê DMZ no router (e não port-fwd só 443)?

Trade-off:
- **DMZ:** simples, todos os ports redirected, mais surface de ataque (mas o servidor só ouve 443 + 22 SSH)
- **Port-fwd 443 só:** mais restritivo, mais limpo, mas tens de port-fwd cada port que vais expor

Para single-tunnel setup, port-fwd 443 só é mais defendível. Para flexibilidade futura (expor mais services), DMZ é mais prático.

## Considerações de segurança

| Vector | Mitigação |
|---|---|
| **Path secret leak** | Rotation periódica (ver `setup-runbook.md` §9). Nunca em git, screenshots, ou ficheiros markdown partilhados |
| **TLS cert expira** | Let's Encrypt + certbot auto-renew (90 dias) |
| **DDoS no port 443** | fail2ban + rate limiting nginx |
| **Compromise do mini-server** | Updates regulares, SSH key-only login (no password), firewall restritivo, tcpdump occasional |
| **Adversary com control da rede restritiva** | TLS-encrypted traffic. Se eles fazem **active MITM**, perderás (mas isso requer corporate-grade DLP que não é típico em hospitais/hotéis) |
| **Path secret discovery por brute-force** | 32-char random = ~10^57 possibilidades. Brute-force impossível. **Mas** se acidentalmente vazas via DNS query / log / screen share, expõe-se imediatamente |

## Limites conhecidos

- ⚠️ **Networks com DPI (Deep Packet Inspection) avançado** — algumas redes corporate fazem TLS fingerprinting (JA3) e podem flag tráfego não-browser. Mitigação parcial: usar wstunnel client com TLS lib que mimica Chrome/Firefox JA3
- ⚠️ **Redes que apenas permitem proxy HTTP explícito** — wstunnel suporta mas requer config extra
- ⚠️ **iOS** — não há cliente wstunnel nativo. Workaround: app **Amnezia VPN** (parameters compatible) ou wireguard-go custom builds
- ⚠️ **Captive portals** — antes do tunnel subir, tens de passar o login portal (que normalmente bloqueia tudo). Conectar a wifi → abrir browser → fazer login → depois iniciar tunnel
- ⚠️ **Latency** — overhead de WebSocket + TLS termination + WireGuard re-encryption = ~10-30ms additional vs WireGuard nativo. Adequado para SSH, RDP, dashboards. Não-ideal para gaming/voice realtime

---

*Próximo: `setup-runbook.md` — passo-a-passo de implementação.*
