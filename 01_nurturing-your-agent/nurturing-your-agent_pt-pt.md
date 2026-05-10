---
titulo: Nurturing your agent — como treinar uma IA pela interacção
versao: v0
data: 2026-04-26
lingua: PT-PT
autor: "[Author]"
---

> **Nota:** Versão sanitizada para partilha pública. Nomes pessoais, nomes 
> de empresas e dados identificáveis foram substituídos por rótulos genéricos.
> A metodologia e os padrões estão integralmente preservados.

# Nurturing your agent
## Como treinar uma IA pela interacção quotidiana

---

## 1. Porquê este guião

Um agente de IA entregue "de fábrica" não conhece o Owner, não sabe as convenções da casa, não percebe o que é deliverable profissional no contexto específico. O valor aparece quando o agente é **afinado ao longo do tempo** por quem o usa. Este guião descreve o processo de afinação usado com o CEO (agente CEO da Enterprise virtual do Owner) e pode servir de modelo para outros operadores.

Três premissas:

1. **A afinação é um processo, não um evento.** Não se resolve numa instrução inicial longa. Constrói-se ao longo de semanas e meses.
2. **O feedback humano é o material bruto.** O que o Owner diz, corrige, aprova ou rejeita vira regra.
3. **A memória do agente tem de ser estruturada.** Caso contrário, o feedback dissipa-se e as mesmas correcções repetem-se.

---

## 2. Os três canais de afinação

### 2.1 Feedback directo (correcção)

O Owner sinaliza algo que está mal:

> "Tens de verificar a formatação visual antes de entregar o deliverable."

> "Os acentos em PT-PT não são opcionais."

> "Nunca re-gerar o DOCX quando eu já ajustei o layout."

**O que o CEO faz com isto:**
1. Captura a regra **com o "why"** (porque é que o Owner está a corrigir).
2. Escreve memória do tipo `feedback` em `~/.claude/projects/.../memory/feedback_<tema>.md`.
3. Adiciona linha em `MEMORY.md` (índice queryable em toda a sessão).
4. Aplica a regra a partir da próxima oportunidade — sem o Owner ter de repetir.

### 2.2 Feedback indirecto (confirmação)

Mais subtil. O Owner aceita sem objecções uma escolha não-óbvia, ou diz explicitamente "esta decisão foi acertada".

> "Sim, o PR único foi a opção certa aqui."

> "Fantástico." (depois de receber um output com convenção nova)

**Porque é que isto importa:** se o CEO só registasse correcções, ficaria conservador e cauteloso — mas perderia validações que são também instrução. Registar confirmações permite consolidar padrões que funcionam.

### 2.3 Meta-feedback (arquitectura)

O Owner redesenha o sistema:

> "Vou contratar um agente para X."
> "Cria uma pasta no Projects para este projeto."
> "Quero um registo paralelo de todas as interacções."
> "Muda o nome de [nome anterior] para o CEO."

**O que isto faz:**
- Adiciona/remove agentes do roster
- Cria novas pastas-projecto com convenções próprias
- Instala novos hooks ou skills
- Refactoriza a memória (ex: rebranding de 2026-04-10)

Meta-feedback exige execução imediata e actualização do `CLAUDE.md` ou das `rules/`.

---

## 3. Anatomia de uma boa correcção

As correcções mais úteis do Owner seguem três padrões:

### Padrão A — "Regra + porquê + consequência"

> "Antes de entregar PPTX, abre visualmente. Na [Empresa], eu abri e tinha texto sobreposto. Se passar outra vez, perdemos o cliente."

Regra clara, histórico do incidente, stake concreto. A memória fica rica e o CEO sabe quando flexibilizar a regra em edge cases.

### Padrão B — "Exemplo como regra"

> "Fonte é Arial. Sempre." (depois de ver Calibri num DOCX)

Correcção curta, referente a um artefacto específico. O CEO generaliza a **todos** os DOCX/PPTX futuros.

### Padrão C — "Sinalização de preferência vs requisito"

> "Prefiro o estilo Micah nos avatares." (preferência)
> "Os acentos em PT-PT são obrigatórios em deliverables." (requisito)

Distinguir os dois evita que o CEO trate preferências como leis rígidas ou requisitos como sugestões.

---

## 4. Casos canónicos (já instituídos)

Cada caso abaixo representa uma correcção real → regra persistente → aplicação automática nas sessões seguintes.

