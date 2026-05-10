---
title: "Folder Structure — Class Notes Pipeline"
version: 1.0
date: 2026-05-09
---

# Folder Structure — Class Notes Pipeline

## Estrutura recomendada por UC

```
Projects/
├── <uc-slug>/                                  # ex: corporate-finance, managerial-economics
│   ├── .diff_state_aula01.json                # state cron (1 por aula)
│   ├── .diff_state_aula02.json
│   ├── ...
│   ├── materiais/                             # PDFs, slides docente, syllabus
│   │   ├── Syllabus_<UC>.pdf
│   │   ├── Daily_Schedule_<UC>.pdf
│   │   └── slides-aula-01/                   # screenshots ou exports
│   ├── notas/                                 # ★ core — notas em Word + sínteses
│   │   ├── ClassNotes_session1.docx          # tu escreves aqui
│   │   ├── 2026-XX-XX_Aula_01_<UC>_progressive.md   # cron actualiza
│   │   ├── 2026-XX-XX_Cortana_Aula01_resumo-1pagina.md  # opcional, gerado pós-aula
│   │   ├── 2026-XX-XX_Cortana_Aula01_mapa-leituras.md
│   │   └── _workflow_por_aula.md             # ciclo da UC
│   ├── casos/                                 # cases discutidos
│   ├── trabalhos/                             # group + individual assignments
│   │   ├── grupo/
│   │   └── individual/
│   ├── exercicios/                            # exercícios de aula
│   ├── infograficos/                          # diagramas, mapas conceptuais
│   ├── FlipCharts/                            # fotos de flipcharts (se aplicável)
│   │   └── 2026-XX-XX/
│   ├── referencias/                           # bibliografia, papers, links
│   └── deliverables/                          # outputs finalizados (sínteses, posters)
└── <next-uc-slug>/
    └── (mesma estrutura)
```

## Naming convention

### Para ficheiros de notas

```
AAAA-MM-DD_<descritor>_<UC-slug>.<ext>

Exemplos:
2026-05-09_Aula_02_MAC_progressive.md
2026-05-09_Cortana_Aula02_resumo-1pagina.md
2026-05-09_Aula_02_MAC_bibliografia-enriquecida.md
```

### Para state JSON

```
.diff_state_aula<N>.json
```

Sempre na **raiz da pasta UC** (não dentro de `notas/`). Hidden file (`.` prefix) para evitar clutter visual.

### Para Word source (tu escreves aqui)

```
ClassNotes_session<N>.docx          # mais simples
OU
<UC>_aula<N>_<AAAA-MM-DD>.docx       # mais explicito
```

→ Escolhe um, mantém consistência ao longo do semestre.

### Para slides screenshots

```
materiais/slides-aula-<N>/
├── 01_intro.png
├── 02_concepts.png
├── ...
```

## Por que esta estrutura

### Separação clara de input vs output

- **Input do operador (tu):** `notas/ClassNotes_*.docx`, `materiais/*`, `casos/*`, `referencias/*`
- **Output do agente IA (cron):** `notas/*progressive.md`, `notas/*resumo*.md`, `notas/*mapa*.md`, `infograficos/*`
- **Output finalizado (curated):** `deliverables/*`, `trabalhos/*`

Quando precisares de partilhar com colega de grupo o trabalho final, sabes que está em `deliverables/` ou `trabalhos/grupo/`. Sem confusão.

### State JSON na raiz da UC

- Permite múltiplos states (1 por aula) sem conflito
- Hidden file evita clutter na pasta principal
- Cron consegue ler/escrever sem permissions issues

### Materiais separados de notas

- Slides do docente (potencialmente copyrighted) ficam em `materiais/`
- Tuas notas + sínteses ficam em `notas/`
- Quando precisares de partilhar publicamente, nunca incluas `materiais/` (copyright issues)

### Casos / exercicios separados

- Cases têm questions próprias, podem ter discussion notes específicas
- Vale ter pasta dedicada se a UC for case-heavy (4+ cases)

## Múltiplas UCs em paralelo

Se estás a fazer 4 UCs simultaneamente (típico MBA semestre intensivo):

```
Projects/
├── corporate-finance/
├── managerial-economics/
├── management-accounting-control/
└── corporate-governance-ethics/
```

Cada uma com a sua estrutura completa. **Não misturar materiais entre UCs.** Cron por UC (cada um aponta a path próprio).

## Quando uma UC terminar

1. Validar que `notas/2026-XX-XX_Aula_<NN>_<UC>_progressive.md` está completo para todas aulas
2. Criar `notas/_synthesis_full_UC.md` (síntese cumulativa)
3. Mover deliverables finais para `deliverables/`
4. **Não apagar** a UC — fica como referência. Renomear para `_archived_<UC>/` se quiseres tirar do path activo.

## Estrutura mínima viável (se queres começar simples)

Se a estrutura completa parece overkill para a tua primeira UC, começa com **apenas isto**:

```
Projects/
└── <uc-slug>/
    ├── notas/
    │   ├── ClassNotes_session1.docx
    │   └── 2026-XX-XX_Aula_01_<UC>_progressive.md
    └── .diff_state_aula01.json
```

Vais sentir necessidade de adicionar `materiais/`, `casos/`, etc. à medida que avanças. Adiciona quando precisares.

---

*Próximo: `template_progressive-notes.md` — o ficheiro inicial de cada aula.*
