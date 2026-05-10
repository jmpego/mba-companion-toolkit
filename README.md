---
title: "MBA 2026 — 4 padrões de produtividade IA-aumentada"
version: 1.1
date: 2026-05-10
language: PT-PT (master) + EN (NYA only)
license: CC BY-NC-SA 4.0
---

# 4 padrões de produtividade IA-aumentada para colegas MBA

Este folder reúne **quatro métodos** que tenho usado para aumentar a minha produtividade no MBA. São independentes: podes adoptar um, dois ou todos. Não há código proprietário, dependências comerciais, nem ferramentas obrigatórias.

## Os 4 padrões

### 1. **Nurturing Your Agent** — afinar o teu agente IA

Como transformar um agente IA "de fábrica" (Claude, ChatGPT, Cursor, etc.) num colaborador que conhece as tuas convenções, preferências e padrões. Sem instrução-mãe perfeita; com feedback acumulado ao longo do tempo.

→ `01_nurturing-your-agent/`

**Custo de adopção:** zero. Funciona em qualquer agente IA com memória persistente.

### 2. **Remote Tunnel / VPN** — ligares-te à tua infraestrutura de qualquer rede

Como ter um pequeno servidor doméstico (Raspberry Pi ou similar) acessível de qualquer rede que bloqueie portas standard — eduroam, hospitais, hotéis, redes empresariais. Usa um tunnel HTTPS sobre WebSocket que parece tráfego web normal.

→ `02_remote-tunnel-vpn/`

**Custo de adopção:** ~€80 (Raspberry Pi 5) + 1 fim-de-semana de setup. Aplica-se a casos em que precisas de aceder a NAS, dashboard de casa, agentes IA self-hosted, ou ferramentas de produtividade que vivem em casa.

### 3. **Class Notes Pipeline** — notas de aula vivas, com auto-síntese

Template + workflow para tirares notas em Word durante a aula e teres uma síntese estruturada (com mapa de conceitos, glossário, e referências) automaticamente actualizada de 15 em 15 minutos. Usa um cron + agente IA + convenção visual simples.

→ `03_class-notes-pipeline/`

**Custo de adopção:** zero financeiro. Requer agente IA com capacidade de scheduled tasks (Claude Code tem; ChatGPT e Cursor têm soluções equivalentes via plugins).

### 4. **Quality Gate** — checklist de revisão antes de entregar

Checklist sistemático de 12 itens (9 estruturais bloqueantes + 3 de conteúdo) para passar a qualquer documento antes de o entregar a um leitor externo. Detecta marcas típicas de geração por LLM, naming inconsistente, duplicados, falhas de formatação. Personalizável via Style Book pessoal — o checklist é o esqueleto, tu defines o estilo.

→ `04_quality-gate/`

**Custo de adopção:** zero. ~30 min para responder às questões de personalização e criar o Style Book pessoal. Depois, 5-10 min por deliverable.

## Filosofia comum aos quatro

1. **Markdown como fonte de verdade.** Tudo é texto. Lê em qualquer editor, edita em qualquer máquina, versiona facilmente, sobrevive a obsolescência tecnológica.
2. **Workflow > tooling.** A ferramenta importa menos do que o método. Os três padrões funcionam em multiple stacks.
3. **Privacidade pela arquitectura.** Nenhum dos três pressupõe que dados pessoais saiam do teu controlo. Self-hosted ou local-first por defeito.
4. **Build incremental.** Não tentes adoptar tudo de uma vez. Escolhe o que te dói mais, implementa, valida, depois passa ao seguinte.

## Como ler este folder

Cada subpasta é **auto-suficiente**:

- `README.md` próprio (overview + adoption guide)
- Material principal (md, scripts, templates)
- Sem dependências cruzadas — adoptas um sem ter de adoptar os outros

Recomendação de ordem (por custo/benefício):

1. **Quality Gate primeiro** — zero custo, retorno imediato em qualquer deliverable que entregues
2. **NYA depois** — zero custo, melhora o uso diário do agente IA que já tens
3. **Class Notes Pipeline a seguir** — alto leverage para esta fase do MBA
4. **Remote Tunnel quando** precisares de aceder a recursos de casa em terreno

## Licença

**CC BY-NC-SA 4.0** — Atribuição obrigatória, uso não-comercial, partilha em condições idênticas.

- Usar, modificar, adaptar para uso pessoal/académico
- Partilhar com outros colegas (com atribuição)
- Não vender, integrar em produto comercial, ou apresentar como teu trabalho original

## Limitações honestas

- **Não há suporte formal.** Estes ficheiros são uma fotografia de um momento — a minha versão real está em evolução constante.
- **Não testei tudo em todos os stacks.** O que funcionou para mim pode precisar de adaptação para ti.
- **Não substitui pensar.** Estes padrões são andaimes. O valor vem de tu adicionares a tua perspectiva, dados e prioridades.

---

*Documento sanitizado para partilha entre pares MBA. Sem dados pessoais, institucionais, IPs ou hostnames identificáveis.*
