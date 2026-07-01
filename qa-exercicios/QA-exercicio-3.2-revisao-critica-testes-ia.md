# QA — Exercício 3.2: Revisão Crítica dos Testes Gerados por IA

**Projeto:** NovaTech — Assistente de IA para Atendimento
**Papel:** QA
**Fase:** Cenário-Âncora 3 — Governança e Validação
**Tópico:** Revisão Crítica de Outputs de IA
**Ferramentas utilizadas:** Claude (chat)
**Referências:** Anexo C (o projeto usa **Vitest**, não Jest), AGENTS.md (Cenário 2), rubrica de Testing Standards TS-01 a TS-04 (Exercício QA 2.1)

---

## Contexto

O Copilot gerou 3 testes de integração para o `query endpoint` e o `feedback endpoint`. Antes de aceitá-los no repositório, revisei se eles realmente testam o que deveriam — ou se dão falsa sensação de segurança.

---

## Parte 1 — Minha revisão (antes de usar o Claude)

### Teste 1 — `query endpoint` / "should return a response"

```typescript
describe('query endpoint', () => {
  it('should return a response', async () => {
    const res = await request(app).post('/api/query').send({ question: 'prazo devolução' });
    expect(res.status).toBe(200);
    expect(res.body).toBeDefined();
  });
});
```

**O que testa:** Que o endpoint responde com HTTP 200 e que `res.body` não é `undefined`.

**O que falha em testar:** O conteúdo da resposta. `toBeDefined()` passa para `{}`, para `{ answer: '' }`, ou até para `{ answer: 'O prazo é 30 dias' }` — um valor **errado**. O teste não verifica se a resposta contém "7 dias úteis" nem se cita `POL-001`.

**Risco se o teste "passar" mas o código estiver errado:** Alto. Se um refactor futuro quebrar o retriever e o assistente passar a responder com informação errada ou vazia, este teste continua verde. Ele dá falsa confiança de que o comportamento de negócio está correto — quando na verdade só verifica que o servidor não caiu.

**Violação:** Mesma falha do TS-02 (Testing Standards, Exercício 2.1) — `toBeDefined()` como única assertion de conteúdo.

---

### Teste 2 — `query endpoint edge cases` / "should handle empty question"

```typescript
describe('query endpoint edge cases', () => {
  it('should handle empty question', async () => {
    const res = await request(app).post('/api/query').send({ question: '' });
    expect(res.status).toBe(400);
  });
});
```

**O que testa:** Que uma pergunta vazia retorna HTTP 400. Isso é válido e vale a pena manter.

**O que falha em testar:** É o **único** teste de edge case da suíte — e não exercita nenhum cenário real do domínio NovaTech. Não há teste para: pergunta sobre carga perigosa (guardrail crítico), pergunta sobre tier inexistente (alucinação trap), pergunta sem cobertura na base (gap), pergunta com premissa falsa. Um edge case de input malformado é necessário, mas não é suficiente — a suíte não prova que o pipeline lida corretamente com nenhuma das armadilhas de domínio que sabemos que existem (Anexo A/B).

**Risco:** Médio-alto. A suíte passa a impressão de "cobertura de edge cases" enquanto cobre apenas validação de input, deixando os riscos de negócio (alucinação, inversão de regra, contradição de versão) completamente destestados.

**Violação:** TS-04 (Testing Standards) — dados de teste devem vir do domínio real da NovaTech; aqui o único dado de teste é uma string vazia, que não é um dado de domínio.

---

### Teste 3 — `feedback endpoint` / "should save feedback"

```typescript
describe('feedback endpoint', () => {
  it('should save feedback', async () => {
    const mockCreate = jest.fn().mockResolvedValue({ id: '123' });
    const res = await request(app).post('/api/feedback').send({
      queryId: 'q1', rating: 5, comment: 'great'
    });
    expect(res.status).toBe(200);
    expect(mockCreate).toHaveBeenCalled();
  });
});
```

**O que testa (aparentemente):** Que o endpoint de feedback aceita um payload e retorna 200, e que a função de criação no banco foi chamada.

