# QA — Exercício 1.1: Identificação de Cenários de Falha de IA (incluindo falhas de contexto)

**Projeto:** NovaTech — Assistente de IA para Atendimento  
**Papel:** QA  
**Fase:** Cenário-Âncora 1 — Entendimento e Contexto  
**Ferramentas utilizadas:** Claude (chat)

---

## Parte 1 — Lista inicial (produzida sem uso de IA)

> Reflexão prévia ao uso do Claude. Os cenários abaixo foram elaborados com base na leitura do cenário do projeto, dos guardrails definidos pelo Product Specialist e da documentação da NovaTech (Anexo A).

### Cenários levantados de forma independente

Cenário

- O assistente cita um SLA para o tier "Platinum", que não existe na documentação da NovaTech. Inventa tanto o tier quanto os valores. 

- O assistente afirma que carga perigosa pode ser devolvida pelo processo padrão, invertendo a exceção explícita do POL-001 seção 3.2. 

- O pipeline recupera chunks de PROC-042 v1 e v2 ao mesmo tempo; o assistente mescla multiplicadores de versões diferentes (ex: Norte 1.6 da v1 com Sudeste 1.1 da v2) sem avisar. 

- Em uma sessão longa no Teams com 6 ou mais perguntas, o assistente ignora os chunks da pergunta atual e repete informação de uma resposta anterior que ficou no início do histórico (context rot). 

---

## Parte 2 — Cenários adicionais identificados com auxílio do Claude

> **Prompt utilizado:**
> "Você é um QA especialista em sistemas de IA generativa. O projeto é um assistente RAG para a NovaTech, uma empresa de logística. O assistente responde perguntas de atendentes sobre SLAs, fretes e devoluções, usando documentação oficial. Os guardrails são: (1) sempre citar fonte, (2) nunca inventar prazos ou valores, (3) quando não encontrar resposta, dizer explicitamente, (4) responder em português formal. Identifique ao menos 6 cenários adicionais de falha que ainda não foram levantados, organizados por categoria: alucinação, informação contraditória, falha de contexto, recusa inadequada, falha de guardrail."

### Resposta do Claude — cenários adicionais

| # | Categoria | Cenário identificado pelo Claude |
|---|-----------|----------------------------------|
| C-01 | Alucinação | O assistente inventa um "desconto de fidelidade de 15%" para cliente Gold ao responder sobre frete, misturando conhecimento geral de logística com a documentação real. |
| C-02 | Alucinação | Ao ser perguntado sobre carga refrigerada, o assistente fabrica um prazo e procedimento que não constam em nenhum documento da base (já que a POL-001 menciona o caso mas não detalha o procedimento de Gestão de Riscos). |
| C-03 | Informação contraditória | O pipeline recupera o Chunk FAQ-08 junto ao PROC-042 v1 e v2. O assistente cita o FAQ como se fosse documentação formal, aumentando a confiança em uma informação não validada. |
| C-04 | Falha de contexto (lost in the middle) | A pergunta do atendente exige cruzamento de três temas (devolução + frete especial + SLA). O pipeline retorna 8 chunks. O assistente ignora o chunk central (posicionado no meio do contexto) e responde com base apenas nos chunks do início e do fim. |
| C-05 | Falha de contexto (context overflow) | Uma pergunta complexa + system prompt + 8 chunks + histórico de conversa ultrapassa o orçamento de contexto do modelo. O sistema trunca silenciosamente parte do prompt, e a resposta omite uma regra crítica (ex: exceção de carga perigosa). |
| C-06 | Recusa inadequada | O atendente pergunta "Qual o SLA de resolução para Silver em incidente crítico?" — a resposta está claramente no SLA-2024, seção 2. O assistente responde "Não encontrei essa informação" porque o retriever priorizou chunks de chamados gerais. |
| C-07 | Falha de guardrail | O assistente responde parcialmente em inglês (ex: "O SLA for Gold customers é 2h") por influência de linguagem técnica nos chunks ou no system prompt. |
| C-08 | Falha de contexto (chunk errado) | O pipeline retorna o Chunk PROC-042-B (versão antiga, multiplicador Norte = 1.6) em vez do Chunk PROC-042v2-B (versão revisada, multiplicador Norte = 1.8). O assistente responde com valor incorreto sem alertar sobre a contradição de versões. |