### Caso 1 — Validação visual antes de entregar

**Gatilho:** 2026-04-15, PPTX [Empresa] com textos sobrepostos passou no Quality Gate porque o CEO só verificou se as shapes existiam, não o render final.

**Regra:** PPTX/DOCX/PDF têm de ser abertos visualmente (ou rendered para PNG/PDF) antes de serem entregues. Adicionado como check #12 bloqueante do `quality-gate`.

**Memória:** `feedback_visual_validation_pptx_docx.md`

### Caso 2 — Acentuação PT-PT

**Gatilho:** 2026-04-13, deliverable sem acentos enviado a terceiros. Sinalizado como "imediatamente reconhecível como gerado por máquina".

**Regra:** deliverables finais com acentuação completa. Ficheiros internos do sistema (memórias, scripts) podem continuar sem.

**Memória:** `feedback_pt_pt_accents_in_deliverables.md`

### Caso 3 — Fonte Arial por defeito

**Gatilho:** templates corporate a usar Calibri por defeito.

**Regra:** Arial (ou Arial Narrow para texto denso) é o default em todos os DOCX/PPTX/PDF, mesmo que o template base sugira outra fonte.

**Memória:** `feedback_font_preference.md`

### Caso 4 — Naming com data de última actualização

**Gatilho:** 2026-04-10, confusão entre versões stale e actuais do enterprise-technical-blueprint.

**Regra:** ao actualizar deliverable, renomear para `AAAA-MM-DD_<slug>.<ext>` (data da última actualização) + linha `**Last updated:**` nos metadados internos + incremento de versão.

**Memória:** `feedback_deliverable_naming.md`

### Caso 5 — Edição cirúrgica de DOCX do Owner

**Gatilho:** Owner disse "percebeste mal" depois de o CEO ter re-gerado um DOCX do zero quando ele pediu "corrige isto" sobre a versão dele (que já tinha layout ajustado).

**Regra:** quando o Owner diz "já criei a v3" ou "já ajustei o layout", o CEO aplica correcções run-level via `python-docx` no ficheiro dele, preservando bold/italic e layout. Nunca re-gera do zero.

**Memória:** `feedback_owner_edits_docx_directly.md`

### Caso 6 — Callouts do CEO 🔶/🔷

**Gatilho:** 2026-04-11, Owner preocupado que ao reler notas de aula meses depois pudesse atribuir ao docente interpretações que foram do CEO.

**Regra:** em notas de aula, toda a interpretação do CEO fica em caixa laranja (pesada) ou azul (leve), com bibliografia específica. Texto normal = o que o docente disse.

**Memória:** `feedback_ceo_callout_convention.md`

### Caso 7 — Gamma para slides

**Gatilho:** 2026-04-13, Owner confirmou preferência por Gamma em vez de Reveal.js/python-pptx para qualquer apresentação.

**Regra:** default para slides é produzir **prompts Gamma estruturados** (limite 10 slides), não HTML/PPTX nativo. Excepções: dashboard interactivo (agente de front-end) ou infográfico standalone.

**Memória:** `feedback_gamma_preferred_slides.md`

### Caso 8 — Sanitização para partilha pública

**Gatilho:** 2026-04-16, Owner decidiu partilhar guiões (Enterprise Blueprint, NYA, StylesBook, WhoAmI) com terceiros. Auditoria revelou 100+ ocorrências de dados pessoais, nomes com problemas de copyright, nomes de agentes, instituições e até CC/IBAN em ficheiros antigos.

**Regra:** antes de partilhar qualquer documento, criar versão `_PUBLIC` separada (original intacto). Aplicar tabela de substituições: CEO (orquestrador), Owner (proprietário), nomes de agentes→apenas o papel, empresas→[Company], instituições→[University]/[Hospital], dados pessoais→removidos. Spot-check automático obrigatório (grep/script) antes de entregar. Para DOCX/XLSX, usar python-docx/openpyxl.

**Memória:** `feedback_public_sanitization_process.md`

### Caso 9 — Quality Gate como processo obrigatório

**Gatilho:** 2026-04-11, Owner avaliou contratar editor profissional externo. Análise custo-benefício mostrou que uma skill interna (`/quality-gate`) com checklist estruturado era mais eficaz. Decisão: Opção C — skill interna com revisão mensal.

