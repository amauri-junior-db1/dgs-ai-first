# QA — Exercício 1.2: Design de Critérios de Aceitação para Respostas de IA

**Projeto:** NovaTech — Assistente de IA para Atendimento  
**Papel:** QA  
**Fase:** Cenário-Âncora 1 — Entendimento e Contexto  
**Ferramentas utilizadas:** Claude (chat) + Claude Cowork  

---

## Parte 1 — Avaliação manual das 5 respostas (feita antes da rubrica)

> Avaliação realizada com base nos documentos do Anexo A como fonte de verdade. Cada resposta foi verificada individualmente antes da criação da rubrica.

---

### Resposta #1

| Campo | Detalhe |
|-------|---------|
| **Pergunta** | "Qual o prazo de devolução?" |
| **Resposta do assistente** | "O prazo é de 7 dias úteis, exceto para cargas perigosas classes 1 a 6 da ANTT." |
| **Fonte citada** | POL-001, seção 3.2 |
| **Avaliação** | ✅ **Correta** |
| **Justificativa** | A resposta está alinhada com POL-001 seção 3.1 (prazo geral = 7 dias úteis) e seção 3.2 (exceção para cargas perigosas classes 1 a 6 da ANTT). A fonte citada está correta. A resposta é completa para o escopo da pergunta. **Observação:** A citação aponta a seção 3.2 (exceções), quando a regra principal está na 3.1. Seria mais preciso citar ambas. Não invalida a resposta, mas é um ponto de atenção de completude. |

---

### Resposta #2

| Campo | Detalhe |
|-------|---------|
| **Pergunta** | "Quanto custa frete para 600kg para Manaus?" |
| **Resposta do assistente** | "O frete especial para cargas acima de 500kg para a região Norte tem multiplicador de 1.8 sobre o valor base." |
| **Fonte citada** | PROC-042-v2, seção 2 |
| **Avaliação** | ⚠️ **Parcialmente correta** |
| **Justificativa** | O multiplicador 1.8 para a região Norte está correto de acordo com a PROC-042-v2 seção 2.1. A fonte está correta. Porém, a resposta está **incompleta** por não mencionar: (a) o fator de peso aplicável — para 600kg (faixa 500–1.000kg) o fator é 1.0, ou seja, não há acréscimo; (b) a existência da versão anterior (PROC-042 v1) com multiplicador diferente (1.6 para o Norte), o que é relevante para chamados abertos antes de 01/12/2023 (disposições transitórias da v2, seção 5). O atendente que usar apenas o multiplicador pode calcular o valor errado por não saber como compor a fórmula completa. |

---

### Resposta #3

| Campo | Detalhe |
|-------|---------|
| **Pergunta** | "Qual o SLA do cliente Platinum?" |
| **Resposta do assistente** | "O cliente Platinum tem resposta em até 1h e resolução em até 12h." |
| **Fonte citada** | SLA-2024 |
| **Avaliação** | ❌ **Incorreta — Alucinação** |
| **Justificativa** | **O tier "Platinum" não existe na NovaTech.** O SLA-2024 seção 1 afirma explicitamente: "Não existem outros tiers além dos três listados" (Gold, Silver, Standard). O assistente não apenas inventou os valores de SLA (1h de resposta e 12h de resolução não correspondem a nenhum tier real), como inventou o próprio tier. A citação "SLA-2024" é falsa no contexto — o documento não contém nenhuma informação sobre Platinum. Este é um caso grave de alucinação: resposta confiante, com fonte aparente, mas completamente fabricada. O atendente que usar essa informação vai comprometer a experiência do cliente e violar o contrato. |

---

### Resposta #4

