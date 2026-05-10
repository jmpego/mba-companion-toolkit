---
title: "Workflow per Class — Antes, Durante, Depois"
version: 1.0
date: 2026-05-09
---

# Workflow per Class — Antes / Durante / Depois

## ANTES da aula (idealmente 24-48h antes)

### A.1 Pre-aula brief (10 min)

Cria — manualmente ou via agente IA — um **brief de 1 página** com:

1. **Tema da aula** (do syllabus / daily schedule)
2. **Conceitos esperados** (com base em mapa de leituras + sessões anteriores)
3. **Cases / readings obrigatórios**
4. **Questions / dilemas em aberto** (cross-ref aulas anteriores)
5. **Deadlines pré-aula** (group case post Canvas, write-up submission, etc.)

Ficheiro: `notas/AAAA-MM-DD_<UC>_pre-aula-brief.md`

### A.2 Mapa de leituras (uma vez por UC, no início)

```markdown
# UC <name> — Mapa de Leituras

| Sessão | Tema | Required | Optional support | Deadlines pré-aula |
|---|---|---|---|---|
| 1 | <tema> | <reading 1> | <reading 2,3> | <case post by date> |
| 2 | ... | ... | ... | ... |
```

Ficheiro: `notas/AAAA-MM-DD_<UC>_mapa-leituras.md`

### A.3 Setup files para a aula

```bash
cd <Project>/<UC>/

# 1. Cria new state JSON copiando template
cp <SHARING-MBA-PATH>/03_class-notes-pipeline/template_state-json.json \
   .diff_state_aula<NN>.json

# 2. Cria new progressive notes copiando template
cp <SHARING-MBA-PATH>/03_class-notes-pipeline/template_progressive-notes.md \
   notas/<DATE>_Aula_<NN>_<UC>_progressive.md

# 3. Adapta os placeholders em ambos os ficheiros (paths reais, data, sessão)

# 4. Pelo Word, cria ClassNotes_session<NN>.docx em notas/

# 5. Configura cron com prompt adaptado de template_cron-prompt.md
```

### A.4 Reading prep

Lê **leituras obrigatórias** e tira notas separadas (NÃO no `ClassNotes_sessionN.docx`). Notas de reading vão para:

```
materiais/notas-reading/<reading-slug>.md
```

→ Mantém limpo: `ClassNotes_sessionN.docx` é só para o que escreves **durante** a aula.

---

## DURANTE a aula

### B.1 Setup mental (5 min antes da aula começar)

1. Abre `ClassNotes_session<NN>.docx` no Word
2. Title: `<Date>` no topo
3. Confirma cron está activo (ou inicia o cron agora)
4. Tem o pre-aula brief aberto noutra janela (referência rápida)

### B.2 Tirar notas — convenções recomendadas

**Em texto liso:** o que o docente diz / faz, ipsis verbis quando vale.

```
The course will utilize AI and large language models...

This is a key concept in agency theory:
- Principal-agent problem
- Asymmetric information
```

**Para slides:** **paste image** (Insert → Picture or just `Ctrl+V` after screenshot). Word incorpora.

```
[Insert slide screenshot]

Key takeaway: contribution margin = revenue - variable costs
```

**Para reflexões / questions tuas:** prefixa com algo distinguível para o agente IA mais tarde:

```
> Q: how does this apply to start-ups with negative cash flow?

> Note to self: cross-ref Bhimani Cap. 5 here
```

### B.3 Save reflex

`Ctrl+S` a cada **~5 min**. Word autosave ajuda mas não substitui save manual em redes não-fiáveis (eduroam, hospital).

> Pattern conhecido: OneDrive sync atrasa upload mesmo após Save local. Em ambiente exigente, **valida ícone OneDrive (verde = synced)** antes de fechar laptop ou mudar de network.

### B.4 Não interromper para sintetizar

O cron está a fazer isso. **Foca-te na aula.** Quando o cron faz fire, vais receber um curtíssimo "T<N>: +X linhas, +Y imagens. Tópico: ..." que podes ignorar mid-class.

→ Discipline: **não abrir** o `progressive.md` durante a aula. Distraí-te. Olha-o no intervalo / no fim.

### B.5 Intervalos

No intervalo:

1. Abre `progressive.md` para ver o que o cron capturou
2. Spot-check: a estrutura está coerente? As citações estão correctas?
3. Se algo está errado, **não corrijas no momento** — anota para corrigir pós-aula
4. Volta para o Word para a 2ª metade

---

## DEPOIS da aula (idealmente nas 24h seguintes)

### C.1 Validation pass (10 min)

Abre `progressive.md` completo e:

1. **Verifica citações 🔷** — espelham o que o docente disse?
2. **Verifica sínteses 🔶** — concordas com a leitura do agente?
3. **Verifica imagens 📷** — refs estão correctas?
4. **Identifica gaps** — algo importante que o agente perdeu?

