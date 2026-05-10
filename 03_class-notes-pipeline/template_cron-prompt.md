---
title: "Template Cron Prompt — Class Notes Auto-Diff"
version: 1.0
date: 2026-05-09
---

# Template Cron Prompt — Class Notes Auto-Diff

## Como usar

1. Copia o texto abaixo
2. Substitui os `<placeholders>` por paths reais
3. Cola no teu agente IA via mecanismo de scheduled task (Claude Code `CronCreate`, ChatGPT cron via OS, etc.)
4. Configura schedule: `7,22,37,52 * * * *` (4 fires por hora, off-minute para evitar burst)

## Schedule recomendado

```
7,22,37,52 * * * *
```

Por que 4 fires/hora a `:07/:22/:37/:52`:
- 15 min é granularidade certa (curto = sobrepõe; longo = perdes momentum)
- Off-minute (não :00 / :15 / :30 / :45) evita burst patterns que congestionam APIs / cron daemons partilhados
- Cobre uma aula típica de 2-4h com ~16 fires

## Prompt template

> **Substitui** todos os `<placeholders>` antes de usar.

```
Auto-diff <UC> Aula <NN> — cron 15-min.

1. Lê o source DOCX:
   <absolute_path_to_ClassNotes.docx>

2. Carrega state JSON:
   <absolute_path_to_.diff_state_aulaNN.json>

3. Converte docx → md via pandoc para /tmp/<UC>-aula<NN>-current.md
   (cd para o directório do source antes de pandoc para preservar media refs)

4. Se size=0 ou pandoc falhar com "couldn't unpack docx container":
   reporta inline "T<N>: source ainda vazio (size=0). Cron aguarda primeira escrita."
   e sai (não escreve nada).

5. Compara contagem de linhas (wc -l) e contagem de imagens com last_paragraph_index e last_image_count do state JSON.

6. Se delta < 5 linhas E delta < 1 imagem nova:
   reporta inline "Sem delta — aula em pausa ou intervalo. T<N>: lines=<X> imagens=<Y>"
   e sai.

7. Se delta >= 5 linhas OU >= 1 imagem nova:
   a) Identifica conteúdo NOVO (linhas após last_paragraph_index)
   b) Estrutura em secções §<N+1>+ (continuando numeração da última secção):
      - 🔷 voz docente — citação <EN ou PT> ipsis verbis
      - 🔶 Síntese — em <EN ou PT> com contexto, expansão, cross-refs
      - 📷 image<N>.png — slides referenciados
   c) Anexa ANTES da secção "## Histórico de snapshots" no ficheiro:
      <absolute_path_to_progressive_notes.md>
   d) Update state JSON:
      - last_paragraph_index = novo wc -l
      - last_image_count = novo grep count de '^!\[\]'
      - snapshot_history append: {snapshot:"T<N>", time:"HH:MM", lines, images_total, topic_summary}
   e) Reporta inline curto: "T<N>: +<X> linhas, +<Y> imgs. Tópico: <síntese 1 frase>"

Regras hard:
- <Língua de síntese> com acentuação correcta (se aplicável)
- Sem marcadores LLM (sem ***, sem hedging formulaico tipo "It is important to note that...")
- Preservar citações docente ipsis verbis (EN ou PT — qualquer língua que ele falou)
- Não tocar em parágrafos já processados
- Cross-ref a aulas anteriores onde aplicável (e.g. "ver §4 da Aula 01")
- Se source DOCX não acessível ou pandoc falhar: reporta erro e sai sem corromper state

[Opcional — se a UC tem AI policy restritiva]:
- AI policy reminder: NUNCA escrever a resposta de cases (<lista de cases>) ou Individual Assignment.
  Apenas síntese conceptual e mapeamento.

Cron auto-expira em 7 dias. Owner pode pedir paragem com "para o cron <UC>".
```

## Variantes do prompt

### Variante minimalista (UC light, sem AI policy)