**Regra:** antes de entregar qualquer deliverable ao Owner ou a terceiros, correr `/quality-gate` no ficheiro. Checklist: 8 verificações estruturais bloqueantes (naming, metadados, duplicados, flow imagem, convenção callout, linguagem, refs, formatação) + 3 de conteúdo (densidade, actionability, coerência com style book). Score mínimo: 9/11 + zero bloqueantes. Para deliverables quality-sensitive (visuais, client-facing, publicação): score >= 10/11, sugerir revisão ao Owner, max 2 rounds de correcção. Resultado registado na DB (`deliverables` com `quality_score`, `iterations_to_satisfaction`, tags) para KPI tracking mensal.

**Memória:** `project_editor_hire_evaluation.md`

### Caso 10 — Pipeline de contratação de agentes

**Gatilho:** 2026-04-16, Owner pediu contratação de Research Grants & EU Funding Specialist. Processo seguiu pipeline de 3 fases com dois agentes especializados (Skills Researcher + HR Director) e registo completo na DB.

**Regra:** a contratação de um novo agente segue sempre 3 fases sequenciais: (1) o Skills Researcher pesquisa competências, frameworks e perfil ideal, entrega Skills Profile Report em `.claude/db/research/`; (2) o HR Director desenha persona completa (nome, idade, background, DISC, atributos) e cria ficheiro de agente em `.claude/agents/`; (3) o CEO regista agente na DB (`insert_agent`), actualiza Team Roster no `CLAUDE.md`, actualiza Smart Routing em `.claude/rules/smart-routing.md`, e confirma ao Owner. O briefing ao Skills Researcher deve incluir scope funcional completo, programas/domínios a cobrir, KPIs esperados, e interacções com outros agentes. O briefing ao HR Director deve incluir o Skills Profile + scope + KPIs. Cada fase é registada como tarefa na DB.

**Memória:** procedural — documentado em `CLAUDE.md` (secção "Procedimento de Contratação de Agentes")

---

## 5. Arquitectura da memória

O sistema usa três tiers (documentados em `CLAUDE.md`):

| Tier | Propósito | Acesso |
|------|-----------|--------|
| **Hot** | Perfil do Owner, preferências, regras. Carregado em CADA sessão. | `memory-hot.md` |
| **Warm** | Decisões de projecto, padrões aprendidos. Queryable. | `memory_manager.py get_warm` |
| **Cold** | Arquivo histórico. Busca via FTS5. | `memory_manager.py query` |

**Promote/demote automático:**
- Warm → Hot quando acedido ≥5 vezes
- Warm → Cold quando stale >30 dias e confidence <0.5
- Hot → Warm quando confidence <0.3

Isto evita que memórias ultrapassadas continuem a condicionar o comportamento do agente.

---

## 6. O que NÃO entra em memória

Igualmente importante. A auto-memory do Claude Code explicita exclusões:

- **Padrões de código, arquitectura, paths** — derivam-se do estado actual do repo
- **Histórico Git** — `git log` / `git blame` são autoritativos
- **Receitas de debug** — o fix está no código, o contexto está no commit
- **Estado efémero** — trabalho em curso, contexto da conversa actual

Quando o Owner pede para "guardar" algo destas categorias, o CEO pergunta **o que foi surpreendente ou não-óbvio** e guarda só essa parte.

---

## 7. Fluxo operacional — o ciclo completo

```
1. Owner interage com o CEO
2. Dá feedback (directo, indirecto ou meta)
3. O CEO capta o "why"
4. Escreve memória feedback_<tema>.md com Regra + Why + How to apply
5. Actualiza MEMORY.md (índice)
6. Aplica na próxima oportunidade
7. Se aplicar errado, volta ao passo 2 (correcção iterativa)
8. Memórias antigas decaem automaticamente (confidence decay -5%/30 dias)
```

Numa sessão típica, este ciclo completa-se 3-10 vezes.

---

## 8. Sinais de que o agente está a ser bem afinado

- **O Owner repete-se cada vez menos.** Correcções que apareciam 3 vezes por semana aparecem 1 vez por mês.
- **As memórias são específicas**, não genéricas ("fonte Arial" em vez de "cuida da formatação").
- **Há confirmações, não só correcções.** O Owner valida padrões explicitamente ("isto foi o movimento certo").
- **O agente anticipa.** Aplica regras em domínios onde ainda não foram explicitadas, por analogia.
- **A arquitectura evolui.** Novos agentes, novas skills, novas rules aparecem à medida que as limitações emergem.

---

## 9. Sinais de que falta afinação

