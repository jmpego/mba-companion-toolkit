# Quality Gate — Checklist Detalhado

12 verificações sobre um ficheiro antes de o entregar a um leitor externo.

- **Itens 1-9 (estruturais)** — bloqueantes. Falha = não entregar.
- **Itens 10-12 (conteúdo)** — *warnings*. Falha = corrige se possível, ou documenta excepção.

**Score mínimo recomendado:**
- Entrega geral: **10/12** com zero bloqueantes
- Deliverables sensíveis (visuais, *client-facing*, publicação): **11/12** com zero bloqueantes

---

## Itens estruturais (bloqueantes)

### 1. Naming correcto

**Critério:** o ficheiro segue uma convenção de naming consistente, definida no teu Style Book.

**Sugestão de convenção:** `AAAA-MM-DD_<projecto-slug>_<descritor>.<ext>`
Exemplo: `2026-05-10_grupo-mac_relatorio-final.docx`

**Verificação manual:**
- A data está no formato correcto (ISO 8601: AAAA-MM-DD)?
- O *slug* do projecto está em *kebab-case* (palavras separadas por hífen)?
- O descritor é informativo (não "v2" ou "final" sem contexto)?
- Não há colisões com outros ficheiros no mesmo projecto?

**Pass:** sim a tudo
**Fail:** qualquer não

---

### 2. Metadados presentes

**Critério:** o documento tem metadados que identificam autor, data e versão.

**Por formato:**

| Formato | Metadados esperados |
|---|---|
| Markdown | Frontmatter YAML com `title`, `author`, `date`, `last_updated` (opcional: `version`, `tags`) |
| Word / DOCX | Propriedades do documento preenchidas (Ficheiro → Propriedades): título, autor, palavras-chave |
| PDF | Mesma propriedades que DOCX, ou impressas no rodapé |
| Apresentações | Slide de título com autor, data, contexto |
| Imagens (em documento) | *Caption* / *alt text* definido |

**Verificação manual:** abrir o ficheiro e confirmar.

---

### 3. Sem texto adjacente duplicado

**Critério:** nenhum parágrafo (mais de 10 caracteres) é idêntico ao parágrafo seguinte.

**Causa comum:** conversões de Markdown para Word com Pandoc tendem a duplicar *captions* de imagens (`![caption](path)` é renderizado como imagem com legenda + parágrafo `*caption*` por baixo).

**Verificação automática (bash):**
```bash
awk 'NR>1 && length($0)>10 && $0==prev {print NR": "$0} {prev=$0}' <ficheiro.md>
```

Output esperado: vazio. Qualquer linha listada = duplicado adjacente.

---

### 4. Imagens com flow correcto

**Critério:** as imagens estão integradas no texto de forma legível.

**Regras:**
- Cada imagem tem *caption* único (não repete *caption* de imagem anterior)
- Imagens **não** estão imediatamente após um *heading* sem texto explicativo entre eles
- Ordem típica: `Heading → texto que introduz → imagem → texto que comenta → próximo heading`

**Excepção:** documentos visuais puros (infográficos, *posters*, *slides*) onde a imagem é o conteúdo principal.

---

### 5. Convenção visual consistente

**Critério:** se usas convenções visuais (callouts, caixas coloridas, emojis estruturais, símbolos), aplicaste-as de forma consistente em todo o documento.

**Exemplos de convenções (escolhe a tua, define no Style Book):**
- Caixas tipo "Nota" / "Aviso" / "Exemplo" com formatação distinta
- Símbolos consistentes (📌 para ponto importante, ⚠️ para risco, ✅ para confirmado, etc.)
- Cores consistentes (azul para definição, amarelo para *warning*, etc.)

**Pass:** convenção aplicada uniformemente, ou ausente uniformemente
**Fail:** convenção aplicada parcialmente (algumas caixas marcadas, outras não)

---

### 6. Linguagem e estilo consistentes

**Critério:** o documento usa um único idioma (e variante) consistentemente, com ortografia correcta.

**Sub-itens:**
- Idioma único — não misturar PT-PT com PT-BR sem motivo, ou PT com EN excepto em termos técnicos
- Acentuação correcta (se aplicável ao idioma)
- Tom adequado ao registo (ver Style Book — académico / profissional / pessoal)
- Tipo de letra consistente (definido no Style Book — ex.: Arial, Calibri, Times New Roman)

**Verificação automática para acentos PT (se aplicável):**
```bash
grep -c '[áàâãéêíóôõúçÁÀÂÃÉÊÍÓÔÕÚÇ]' <ficheiro>
```
Resultado esperado: número proporcional ao tamanho. Resultado **0 ou < 10** num documento PT longo = sinal de escrita sem acentos = **fail**.

**Verificação tipo de letra (DOCX):**
Abrir Word → seleccionar tudo → confirmar que o tipo de letra é o do Style Book. Se há mistura (ex.: Calibri por defeito + Arial em algumas tabelas), uniformizar.

---

### 7. Referências cruzadas válidas

**Critério:** todos os *links* internos, citações e referências apontam para alvos que existem e são acessíveis.

