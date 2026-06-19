# QA — Exercício 1.3: Plano de Testes para Pipeline de RAG

**Projeto:** NovaTech — Assistente de IA para Atendimento  
**Papel:** QA  
**Fase:** Cenário-Âncora 1 — Entendimento e Contexto  
**Ferramentas utilizadas:** Claude (chat) + Claude Cowork  
**Referências:** Anexo A (documentação NovaTech), Anexo B (chunks de referência e mapa de cobertura)

---

## Introdução e premissas

Testar um pipeline de RAG é diferente de testar sistemas determinísticos. As principais diferenças que este plano considera:

| Característica | Sistema tradicional | Sistema RAG/LLM |
|---------------|--------------------|--------------------|
| Resultado | Determinístico (pass/fail) | Probabilístico (graus de qualidade) |
| Critério de aprovação | Binário | Score em rubrica (ex: ≥ 10/12) |
| Regressão | "Mudou = quebrou" | "Mudou = pode ter melhorado ou piorado" |
| Unidade testável | Função, endpoint | Etapa do pipeline (ingestão, retrieval, geração) |
| Ambiente de teste | Estável | Pode variar com temperatura do modelo |

Por isso, este plano organiza os testes por **etapa do pipeline** e define critérios de aceitação baseados em rubrica — não em pass/fail absoluto.

---

## Visão geral do pipeline testado

```
[Fontes] → [Extração] → [Chunking] → [Embedding] → [Vector Store]
                                                            ↓
[Atendente] → [Pergunta] → [Embedding da pergunta] → [Retrieval]
                                                            ↓
                                               [Montagem do contexto]
                                                            ↓
                                                   [LLM (GPT-4o)]
                                                            ↓
                                               [Resposta com fonte]
```

---

## Bloco 1 — Testes de Ingestão

**Objetivo:** Verificar que os documentos foram corretamente extraídos, convertidos para texto, divididos em chunks e indexados no Azure AI Search.

**Natureza dos testes:** Determinísticos. Podem ser automatizados com scripts de validação pós-ingestão.

### T-ING-01 — Completude da ingestão

| Campo | Detalhe |
|-------|---------|
| **Descrição** | Verificar que todos os documentos esperados foram indexados e nenhum foi silenciosamente ignorado. |
| **Pré-condição** | Pipeline de ingestão executado com o conjunto completo de documentos da NovaTech. |
| **Procedimento** | Consultar o índice do Azure AI Search e contar os documentos. Comparar com a lista de origem (800 PDFs + 400 páginas wiki + 50 planilhas = 1.250 fontes). |
| **Critério de aceitação** | 100% dos documentos esperados estão presentes no índice. Qualquer documento ausente é registrado como falha com o caminho de origem. |
| **Verificação automatizável** | ✅ Script que consulta o índice e compara com manifesto de arquivos de origem. |

---

### T-ING-02 — Fidelidade do conteúdo extraído (PDFs com tabelas)

| Campo | Detalhe |
|-------|---------|
| **Descrição** | Verificar que tabelas em PDFs foram corretamente extraídas como texto estruturado, sem perda de células ou mistura de colunas. |
| **Pré-condição** | POL-001, PROC-042, PROC-042-v2 e SLA-2024 ingeridos. |
| **Procedimento** | Recuperar os chunks que deveriam conter as tabelas de multiplicadores (PROC-042-B e PROC-042v2-B) e comparar os valores extraídos com os originais. |
| **Dado de teste** | Chunk PROC-042v2-B deve conter: Sul=1.3, Sudeste=1.1, Centro-Oeste=1.4, Nordeste=1.5, Norte=1.8. |
| **Critério de aceitação** | Todos os valores da tabela presentes e corretos no chunk extraído. Nenhum valor trocado entre linhas (ex: Sul com valor de Norte). |
| **Verificação automatizável** | ✅ Script que busca o chunk por ID e valida os valores esperados com regex ou parsing. |

---

### T-ING-03 — Metadados de versionamento nos chunks

| Campo | Detalhe |
|-------|---------|
| **Descrição** | Verificar que os chunks contêm metadados de versão e data de vigência para permitir que o modelo priorize a versão correta. |
| **Pré-condição** | PROC-042 (v1) e PROC-042-v2 ingeridos. |
| **Procedimento** | Inspecionar os metadados dos chunks PROC-042-B e PROC-042v2-B no índice. Verificar presença de campos: `documento_versao`, `data_emissao`, `status_vigencia`. |
| **Critério de aceitação** | Todos os chunks possuem os 3 campos de metadado preenchidos. Chunks da v2 têm `data_emissao = 2023-11-10`. |
| **Verificação automatizável** | ✅ Script de validação de schema de metadados no índice. |

---

### T-ING-04 — Ausência de chunks truncados em fronteiras críticas