```
Auto-diff <UC>. Lê <docx_path>. Compare com <state_json_path>.
Se delta >= 5 linhas: estrutura novo content em §<N>+ com 🔷 (docente) / 🔶 (síntese) / 📷 (slides),
anexa a <progressive_md_path> antes de "## Histórico de snapshots", update state.
Senão: reporta "sem delta".
Convenção: PT-PT (ou EN), citações ipsis verbis, sem markers LLM.
```

### Variante com AI policy crítica

Adiciona à parte "Regras hard":

```
- AI policy reminder: NUNCA escrever a resposta de:
  - Case write-ups (Q1 sobre <empresa1>, Q2 sobre <empresa2>, ...)
  - Individual Assignment (exam in-class)
  - Group case discussion comments
  Apenas síntese conceptual e mapeamento dos conceitos da aula.
- Se na próxima síntese aproximas-te demais de um conteúdo de case answer, recua e reformula como questão / framework / mapping.
```

### Variante com flipcharts (UCs com bastante whiteboard work)

Adiciona à parte "estrutura":

```
- 📋 = flipchart / whiteboard (foto tirada pelo Owner durante aula)
- Se imagem em /flipcharts/AAAA-MM-DD/<filename>: trata como flipchart (não slide), descreve conteúdo se possível
```

## Calibração de threshold

O threshold default é "≥ 5 linhas OU ≥ 1 imagem nova". Adapta conforme a UC:

| UC type | Threshold sugerido |
|---|---|
| Lecture-heavy (muito texto, poucos slides) | 5 linhas, 1 imagem |
| Case-heavy (discussion, slides menos densos) | 3 linhas, 1 imagem |
| Exercise-heavy (ex: matemática, cálculos) | 2 linhas, 1 imagem |
| Discussion-heavy (oral, pouca escrita) | 10 linhas, 1 imagem (evita ruido de typos) |

## Optimização de tokens

Cron 4×/h × 4h aula × 4 aulas/semana × 16 semanas = **1024 fires por semestre**. Maioria vai retornar "sem delta". Optimização:

1. **Threshold sensível** — não processar mudanças < 5 linhas
2. **No-op silencioso** — quando sem delta, não logging verboso
3. **Pandoc local** (não API call) — conversão docx→md é local, zero tokens
4. **Comparação numérica antes de leitura** — só lê md se size mudou

Custo típico (Claude Code, modelo Haiku):
- Fire sem delta: ~1.5K tokens output
- Fire com delta médio (10 linhas, 2 imagens): ~6-10K tokens output
- Fire com delta grande (50+ linhas, 5+ imagens): ~20-30K tokens output

Total estimado por semestre (200 horas de aula): **~5-15M tokens**, dependendo do modelo. Budget de €10-50 dependendo plano.

## Fallback se cron não disponível na tua stack IA

Workaround manual:
1. Cria um shortcut que invoca o agente IA com o prompt acima
2. Atalha-o para tecla rápida (e.g. `Ctrl+Shift+A`)
3. Pressionas a cada 15 min durante aula

Menos elegante mas funciona. Tens controlo total sobre quando o agente lê.

## Limites a saber

- **Pandoc falha em ficheiros corruptos / em uso simultâneo.** Se Word está a guardar exactamente quando cron corre, pode dar erro temporário. Próximo fire vai recuperar.
- **Multi-language switching** (docente alterna EN ↔ PT) pode confundir o agente. Ser explícito no prompt: "preserve original language of docente".
- **Tabelas complexas em Word** (merged cells, nested) podem perder estrutura na conversão para markdown. Workaround: usar tabelas simples no Word, ou desenhar tabela à mão depois no markdown.
- **Equações matemáticas** — Word equation editor exporta às vezes com glitches. Para aulas com matemática (Finance, Stats), considerar começar a escrever LaTeX direto no Word como código, não como equation.

---

*Próximo: `workflow_per-class.md` — o ciclo completo de cada aula (antes / durante / depois).*