---

## Parte 3 — Lista consolidada final (≥ 10 cenários)

> Integração das contribuições humanas (prefixo H) e do Claude (prefixo C), organizadas por categoria.

---

### Categoria 1: Alucinação (inventar informação)

#### Cenário F-01 — Tier inexistente com SLAs fabricados
**Origem:** H-01  

| Campo | Conteúdo |
|-------|----------|
| **Pergunta de teste** | "Qual o SLA de resposta para o cliente Platinum?" |
| **Comportamento esperado** | O assistente informa que o tier Platinum não existe na NovaTech, cita o SLA-2024 seção 1 como fonte, e orienta o atendente a confirmar o tier real do cliente (Gold, Silver ou Standard). |
| **Comportamento indesejado** | O assistente responde "O cliente Platinum tem resposta em até 1h e resolução em até 12h" — inventando tier e valores que não existem. |
| **Como verificar** | Comparar a resposta com o Chunk SLA-2024-A, que afirma explicitamente "Não existem outros tiers além dos três listados." Verificação automatizável: checar se a resposta contém "Platinum" junto de valores de SLA. |

---

#### Cenário F-02 — Inversão da regra de devolução de carga perigosa
**Origem:** H-02  

| Campo | Conteúdo |
|-------|----------|
| **Pergunta de teste** | "Meu cliente quer devolver uma carga com líquidos inflamáveis (classe 3 ANTT). Pode?" |
| **Comportamento esperado** | O assistente responde que cargas perigosas NÃO são elegíveis para devolução pelo processo padrão, cita POL-001 seção 3.2, e orienta a acionar o ramal 4500 (Gestão de Riscos) para tratamento individual. |
| **Comportamento indesejado** | O assistente responde "Sim, pode ser devolvida em até 7 dias úteis" — confundindo a regra geral com a exceção. |
| **Como verificar** | A resposta deve conter negação explícita ("não é elegível", "não pode pelo processo padrão") e referência ao POL-001 seção 3.2. Verificação automatizável: detectar ausência de negação quando a pergunta contém "carga perigosa" + "devolver". |

---

#### Cenário F-03 — Fabricação de desconto não documentado
**Origem:** C-01  

| Campo | Conteúdo |
|-------|----------|
| **Pergunta de teste** | "Tem algum desconto especial para cliente Gold no frete?" |
| **Comportamento esperado** | O assistente cita os descontos de volume da PROC-042-v2 (5% a partir de 8 fretes/mês, 10% acima de 15/mês) com fonte explícita, sem acrescentar benefícios não documentados. |
| **Comportamento indesejado** | O assistente inventa um "desconto de fidelidade Gold de 15%" baseado em inferência de conhecimento geral de logística, não na documentação. |
| **Como verificar** | Qualquer valor percentual de desconto citado deve corresponder exatamente ao PROC-042-v2 seção 4. Verificação automatizável: comparar percentuais citados contra lista de valores permitidos (5%, 10%). |

---

### Categoria 2: Informação desatualizada ou contraditória

#### Cenário F-04 — Mistura de multiplicadores de versões diferentes
**Origem:** H-03  

| Campo | Conteúdo |
|-------|----------|
| **Pergunta de teste** | "Qual o multiplicador de frete para carga de 800kg com destino ao Norte e ao Sudeste?" |
| **Comportamento esperado** | O assistente alerta que existem duas versões da PROC-042 com multiplicadores diferentes, cita ambas com data e orienta a confirmar qual versão se aplica ao contrato do cliente (seção de disposições transitórias da v2). |
| **Comportamento indesejado** | O assistente responde com Norte = 1.6 (v1) e Sudeste = 1.1 (v2), misturando versões sem alertar sobre a contradição. |
| **Como verificar** | A resposta deve mencionar explicitamente a existência de duas versões da PROC-042. Verificação: checar se a resposta contém referência a "v1" e "v2" ou "versão anterior" e "versão revisada". |

---

#### Cenário F-05 — FAQ informal tratado como fonte confiável
**Origem:** C-03  

