---
title: "Class Notes Pipeline — Adoption Guide"
version: 1.0
date: 2026-05-09
license: CC BY-NC-SA 4.0
---

# Class Notes Pipeline — como adoptar

## O problema que resolve

Tiras notas de aula em Word. Querias ter:

1. **Síntese estruturada** das tuas notas (com mapa de conceitos, glossário, refs)
2. **Auto-actualização durante a aula** (não esperar pelo fim para começar a sintetizar)
3. **Distinção clara entre voz docente e tuas próprias reflexões**
4. **Sem perda de conteúdo** se o portátil reiniciar / IDE crashar / sync falhar
5. **Acumulação ao longo do semestre** — síntese cumulativa cross-aulas

Em vez de:
- Notas dispersas em vários ficheiros sem relação
- Pós-aula gastar 1-2h a re-organizar (que nunca fazes)
- Esquecer 60% do que ouviste no momento

Este pipeline foi validado em 4 UCs MBA simultâneas durante 2 meses, com taxas de captura ~80-90% do conteúdo de aula em síntese estruturada **disponível para revisão durante a própria aula**.

## Como funciona em 1 minuto

```
┌─────────────────────────┐         ┌─────────────────────────┐
│ Tu, durante a aula      │  Save   │ Word doc local          │
│ - escreves notas em Word│ ──────▶ │ (sync OneDrive/iCloud)  │
│ - cola screenshots de   │         │                         │
│   slides do docente     │         │                         │
└─────────────────────────┘         └────────────┬────────────┘
                                                 │
                                                 ▼  cron 15-min
                                    ┌──────────────────────────┐
                                    │ Agente IA                │
                                    │ - converte docx → md     │
                                    │ - compara com snapshot   │
                                    │ - identifica novo content│
                                    │ - estrutura com:         │
                                    │   🔷 voz docente (EN/PT) │
                                    │   🔶 síntese tua (PT)    │
                                    │   📷 slides              │
                                    │ - anexa a progressive .md│
                                    └────────────┬─────────────┘
                                                 │
                                                 ▼
                                    ┌──────────────────────────┐
                                    │ progressive_notes.md     │
                                    │ Actualizado de 15-15 min │
                                    │ Estruturado, navegável,  │
                                    │ com índice + glossário   │
                                    └──────────────────────────┘
```

## O que vais conseguir

Ao fim de cada aula, tens um ficheiro markdown estruturado com:

- **Conceitos numerados §1, §2, ...** (cada conceito uma secção)
- **Citações ipsis verbis do docente** (entre 🔷, em EN ou PT-PT preservando original)
- **Tua síntese e contexto** (entre 🔶, em PT-PT — ou EN, se preferires)
- **Imagens dos slides** referenciadas (📷 image1.png, image2.png, ...)
- **Cross-refs internos** (§3 → §15 quando há conexão)
- **Mapa conceptual final** + **glossário** + **pendentes**

Pronto para revisão pré-exame, partilha com colegas de grupo, ou base para o trabalho final.

## Conteúdo desta pasta

| Ficheiro | Para que serve |
|---|---|
| `README.md` (este) | Overview + adopção |
| `folder-structure.md` | Estrutura recomendada por UC + naming convention |
| `template_progressive-notes.md` | Template inicial para uma aula nova |
| `template_state-json.json` | State JSON que o cron usa para rastrear delta |
| `template_cron-prompt.md` | Prompt template para o agente IA correr de 15 em 15 min |
| `workflow_per-class.md` | Ciclo antes/durante/depois de cada aula |

## Pré-requisitos

| Item | Necessário | Custo |
|---|---|---|
| Editor de markdown | VS Code (free) ou similar | 0 |
| Word (ou Pages, ou Google Docs com export para .docx) | Sim, para tirar notas | Provavelmente já tens |
| Cloud sync | OneDrive, iCloud, Dropbox, Google Drive — qualquer um | Provavelmente já tens |
| **Pandoc** (CLI) | Sim — converte docx para markdown | 0, free, brew/apt install |
| **Agente IA com scheduled tasks** | Crítico | depends — ver abaixo |

## Stack de agente IA — escolhe **um**

