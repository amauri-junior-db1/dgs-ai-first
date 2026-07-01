# QA — Exercício 3.1: Revisão Crítica das Respostas do Assistente (Fase de Governança)

**Projeto:** NovaTech — Assistente de IA para Atendimento
**Papel:** QA
**Fase:** Cenário-Âncora 3 — Governança e Validação
**Tópico:** Revisão Crítica de Outputs de IA
**Ferramentas utilizadas:** Claude (chat) + Claude Cowork

**Rubrica aplicada** (criada no Cenário 2, simulada): 4 dimensões, escala 1–3 cada — **Precisão factual**, **Citação de fonte**, **Aderência a guardrails**, **Completude**. Score total por resposta: 4–12. Referência de verdade: **Anexo A — Documentação Simulada da NovaTech**.

---

## Parte 1 — Avaliação individual (antes de consultar o Claude)

| # | Pergunta | Precisão | Fonte | Guardrails | Completude | Total /12 | Veredito |
|---|----------|:-:|:-:|:-:|:-:|:-:|----------|
| 1 | Prazo de devolução? | 3 | 3 | 3 | 3 | 12 | ✅ Aprovada |
| 2 | Devolução carga perigosa? | 3 | 3 | 3 | 2 | 11 | ✅ Aprovada (ressalva) |
| 3 | SLA Gold resolução? | 3 | 3 | 3 | 3 | 12 | ✅ Aprovada |
| 4 | SLA Platinum? | 3 | 2 | 3 | 3 | 11 | ✅ Aprovada (ressalva) |
| 5 | Frete 600kg Manaus? | 3 | 3 | 3 | 3 | 12 | ✅ Aprovada |
| 6 | Frete 600kg sem destino? | 1 | 2 | 1 | 1 | 5 | ❌ Reprovada |
| 7 | Receita de bolo? | 3 | — | 3 | 3 | 12 | ✅ Aprovada (fora de escopo, reconhecido corretamente) |
| 8 | "What is the return policy?" | 3 | 3 | 1 | 3 | 10 | ❌ Reprovada |

### Justificativa das notas relevantes

- **#1** — "7 dias úteis" e a exceção de carga perigosa batem com POL-001 seção 3.1/3.2. Fonte citada corretamente. Nenhum problema.
- **#2** — Correto que carga perigosa não é elegível ao processo padrão e que precisa escalar. Completude 2 (não 3) porque a resposta resumida diz apenas "escalar supervisor", sem citar o canal formal correto (setor de Gestão de Riscos, ramal 4500 — POL-001 seção 3.2). Não é um erro factual grave, mas é uma omissão que vale corrigir antes do go-live.
- **#3** — SLA Gold, chamados gerais, resolução em até 24h úteis — confere com SLA-2024 seção 2. Correto.
- **#4** — Reconhecer que "Platinum" não existe e sugerir confirmar o tier é o comportamento esperado (SLA-2024 seção 1 é explícita: "não existem outros tiers além dos três listados"). Fonte 2 (não 3) porque a resposta não cita essa nota da SLA-2024 explicitamente — só diz "tier não encontrado". Correto no mérito, incompleto na citação.
- **#5** — Manaus = região Norte. PROC-042-v2: fator de peso para 600kg (faixa 500–1.000kg) = 1.0; multiplicador Norte = 1.8. A resposta cita "multiplicador 1.8" e a fonte PROC-042-v2 — consistente.
- **#6 (reprovada)** — O atendente não informou o destino, e a resposta afirma "o frete para o Sudeste é 1.1" como se fosse um fato, sem alertar que assumiu o destino. Isso é uma **alucinação por inferência não solicitada**: o assistente deveria ter perguntado o destino antes de calcular. Guardrail 1 (não inventar valores sem base) é violado porque o destino foi inventado, não o multiplicador em si.
- **#7** — Recusa correta e dentro do escopo esperado ("não tenho informações sobre receitas, posso ajudar com logística"). Não há dimensão de "fonte" aplicável a uma recusa por escopo — tratado como neutro/adequado.
- **#8 (reprovada)** — O conteúdo está correto e a fonte é citada, mas a resposta foi dada em inglês. Isso viola diretamente o guardrail de idioma (respostas devem ser em português formal). Um guardrail violado é suficiente para reprovar a resposta, independentemente da nota nas outras dimensões — não é um erro de conteúdo, é um erro de conformidade.

---

## Parte 2 — Segunda avaliação (Claude como revisor adicional)