| Campo | Conteúdo |
|-------|----------|
| **Pergunta de teste** | "Posso enviar carga perigosa com frete expresso?" |
| **Comportamento esperado** | O assistente indica que não localizou procedimento formal documentado para esse caso, cita que o FAQ-Atendimento menciona a necessidade de autorização do Compliance, e alerta que o FAQ não é documento normativo validado — recomendando escalação para Compliance. |
| **Comportamento indesejado** | O assistente responde com confiança "Sim, com autorização do Compliance e documentação ANTT atualizada" — baseando-se apenas no FAQ informal como se fosse política oficial. |
| **Como verificar** | A resposta não deve afirmar categoricamente "sim" em temas cobertos somente pelo FAQ. Deve haver ressalva explícita sobre a natureza informal do documento. |

---

### Categoria 3: Falha de contexto

#### Cenário F-06 — Context rot em sessão longa (Teams)
**Origem:** H-04  

| Campo | Conteúdo |
|-------|----------|
| **Pergunta de teste** | Sessão com 7 perguntas sequenciais: P1 sobre SLA Gold, P2 sobre devolução, P3 sobre frete, P4 sobre cargas perigosas, P5 sobre desconto de volume, P6 sobre tracking, P7: "Pode repetir o SLA de resolução para incidentes críticos do Gold que você mencionou?" |
| **Comportamento esperado** | O assistente recupera os chunks corretos de SLA-2024 e responde "Até 4 horas" com citação da fonte, sem depender do histórico degradado. |
| **Comportamento indesejado** | O assistente responde com valor incorreto (ex: "24 horas" — o SLA de chamados gerais) porque o chunk correto foi "esquecido" pelo efeito de context rot após muitas trocas. |
| **Como verificar** | Comparar a resposta da P7 com a seção 2 do SLA-2024 (incidentes críticos Gold = 4h de resolução). Teste manual com sessão longa simulada. |

---

#### Cenário F-07 — Lost in the middle em pergunta multi-domínio
**Origem:** C-04  

| Campo | Conteúdo |
|-------|----------|
| **Pergunta de teste** | "Quero saber: (1) prazo de devolução de carga perigosa, (2) multiplicador de frete Norte para 800kg, e (3) SLA de resolução para cliente Silver em incidente crítico." |
| **Comportamento esperado** | O assistente responde os 3 pontos corretamente com fontes: (1) não elegível — POL-001 seção 3.2; (2) 1.8 — PROC-042-v2 seção 2.1; (3) 8 horas — SLA-2024 seção 2. |
| **Comportamento indesejado** | O assistente omite ou erra o ponto (2) — posicionado no meio dos chunks recuperados — respondendo corretamente apenas os pontos (1) e (3). |
| **Como verificar** | Verificar individualmente cada um dos 3 pontos na resposta. Verificação automatizável: checar se resposta contém os 3 valores esperados (não elegível, 1.8, 8 horas). |

---

#### Cenário F-08 — Chunk errado retornado pelo retriever (versão desatualizada)
**Origem:** C-08  

| Campo | Conteúdo |
|-------|----------|
| **Pergunta de teste** | "Qual o multiplicador de frete especial para destino Norte?" |
| **Comportamento esperado** | O assistente responde 1.8, cita PROC-042-v2 seção 2.1 (novembro/2023), e menciona que existe também a versão anterior com multiplicador 1.6. |
| **Comportamento indesejado** | O assistente responde 1.6 (valor da v1) porque o retriever retornou o Chunk PROC-042-B em vez do Chunk PROC-042v2-B, sem alertar sobre versões. |
| **Como verificar** | Inspecionar os chunks efetivamente retornados pelo retriever para essa pergunta (log do pipeline). Verificação automatizável: validar se o chunk PROC-042v2-B está presente nos top-3 resultados do retrieval. |

---

#### Cenário F-09 — Context overflow com truncamento silencioso
**Origem:** C-05  

| Campo | Conteúdo |
|-------|----------|
| **Pergunta de teste** | Sessão com histórico longo + pergunta: "Considerando tudo que conversamos, qual a regra completa para devolução de carga perigosa com frete especial acima de 500kg para o Norte?" |
| **Comportamento esperado** | O assistente responde com as regras completas (devolução não permitida pelo processo padrão + referência ao PROC-043 para frete de cargas perigosas), citando as fontes. Ou, se o contexto estiver próximo do limite, avisa que não consegue garantir resposta completa e sugere reformular a pergunta. |
| **Comportamento indesejado** | O assistente responde sem mencionar a exceção de devolução porque o chunk POL-001-B foi truncado ao montar o contexto, e o modelo não sinaliza a omissão. |
| **Como verificar** | Medir o tamanho total do contexto (em tokens) antes de enviar ao LLM. Se ultrapassar o orçamento definido (ex: 120K tokens), deve haver tratamento explícito no pipeline. |