| Campo | Detalhe |
|-------|---------|
| **Pergunta** | "Posso devolver carga perigosa?" |
| **Resposta do assistente** | "Sim, cargas perigosas podem ser devolvidas em até 7 dias úteis." |
| **Fonte citada** | POL-001, seção 3.2 |
| **Avaliação** | ❌ **Incorreta — Inversão de regra** |
| **Justificativa** | A POL-001 seção 3.2 afirma o oposto: cargas perigosas **NÃO são elegíveis** para devolução pelo processo padrão. O assistente citou a seção correta, mas inverteu completamente o sentido da regra. Esta é uma falha crítica de interpretação: a fonte é real, mas o conteúdo da resposta contradiz diretamente o que a fonte diz. O impacto operacional é alto — um atendente que siga essa resposta vai gerar promessas indevidas ao cliente e criar um chamado de devolução que será rejeitado, além de potencial risco regulatório com cargas ANTT. |

---

### Resposta #5

| Campo | Detalhe |
|-------|---------|
| **Pergunta** | "Qual o multiplicador de frete para o Sudeste?" |
| **Resposta do assistente** | "O multiplicador regional para o Sudeste é 1.1." |
| **Fonte citada** | PROC-042-v2, seção 2 |
| **Avaliação** | ⚠️ **Parcialmente correta** |
| **Justificativa** | O valor 1.1 está correto para a PROC-042-v2 (versão revisada, novembro/2023). A fonte citada está correta. Porém, a resposta omite que existe uma versão anterior (PROC-042 v1) com multiplicador diferente para o Sudeste (1.0), e que chamados anteriores a 01/12/2023 ainda usam a v1. Para uma pergunta com impacto financeiro direto, a omissão da ressalva sobre versões é um ponto relevante de incompletude — o atendente pode aplicar o multiplicador errado em chamados em transição. |

---

### Resumo da avaliação manual

| # | Status | Principal problema |
|---|--------|--------------------|
| Resposta 1 | ✅ Correta | Citação poderia ser mais precisa (seção 3.1 + 3.2) |
| Resposta 2 | ⚠️ Parcial | Fórmula incompleta + omissão de disposições transitórias |
| Resposta 3 | ❌ Incorreta | Alucinação — tier e valores fabricados |
| Resposta 4 | ❌ Incorreta | Inversão da regra — POL-001 seção 3.2 diz o contrário |
| Resposta 5 | ⚠️ Parcial | Omissão de versão anterior e disposições transitórias |

---

## Parte 2 — Rubrica de avaliação (elaborada com o Claude)

> **Prompt utilizado:**
> "Crie uma rubrica de avaliação de respostas de um assistente de IA para atendimento em logística. A rubrica deve ter 4 dimensões: precisão factual, citação de fonte, aderência aos guardrails e completude. Cada dimensão deve ter escala de 1 a 3, com descrição clara de cada nível, objetiva o suficiente para que dois avaliadores diferentes cheguem à mesma pontuação."

---

### Rubrica de Avaliação de Respostas do Assistente NovaTech

#### Dimensão 1: Precisão Factual (PF)

Avalia se as informações afirmadas na resposta são verdadeiras e coerentes com a documentação oficial.

| Nível | Pontuação | Critério |
|-------|-----------|----------|
| **Inadequada** | 1 | A resposta contém ao menos uma informação factualmente errada (valor incorreto, regra invertida, entidade inexistente). A resposta pode citar fonte, mas o conteúdo contradiz o que a fonte diz. |
| **Parcialmente correta** | 2 | Todas as informações afirmadas são corretas, mas a resposta omite dados relevantes que alterariam a conduta do atendente (ex: omitir exceção, omitir versão alternativa com impacto financeiro). |
| **Correta e completa** | 3 | Todas as informações afirmadas são corretas e as informações relevantes para o contexto da pergunta estão presentes, incluindo exceções e variações que impactam a tomada de decisão. |

---

#### Dimensão 2: Citação de Fonte (CF)

Avalia se a resposta indica a origem das informações de forma identificável e verificável.

