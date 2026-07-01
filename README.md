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
├── exercicio-fase-3-governanca.md          ← Enunciado — Cenário 3
│
├── 📁 documentacao-novatech/               ← Fonte de verdade do projeto
│   ├── anexo-a-documentacao-simulada-novatech.md
│   ├── anexo-b-chunks-referencia-rag.md
│   ├── anexo-c-estrutura-repositorio.md
│   ├── FAQ-atendimento.md
│   ├── POL-001-politica-devolucao.md
│   ├── PROC-042-frete-especial-v1.md
│   ├── PROC-042-v2-frete-especial-revisado.md
│   └── SLA-2024-tabela-sla-clientes.md
│
└── 📁 qa-exercicios/                       ← Entregáveis do papel QA (Cenário 3)
    ├── QA-exercicio-3.1-revisao-respostas-assistente.md
    └── QA-exercicio-3.2-revisao-testes-ia.md
```

> Os exercícios 1.1–1.3 (Cenário 1) ficam só na branch [`cenario-1`](../../tree/cenario-1); os 2.1–2.3 (Cenário 2) só na [`cenario-2`](../../tree/cenario-2). Cada branch contém apenas os entregáveis da própria fase.

---

## Cenário 3 — Fase de Governança e Validação

Foco: Harness Engineering (HITL e Structured Outputs), Revisão Crítica de Outputs de IA.

### Exercício 3.1 — Revisão Crítica das Respostas do Assistente
`qa-exercicios/QA-exercicio-3.1-revisao-respostas-assistente.md`

Aplicação da rubrica de 4 dimensões (Cenário 1, Exercício 1.2) a 8 respostas do assistente em staging, com avaliação própria antes do Claude, segunda avaliação do Claude, comparação, e relatório de qualidade gerado no Claude Cowork com parecer de go-live.

| Item | Detalhe |
|------|---------|
| Respostas avaliadas | 8 |
| Reprovações identificadas | #6 (assumiu destino não informado) e #8 (respondeu em inglês, violando guardrail de idioma) |
| Score médio | 10,6 / 12 |
| Parecer de go-live | Não recomendado sem corrigir #6 e #8 — com monitoramento reforçado após a correção |

---

### Exercício 3.2 — Revisão Crítica dos Testes Gerados por IA
`qa-exercicios/QA-exercicio-3.2-revisao-testes-ia.md`

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
| `anexo-c-estrutura-repositorio.md` | Estrutura do repositório `novatech-assistant` (specs, skills, MCP, Vitest) |
| `POL-001-politica-devolucao.md` | Regras de devolução, exceções e carga perigosa |
| `PROC-042-frete-especial-v1.md` | Multiplicadores de frete — versão original (desatualizada) |
| `PROC-042-v2-frete-especial-revisado.md` | Multiplicadores de frete — versão vigente |
| `SLA-2024-tabela-sla-clientes.md` | SLAs por tier (Gold, Silver, Standard) |
| `FAQ-atendimento.md` | Conhecimento informal — usar com cautela |
