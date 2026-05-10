# Marcadores de Uso de LLM — Lista a Evitar

Lista exaustiva de artefactos típicos da geração por modelos de linguagem (Claude, ChatGPT, Gemini, Llama, etc.) que devem ser removidos antes de entregar um documento. Mesmo que o conteúdo seja bom, estes marcadores sinalizam ao leitor que o texto **não foi revisto** depois da geração — e isso compromete a percepção de qualidade.

---

## A. Marcadores tipográficos

### A.1 — Triple asterisks como separador
```
***
```
**Substituir por:** quebra de linha simples, *heading* explícito, ou `---` (separador horizontal Markdown padrão se necessário).

### A.2 — Em-dashes como separadores de secção
```
———
— —
```
**Substituir por:** `---` ou *heading* explícito.

### A.3 — Bullets com caracteres não-teclaveis
```
• Item
‣ Item
∙ Item
◦ Item
```
**Substituir por:** `-` ou `*` (Markdown padrão), ou números se a ordem importa.

### A.4 — Aspas tipográficas em contexto técnico
```
"texto"  →  em vez de  "texto" (em código ou referências técnicas)
```
**Substituir por:** aspas rectas (`"`) em código, *paths*, *commands*, identificadores. Aspas tipográficas (`""`) só em prosa corrente.

### A.5 — Reticências unicode
```
…
```
**Substituir por:** `...` (três pontos teclaveis), ou re-escrever a frase para não precisar.

---

## B. Frases formulaicas

### B.1 — Aberturas formulaicas
- "Certamente!"
- "Com certeza!"
- "Claro!"
- "Fico feliz em ajudar."
- "É um prazer poder explicar."
- "Excelente pergunta!"

**Por que evitar:** soam servis e não acrescentam informação. Em documentação escrita são **sempre** removíveis.

### B.2 — Conectores formulaicos
- "É importante notar que..."
- "Vale a pena mencionar que..."
- "Não se pode esquecer que..."
- "Convém destacar que..."

**Substituir por:** afirmação directa. Se a informação é importante, di-la directamente.

❌ "É importante notar que o orçamento foi reduzido em 15%."
✅ "O orçamento foi reduzido em 15%."

### B.3 — Fechos formulaicos
- "Em resumo,"
- "Em suma,"
- "Para concluir,"
- "Em última análise,"
- "Em conclusão,"
- "Espero que isto ajude!"
- "Se tiveres mais perguntas, fico ao dispor."

**Substituir por:** *heading* "Conclusão" ou "Recomendação" se realmente fechas o documento. Sem fórmula vazia.

### B.4 — Hedging excessivo
- "pode potencialmente"
- "é possível que talvez"
- "geralmente tende a"
- "tipicamente costuma"
- "frequentemente tem a tendência de"

**Por que evitar:** é redundância (potencialmente *já* implica possibilidade; talvez *já* implica incerteza). Acumular dois hedges enfraquece a frase sem ganhar precisão.

❌ "Esta abordagem pode potencialmente reduzir custos."
✅ "Esta abordagem pode reduzir custos." OU "Esta abordagem reduz custos em 12-18% (n=14 estudos)."

---

## C. Estruturas excessivas

### C.1 — Paralelismo sintáctico excessivo

Três ou mais pontos consecutivos com construção sintáctica idêntica soam mecânicos:

❌
```
- Aumenta a eficiência operacional
- Reduz os custos administrativos
- Melhora a satisfação dos clientes
- Optimiza os processos internos
- Maximiza o retorno do investimento
```

✅
```
- Eficiência operacional sobe ~15% (medido em tempo de ciclo)
- Custos administrativos descem €4-6k/mês após 6 meses
- Satisfação dos clientes melhora (NPS +12 pontos no piloto)
```

**Princípio:** se cada *bullet* tem o mesmo número de palavras e a mesma estrutura verbal, está formulaico. Variar comprimento e estrutura.

### C.2 — Listas de 5+ itens com 1 frase cada

LLMs tendem a produzir listas longas onde cada item é uma frase isolada. Se a lista tem 7+ *items* e cada um é uma frase de tamanho similar, suspeita.

