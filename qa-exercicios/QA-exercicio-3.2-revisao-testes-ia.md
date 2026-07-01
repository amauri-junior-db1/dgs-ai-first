# QA — Exercício 3.2: Revisão Crítica dos Testes Gerados por IA (Fase de Governança)

**Projeto:** NovaTech — Assistente de IA para Atendimento
**Papel:** QA
**Fase:** Cenário-Âncora 3 — Governança e Validação
**Tópico:** Revisão Crítica de Outputs de IA
**Ferramentas utilizadas:** Claude (chat)
**Referências:** Anexo C — Estrutura do Repositório (framework de teste do projeto: **Vitest**, não Jest); AGENTS.md do cenário 2 (TypeScript strict, Zod, pino para logs, nunca logar dados pessoais).

---

## Parte 1 — Avaliação individual (antes de consultar o Claude)

### Teste 1 — `query endpoint › should return a response`

```typescript
expect(res.status).toBe(200);
expect(res.body).toBeDefined();
```

**O que testa:** apenas que a requisição não falhou (status 200) e que existe *algum* corpo de resposta.

**O que falha em testar:** o conteúdo da resposta. Não verifica se a resposta contém o prazo correto ("7 dias úteis"), se cita a fonte (`source_document`), nem se respeita o formato estruturado esperado (`answer`, `source_document`, `confidence_score`).

**Risco se "passar" com código errado:** um bug que faz o endpoint retornar `{ answer: "não sei" }` com status 200 passaria neste teste. É um **falso positivo de cobertura** — o teste dá a impressão de que o endpoint funciona, mas não verifica a única coisa que importa: a resposta está certa e citada?

**Classificação:** Insuficiente (assertion vaga).

---

### Teste 2 — `query endpoint edge cases › should handle empty question`

```typescript
const res = await request(app).post('/api/query').send({ question: '' });
expect(res.status).toBe(400);
```

**O que testa:** um único edge case de validação de input (string vazia retorna 400).

**O que falha em testar:** nenhuma pergunta real do domínio de logística (prazo, frete especial, SLA, carga perigosa). O "edge case" testado é de input inválido, não de comportamento do assistente diante de perguntas reais — que é o que realmente importa para a NovaTech.

**Risco se "passar" com código errado:** o pipeline de retrieval e geração pode estar completamente quebrado para perguntas reais (ex: sempre retornando "não encontrei"), e essa suíte continuaria passando 100%, porque nenhum teste exercita o caminho principal.

**Classificação:** Incompleto (cobre apenas validação de borda, não o domínio).

---

### Teste 3 — `feedback endpoint › should save feedback`

```typescript
const mockCreate = jest.fn().mockResolvedValue({ id: '123' });
const res = await request(app).post('/api/feedback').send({...});
expect(res.status).toBe(200);
expect(mockCreate).toHaveBeenCalled();
```

**O que testa (aparentemente):** que o feedback é salvo no banco.

**O que realmente testa:** nada relacionado à persistência real. `mockCreate` é uma função solta, **nunca injetada** no handler — o handler real usa `container.items.create(...)` de uma instância própria do `CosmosClient`, que não tem nenhuma relação com esse mock. Ou seja, `mockCreate` nunca será chamado de fato pelo código de produção; o teste está verificando um mock que não está conectado ao *system under test* (SUT).

**Risco se "passar" com código errado:** o teste passa **mesmo que a gravação real no banco falhe silenciosamente**, porque `expect(mockCreate).toHaveBeenCalled()` está testando uma peça de teatro, não o comportamento real. Pior: também não verifica que dados pessoais (`attendantEmail`) não estão sendo logados — um teste que "passa" aqui dá falsa confiança tanto sobre persistência quanto sobre um requisito de segurança/privacidade do AGENTS.md.

**Classificação:** Perigoso — mock desconectado do SUT, gera falso sentido de segurança.

### Ponto de atenção transversal (nos 3 testes)

Os três testes usam a API do **Jest** (`jest.fn()`), mas o Anexo C especifica que o projeto usa **Vitest** como framework de testes. Isso é uma inconsistência com a convenção do repositório (deveria ser `vi.fn()` do módulo `vitest`) — sinal de que o código gerado por IA não seguiu o contexto do projeto (AGENTS.md / Anexo C), e que ninguém rodou os testes de fato antes de considerá-los prontos (senão o erro de import teria aparecido imediatamente).

---

## Parte 2 — Segunda revisão (Claude)