**O problema real:** `mockCreate` é um `jest.fn()` criado **solto**, sem nenhuma injeção no client Cosmos real usado pelo handler (comparar com o `feedback-handler.ts` do Exercício Dev 3.2, que instancia `CosmosClient` diretamente dentro da função via `require`). Isso significa uma de duas coisas, e as duas são graves:

1. **Se não há `jest.mock('@azure/cosmos', ...)` em nenhum lugar do arquivo** (não mostrado): `mockCreate` nunca é de fato chamado pelo código real, então `expect(mockCreate).toHaveBeenCalled()` deveria falhar — a menos que o teste esteja quebrado/flaky, ou que exista um mock global escondido em outro arquivo de setup, o que é ainda pior porque não fica visível a quem lê o teste.
2. **Se existe um mock global do Cosmos** que sempre resolve com sucesso: o teste nunca prova que os dados foram validados antes de persistir. O handler real (visto no Exercício Dev 3.2) não valida `rating` (aceita `999`), não valida `queryId`, e loga `attendantEmail` (dado pessoal) — **nenhum desses problemas quebraria este teste**, porque o mock sempre retorna sucesso independentemente do que foi enviado.

**Risco:** Crítico. Este é exatamente o padrão "mock que mascara bug": o teste dá 100% de confiança falsa. Um handler completamente sem validação de input passa neste teste com a mesma facilidade que um handler correto.

**Violação:** Testa a implementação (chamada de uma função) e não o comportamento observável (o que foi de fato persistido, e se dados inválidos foram rejeitados).

---

### Ponto de atenção adicional — Jest vs. Vitest

Os três testes usam APIs do **Jest** (`jest.fn()`, `jest.mock`, sintaxe implícita de `describe`/`it` do Jest). Porém, conforme o **Anexo C** (estrutura do repositório) e o `AGENTS.md` do projeto (Cenário 2), o NovaTech Assistant usa **Vitest**, não Jest. Isso não é só estilo:

- `jest.fn()` e `jest.mock()` não existem no ambiente Vitest sem um shim de compatibilidade — o correto seria `vi.fn()` e `vi.mock()`.
- Se esses testes forem commitados como estão, eles provavelmente **nem executam** no pipeline de CI real do projeto, ou dependem de uma configuração de compatibilidade Jest→Vitest que mascara o problema de tooling errado sendo usado pelo Copilot.

Isso reforça um ponto mais amplo: o Copilot gerou os testes com o "conhecimento genérico" mais comum (Jest é mais popular que Vitest), sem checar o `AGENTS.md`/Anexo C do projeto — o mesmo tipo de falha de "ignorar contexto do repositório" identificado no Exercício Dev 3.2 (uso de `require` dinâmico, `console.log` em vez de `pino`).

---

## Parte 2 — Segunda revisão (Claude)

> Pedi ao Claude para revisar os mesmos 3 testes de forma independente, sem mostrar minha análise antes.

| Teste | Avaliação do Claude | Convergência com a minha análise |
|-------|----------------------|-------------------------------------|
| 1 | "`toBeDefined()` é uma assertion de presença, não de correção. O teste não distingue uma resposta certa de uma resposta qualquer." | ✅ Total. |
| 2 | "O teste de input vazio é válido, mas isolado ele testa apenas uma camada de validação de schema, não o comportamento de domínio do assistente." O Claude também sugeriu que faltam testes para pelo menos um caso de guardrail (carga perigosa) e um caso de gap de cobertura. | ✅ Total — o Claude foi um pouco mais específico ao sugerir quais casos de domínio faltam (guardrail de carga perigosa, gap de cobertura), o que eu tinha deixado mais genérico. |
| 3 | "O mock não está claramente conectado ao código sob teste. Mesmo assumindo que exista um mock module-level, o teste não verifica nenhuma regra de negócio (validação de rating, ausência de campos, dados sensíveis) — apenas que uma função mockada foi chamada, o que é quase tautológico." | ✅ Total. O Claude também apontou algo que eu não tinha formalizado: a asserção `expect(mockCreate).toHaveBeenCalled()` é **tautológica** se o mock estiver corretamente instalado — ela testa que o mock existe, não que o comportamento é correto. Concordo e incorporo essa formulação. |
| Jest vs. Vitest | Identificado de forma independente pelo Claude, citando a mesma inconsistência com o Anexo C. | ✅ Total. |