**Solução:** consolidar em parágrafos agrupados, ou cortar para os 3-5 *items* essenciais.

### C.3 — Headings em cascata sem conteúdo

```
## Secção
### Sub-secção
#### Sub-sub-secção
##### Sub-sub-sub-secção
- Apenas um bullet aqui
```

LLMs sobre-estruturam quando não têm conteúdo. Se uma sub-sub-secção tem 1 *bullet*, achata para texto corrente.

---

## D. Conteúdo "filler"

### D.1 — Definições óbvias
- "O lucro é a diferença entre receitas e custos."
- "Marketing é o processo de promover produtos ou serviços."

**Por que evitar:** se o leitor é do domínio, é redundante. Se não é, deves explicar **uma vez** no início e referenciar.

### D.2 — Tautologias
- "Os custos fixos são fixos."
- "A análise quantitativa baseia-se em quantidades."

**Detectar:** procurar repetição de raiz da palavra na mesma frase.

### D.3 — Listas de "considerações"

LLMs gostam de listas tipo:

❌ "Considerações importantes a ter em conta:"
   "- Aspecto financeiro"
   "- Aspecto operacional"
   "- Aspecto estratégico"

Sem dizer **o que** considerar em cada aspecto. Se a lista não acrescenta informação concreta, eliminar.

---

## E. Inconsistências de geração

### E.1 — Mudança abrupta de tom

LLMs por vezes mudam de tom no meio de um documento (de formal para informal, ou vice-versa). Re-ler pelo menos a primeira e última frase de cada secção para verificar consistência.

### E.2 — Repetição da pergunta

❌ "A pergunta é: qual é o melhor método?
A pergunta sobre qual é o melhor método é importante porque..."

LLMs tendem a re-afirmar a pergunta antes de responder. Em documentação escrita, é redundante.

### E.3 — Auto-referência ao processo

❌ "Vamos analisar este caso passo a passo. Primeiro, vamos ver..."
❌ "Na resposta abaixo vou explorar..."
❌ "Como mencionei anteriormente..."

**Substituir por:** *headings* claros e remoção de auto-referência. O leitor já sabe que está a ler um documento; não precisa de narrador.

---

## F. Verificação automatizada

Comando *grep* para detectar os marcadores mais frequentes (PT-PT):

```bash
grep -Pn '^\*{3,}$|^•|Certamente!|Com certeza!|Fico feliz|importante notar|Em resumo,|Em suma,|Para concluir,|Espero que isto ajude|pode potencialmente|geralmente tende' <ficheiro>
```

Output esperado: vazio. Cada match = candidato a remover.

Para EN:
```bash
grep -Pn '^\*{3,}$|^•|Certainly!|Of course!|I.m happy to|Important to note|In summary,|In conclusion,|It.s worth noting|might potentially|tends to typically' <ficheiro>
```

---

## G. Quando manter

Há contextos onde alguns destes marcadores são aceitáveis:

- **Emails muito informais** entre colegas próximos: "Espero que isto ajude!" pode ser apropriado
- **Slides** onde *bullets* paralelos são uma convenção visual
- **Documentação técnica** onde "É importante notar que..." pode marcar uma *gotcha* genuína (mas então seria melhor uma caixa "⚠️ Atenção:")

Regra geral: **na dúvida, remover**. O documento fica mais curto e mais directo.

---

## H. Por que isto importa

Estes marcadores não são "errados" no sentido gramatical. Mas:

1. **Sinalizam ausência de revisão.** O leitor percebe que o texto saiu do LLM e foi colado sem retoque.
2. **Diluem informação.** Cada palavra formulaica é uma palavra que o leitor lê sem ganhar informação.
3. **Comprometem credibilidade.** Em contextos profissionais (consultoria, academia, parecer técnico), texto com marcas LLM é lido como menos sério.
4. **Acumulam.** Um único "Em resumo," não estraga o texto. Mas se aparecem 4-5 marcadores num documento, o efeito é cumulativo.

A remoção destes marcadores é tipicamente **a edição com maior retorno** depois de gerar texto com LLM. 5-10 minutos de revisão removem 80% do "*LLM smell*" do documento.