**Prompt utilizado:**
> "Você é um segundo avaliador de QA revisando respostas de um assistente de IA para a NovaTech (logística). Use a documentação em anexo como fonte de verdade e avalie cada uma das 8 respostas abaixo pelas dimensões: precisão factual, citação de fonte, aderência a guardrails (idioma português, nunca assumir dados não informados, sempre citar fonte) e completude. Aponte especificamente qualquer alucinação por inferência (dado assumido sem ter sido informado) e qualquer violação de guardrail de idioma."

**Resposta do Claude (síntese):**

Confirma a reprovação das respostas #6 e #8 pelos mesmos motivos, e acrescenta dois pontos que a primeira leitura tratou como observação menor mas que merecem destaque:

1. **#6** é uma alucinação particularmente perigosa (mais do que um erro de arredondamento) porque o valor "1.1" é apresentado com o mesmo grau de confiança que uma resposta correta — não há sinalização de incerteza nem pedido de esclarecimento. Do ponto de vista de produto, isso é indistinguível de uma resposta certa aos olhos do atendente, o que aumenta o risco de o atendente repassar um valor de frete errado ao cliente.
2. **#4** deveria, idealmente, citar a nota explícita da SLA-2024 ("não existem outros tiers além dos três listados") em vez de apenas dizer "tier não encontrado". Isso não muda o veredito (segue aprovada), mas reforça o guardrail de sempre ancorar a resposta em uma fonte, mesmo quando a resposta é uma negação.

O Claude não alterou nenhum veredito da Parte 1.

---

## Parte 3 — Comparação entre avaliação humana (QA) e avaliação do Claude

| # | Minha avaliação | Avaliação do Claude | Concordância |
|---|------------------|----------------------|---------------|
| 1 | Aprovada | Aprovada | ✅ Total |
| 2 | Aprovada (ressalva) | Aprovada (ressalva) | ✅ Total |
| 3 | Aprovada | Aprovada | ✅ Total |
| 4 | Aprovada (ressalva) | Aprovada (ressalva, reforça citação da nota SLA-2024) | ✅ Total, com detalhe adicional |
| 5 | Aprovada | Aprovada | ✅ Total |
| 6 | Reprovada | Reprovada (reforça gravidade: confiança alta em dado assumido) | ✅ Total, com detalhe adicional |
| 7 | Aprovada | Aprovada | ✅ Total |
| 8 | Reprovada | Reprovada | ✅ Total |

**Honestidade da comparação:** não houve divergência de veredito entre a avaliação individual e a do Claude nesta rodada — a segunda leitura confirmou os dois problemas centrais (assunção indevida em #6, idioma em #8) e adicionou nuance de severidade e de completude de citação, sem mudar nenhuma nota de forma relevante. Isso é esperado quando a rubrica e os guardrails já estão bem definidos: o valor do segundo avaliador aqui é mais em profundidade de justificativa do que em detecção de novos problemas.

---

## Parte 4 — Relatório de Qualidade (gerado com apoio do Claude Cowork)

**NovaTech — Assistente de Atendimento**
**Relatório de Qualidade Pré-Go-Live | Amostra: 8 respostas em staging | Data: 01/07/2026**

**Score médio da amostra:** 10,6 / 12 (≈ 88%)

**Respostas reprovadas:**

| # | Pergunta | Motivo da reprovação |
|---|----------|------------------------|
| 6 | Frete 600kg sem destino informado | Assumiu "Sudeste" sem o dado ter sido fornecido e apresentou o valor como fato, sem sinalizar incerteza nem pedir esclarecimento. Risco de frete cobrado incorretamente ao cliente. |
| 8 | "What is the return policy?" | Respondeu em inglês — violação do guardrail de idioma (respostas devem ser em português formal), mesmo com conteúdo e fonte corretos. |

**Aprovadas com ressalva (monitorar, não bloqueante):**
- #2 e #4 — corretas no mérito, mas incompletas na citação do canal/nota formal de apoio (ramal 4500 / Gestão de Riscos; nota de tiers inexistentes na SLA-2024).

**Parecer de go-live:** **não recomendado o go-live sem antes corrigir os itens #6 e #8.** São duas correções de baixo esforço — um guardrail de idioma (forçar resposta em português) e uma regra de "pedir o dado obrigatório antes de calcular, nunca assumir" — mas de alto risco caso cheguem à produção sem tratamento, especialmente #6 (frete calculado errado repassado ao cliente). Com essas duas correções aplicadas e reavaliadas, os 88% de aderência da amostra atual sustentam um go-live controlado, com monitoramento reforçado nas duas primeiras semanas para: (a) perguntas de frete especial com múltiplas variáveis, e (b) completude de citação em respostas de negação/exceção.

---

*Documento produzido para o Exercício QA 3.1 — Cenário-Âncora 3, NovaTech.*