**Sub-itens:**
- *Links* internos para outros ficheiros do projecto apontam para ficheiros que existem
- Caminhos relativos funcionam a partir do local do ficheiro
- Citações bibliográficas têm correspondência na lista de referências
- *Links* externos (URLs) abrem (verificar pelo menos os críticos)
- Referências a tabelas / figuras / equações dentro do documento usam a numeração correcta

---

### 8. Tabelas e caixas bem formatadas

**Critério:** elementos estruturais (tabelas, listas, *code blocks*) estão sintacticamente correctos.

**Sub-itens:**
- Tabelas Markdown têm o mesmo número de colunas em todas as linhas
- Listas (numeradas ou *bullets*) têm indentação consistente; sem *items* órfãos
- *Code blocks* têm linguagem declarada quando relevante (` ```python `, ` ```bash `)
- Tabelas em Word têm largura razoável (não cortadas na margem)
- Caixas / *callouts* alinhadas verticalmente

---

### 9. Sem marcadores de uso de LLM

**Critério:** o documento não contém artefactos típicos de geração por *Large Language Models* (Claude, ChatGPT, Gemini, etc.).

**Lista exaustiva:** ver `llm-marks-to-avoid.md`.

**Marcadores mais frequentes:**
- `***` (separadores de três asteriscos)
- *Em-dashes* (`—`) usados como separadores entre secções
- *Bullets* com caracteres não-teclaveis (`•` em vez de `-` ou `*`)
- Frases formulaicas: "Certamente!", "Com certeza!", "Fico feliz em ajudar", "É importante notar que", "Em resumo,", "Em suma,"
- Hedging excessivo: "pode potencialmente", "é possível que talvez", "geralmente tende a"
- Estruturas paralelistas excessivas (3+ pontos consecutivos com construção sintáctica idêntica)

**Verificação automática (bash):**
```bash
grep -Pn '^\*{3,}$|^•|Certamente!|Com certeza!|Fico feliz|importante notar|Em resumo,|Em suma,' <ficheiro>
```
Output esperado: vazio.

**Aplica-se a:** deliverables finais, documentos públicos, propostas, *emails* formais, apresentações, PDFs.
**Excepção:** ficheiros internos de trabalho, *drafts*, notas privadas.

---

## Itens de conteúdo (warning)

### 10. Densidade adequada

**Critério:** o documento tem densidade informativa adequada — nem diluído (excesso de palavras vazias), nem denso a ponto de ser ilegível.

**Sinais de baixa densidade:**
- Frases longas com pouca informação (`"É importante mencionar que talvez seja útil considerar..."`)
- Repetição da mesma ideia em parágrafos consecutivos
- Excesso de adjectivos sem suporte factual

**Sinais de excesso de densidade:**
- Parágrafos com mais de 6 frases sem pausa visual
- Tabelas sem espaço de respiração entre elas
- Falta de *headings* / *subheadings* num documento longo

**Calibração:** documento com 5.000 palavras deve ter pelo menos 8-10 *headings* / *subheadings*.

---

### 11. Actionability

**Critério:** quem lê o documento sabe **o que fazer a seguir**.

**Por tipo de deliverable:**

| Tipo | Actionability esperada |
|---|---|
| Relatório de análise | Recomendações claras com responsável e prazo |
| Memo executivo | Decisão pedida + opções com prós/contras |
| Manual / runbook | *Steps* numerados, cada um executável sem ambiguidade |
| Notas de aula | Tópicos identificados + ligação a leituras / próximas aulas |
| Paper académico | Implicações claras + *next steps* de investigação |

**Pass:** documento responde implícita ou explicitamente a "e agora?"
**Fail:** documento descreve sem direcção.

---

### 12. Coerência com Style Book

**Critério:** o documento respeita as convenções do teu Style Book pessoal/de equipa.

**Sub-itens (depende do Style Book):**
- Tom adequado ao registo escolhido (académico vs profissional vs pessoal)
- Estrutura típica do registo (ex.: paper académico tem IMRaD; memo executivo tem TL;DR + recomendação + análise)
- Convenções visuais consistentes
- Vocabulário coerente com domínio

**Pass:** alinhado com Style Book
**Fail:** desvio sem justificação documentada

---

## Como aplicar — workflow rápido

```
1. Abrir o ficheiro
2. Percorrer items 1-9 (bloqueantes) — anotar fails
3. Se algum #1-#9 falha → corrigir antes de continuar
4. Percorrer items 10-12 (warnings) — anotar fails
5. Calcular score: passes / 12
6. Decisão:
   - 12/12 → entregar
   - 10-11/12 com zero bloqueantes → entregar (anotar warnings)
   - <10/12 ou qualquer bloqueante fail → corrigir e re-correr
```

**Tempo médio para um documento de 5-10 páginas:** 10-15 minutos a primeira vez, 5-7 minutos depois de habituado.

---

## Adaptação por domínio

Algumas profissões têm requisitos adicionais que se sobrepõem a este checklist:

- **Medicina / saúde:** revisão de conformidade com guidelines clínicas, citações verificáveis em PubMed
- **Direito:** referências legislativas com data de publicação, sem ambiguidade nas conclusões
- **Engenharia:** unidades SI, equações numeradas, dimensões verificáveis
- **Negócios / consultoria:** TL;DR no topo, *executive summary* obrigatório, dados com fonte

Adiciona estes requisitos ao teu Style Book na secção de domínio.
