# dgs-ai-first — Trilha de Certificação AI First | Papel: QA

Repositório dos entregáveis do papel **QA** na Trilha de Formação para Certificação AI First da DGS.

O cenário-âncora é a **NovaTech** — empresa de logística que está construindo um assistente de atendimento ao cliente baseado em IA com pipeline RAG sobre documentação interna.

---

## Organização do repositório

Os exercícios de cada cenário são entregues em branches separadas:

| Branch | Fase | Prazo |
|--------|------|-------|
| `cenario-1` | Fase de Entendimento e Contexto | 06/06 |
| `cenario-2` | Fase de Estruturação do Trabalho | 18/06 |
| `cenario-3` | Fase de Validação e Entrega | 27/06 |

A branch `main` contém a documentação de referência do projeto (Anexos A e B + documentos NovaTech).

---

## Estrutura de pastas

```
dgs-ai-first/
├── README.md
├── exercicio-fase-1-entendimento.md        ← Enunciado — Cenário 1
├── exercicio-fase-3-governanca.md          ← Enunciado — Cenário 3
│
├── 📁 documentacao-novatech/               ← Fonte de verdade do projeto
│   ├── anexo-a-documentacao-simulada-novatech.md
│   ├── anexo-b-chunks-referencia-rag.md
│   ├── FAQ-atendimento.md
│   ├── POL-001-politica-devolucao.md
│   ├── PROC-042-frete-especial-v1.md
│   ├── PROC-042-v2-frete-especial-revisado.md
│   └── SLA-2024-tabela-sla-clientes.md
│
└── 📁 qa-exercicios/                       ← Entregáveis do papel QA
    ├── QA-exercicio-1.1-cenarios-de-falha.md
    ├── QA-exercicio-1.2-criterios-de-aceitacao.md
    ├── QA-exercicio-1.3-plano-de-testes-rag.md
    ├── QA-exercicio-2.1-testing-standards-agents-md.md
    ├── QA-exercicio-2.2-spec-sdd-query-endpoint.md
    ├── QA-exercicio-2.3-skill-create-integration-test.md
    ├── QA-exercicio-3.1-revisao-critica-respostas.md
    └── QA-exercicio-3.2-revisao-critica-testes-ia.md
```

---

## Cenário 1 — Fase de Entendimento e Contexto

Foco: Fundamentos de IA Generativa, Engenharia de Prompt, Engenharia de Contexto, RAG e MCP.

### Exercício 1.1 — Identificação de Cenários de Falha de IA
`qa-exercicios/QA-exercicio-1.1-cenarios-de-falha.md`

| Item | Detalhe |
|------|---------|
| Cenários próprios (sem IA) | 4 cenários |
| Cenários via Claude | 8 cenários |
| **Total consolidado** | **11 cenários** |

Categorias cobertas:
- **Alucinação** — 3 cenários (tier inexistente, inversão de regra, desconto fabricado)
- **Informação contraditória** — 2 cenários (mistura de versões v1/v2, FAQ como fonte formal)
- **Falha de contexto** — 4 cenários (context rot, lost in the middle, chunk errado, context overflow)
- **Recusa inadequada** — 1 cenário
- **Falha de guardrail** — 1 cenário

---

### Exercício 1.2 — Design de Critérios de Aceitação para Respostas de IA
`qa-exercicios/QA-exercicio-1.2-criterios-de-aceitacao.md`

| Item | Detalhe |
|------|---------|
| Avaliação manual | 5 respostas avaliadas antes da rubrica |
| Respostas incorretas identificadas | R3 (alucinação — tier Platinum) e R4 (inversão — carga perigosa) |
| Rubrica | 4 dimensões × escala 1–3 = máximo 12 pontos |
| Template | Reutilizável para qualquer lote de respostas |

Dimensões da rubrica:

| Sigla | Dimensão |
|-------|----------|
| PF | Precisão Factual |
| CF | Citação de Fonte |
| AG | Aderência aos Guardrails |
| CO | Completude |

---

### Exercício 1.3 — Plano de Testes para Pipeline de RAG
`qa-exercicios/QA-exercicio-1.3-plano-de-testes-rag.md`

| Bloco | Foco | Casos |
|-------|------|-------|
| 1 — Ingestão | Extração, chunking, metadados | 4 |
| 2 — Retrieval | Chunks corretos recuperados | 6 |
| 3 — Geração | Qualidade da resposta do LLM | 3 |
| 4 — Contexto | Context rot, lost in the middle, overflow | 4 |
| 5 — Ponta a ponta | Fluxo completo com ground truth | 5 |
| 6 — Regressão | Gatilhos e critério de baseline | variável |
| **Total fixo** | | **22 casos** |

---

## Cenário 2 — Fase de Estruturação do Trabalho

Foco: AI Agents, Recorte de Domínio e SDD, AGENTS.md, Skills.

### Exercício 2.1 — Testing Standards para o AGENTS.md
`qa-exercicios/QA-exercicio-2.1-testing-standards-agents-md.md`

