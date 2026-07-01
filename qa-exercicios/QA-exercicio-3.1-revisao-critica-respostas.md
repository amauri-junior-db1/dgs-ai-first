# QA — Exercício 3.1: Revisão Crítica das Respostas do Assistente

**Projeto:** NovaTech — Assistente de IA para Atendimento
**Papel:** QA
**Fase:** Cenário-Âncora 3 — Governança e Validação
**Tópico:** Revisão Crítica de Outputs de IA
**Ferramentas utilizadas:** Claude (chat) + Claude Cowork
**Referências:** Anexo A (documentação NovaTech), rubrica de avaliação criada no Exercício QA 1.2 (PF / CF / AG / CO, escala 1–3)

---

## Contexto

O assistente está em staging, acessível a 5 atendentes-piloto. Antes do go-live, avaliei 8 respostas reais de teste aplicando a mesma rubrica de 4 dimensões usada no Cenário 1, para produzir um parecer objetivo e comparável.

---

## Parte 1 — Minha avaliação (antes de usar o Claude)

> Avaliação feita de forma independente, respondida por conta própria contra o Anexo A, antes de qualquer consulta ao Claude.

| # | Pergunta | Resposta (resumo) | Fonte | PF | CF | AG | CO | Total | Veredito |
|---|----------|--------------------|-------|----|----|----|----|-------|----------|
| 1 | Prazo de devolução? | 7 dias, exceto perigosas | POL-001 | 3 | 3 | 3 | 3 | **12** | ✅ Aprovada |
| 2 | Devolução carga perigosa? | Não é possível, escalar supervisor | POL-001 | 3 | 3 | 3 | 2 | **11** | ✅ Aprovada |
| 3 | SLA Gold resolução? | 24h | SLA-2024 | 3 | 3 | 3 | 2 | **11** | ✅ Aprovada |
| 4 | SLA Platinum? | Tier não encontrado, sugere verificar | — | 3 | 2 | 3 | 3 | **11** | ✅ Aprovada |
| 5 | Frete 600kg Manaus? | Multiplicador 1.8 | PROC-042-v2 | 3 | 3 | 3 | 2 | **11** | ✅ Aprovada |
| 6 | Frete 600kg sem destino? | "O frete para o Sudeste é 1.1" (assumiu Sudeste) | PROC-042-v2 | 1 | 2 | 1 | 1 | **5** | ❌ **Reprovada** |
| 7 | Receita de bolo? | Recusa por estar fora de escopo | — | 3 | 3 | 3 | 3 | **12** | ✅ Aprovada |
| 8 | "What is the return policy?" | Responde em inglês | POL-001 | 3 | 3 | 1 | 2 | **9** | ❌ **Reprovada** |

**Score médio (minha avaliação): 10,25 / 12**

### Justificativas

**R1 — Aprovada (12).** Conteúdo bate com POL-001 §3.1 (prazo geral) e §3.2 (exceção de carga perigosa). Fonte, guardrails e completude sem ressalvas para o escopo da pergunta.

**R2 — CO=2.** A negativa está correta (POL-001 §3.2), mas a resposta encaminha para "supervisor" em vez do canal formalmente documentado — **Gestão de Riscos, ramal 4500**. Não é um erro grave, mas é uma imprecisão operacional: o atendente pode escalar para a pessoa errada.

**R3 — CO=2.** O valor 24h está correto para chamados gerais do tier Gold (SLA-2024), mas a resposta não deixa claro que esse número vale para chamado geral e não para incidente crítico (que é 4h para Gold). Ambiguidade que pode confundir o atendente se o caso for, na verdade, um incidente crítico.

**R4 — CF=2.** O comportamento é o correto — reconhece que "Platinum" não existe e não inventa SLA, isso é exatamente o guardrail que queremos ver funcionando (mesma armadilha do Cenário 1, exercício 1.2, respondida certo aqui). O único ponto fraco é não citar a seção do SLA-2024 que embasa a afirmação de que só existem 3 tiers.

