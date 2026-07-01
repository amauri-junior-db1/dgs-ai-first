# dgs-ai-first — Trilha de Certificação AI First | Papel: QA

Repositório dos entregáveis do papel **QA** na Trilha de Formação para Certificação AI First da DGS.

O cenário-âncora é a **NovaTech** — empresa de logística que está construindo um assistente de atendimento ao cliente baseado em IA com pipeline RAG sobre documentação interna.

---

## Organização do repositório

Esta branch (`main`) contém **apenas a documentação de referência** do projeto — a fonte de verdade usada em todos os exercícios. Os entregáveis de cada fase ficam em branches separadas:

| Branch | Fase | Prazo |
|--------|------|-------|
| [`cenario-1`](../../tree/cenario-1) | Fase de Entendimento e Contexto | 06/06 |
| [`cenario-2`](../../tree/cenario-2) | Fase de Estruturação do Trabalho | 18/06 |
| [`cenario-3`](../../tree/cenario-3) | Fase de Governança e Validação | 27/06 |

---

## Estrutura de pastas

```
dgs-ai-first/ (main)
├── README.md
│
└── 📁 documentacao-novatech/               ← Fonte de verdade do projeto
    ├── anexo-a-documentacao-simulada-novatech.md
    ├── anexo-b-chunks-referencia-rag.md
    ├── anexo-c-estrutura-repositorio.md
    ├── FAQ-atendimento.md
    ├── POL-001-politica-devolucao.md
    ├── PROC-042-frete-especial-v1.md
    ├── PROC-042-v2-frete-especial-revisado.md
    └── SLA-2024-tabela-sla-clientes.md
```

---

## Documentação de referência

| Arquivo | Uso principal |
|---------|---------------|
| `anexo-a-documentacao-simulada-novatech.md` | Fonte de verdade — avaliação de respostas |
| `anexo-b-chunks-referencia-rag.md` | Gabarito de retrieval (mapa de cobertura RAG) |
| `anexo-c-estrutura-repositorio.md` | Estrutura do repositório `novatech-assistant` (usada a partir do Cenário 2) |
| `POL-001-politica-devolucao.md` | Regras de devolução, exceções e carga perigosa |
| `PROC-042-frete-especial-v1.md` | Multiplicadores de frete — versão original (desatualizada) |
| `PROC-042-v2-frete-especial-revisado.md` | Multiplicadores de frete — versão vigente |
| `SLA-2024-tabela-sla-clientes.md` | SLAs por tier (Gold, Silver, Standard) |
| `FAQ-atendimento.md` | Conhecimento informal — usar com cautela |