Seção `Testing Standards` machine-readable para o `AGENTS.md` do projeto NovaTech. Define 4 padrões prescritivos para geração de testes por agentes de IA.

| Padrão | Regra |
|--------|-------|
| TS-01 | Estrutura AAA obrigatória com labels `// Arrange`, `// Act`, `// Assert` |
| TS-02 | Assertions de conteúdo — proibido `toBeDefined()` como única verificação |
| TS-03 | Nomenclatura descritiva — `it(...)` deve descrever o comportamento sem precisar ler o corpo |
| TS-04 | Dados de domínio reais — inputs e expected values derivados de POL-001, SLA-2024, PROC-042-v2 |

Inclui: reescrita completa do teste ruim fornecido como referência + 3 critérios de review objetivos (RC-01 a RC-03).

---

### Exercício 2.2 — Spec SDD do Query Endpoint
`qa-exercicios/QA-exercicio-2.2-spec-sdd-query-endpoint.md`

Spec de testes derivada dos Verification Criteria do endpoint de query do NovaTech Assistant.

| VC | Comportamento verificado | Fonte |
|----|--------------------------|-------|
| VC-01 | Prazo de devolução: 7 dias úteis com citação POL-001 | POL-001 §3.1 |
| VC-02 | SLA por tier de cliente com citação SLA-2024 | SLA-2024 §2 |
| VC-03 | Guardrail carga perigosa: recusa + ramal 4500 | POL-001 §3.2 |
| VC-04 | Multiplicador de frete: versão vigente (PROC-042-v2) | PROC-042-v2 §2.1 |
| VC-05 | Gap na base: declarar explicitamente que não encontrou | Guardrail do sistema |

10 cenários funcionais (happy path + edge case por VC) + 4 testes de robustez de IA (prompt injection, idioma, ambiguidade, falsa premissa).

---

### Exercício 2.3 — Skill: create-integration-test
`qa-exercicios/QA-exercicio-2.3-skill-create-integration-test.md`

Skill reutilizável para geração de testes de integração por agentes de IA.

| Componente | Conteúdo |
|------------|----------|
| Template | Esqueleto com placeholders e guia de preenchimento |
| Exemplos DO | 2 testes corretos completos (guardrail carga perigosa + SLA Gold) |
| Anti-padrões DON'T | 4 erros comuns com código, consequência e correção |
| Checklist | 8 itens binários verificáveis em < 2 min |

Dependências declaradas: `[[testing-standards]]` + `[[novatech-domain-policy]]`.

---

## Cenário 3 — Fase de Governança e Validação

Foco: Harness Engineering (HITL e Structured Outputs), Revisão Crítica de Outputs de IA.

### Exercício 3.1 — Revisão Crítica das Respostas do Assistente
`qa-exercicios/QA-exercicio-3.1-revisao-critica-respostas.md`

Aplicação da rubrica de 4 dimensões (Cenário 1, Exercício 1.2) a 8 respostas do assistente em staging, com avaliação própria antes do Claude, segunda avaliação do Claude, comparação, e relatório de qualidade gerado no Claude Cowork com parecer de go-live.

| Item | Detalhe |
|------|---------|
| Respostas avaliadas | 8 |
| Reprovações identificadas | R6 (assumiu destino não informado) e R8 (respondeu em inglês, violando guardrail de idioma) |
| Score médio | 10,0 / 12 |
| Parecer de go-live | Pronto com ressalvas — 2 itens bloqueantes antes do lançamento |

---

### Exercício 3.2 — Revisão Crítica dos Testes Gerados por IA
`qa-exercicios/QA-exercicio-3.2-revisao-critica-testes-ia.md`

Revisão de 3 testes de integração gerados pelo Copilot, com avaliação própria antes do Claude, segunda avaliação do Claude, comparação, e reescrita do teste com assertions vagas.

| Teste | Problema identificado |
|-------|------------------------|
| 1 | Assertions vagas (`toBeDefined()`) — não verifica correção do conteúdo |
| 2 | Dados de teste irreais — não exercita o domínio NovaTech |
| 3 | Mock desconectado/permissivo que mascara ausência de validação de input |
| Transversal | Testes escritos em Jest num projeto que usa Vitest |

---

## Documentação de referência

| Arquivo | Uso principal |
|---------|---------------|
| `anexo-a-documentacao-simulada-novatech.md` | Fonte de verdade — avaliação de respostas |
| `anexo-b-chunks-referencia-rag.md` | Gabarito de retrieval (mapa de cobertura RAG) |
| `POL-001-politica-devolucao.md` | Regras de devolução, exceções e carga perigosa |
| `PROC-042-frete-especial-v1.md` | Multiplicadores de frete — versão original (desatualizada) |
| `PROC-042-v2-frete-especial-revisado.md` | Multiplicadores de frete — versão vigente |
| `SLA-2024-tabela-sla-clientes.md` | SLAs por tier (Gold, Silver, Standard) |
| `FAQ-atendimento.md` | Conhecimento informal — usar com cautela |
