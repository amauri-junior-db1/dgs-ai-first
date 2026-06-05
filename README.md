# NovaTech — Cenário-Âncora 1 | Papel: QA

Repositório dos entregáveis do **Exercício QA — Fase 1: Entendimento e Contexto**.

---

## Estrutura de pastas

```
PRÁTICA 1/
├── README.md                                        ← Este arquivo
├── exercicio-fase-1-entendimento.md                 ← Enunciado completo de todos os papéis
│
├── 📁 documentacao-novatech/                        ← Fonte de verdade do projeto
│   ├── anexo-a-documentacao-simulada-novatech.md    ← Todos os 5 documentos NovaTech
│   ├── anexo-b-chunks-referencia-rag.md             ← Chunks + mapa de cobertura RAG
│   ├── FAQ-atendimento.md                           ← FAQ informal do time de suporte
│   ├── POL-001-politica-devolucao.md                ← Política de devolução
│   ├── PROC-042-frete-especial-v1.md                ← Procedimento de frete (v1)
│   ├── PROC-042-v2-frete-especial-revisado.md       ← Procedimento de frete (v2 revisado)
│   └── SLA-2024-tabela-sla-clientes.md              ← Tabela de SLA por tier
│
└── 📁 qa-exercicios/                                ← Entregáveis do papel QA
    ├── QA-exercicio-1.1-cenarios-de-falha.md        ← Exercício 1.1
    ├── QA-exercicio-1.2-criterios-de-aceitacao.md   ← Exercício 1.2
    └── QA-exercicio-1.3-plano-de-testes-rag.md      ← Exercício 1.3
```

> **Dica VSCode:** Abre qualquer `.md` e pressiona `Ctrl+K V` para visualizar o preview renderizado lado a lado.

---

## Entregáveis — QA

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

| Arquivo | Pasta | Uso principal |
|---------|-------|---------------|
| `anexo-a-documentacao-simulada-novatech.md` | `documentacao-novatech/` | Fonte de verdade — avaliação de respostas |
| `anexo-b-chunks-referencia-rag.md` | `documentacao-novatech/` | Gabarito de retrieval (mapa de cobertura) |
| `POL-001-politica-devolucao.md` | `documentacao-novatech/` | Regras de devolução e exceções |
| `PROC-042-frete-especial-v1.md` | `documentacao-novatech/` | Multiplicadores de frete — versão original |
| `PROC-042-v2-frete-especial-revisado.md` | `documentacao-novatech/` | Multiplicadores de frete — versão revisada |
| `SLA-2024-tabela-sla-clientes.md` | `documentacao-novatech/` | SLAs por tier (Gold, Silver, Standard) |
| `FAQ-atendimento.md` | `documentacao-novatech/` | Conhecimento informal — usar com cautela |
| `exercicio-fase-1-entendimento.md` | raiz `PRÁTICA 1/` | Enunciado completo de todos os papéis |
