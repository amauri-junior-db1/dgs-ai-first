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
├── exercicio-fase-2-estruturacao.md        ← Enunciado — Cenário 2
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
└── 📁 qa-exercicios/                       ← Entregáveis do papel QA (Cenário 2)
    ├── QA-exercicio-2.1-testing-standards-agents-md.md
    ├── QA-exercicio-2.2-spec-sdd-query-endpoint.md
    └── QA-exercicio-2.3-skill-create-integration-test.md
```

> Os exercícios 1.1–1.3 (Cenário 1) ficam só na branch [`cenario-1`](../../tree/cenario-1); os 3.1–3.2 (Cenário 3) só na [`cenario-3`](../../tree/cenario-3). Cada branch contém apenas os entregáveis da própria fase.

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
