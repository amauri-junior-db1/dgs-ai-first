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
| `cenario-3` | Fase de Governança e Validação | 27/06 |

A branch `main` contém a documentação de referência do projeto (Anexos A, B e C + documentos NovaTech).

---

## Estrutura de pastas

```
dgs-ai-first/
├── README.md
├── exercicio-fase-1-entendimento.md        ← Enunciado — Cenário 1
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
└── 📁 qa-exercicios/                       ← Entregáveis do papel QA (Cenário 1)
    ├── QA-exercicio-1.1-cenarios-de-falha.md
    ├── QA-exercicio-1.2-criterios-de-aceitacao.md
    └── QA-exercicio-1.3-plano-de-testes-rag.md
```

> Os exercícios 2.1–2.3 (Cenário 2) ficam só na branch [`cenario-2`](../../tree/cenario-2); os 3.1–3.2 (Cenário 3) só na [`cenario-3`](../../tree/cenario-3). Cada branch contém apenas os entregáveis da própria fase.

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