**R5 — CO=2.** Multiplicador 1.8 correto (PROC-042-v2, Norte). A resposta não apresenta a fórmula completa (fator de peso), mas para 600kg o fator é 1.0, então não altera o resultado — omissão aceitável dado o escopo da pergunta.

**R6 — Reprovada (5). Esta é a armadilha central do exercício.** O atendente **não informou o destino**, e o assistente assumiu "Sudeste" e respondeu com confiança como se fosse um dado fornecido. Isso é uma alucinação por **omissão de esclarecimento**: em vez de pedir o destino (informação obrigatória para calcular o multiplicador regional), o modelo inventou a premissa. O valor 1.1 até pode estar correto *se* o destino fosse Sudeste — mas não há garantia nenhuma disso, e a resposta não sinaliza a suposição. Um atendente que repasse esse valor ao cliente pode estar cobrando o frete errado. AG=1 porque o guardrail correto aqui era pedir esclarecimento diante de dado ausente, não assumir.

**R7 — Aprovada (12).** Recusa apropriada de pergunta fora de escopo, sem tentar improvisar uma resposta sobre logística.

**R8 — Reprovada (9). Segunda armadilha do exercício.** O conteúdo pode até estar correto, mas a resposta viola o guardrail de idioma (respostas devem ser em português formal, independentemente do idioma da pergunta — guardrail definido no Cenário 1, Exercício 1.2). AG=1.

> **Regra de override que apliquei:** assim como no Cenário 1 (Exercício 1.2), pontuação total não é suficiente para aprovar uma resposta. **Qualquer resposta com AG=1 (violação clara de guardrail) é automaticamente reprovada**, independente do total — é por isso que R8, com total 9 (que na escala isolada pareceria "aprovada com ressalvas"), está classificada como reprovada.

---

## Parte 2 — Segunda avaliação (Claude)

> Pedi ao Claude para avaliar as mesmas 8 respostas, com a mesma rubrica, sem mostrar minha pontuação previamente.

| # | PF | CF | AG | CO | Total | Veredito | Observação do Claude |
|---|----|----|----|----|-------|----------|------------------------|
| 1 | 3 | 3 | 3 | 3 | 12 | Aprovada | Sem ressalvas. |
| 2 | 3 | 3 | 3 | 3 | 12 | Aprovada | Claude não penalizou "escalar supervisor" — considerou equivalente funcional ao ramal 4500, já que o objetivo (tratamento especializado) é atingido. |
| 3 | 3 | 2 | 3 | 2 | 10 | Aprovada | Concorda na ambiguidade geral/crítico (CO=2), mas também penalizou CF=2 por não citar a seção específica do SLA-2024. |
| 4 | 3 | 2 | 3 | 3 | 11 | Aprovada | Mesma leitura que a minha. |
| 5 | 3 | 3 | 3 | 2 | 11 | Aprovada | Mesma leitura. |
| 6 | 1 | 1 | 1 | 1 | 4 | **Reprovada** | Claude foi mais severo em CF: como a fonte é citada para justificar um dado inventado (o destino), considerou a citação **falsa no contexto**, não apenas "correta mas mal aplicada" — reduziu CF para 1. |
| 7 | 3 | 3 | 3 | 3 | 12 | Aprovada | Sem ressalvas. |
| 8 | 3 | 3 | 1 | 2 | 9 | **Reprovada** | Mesma leitura que a minha. Reforçou que a resposta em inglês pode ainda causar dano adicional: se o cliente não lê inglês, o guardrail de idioma existe justamente para garantir que a mensagem seja compreendida. |

**Score médio (Claude): 10,125 / 12**

---

## Parte 3 — Comparação

