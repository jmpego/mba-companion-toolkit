---
title: "Remote Tunnel / VPN — Adoption Guide"
version: 1.0
date: 2026-05-09
license: CC BY-NC-SA 4.0
---

# Remote Tunnel / VPN — como adoptar

## O problema que resolve

Tens uma infraestrutura em casa — NAS, dashboard, agentes IA self-hosted, ferramentas de produtividade que vivem num mini-servidor (Raspberry Pi, NUC, Mac mini). Mas em **redes restritivas** — hospitais, hotéis, universidades, corporate networks, eduroam — não consegues aceder porque:

- **WireGuard nativo (UDP) é bloqueado** em quase todas estas redes
- **Portas custom (não-443) são bloqueadas** por proxy/firewall
- **DNS hijack / SNI inspection** detecta tunnels e bloqueia
- **Apenas tráfego HTTPS:443 sai limpo**

Solução clássica: tunnel WireGuard **encapsulado em HTTPS WebSocket** (`wstunnel`). O tráfego parece WebSocket comum sobre TLS:443 — indistinguível de browsing normal. O servidor em casa termina o tunnel e o tráfego flui para a tua rede doméstica.

## Para que casos faz sentido

| Caso | Aplicabilidade |
|---|---|
| Acedo ao meu NAS / dashboard de casa em terreno | ✅ Alto valor |
| Trabalho remoto via SSH / RDP a um servidor de casa | ✅ Alto valor |
| Self-hosted AI/LLM accessível de qualquer rede | ✅ Alto valor |
| Backup remoto / sync seguro | ✅ Alto valor |
| Browsing genérico (substituir VPN comercial) | ⚠️ Possível mas overkill |
| Streaming Netflix de outro país | ❌ Não é o objectivo deste setup |

## Conteúdo desta pasta

| Ficheiro | Para que serve |
|---|---|
| `README.md` (este) | Overview + decisão de adopção |
| `architecture.md` | Diagrama topológico + componentes + decisões de design |
| `setup-runbook.md` | Passo-a-passo genérico (servidor + cliente Windows + cliente macOS) |
| `troubleshooting.md` | Top 6 problemas comuns + diagnóstico estruturado |

## Stack assumida

- **Servidor em casa:** mini-PC ou Raspberry Pi com Linux (Debian/Ubuntu/Raspbian recomendado)
- **Router doméstico:** capaz de DMZ ou port-forward 443 → servidor
- **DDNS:** DuckDNS, no-ip, ou DNS próprio gratuito
- **TLS cert:** Let's Encrypt via certbot (gratuito, auto-renew)
- **Cliente:** Windows (PowerShell), macOS (Terminal), iOS/Android (Amnezia VPN ou wireguard-go custom builds)

## Custo de adopção

| Item | Custo |
|---|---|
| Mini-PC (se não tiveres) | ~€80-150 (Raspberry Pi 5 8GB + caixa + PSU + microSD) |
| Tempo de setup inicial | 1 fim-de-semana (4-8h) |
| Manutenção mensal | ~30 min (apt update, cert renew automático) |
| Tempo de adaptação a uma rede nova restritiva | 5-30 min (escalation ladder em `troubleshooting.md`) |

## Avisos honestos

- ⚠️ **Não substitui VPN corporate enterprise-grade.** Não tem alta disponibilidade, failover automático, ou suporte 24/7. É **single-home setup pessoal**.
- ⚠️ **Path secret é o único factor de segurança contra discovery casual.** Se vazares, qualquer um pode tentar TLS handshake. **Não comites em git, não partilhes em chats, rotaciona regularmente** (script de rotation incluído em `setup-runbook.md` §9).
- ⚠️ **Algumas redes hostis bloqueiam mesmo com SNI override.** Existe uma escalation ladder de 5 níveis no `troubleshooting.md`. Se o nível 5 falhar, é improvável conseguires nessa rede sem pivot via 4G hotspot.
- ⚠️ **Legal:** verifica a política de uso aceitável (AUP) da rede onde te conectas. Em algumas redes corporate, tunneling para fora é violação contratual.

## Filosofia de design

1. **Privacy by self-hosting.** Os teus dados ficam em casa. Sem terceiro a inspeccionar tráfego.
2. **Plausible deniability.** Tráfego parece HTTPS normal. Não sticks out.
3. **Resilience over polish.** Setup deliberadamente boring — boring tech que funciona em produção há anos.
4. **Observable.** Logs, secrets rotation, monitoring básico. Não é black box.

## Adopção em 5 passos

1. **Lê `architecture.md`** (10 min) — percebe o que vais montar
2. **Compra/prepara o mini-server** (1h se já tens; ~24h se vais comprar Raspberry Pi)
3. **Segue `setup-runbook.md`** secção a secção (2-4h primeira vez)
4. **Testa em casa primeiro** (mesmo LAN — confirma que tunnel sobe)
5. **Testa em rede restritiva** (cafetaria, hotel — não comeces logo no hospital onde precisas urgentemente)

→ Quando precisares no contexto crítico (rede do hospital, sala de exame, sala de reunião), já vais ter feito o setup com calma.

## Stack alternativo

Se este setup é overkill para ti:

- **Tailscale** (€/mês ou free tier) — magic, just works, mas dados passam por relays Tailscale (legítimos mas not self-hosted)
- **ZeroTier** (similar a Tailscale)
- **Cloudflare Tunnel** (free) — bom para HTTP services específicos, não para WireGuard tunnel completo
- **VPN comercial** (Mullvad, ProtonVPN) — para browsing privacy, mas não te liga à tua casa

Este setup faz sentido se valorizas **self-hosting + auditável + zero terceiros**. Senão, Tailscale é provavelmente o caminho.

---

*Material derivado de setup pessoal validado em produção desde 2026-04. Sanitizado para partilha entre pares — IPs, hostnames, secrets, e contexto institucional foram abstraídos.*