| Nível | Pontuação | Critério |
|-------|-----------|----------|
| **Ausente ou falsa** | 1 | A resposta não cita nenhuma fonte, ou cita uma fonte que não suporta o conteúdo afirmado (ex: "SLA-2024" para informação sobre tier inexistente). |
| **Parcialmente correta** | 2 | A fonte citada é real e existe na base, mas está incompleta (ex: cita o documento mas não a seção) ou é genérica demais para verificação rápida. |
| **Correta e específica** | 3 | A fonte citada é real, existe na base, aponta para o documento e a seção relevante, e o conteúdo da resposta é verificável diretamente naquela seção. |

---

#### Dimensão 3: Aderência aos Guardrails (AG)

Avalia se a resposta segue os 4 guardrails definidos: (1) citar fonte, (2) nunca inventar prazos/valores, (3) dizer explicitamente quando não encontrou, (4) responder em português formal.

| Nível | Pontuação | Critério |
|-------|-----------|----------|
| **Viola guardrail(s)** | 1 | A resposta viola ao menos um guardrail de forma clara (ex: afirma valor que não está em nenhum documento; usa idioma diferente do português; não cita fonte mesmo citando um dado específico). |
| **Aderente com ressalvas** | 2 | A resposta segue os guardrails principais, mas há desvio menor (ex: linguagem levemente informal; citação de fonte presente mas incompleta; não menciona que "não encontrou" para uma parte da pergunta). |
| **Totalmente aderente** | 3 | A resposta segue todos os guardrails sem desvios. Se há informação não disponível na base, isso é declarado explicitamente. O português é formal e claro. Fonte está citada. Nenhum valor ou prazo foi inventado. |

---

#### Dimensão 4: Completude (CO)

Avalia se a resposta responde todos os aspectos da pergunta sem deixar lacunas que obrigam o atendente a buscar informação complementar.

| Nível | Pontuação | Critério |
|-------|-----------|----------|
| **Incompleta** | 1 | A resposta responde menos da metade do que foi perguntado, ou omite uma informação que é parte central da pergunta. O atendente precisaria buscar informação em outra fonte para agir. |
| **Parcialmente completa** | 2 | A resposta cobre o núcleo da pergunta, mas omite contexto relevante que o atendente deveria saber para agir com segurança (ex: exceção aplicável, versão alternativa de documento, necessidade de escalar para outro setor). |
| **Completa** | 3 | A resposta cobre todos os aspectos da pergunta e indica proativamente contexto relevante (exceções, versões, procedimentos de escalação) quando aplicável. |

---

## Parte 3 — Template de avaliação reutilizável

> Template produzido para uso do time de QA em avaliação de qualquer lote de respostas do assistente.

---

```markdown
# Template de Avaliação de Respostas — Assistente NovaTech

**Data da avaliação:** _______________  
**Avaliador:** _______________  
**Lote de respostas:** _______________  
**Versão do sistema avaliado:** _______________  

---

## Instruções de uso

1. Para cada resposta, leia a pergunta original, a resposta do assistente e a fonte citada.
2. Consulte o documento de referência indicado na coluna "Fonte esperada" do Anexo A.
3. Atribua pontuação de 1 a 3 em cada dimensão conforme a rubrica abaixo.
4. Registre observações específicas que justifiquem a pontuação.
5. Calcule a pontuação total (máximo: 12 por resposta).

## Rubrica resumida

| Dimensão | 1 — Inadequada | 2 — Parcial | 3 — Completa/Correta |
|----------|----------------|-------------|----------------------|
| Precisão Factual (PF) | Informação errada ou fabricada | Correto, mas incompleto | Correto e completo |
| Citação de Fonte (CF) | Ausente ou falsa | Fonte real, seção ausente | Fonte + seção verificável |
| Aderência Guardrails (AG) | Viola ao menos 1 guardrail | Desvio menor | Todos os guardrails respeitados |
| Completude (CO) | Responde < 50% da pergunta | Núcleo respondido, contexto ausente | Cobre pergunta + contexto relevante |

## Classificação por pontuação total

| Pontuação | Classificação | Ação recomendada |
|-----------|---------------|-----------------|
| 10–12 | ✅ Aprovada | Nenhuma ação necessária |
| 7–9 | ⚠️ Aprovada com ressalvas | Documentar lacunas para revisão do prompt |
| 4–6 | 🔶 Reprovada — falha moderada | Investigar causa; revisar chunking ou system prompt |
| 1–3 | ❌ Reprovada — falha crítica | Bloquear uso em produção; investigar imediatamente |

---

## Registro de avaliação

| # | Pergunta (resumo) | Resposta (resumo) | Fonte citada | PF | CF | AG | CO | Total | Classificação | Observações |
|---|-------------------|-------------------|--------------|----|----|----|----|-------|---------------|-------------|
| 1 | | | | | | | | | | |
| 2 | | | | | | | | | | |
| 3 | | | | | | | | | | |
| 4 | | | | | | | | | | |
| 5 | | | | | | | | | | |

**Pontuação média do lote:** _____ / 12  
**Percentual de respostas aprovadas (≥ 10):** _____%  
**Percentual de respostas com falha crítica (≤ 3):** _____%  

---

## Ações identificadas

| Problema recorrente | Quantidade de ocorrências | Ação sugerida | Responsável | Prazo |
|--------------------|--------------------------|---------------|-------------|-------|
| | | | | |

---

*Rubrica v1.0 — NovaTech Assistente de IA — Time de QA*
```