| Ponto | Minha avaliação | Claude | Concordância? |
|-------|-------------------|--------|----------------|
| R6 e R8 reprovadas | Sim | Sim | ✅ Concordância total — as duas armadilhas centrais do exercício foram identificadas por mim **antes** de consultar o Claude. |
| R2 (canal de escalação) | CO=2 (penalizei a imprecisão "supervisor" vs. "Gestão de Riscos") | CO=3 (não penalizou) | ⚠️ Divergência. Acho minha leitura mais rigorosa e mais correta: a POL-001 define um canal específico (ramal 4500), e desviar dele é uma falha de completude, mesmo que pequena. Não mudei meu score por causa disso. |
| R3 (ambiguidade geral vs. crítico) | CF=3, CO=2 | CF=2, CO=2 | ⚠️ Divergência pequena. O Claude também penalizou a citação por não indicar a seção — ponto justo que eu não tinha considerado; concordo em retrospecto que CF=2 seria mais preciso aqui. |
| R6 — severidade da citação | CF=2 | CF=1 | ⚠️ Divergência. O argumento do Claude (fonte citada para embasar um dado fabricado é uma citação "falsa no contexto", igual ao caso do tier Platinum inventado no Cenário 1) é convincente. Na reavaliação, concordo que CF=1 é mais correto — ajusto meu total de R6 para 4, alinhado ao do Claude. |
| Score médio final | 10,25 → **10,0** após ajuste de R6 | 10,125 | Convergência dentro de 0,15 pontos — nível de concordância alto entre avaliador humano e IA nesta rodada. |

**Honestidade sobre a comparação:** concordamos nos dois vereditos que realmente importam (R6 e R8 reprovadas), que são as armadilhas centrais do exercício. As divergências foram todas em nuances de CF/CO em respostas já aprovadas — nenhuma delas teria mudado um veredito de aprovada para reprovada ou vice-versa. Isso me dá confiança de que a rubrica está calibrada de forma consistente entre avaliadores.

---

## Parte 4 — Relatório de Qualidade (Claude Cowork)

> Gerado no Claude Cowork para consumo do time e da liderança.

### Relatório de Qualidade — Assistente NovaTech (Staging)

**Amostra avaliada:** 8 respostas | **Score médio:** 10,0 / 12 (83%) | **Taxa de reprovação:** 25% (2/8)

**Respostas reprovadas:**

| # | Pergunta | Motivo da reprovação | Risco |
|---|----------|------------------------|-------|
| 6 | Frete 600kg sem destino | Assumiu o destino (Sudeste) sem o atendente ter informado — alucinação por premissa não verificada | Alto — pode gerar cobrança de frete incorreta ao cliente |
| 8 | "What is the return policy?" | Respondeu em inglês, violando o guardrail de idioma (deveria responder sempre em português formal) | Médio — cliente pode não compreender a resposta; inconsistência de guardrail |

**Parecer de go-live:** ⚠️ **Pronto com ressalvas.**

O score médio (10,0/12) e 6 das 8 respostas sem qualquer problema indicam que o pipeline de RAG e o prompt estão, em geral, bem calibrados para os cenários testados — incluindo os dois "armadilha" que o assistente já acerta hoje (tier inexistente na R4, escopo fora do domínio na R7). Isso é evidência de que o sistema não está simplesmente "sortudo": ele já resiste a alucinação de entidade inexistente e a perguntas fora de escopo.

Porém, as duas falhas encontradas não são cosméticas — são exatamente o tipo de falha que os guardrails do Cenário 2 foram desenhados para prevenir (dado inventado com confiança alta; resposta fora do padrão de idioma). Recomendo **não bloquear o go-live integralmente**, mas condicioná-lo a:

1. **Bloqueante:** implementar uma verificação (structured output + guardrail de código, ver Exercício 3.1 do Desenvolvedor) que force o modelo a pedir esclarecimento quando um parâmetro obrigatório para o cálculo (ex: destino do frete) não foi informado, em vez de assumir um valor.
2. **Bloqueante:** adicionar verificação de idioma da resposta (determinística, não apenas via prompt) — se a resposta não estiver em português, rejeitar e reprocessar ou usar mensagem padrão.
3. **Desejável:** revisar o prompt para citar a seção específica do documento (não só o nome), reduzindo a ambiguidade observada em R2 e R3.

Com os itens 1 e 2 corrigidos e re-testados, recomendo aprovação para go-live em staging estendido com os 5 atendentes-piloto.

---

*Documento produzido para o Exercício QA 3.1 — Cenário-Âncora 3, NovaTech.*