| Campo | Detalhe |
|-------|---------|
| **Descrição** | Verificar que o chunking não cortou um bloco de informação crítica ao meio (ex: tabela de multiplicadores pela metade, ou regra de exceção separada do seu contexto). |
| **Pré-condição** | Chunking executado com a estratégia definida (por seção + overlap de 10%). |
| **Procedimento** | Inspecionar manualmente os primeiros e últimos tokens de cada chunk que contém tabela ou lista de exceções. |
| **Critério de aceitação** | Nenhum chunk começa no meio de uma linha de tabela ou no meio de uma lista numerada sem o cabeçalho da lista no início do chunk. |
| **Verificação automatizável** | ⚠️ Parcial — script pode detectar chunks que começam com `|` (linha de tabela sem cabeçalho) ou com item numerado solto. |

---

## Bloco 2 — Testes de Retrieval

**Objetivo:** Verificar que, dada uma pergunta, o pipeline recupera os chunks corretos — nem mais, nem menos.

**Referência:** Mapa de cobertura do Anexo B (pergunta → chunks esperados).

**Natureza dos testes:** Semi-automatizáveis. O score de similaridade é determinístico dado o mesmo modelo de embedding; os resultados podem ser validados por script.

### T-RET-01 a T-RET-05 — Pares pergunta → chunk esperado

| # | Pergunta | Chunks esperados (Anexo B) | Chunks que NÃO devem aparecer no top-3 |
|---|----------|---------------------------|----------------------------------------|
| T-RET-01 | "Qual o prazo de devolução?" | POL-001-A, POL-001-B | Nenhuma restrição específica |
| T-RET-02 | "Posso devolver carga perigosa?" | POL-001-B | Chunk que inverta a regra (FAQ-03 pode aparecer, mas não deve ser top-1) |
| T-RET-03 | "Qual o SLA do cliente Gold?" | SLA-2024-B | SLA-2024-C (incidentes críticos) não deve ser top-1 para chamados gerais |
| T-RET-04 | "Qual o SLA do cliente Platinum?" | SLA-2024-A (contém "não existem outros tiers") | Qualquer chunk que contenha valores de SLA sem mencionar que Platinum não existe |
| T-RET-05 | "Qual o multiplicador de frete para o Norte?" | PROC-042v2-B (versão revisada) | PROC-042-B (v1) NÃO deve ser top-1 — deve aparecer somente como contexto secundário com ressalva |

**Critério de aceitação geral para testes de retrieval:**
- Os chunks esperados devem estar presentes nos top-5 resultados.
- Para T-RET-04 e T-RET-05, o chunk com a informação mais recente/correta deve ter score maior que o chunk desatualizado.
- Nenhum chunk irrelevante deve ter score acima de 0.85 (threshold de alta confiança) para as perguntas acima.

**Verificação automatizável:** ✅ Script que envia cada pergunta ao retriever, captura os top-5 chunks e verifica presença dos IDs esperados.

---

### T-RET-06 — Pergunta sem cobertura na base

| Campo | Detalhe |
|-------|---------|
| **Pergunta** | "Qual o valor do frete padrão para 300kg para o Nordeste?" |
| **Comportamento esperado do retriever** | Retornar chunks com score baixo (< 0.7) ou nenhum chunk com alta confiança — indicando ausência de cobertura. |
| **Comportamento esperado do assistente** | Dizer explicitamente que não encontrou informação sobre frete padrão (< 500kg) na documentação disponível. |
| **Comportamento indesejado** | Retriever retorna PROC-042v2-B com score alto e o assistente responde com a fórmula de frete especial (válida apenas para > 500kg) como se fosse a resposta correta. |
| **Verificação automatizável** | ✅ Verificar score máximo dos chunks retornados. Se score > 0.8 para pergunta sabidamente sem cobertura, registrar como falso positivo de retrieval. |

---

## Bloco 3 — Testes de Geração

**Objetivo:** Verificar que, dados os chunks corretos, o LLM gera a resposta adequada.

**Natureza dos testes:** Probabilísticos. Usar temperatura=0 para maximizar determinismo nos testes. Resultado avaliado pela rubrica do Exercício 1.2 (pontuação ≥ 10/12 para aprovação).

### T-GEN-01 — Resposta correta com chunks corretos

| Campo | Detalhe |
|-------|---------|
| **Setup** | Injetar manualmente o Chunk POL-001-B no contexto + pergunta "Posso devolver carga perigosa?" |
| **Comportamento esperado** | A resposta deve afirmar que NÃO é elegível pelo processo padrão, citar POL-001 seção 3.2, e indicar o ramal 4500. |
| **Critério de aceitação** | Pontuação PF=3 na rubrica. A resposta deve conter negação explícita. |

---

### T-GEN-02 — Comportamento com chunks contraditórios