**Comparação honesta:** convergência completa nos 3 problemas centrais e na inconsistência de tooling. A única diferença de valor foi o Claude ter sido mais específico ao indicar *quais* cenários de domínio deveriam ser adicionados ao Teste 2 (guardrail de carga perigosa e gap de cobertura) — vou incorporar essa sugestão na recomendação final. Não houve nenhum ponto em que o Claude discordou da minha análise ou identificou um problema que eu tivesse classificado como "correto" incorretamente.

---

## Parte 3 — Teste 1 Reescrito

> Reescrito em **Vitest** (não Jest), verificando conteúdo real da resposta em vez de apenas existência.

### Antes (problema)

```typescript
describe('query endpoint', () => {
  it('should return a response', async () => {
    const res = await request(app).post('/api/query').send({ question: 'prazo devolução' });
    expect(res.status).toBe(200);
    expect(res.body).toBeDefined();
  });
});
```

### Depois (corrigido)

```typescript
import { describe, it, expect } from 'vitest';
import request from 'supertest';
import { app } from '../app';

describe('POST /api/query', () => {
  it('returns the 7-day return deadline and cites POL-001 for a standard return question', async () => {
    // Arrange
    const query = { question: 'Qual o prazo de devolução?' };

    // Act
    const res = await request(app).post('/api/query').send(query);

    // Assert
    expect(res.status).toBe(200);
    expect(res.body.answer).toContain('7 dias úteis');
    expect(res.body.source_document).toContain('POL-001');
  });
});
```

### O que mudou e por quê

| Mudança | Justificativa |
|---------|----------------|
| `jest.fn()`/sintaxe Jest → Vitest (`import { describe, it, expect } from 'vitest'`) | O projeto usa Vitest (Anexo C / AGENTS.md); usar Jest é uma inconsistência de contexto que pode impedir o teste de rodar no CI real. |
| `it('should return a response')` → `it('returns the 7-day return deadline and cites POL-001...')` | Nome descreve o comportamento exato esperado — falha autoexplicativa (TS-03, Exercício 2.1). |
| `expect(res.body).toBeDefined()` → `expect(res.body.answer).toContain('7 dias úteis')` | Verifica **conteúdo correto**, não apenas presença. Um refactor que quebre o retriever ou devolva o prazo errado agora faz o teste falhar. |
| Adicionado `expect(res.body.source_document).toContain('POL-001')` | Garante que a citação de fonte (guardrail obrigatório, ver Exercício Dev 3.1) está presente e correta — não apenas que existe uma resposta. |
| Estrutura AAA explícita (`// Arrange`, `// Act`, `// Assert`) | Alinhado ao TS-01 do AGENTS.md (Exercício 2.1). |

---

## Resumo dos problemas identificados

| Teste | Problema | Severidade |
|-------|----------|------------|
| 1 | Assertions vagas (`toBeDefined()`) — não verifica correção do conteúdo | Alta |
| 2 | Dados de teste irreais — nenhum caso exercita o domínio NovaTech (guardrails, alucinação, gaps) | Média-alta |
| 3 | Mock desconectado/permissivo que mascara ausência de validação de input | Crítica |
| Transversal | Testes escritos em Jest num projeto que usa Vitest — inconsistência de contexto | Média (bloqueia execução real em CI) |

**Recomendação para o Tech Lead:** nenhum dos 3 testes deve ser mergeado como está. O Teste 1 tem correção pronta acima. Os Testes 2 e 3 precisam ser reescritos do zero: o 2 incorporando ao menos um caso de guardrail de domínio (carga perigosa) e um caso de gap de cobertura; o 3 substituindo o mock solto por um mock de módulo real (`vi.mock('@azure/cosmos', ...)`) que permita verificar o payload efetivamente persistido, incluindo um caso negativo onde o payload inválido (ex: `rating: 999`) é rejeitado antes de chegar ao banco.

---

*Documento produzido para o Exercício QA 3.2 — Cenário-Âncora 3, NovaTech.*