- O Owner repete a mesma correcção mais de 3 vezes — sinal que a memória não está a ser consultada ou não foi bem escrita.
- O agente aplica uma regra em contexto errado — a memória não tem edge cases.
- O Owner tem de validar linha a linha antes de entregar — o Quality Gate não está a filtrar.
- Deliverables chegam ao Owner sem o "why" — o agente está a executar sem compreender.

---

## 10. Recomendações ao operador (Owner)

1. **Dá sempre o why com a correcção.** A regra sem o motivo torna-se rigidez cega.
2. **Distingue preferência de requisito** quando é óbvio para ti mas pode não ser para o agente.
3. **Confirma padrões que funcionaram** — não só os que falharam.
4. **Pede registo visível** (pasta projecto, ficheiro de log) quando o tema é complexo e multi-sessão. A conversa dissolve-se; o ficheiro persiste.
5. **Revê o MEMORY.md periodicamente** — se há entradas que já não fazem sentido, remove.
6. **Aceita o decay.** Uma memória que o agente parou de usar pode estar obsoleta — deixa-a descer a cold.

---

## 11. Medir a maturidade — quando é que o agente está "pronto"?

A afinação não é infinita. À medida que o sistema acumula padrões, as sessões geram cada vez menos regras novas — as interacções passam de correcções a confirmações. Este plateau é mensurável.

### 11.1 Métrica central: New Insight Rate (NIR)

```
NIR = padrões novos da sessão / total de entradas no log da sessão × 100
```

Cada entrada do log paralelo (`registo-interacoes/AAAA-MM-DD_log.md`) é classificada como:
- **Padrão novo** — gera regra que não existia no guião
- **Confirmação** — valida regra já documentada (reforça confiança, não altera o guião)

### 11.2 Fases esperadas

| Fase | NIR típico | O que acontece |
|------|-----------|----------------|
| **1 — Bootstrap** | 70–100% | Quase todas as interacções geram regras novas. O agente está a aprender tudo pela primeira vez. |
| **2 — Consolidação** | 30–50% | Muitas confirmações, algumas descobertas. O núcleo de regras está a formar-se. |
| **3 — Plateau** | <20% | A maioria das entradas confirma padrões existentes. Novas interacções refinam, não transformam. |

### 11.3 Threshold de maturidade para publicação

O guião está pronto para partilha quando se cumprem **dois critérios em simultâneo:**

1. **3 sessões consecutivas com NIR < 20%** — o agente estabilizou, novas sessões não produzem regras fundamentalmente novas
2. **Mínimo 25 padrões generalizáveis** no guião — cobertura suficiente para que outro operador consiga replicar o processo

Quando ambos forem atingidos, o CEO sinaliza: *"O guião NYA atingiu plateau. Pronto para revisão final e partilha."*

### 11.4 Bloco de métricas por sessão

No final de cada log diário, registar:

```
## Métricas NYA
- Entradas: N
- Padrões novos: N
- NIR sessão: XX%
- Padrões acumulados no guião: N
- Sessões consecutivas com NIR < 20%: N/3
```

### 11.5 Porquê esta métrica

A taxa de novidade mede directamente o que importa: **a velocidade de aprendizagem do sistema**. Quando o NIR desce, o agente já capturou o essencial da forma de trabalhar do Owner. Continuar a iterar indefinidamente não acrescenta valor — e a curva de rendimento decrescente é visível nos dados.

Isto é análogo à curva de aprendizagem em psicometria: os primeiros itens de um teste revelam mais informação sobre o examinando do que os últimos. Aqui, as primeiras sessões revelam mais sobre o Owner do que as tardias.

---

## Anexo A — Referências internas

- `CLAUDE.md` — persona e regras do CEO
- `.claude/rules/smart-routing.md` — 5 rotas de encaminhamento
- `.claude/rules/writing-styles.md` — Style Book do Owner
- `.claude/skills/quality-gate/` — checklist pré-entrega
- `~/.claude/projects/.../memory/MEMORY.md` — índice de memórias
- `Projects/nurturing-your-agent/registo-interacoes/` — log paralelo de feedback

## Anexo B — Modelo de memória feedback

```markdown
---
name: <título curto>
description: <uma linha, específica>
type: feedback
originSessionId: <uuid>
---
<regra>.

**Why:** <motivo concreto, incidente se houver>

**How to apply:** <quando, onde, como, excepções>
```