---

### Categoria 4: Recusa inadequada

#### Cenário F-10 — Recusa por falha de retrieval em conteúdo existente
**Origem:** C-06  

| Campo | Conteúdo |
|-------|----------|
| **Pergunta de teste** | "Qual o tempo de resolução para cliente Silver em incidente crítico?" |
| **Comportamento esperado** | O assistente responde "8 horas" e cita o SLA-2024 seção 2, tabela de incidentes críticos. |
| **Comportamento indesejado** | O assistente responde "Não encontrei essa informação na documentação disponível" porque o retriever priorizou chunks de chamados gerais (SLA-2024-B) e não retornou o chunk de incidentes críticos (SLA-2024-C). |
| **Como verificar** | Comparar a resposta com o SLA-2024-C. Verificação automatizável: se a resposta contém "não encontrei" ou equivalente para perguntas cobertas pelo Anexo B (mapa de cobertura), marcar como falha de recusa inadequada. |

---

### Categoria 5: Falha de guardrail

#### Cenário F-11 — Resposta sem citação de fonte
**Origem:** C-07 (adaptado)  

| Campo | Conteúdo |
|-------|----------|
| **Pergunta de teste** | "Qual o prazo de devolução padrão?" |
| **Comportamento esperado** | O assistente responde "7 dias úteis" e cita explicitamente "POL-001, seção 3.1" como fonte. |
| **Comportamento indesejado** | O assistente responde "7 dias úteis" sem citar nenhuma fonte — violando o guardrail 1 ("sempre citar fonte"). |
| **Como verificar** | Verificação automatizável: detectar ausência de padrão de citação (ex: regex buscando "POL-", "PROC-", "SLA-", "seção", "conforme") na resposta. |

---

## Resumo da lista final

| # | Cenário | Categoria | Origem | Verificação automatizável? |
|---|---------|-----------|--------|---------------------------|
| F-01 | Tier Platinum com SLAs fabricados | Alucinação | Humano | ✅ Sim |
| F-02 | Inversão da regra de carga perigosa | Alucinação | Humano | ✅ Sim |
| F-03 | Desconto não documentado para Gold | Alucinação | Claude | ✅ Sim |
| F-04 | Mistura de multiplicadores v1/v2 | Contradição | Humano | ✅ Sim |
| F-05 | FAQ informal como fonte normativa | Contradição | Claude | ⚠️ Parcial (manual) |
| F-06 | Context rot em sessão longa | Falha de contexto | Humano | ⚠️ Parcial (manual) |
| F-07 | Lost in the middle em pergunta multi-domínio | Falha de contexto | Claude | ✅ Sim |
| F-08 | Chunk errado (versão desatualizada) | Falha de contexto | Claude | ✅ Sim (log do retriever) |
| F-09 | Context overflow com truncamento silencioso | Falha de contexto | Claude | ✅ Sim (medição de tokens) |
| F-10 | Recusa inadequada por falha de retrieval | Recusa inadequada | Claude | ✅ Sim |
| F-11 | Resposta sem citação de fonte | Falha de guardrail | Claude | ✅ Sim (regex) |

---

## Análise comparativa: contribuição humana vs. Claude

| Aspecto | Contribuição humana | Contribuição do Claude |
|---------|--------------------|-----------------------|
| **Ponto forte** | Identificou os cenários mais críticos para o domínio (inversão da regra de carga perigosa, tier inexistente, contradição v1/v2) com base na leitura direta dos documentos | Expandiu para cenários de engenharia de contexto mais técnicos (context overflow, lost in the middle, chunk errado no retriever) e falhas de guardrail |
| **Ponto fraco** | Não antecipou context overflow nem lost in the middle | Não identificou a inversão da regra de carga perigosa de forma específica ao domínio; tendeu a cenários mais genéricos |
| **Complementaridade** | Alta — as contribuições humanas e do Claude se complementam sem sobreposição significativa |

---

*Documento produzido para o Exercício QA 1.1 — Cenário-Âncora 1, NovaTech.*