---

## Parte 4 — Aplicação da rubrica às 5 respostas

| # | PF | CF | AG | CO | Total | Classificação |
|---|----|----|----|----|-------|---------------|
| **R1** — Prazo de devolução | 3 | 2 | 3 | 2 | **10** | ✅ Aprovada |
| **R2** — Frete 600kg Manaus | 2 | 3 | 2 | 1 | **8** | ⚠️ Aprovada com ressalvas |
| **R3** — SLA Platinum | 1 | 1 | 1 | 1 | **4** | 🔶 Reprovada — falha moderada* |
| **R4** — Devolver carga perigosa | 1 | 2 | 1 | 1 | **5** | ❌ Reprovada — falha crítica** |
| **R5** — Multiplicador Sudeste | 2 | 3 | 3 | 2 | **10** | ✅ Aprovada com ressalvas*** |

> *\*R3 recebeu pontuação total 4, mas por natureza é uma falha crítica (alucinação de tier inexistente + valores fabricados). A classificação "falha moderada" pela pontuação subestima o risco. Recomendação: adicionar critério de override — qualquer resposta com PF=1 causado por entidade fabricada deve ser automaticamente reclassificada como falha crítica.*

> *\*\*R4 é a resposta de maior risco operacional: a inversão da regra de carga perigosa pode gerar promessas ao cliente que violam a POL-001 e criam risco regulatório ANTT.*

> *\*\*\*R5 recebeu total 10 mas a nota parcial em CO reflete a omissão das disposições transitórias — relevante para contratos em período de transição.*

### Justificativas detalhadas

**R1 — CF=2:** A fonte citada (seção 3.2) é real, mas a regra principal do prazo está na seção 3.1. Citar somente 3.2 pode confundir quem tentar verificar a fonte.

**R2 — AG=2:** A resposta não inventou valores, mas não mencionou as disposições transitórias da PROC-042-v2 seção 5, que são críticas para chamados em transição — desvio menor de completude que impacta a aplicação correta da regra.

**R2 — CO=1:** A fórmula completa (valor base × multiplicador × fator de peso) não foi apresentada. O atendente recebe o multiplicador, mas não sabe como compor o valor final.

**R3 — todos os campos = 1:** Tier inexistente + valores fabricados + fonte falseada = falha em todas as dimensões.

**R4 — CF=2:** A fonte (POL-001 seção 3.2) é real. O problema não está na citação, mas na inversão do conteúdo — por isso CF=2 e não 1.

---

*Documento produzido para o Exercício QA 1.2 — Cenário-Âncora 1, NovaTech.*