| Campo | Detalhe |
|-------|---------|
| **Setup** | Injetar manualmente Chunk PROC-042-B (v1, Norte=1.6) E Chunk PROC-042v2-B (v2, Norte=1.8) no mesmo contexto + pergunta "Qual o multiplicador para o Norte?" |
| **Comportamento esperado** | O assistente deve alertar sobre a contradição entre versões, apresentar ambos os valores com data, e orientar a confirmar qual versão se aplica ao contrato. |
| **Comportamento indesejado** | O assistente escolhe um dos valores sem mencionar a contradição, ou mescla os dois sem aviso. |
| **Critério de aceitação** | A resposta deve conter referência explícita a "duas versões" ou "v1" e "v2". |

---

### T-GEN-03 — Comportamento com ausência de informação

| Campo | Detalhe |
|-------|---------|
| **Setup** | Contexto sem nenhum chunk relevante + pergunta "Qual o valor do seguro de carga para mercadoria de R$ 50.000?" |
| **Comportamento esperado** | O assistente responde que não encontrou essa informação na documentação oficial disponível, sem inventar percentuais. |
| **Comportamento indesejado** | O assistente responde com percentuais do FAQ-22 (0,3% ou 0,8%) como se fossem política oficial, ou inventa valores. |
| **Critério de aceitação** | AG=3 (guardrail 3 respeitado: "dizer explicitamente quando não encontrou"). |

---

## Bloco 4 — Testes de Contexto

**Objetivo:** Verificar que o gerenciamento do contexto (tamanho, posicionamento, histórico) não degrada a qualidade das respostas.

**Natureza dos testes:** Majoritariamente manuais. Alguns verificáveis por instrumentação do pipeline.

### T-CTX-01 — Orçamento de contexto (não ultrapassar limite)

| Campo | Detalhe |
|-------|---------|
| **Descrição** | Verificar que o contexto total montado (system prompt + metadados do cliente + chunks + histórico + pergunta) não ultrapassa o orçamento definido (ex: 120K tokens para GPT-4o com janela de 128K). |
| **Procedimento** | Instrumentar o pipeline para logar o total de tokens antes de cada chamada ao LLM. Testar com sessão de 10 perguntas e histórico acumulado. |
| **Critério de aceitação** | 100% das chamadas com total de tokens < 120K. Se ultrapassar, o pipeline deve truncar o histórico mais antigo (não os chunks nem o system prompt) e logar o evento. |
| **Verificação automatizável** | ✅ Log de tokens por chamada + alerta se > threshold. |

---

### T-CTX-02 — Context rot em sessão longa

| Campo | Detalhe |
|-------|---------|
| **Descrição** | Verificar que respostas tardias em uma sessão não dependem de informações fornecidas apenas no início do histórico. |
| **Procedimento** | Simular sessão com 8 perguntas sequenciais. A pergunta 8 deve ser equivalente à pergunta 1 (mesmo tema, diferente formulação). Comparar a qualidade das respostas 1 e 8 pela rubrica. |
| **Critério de aceitação** | Diferença de pontuação entre resposta 1 e resposta 8 ≤ 2 pontos (na escala de 12). Se a diferença for > 2, indica degradação por context rot. |
| **Verificação automatizável** | ⚠️ Parcial — requer avaliação humana das respostas pela rubrica. |

---

### T-CTX-03 — Lost in the middle em pergunta multi-domínio

| Campo | Detalhe |
|-------|---------|
| **Descrição** | Verificar que informações posicionadas no meio do contexto são processadas corretamente. |
| **Procedimento** | Montar contexto com 8 chunks na ordem: [SLA] [SLA] [FRETE] [FRETE] [DEVOLUÇÃO] [DEVOLUÇÃO] [SLA] [SLA]. Fazer pergunta que depende especificamente dos chunks de FRETE (posição central). Verificar se a resposta usa os chunks centrais. |
| **Critério de aceitação** | A resposta sobre frete deve estar correta (PF=3) mesmo com os chunks de frete posicionados no meio do contexto. |
| **Verificação automatizável** | ⚠️ Parcial — verificação do conteúdo da resposta é manual pela rubrica. |

---

### T-CTX-04 — Sessão no Teams com múltiplas perguntas sobre temas diferentes

| Campo | Detalhe |
|-------|---------|
| **Descrição** | Simular o fluxo real de um atendente que, numa mesma sessão no Teams, faz perguntas sobre SLA, depois sobre frete, depois sobre devolução. Verificar que o histórico não contamina respostas posteriores. |
| **Procedimento** | P1: SLA Gold. P2: Multiplicador Norte. P3: Prazo de devolução padrão. P4: Devolver carga perigosa? Avaliar cada resposta pela rubrica. |
| **Critério de aceitação** | Todas as 4 respostas com pontuação ≥ 10/12 na rubrica. Nenhuma resposta usa informação da resposta anterior como se fosse dado corrente. |

