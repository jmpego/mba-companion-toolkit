# Quality Gate — Guião de Revisão de Deliverables

**O que é isto?** Um *checklist* sistemático para revisão de qualidade antes de entregar qualquer documento (académico, profissional, pessoal) a um leitor externo. Foi desenhado para deliverables produzidos com apoio de assistentes de IA (Claude, ChatGPT, Gemini, etc.) mas aplica-se a qualquer documento.

**Para quem?** Estudantes, profissionais, equipas que produzem documentação escrita e querem um padrão consistente de qualidade antes de partilhar.

**O que faz:** corre 12 verificações sobre um ficheiro (9 estruturais bloqueantes + 3 de conteúdo recomendadas) e devolve um *score* de 0-12. Mínimo recomendado para entrega: **10/12 com zero bloqueantes** (11/12 para deliverables sensíveis: visuais, *client-facing*, publicação).

---

## Estrutura desta pasta

| Ficheiro | Para que serve |
|---|---|
| `README.md` | Este ficheiro — overview e como adoptar |
| `checklist.md` | Os 12 itens detalhados, com critério de pass/fail |
| `llm-marks-to-avoid.md` | Lista exaustiva de marcadores típicos de uso de LLM a remover antes de entregar |
| `style-book-template.md` | Template para criares o teu próprio Style Book (registos académico / profissional / pessoal) |
| `personalization-questions.md` | Questões para responderes **antes** de adoptar — adapta o checklist ao teu contexto |

---

## Como adoptar — 4 passos

### Passo 1 — Responder às perguntas de personalização

Abre `personalization-questions.md` e responde a cada pergunta. As respostas vão definir:

- Idioma principal e variante (PT-PT, PT-BR, EN-UK, EN-US, ES, FR, …)
- Tipo de letra preferida (Arial, Calibri, Times New Roman, Inter, Source Sans, …)
- Convenção de naming de ficheiros
- Registo de escrita (académico / profissional / pessoal)
- Domínio principal (medicina / engenharia / negócios / direito / …)
- Convenções visuais (callouts, ênfases, emojis, …)

Guarda as respostas no teu próprio Style Book (ver Passo 2).

### Passo 2 — Criar o teu Style Book

Copia `style-book-template.md` para um ficheiro pessoal (ex.: `meu-style-book.md`) e preenche com as respostas do Passo 1. Este ficheiro passa a ser **a tua fonte de verdade** para qualquer revisão futura.

### Passo 3 — Aplicar o checklist a cada deliverable

Antes de entregar qualquer documento:

1. Abre `checklist.md`
2. Percorre os 12 itens
3. Marca cada um como ✅ pass ou ❌ fail
4. Se algum bloqueante (#1-#9) falha → corrige antes de entregar
5. Se algum *warning* (#10-#12) falha → corrige se houver tempo, ou anota como conhecido

### Passo 4 — Iterar

Após algumas semanas de uso, ajusta o teu Style Book com base em padrões que vais notar (ex.: tipo de erro recorrente, marca LLM específica que aparece na tua escrita assistida).

---

## Filosofia

Este Quality Gate assenta em três princípios:

1. **Bloqueante > advisory.** Os 9 itens estruturais não são "sugestões" — se falham, o documento não sai. Os 3 itens de conteúdo são *warnings* que toleras se justificado.
2. **Verificação automatizável > inspecção subjectiva.** Sempre que possível, cada item tem um comando ou *grep* concreto. Reduz tempo de revisão e remove subjectividade.
3. **Personalização > prescrição.** Não há um único "estilo correcto". O checklist define o **esqueleto** (estrutura, ausência de marcas LLM, naming consistente). Tu defines o **estilo** (tipo de letra, registo, idioma, convenções visuais).

---

## O que este Quality Gate **não** faz

- Não verifica correcção factual do conteúdo (não substitui revisão de domínio)
- Não verifica originalidade / plágio (precisa de Turnitin ou equivalente)
- Não substitui *peer review* académico
- Não corrige automaticamente — só sinaliza problemas

---

## Licença

**CC BY-NC-SA 4.0** (Creative Commons — Atribuição, Não-Comercial, Partilha em condições idênticas).

- ✅ Usar, modificar, adaptar para uso pessoal/académico
- ✅ Partilhar com outros colegas (com atribuição)
- ❌ Vender, integrar em produto comercial, ou apresentar como trabalho original

Detalhes na licença do repositório.

## Contribuições

*Issues* e *pull requests* são bem-vindos. Se descobrires uma marca LLM nova, um item bloqueante adicional, ou tiveres um Style Book de domínio (ex.: medicina, engenharia, direito) que valha a pena partilhar, abre uma PR.