Este pipeline assume que tens um agente IA capaz de:
- Ler ficheiros locais
- Correr comandos shell (`pandoc`, `wc`, `grep`)
- Editar markdown files
- Ser invocado por cron / scheduled task

**Opções:**

### A) Claude Code (Anthropic)
- ✅ Tem `CronCreate` built-in
- ✅ Acesso a filesystem local
- ✅ Tem skills/hooks
- 💰 Subscription mensal (~$20-100 dependendo plano)
- **Recomendação:** se já tens, este é o caminho mais directo. Os templates desta pasta foram feitos para Claude Code.

### B) Claude Desktop App + MCP
- ✅ Free com plano gratuito Claude (limited)
- ✅ MCP filesystem connector
- ⚠️ Sem cron nativo — precisas externalizar (cron OS + invoke via CLI)

### C) ChatGPT Custom GPT + Code Interpreter
- ✅ Plus subscription €20/mês
- ⚠️ Sem cron — workaround via launchd/cron OS + Playwright para POST manual
- ⚠️ Código gerado vive no sandbox, mais friction para escrita persistente

### D) Cursor / Continue / outras IDEs com IA
- Funciona mas requer adaptação manual dos templates

### E) Local LLM (Ollama, llama.cpp)
- Maior privacy, zero $ ongoing
- Requer hardware decente (Mac M-series ou GPU)
- Templates precisam adaptação para tooling (não há `CronCreate` standard)

→ **Templates nesta pasta** assumem **Claude Code (A)**. Para outras stacks (B-E), os ficheiros base (markdown templates, state JSON, workflow) são reutilizáveis; o **scheduling** e o **invocation prompt** precisam adaptação.

## Adopção em 5 passos

1. **Confirma o teu stack de IA** (A-E acima). Se ainda não tens, escolhe e instala.
2. **Cria a estrutura de pastas** para a tua primeira UC seguindo `folder-structure.md`
3. **Copia os templates** (`template_*`) e renomeia para a aula (`2026-05-09_Aula_01_<UC>_progressive.md`, etc.)
4. **Configura o cron** com o prompt de `template_cron-prompt.md` adaptado a paths teus
5. **Tira notas em Word numa aula** e observa o cron a fazer seu trabalho

→ Primeira aula: ~30 min de setup. Aulas seguintes: 30 segundos (criar novo state JSON + novo progressive notes a partir do template).

## Custo de adopção

| Item | Custo |
|---|---|
| Setup primeira UC | 1-2h |
| Adicionar nova UC | 5-10 min (copy + adaptar templates) |
| Criar nova aula dentro UC | 30 segundos |
| Manutenção cron | 0 (auto) |
| Storage (40 aulas / semestre × ~50KB md) | ~2MB |

## Limitações honestas

- ⚠️ **Tens de continuar a tirar notas tu próprio.** O agente sintetiza o que tu escreves; não faz a aula por ti
- ⚠️ **Qualidade da síntese depende da qualidade das notas.** Garbage in, garbage out
- ⚠️ **AI policies de UCs.** Algumas UCs proíbem AI assistance em case answers / individual assignments. Verifica syllabus. Este pipeline é para **organizar e sintetizar TUAS notas** (que tu escreveste), não para gerar respostas. Mas se a UC for muito restrictiva, valida com o docente
- ⚠️ **Custo de API IA.** Cron 4×/h × 4h aula × N aulas/semana pode acumular. Optimização: threshold "no delta = sem trabalho" minimiza tokens
- ⚠️ **Privacidade.** Notas vão para a API do agente IA (Anthropic, OpenAI, etc.). Se a UC tiver conteúdo sensível, valida policy do agente. Local LLMs (E) eliminam este risco mas têm trade-off em qualidade

## Filosofia

1. **Tu és o autor.** Agente é editor + organizador
2. **Notas como produto da aula.** Treinas a tua atenção a estruturar; agente formaliza
3. **Resilience by markdown SoT.** Se a IA falha, tens sempre as notas raw em Word + os progressive notes
4. **Iteração curta.** 15 min é o ciclo certo. Mais curto sobreposições; mais longo perdes momentum
5. **Convenção visual fixa.** 🔷/🔶/📷 são os símbolos. Não os mudes mid-semestre

---

*Próximo: lê `folder-structure.md` para perceberes a organização recomendada.*