---

## Bloco 5 — Testes de Ponta a Ponta

**Objetivo:** Testar o fluxo completo (pergunta do atendente → resposta com fonte) com um conjunto de perguntas e respostas esperadas derivadas da documentação real.

**Referência:** Mapa de cobertura do Anexo B.

| # | Pergunta de entrada | Resposta esperada (resumo) | Fonte esperada | Critério mínimo |
|---|--------------------|-----------------------------|----------------|-----------------|
| E2E-01 | "Qual o prazo de devolução padrão?" | 7 dias úteis, exceto cargas perigosas | POL-001, seção 3.1 e 3.2 | PF=3, CF=3 |
| E2E-02 | "Qual o SLA de resolução para cliente Silver em incidente crítico?" | 8 horas | SLA-2024, seção 2 | PF=3, CF=3 |
| E2E-03 | "Qual o multiplicador de frete para o Norte na versão mais recente?" | 1.8 (PROC-042-v2) com menção à versão anterior (1.6) | PROC-042-v2, seção 2.1 | PF=3, CF≥2 |
| E2E-04 | "Existe o tier Platinum?" | Não existe; tiers são Gold, Silver e Standard | SLA-2024, seção 1 | PF=3, AG=3 |
| E2E-05 | "Quanto vale frete padrão para 200kg?" | Não encontrado na documentação disponível | N/A | AG=3 (resposta de ausência) |

**Critério de aceitação geral E2E:** ≥ 80% das respostas com pontuação ≥ 10/12. Nenhuma resposta com PF=1 por alucinação.

---

## Bloco 6 — Testes de Regressão

**Objetivo:** Garantir que mudanças no prompt, na documentação ou no pipeline não degradam respostas previamente aprovadas.

**Quando executar:** Automaticamente em cada um dos seguintes gatilhos:

| Gatilho | Escopo da regressão |
|---------|---------------------|
| Alteração no system prompt | Todos os testes de geração (Bloco 3) + E2E (Bloco 5) |
| Publicação de novo documento na base | T-RET correspondente ao tema do documento + E2E relevantes |
| Atualização de documento existente (ex: nova PROC-042-v3) | T-ING-03 (metadados) + T-RET-05 + T-GEN-02 + E2E-03 |
| Mudança no modelo de LLM ou versão | Suite completa (todos os blocos) |
| Mudança na estratégia de chunking | T-ING-04 + todos os T-RET |

**Critério de regressão:** Uma mudança é aprovada se a pontuação média do lote de regressão não cair mais de 1 ponto (na escala de 12) em relação à baseline anterior.

**Baseline:** Estabelecida na primeira execução completa aprovada da suite (go-live). Armazenada como artefato versionado junto ao código do pipeline.

---

## Considerações especiais sobre testes de IA

### Não-determinismo
O LLM pode gerar respostas ligeiramente diferentes para a mesma entrada, especialmente com temperatura > 0. Para testes de regressão, usar temperatura=0. Para testes de qualidade, executar cada caso 3 vezes e usar a pontuação mediana.

### Testes de alucinação são negativos por natureza
Diferente de testes funcionais, a verificação de alucinação busca a **ausência** de um comportamento (o assistente NÃO deve inventar). Isso exige que o conjunto de testes inclua perguntas sobre entidades inexistentes (ex: tier Platinum), procedimentos sem cobertura, e versões desatualizadas — como armadilhas deliberadas.

### Ground truth como artefato versionado
O conjunto de perguntas + respostas esperadas (Bloco 5) deve ser versionado junto com o código do pipeline. Quando a documentação da NovaTech muda, o ground truth também muda — isso deve ser um processo controlado, não uma atualização silenciosa.

---

## Resumo do plano

| Bloco | Nome | Qtde testes | Automatizável | Frequência |
|-------|------|-------------|---------------|------------|
| 1 | Ingestão | 4 | ✅ 3 / ⚠️ 1 | A cada execução do pipeline de ingestão |
| 2 | Retrieval | 6 | ✅ 5 / ⚠️ 1 | A cada mudança de embedding ou chunking |
| 3 | Geração | 3 | ⚠️ 2 / ❌ 1 | A cada mudança no system prompt |
| 4 | Contexto | 4 | ✅ 1 / ⚠️ 3 | Semanal + após mudança de modelo |
| 5 | Ponta a ponta | 5 | ⚠️ Parcial | A cada deploy |
| 6 | Regressão | Variável | ✅ Por gatilho | Conforme gatilhos definidos |

**Total de casos de teste fixos:** 22  
**Responsável pela manutenção do plano:** QA (Amauri)  
**Revisão do plano:** A cada sprint ou quando houver mudança arquitetural no pipeline

---

*Documento produzido para o Exercício QA 1.3 — Cenário-Âncora 1, NovaTech.*