**Prompt utilizado:**
> "Você é um segundo revisor de QA analisando 3 testes de integração gerados por Copilot para um assistente de IA (Azure Functions + TypeScript + Vitest). Aponte, para cada teste, o que ele realmente verifica, o que não verifica, e o risco concreto de um bug passar sem ser detectado. Preste atenção especial a mocks desconectados do código real e a inconsistências de framework de teste."

**Resposta do Claude (síntese, complementando a Parte 1):**

- **Teste 1:** concorda que a assertion é vaga, e acrescenta que também falta um teste do **caminho negativo** — por exemplo, uma pergunta sem resposta na base (ex: "qual o SLA do tier Platinum?") deveria retornar uma resposta explícita de "não encontrado", não silêncio ou erro genérico.
- **Teste 2:** concorda que falta cobertura de domínio, e acrescenta que o teste testa a *rejeição* de input inválido mas não testa nenhum caso de sucesso "de borda" dentro do domínio real (ex: pergunta ambígua que deveria acionar HITL).
- **Teste 3:** confirma o diagnóstico do mock desconectado, e nomeia o padrão de forma mais direta: é um **"vanity test"** — dá a aparência de teste de integração, mas não integra nada. Reforça que a ausência de verificação sobre `attendantEmail` no log é o risco mais grave dos três testes, porque é uma violação de dado pessoal, não só um problema de cobertura.

---

## Parte 3 — Comparação entre avaliação humana (QA) e avaliação do Claude

| Teste | Minha avaliação | Avaliação do Claude | Concordância |
|-------|------------------|----------------------|---------------|
| 1 | Insuficiente (assertion vaga) | Concorda; acrescenta ausência de teste de caminho negativo | ✅ Total, com adição |
| 2 | Incompleto (não exercita o domínio) | Concorda; acrescenta ausência de caso de borda "de sucesso" no domínio (ex: gatilho de HITL) | ✅ Total, com adição |
| 3 | Perigoso (mock desconectado do SUT) | Concorda; classifica como "vanity test" e destaca a falha de privacidade como o risco mais grave | ✅ Total, com adição |
| Transversal | Jest usado onde deveria ser Vitest | Não apontado espontaneamente — precisei perguntar diretamente sobre inconsistência de framework | ⚠️ Parcial — esse ponto exigiu direcionamento explícito no prompt |

**Honestidade da comparação:** o Claude não discordou de nenhuma classificação, mas agregou valor real em dois pontos (caminho negativo do Teste 1, e nomear com precisão o risco de privacidade do Teste 3). O único ponto onde a minha leitura inicial foi mais direta que a do Claude foi a inconsistência Jest/Vitest — o Claude só a reforçou depois de eu perguntar especificamente sobre isso, o que sugere que **inconsistências de convenção de projeto (não de lógica) são mais fáceis de passar batido para um revisor de IA sem contexto explícito do repositório.**

---

## Parte 4 — Teste 1 reescrito

Versão que verifica o **conteúdo** da resposta (não apenas sua existência), o formato estruturado esperado e a citação de fonte — usando `vitest` (framework correto do projeto) em vez de `jest`:

```typescript
// query.test.ts — Teste 1 reescrito (Vitest)
import { describe, it, expect } from 'vitest';
import request from 'supertest';
import { app } from '../src/app';

describe('query endpoint', () => {
  it('deve responder o prazo de devolução correto, no formato estruturado, citando a fonte', async () => {
    const res = await request(app)
      .post('/api/query')
      .send({ question: 'Qual o prazo de devolução para produtos standard?' });

    expect(res.status).toBe(200);

    // valida o formato estruturado obrigatório (schema: answer, source_document, confidence_score)
    expect(res.body).toHaveProperty('answer');
    expect(res.body).toHaveProperty('source_document');
    expect(res.body).toHaveProperty('confidence_score');

    // valida o CONTEÚDO da resposta — não apenas que "existe algo"
    expect(res.body.answer).toMatch(/7\s*dias\s*úteis/i);
    expect(res.body.source_document).toBe('POL-001');
    expect(['Alta', 'Média']).toContain(res.body.confidence_score);
  });

  it('deve reconhecer explicitamente quando não encontra a resposta na base', async () => {
    const res = await request(app)
      .post('/api/query')
      .send({ question: 'Qual o SLA de resposta para o tier Platinum?' });

    expect(res.status).toBe(200);
    expect(res.body.answer).toMatch(/não (existe|encontrei|há)/i);
    expect(res.body.confidence_score).toBe('Baixa');
  });
});
```

A segunda `it` foi incluída atendendo à sugestão do Claude na Parte 2 (cobrir o caminho negativo além do caminho feliz).

---

*Documento produzido para o Exercício QA 3.2 — Cenário-Âncora 3, NovaTech.*