Se houver erros: edita directamente o markdown (manual). Se houver gaps: adiciona secções §X.5 ou §Y novos a mão.

### C.2 Resumo de 1 página (15 min)

Cria — ou pede ao agente IA — `notas/<Date>_<UC>_Aula<NN>_resumo-1pagina.md`:

- 5-7 conceitos-chave
- 1 frase-chave do docente
- Cross-refs a aulas anteriores
- Questions abertas para próxima aula

Útil para revisão pré-exame e para comunicar com colegas de grupo.

### C.3 Bibliografia enriquecida (opcional — para UCs académicas)

Se a UC envolve papers/textbook references, criar `notas/<Date>_<UC>_Aula<NN>_bibliografia-enriquecida.md` com:

- Autores citados
- Conceitos com anos de origem (e.g. "agency problem — Jensen & Meckling 1976")
- Cross-refs ao manual de referência (e.g. "Bhimani et al. 2015 Cap. 9")
- Links a papers (DOI ou arXiv)

→ Vira um **mini-glossary acumulativo da UC**.

### C.4 Trabalho pós-aula (deadlines)

Se houver group case write-up due em 24-48h, **agora** é o momento:

1. Discutir com grupo (Slack / WhatsApp / call)
2. Tu / grupo escreve a resposta
3. **Não usar AI para escrever** se a UC AI policy proíbe (verifica syllabus)
4. Submeter dentro do deadline

### C.5 Trigger para Aula <NN+1>

Mesmo dia ou no dia seguinte, executar A.1 + A.3 para a próxima aula. Acumulação semanal evita panic-mode.

---

## Padrão de tempo total

| Etapa | Tempo |
|---|---|
| ANTES (A.1-A.4) | ~30-60 min por aula |
| DURANTE (B.1-B.5) | tempo de aula (3-4h) |
| DEPOIS (C.1-C.4) | ~30-90 min por aula |
| **Total per class** | ~5h por aula (incluindo aula + prep + post) |

→ Compatível com expectativa MBA típica de "5h prep per session" indicada por muitos syllabi.

## Padrão acumulativo cross-aulas

Após 4-6 aulas, vais ter:

- **N progressive.md** files (1 por aula, ~30-50KB cada)
- **N resumos-1página** (referência rápida)
- **1-2 bibliografia files** acumulativos
- **1 mapa de leituras** (estável)
- **Cases / write-ups completos**

Tudo organizado, navegável, partilhável com colegas. **Pré-exame**: leituras dos resumos + bibliografia enriquecida = 1-2h de revisão estruturada vs panic re-reading.

## Anti-patterns

❌ **Tirar notas no progressive.md directamente.** O `progressive.md` é output do agente. Se editas, conflicts com cron next fire.

❌ **Esperar pelo fim da aula para criar state JSON.** Demoras 5 min a fazer setup pré-aula. Não tens 5 min mid-aula.

❌ **Ignorar o cron output mid-aula.** É distracção mas valioso ler nos intervalos para confirmar que tudo está OK.

❌ **Mudar a convenção 🔷/🔶 a meio do semestre.** Mantém estável. Output torna-se cumulativamente legível.

❌ **Aplicar o pipeline a toda UC sem discriminação.** UCs com poucas aulas (e.g. 3 sessões intensivas) beneficiam mais. UCs com 30 aulas dispersas e leituras pesadas talvez não justifique o overhead — adapta.

❌ **Usar IA para escrever cases / individual assignments.** A AI policy é tua responsabilidade. O pipeline é para sintetizar **TUAS** notas. Não substitui pensar.

---

## Adaptação por tipo de UC

### UC case-heavy (e.g. Strategy, Marketing, Governance)
- Mais ênfase em §B.2 (capturar discussion oral, não só slides)
- `materiais/cases/` separado por case
- Resumo pós-aula deve incluir 1 frase per case sobre lição-chave

### UC quantitative (Finance, Statistics, Operations)
- Word para texto + capturar quadro com fotos (cell phone → upload)
- Excel para cálculos paralelos
- LaTeX direct no Word para fórmulas (mais limpo que Equation Editor)

### UC discussion-only (e.g. Leadership, Ethics)
- Threshold cron mais alto (10 linhas — evita ruído)
- Foco em mapping conceptual + decision frameworks (não citações ipsis verbis)
- Pos-aula: 1-página de "key tensions / dilemmas discussed"

---

## Quando o pipeline não vale o esforço

- UC com 1-2 aulas curtas (não vale setup)
- UC online assíncrona (lectures pré-gravadas — usa transcripts directamente, sem cron)
- UC com NDA / confidencialidade extrema (data sensível não pode ir para API IA)

→ Para estes casos, mantém o método mas faz tudo manualmente / a posteriori.

---

*Fim do workflow. Combina com `folder-structure.md`, `template_progressive-notes.md`, `template_state-json.json`, `template_cron-prompt.md`. Bom semestre.*
