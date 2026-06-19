# QA — Exercício 2.1: Testing Standards para o AGENTS.md

## Contexto

Este documento define a seção **Testing Standards** a ser incluída no `AGENTS.md` do projeto NovaTech Assistant. Qualquer agente (Copilot, Claude Code, etc.) que leia esta seção antes de gerar testes deve produzir código melhor do que o teste ruim fornecido como referência.

---

## Seção a inserir no AGENTS.md

```markdown
## Testing Standards

> Machine-readable section. Agents generating tests for this project MUST read and follow
> these standards in full before writing any test code. Deviating from these rules is not
> acceptable — rewrite until compliant.

### TS-01 — Structure: Arrange / Act / Assert (AAA)

Every test MUST be structured in three explicit, labeled sections.

MUST:
- Include three comment labels: `// Arrange`, `// Act`, `// Assert`
- Place exactly ONE operation (one `await` call) in the Act section
- Set up all inputs and expected values in Arrange, before Act

MUST NOT:
- Skip any of the three labels
- Place assertions before the Assert section
- Execute more than one operation in Act (split into separate tests)

Example:
  it('describes exact behavior', async () => {
    // Arrange
    const query = 'domain-specific input value'
    const expectedValue = 'specific string from documentation'

    // Act
    const response = await functionUnderTest(query)

    // Assert
    expect(response.answer).toContain(expectedValue)
  })

### TS-02 — Assertion Quality

MUST use matchers that verify CONTENT, not existence:
- `.toContain(specificString)` for text in a response
- `.toMatch(/pattern/)` for flexible content matching
- `.toBe(exactValue)` for precise equality
- `.not.toContain(forbiddenString)` for guardrail tests (verify what must NOT appear)

MUST NOT use these as the only assertion:
- `toBeDefined()` — passes for empty string `""`, `null`, or wrong answer
- `toBeTruthy()` — same problem
- `toNotBeNull()` — same problem

Why this matters: a test with only `toBeDefined()` passes even when the assistant
returns the wrong deadline, a hallucinated SLA value, or an empty string.

### TS-03 — Test Naming

MUST:
- `describe` block: exact component name as called in code (e.g., `queryAssistant`)
- `it` block: completes the phrase "it returns / refuses / cites..."
- Be specific enough that the failure message is self-documenting without reading the body

MUST NOT:
- Use generic names: `'test query'`, `'works'`, `'test 1'`, `'it works'`
- Repeat the function name without describing behavior: `it('queryAssistant')`
- Use names that require reading the test body to understand what is tested

### TS-04 — Domain Data

MUST use realistic NovaTech domain data in inputs and expected values:
- Return policy: "7 dias úteis", "POL-001", ANTT classes 1–6 for dangerous cargo
- SLA: tiers Gold / Silver / Standard, specific values from SLA-2024
- Freight: multipliers from PROC-042-v2 (Norte: 1.8, Sudeste: 1.1, Nordeste: 1.5)

MUST NOT use placeholder data:
- Inputs: `'test'`, `'hello'`, `'foo'`, `'query'`
- Expected values: invented deadlines, SLA values not in SLA-2024, multipliers not in PROC-042-v2
```

---

## Teste ruim fornecido (BEFORE)

```javascript
test('test query', async () => {
  const result = await queryAssistant('return policy')
  expect(result).toBeDefined()
})
```

### Problemas identificados

| Problema | Regra violada |
|----------|---------------|
| `'test query'` é genérico — falha não diz o que quebrou | TS-03 |
| `toBeDefined()` como única assertion — passa para string vazia ou resposta errada | TS-02 |
| Sem comentários `// Arrange`, `// Act`, `// Assert` | TS-01 |
| `'return policy'` é placeholder, não query de domínio NovaTech | TS-04 |

---

## Teste reescrito (AFTER)

```javascript
describe('queryAssistant', () => {
  it('refuses standard return process for dangerous cargo and cites POL-001 section 3.2', async () => {
    // Arrange
    const query = 'Posso devolver carga perigosa classe 3 (líquidos inflamáveis)?'
    const forbiddenDeadline = '7 dias úteis'

    // Act
    const response = await queryAssistant(query)

    // Assert
    expect(response.answer).toMatch(/não.*elegível|não.*processo padrão/i)
    expect(response.answer).not.toContain(forbiddenDeadline)  // guardrail: prazo geral NÃO se aplica
    expect(response.source).toContain('POL-001')
  })
})
```

### Cada melhoria explicada

1. **TS-03** — `it('refuses standard return...')` descreve o comportamento exato; a mensagem de falha é autoexplicativa
2. **TS-02** — `.toMatch(/não.*elegível|não.*processo padrão/i)` verifica conteúdo específico da resposta
3. **TS-02** — `.not.toContain(forbiddenDeadline)` verifica que o guardrail não foi invertido (carga perigosa não recebe prazo geral)
4. **TS-02** — `.toContain('POL-001')` verifica citação de fonte obrigatória
5. **TS-01** — três labels explícitos: Arrange / Act / Assert
6. **TS-04** — query usa terminologia ANTT real; `forbiddenDeadline` usa valor exato da POL-001

---

## Critérios de review (3 critérios objetivos)

Um teste **passa** a revisão de QA se e somente se:

**RC-01 — AAA labeled**
O corpo do teste contém exatamente três labels (`// Arrange`, `// Act`, `// Assert`) e a seção Act contém exatamente uma operação (um único `await`).
> Verificação: contar ocorrências dos comentários e do `await` — resultado determinístico.

**RC-02 — Behavioral assertion**
Ao menos uma assertion usa `.toContain()`, `.toMatch()`, `.toBe()`, ou equivalente que verifique **conteúdo**. Um teste que usa apenas `toBeDefined()`, `toBeTruthy()` ou `toNotBeNull()` **falha** este critério, independente de outras assertions.
> Verificação: grep por `toBeDefined\(\)` como única assertion no bloco Assert.

**RC-03 — Descriptive `it` block**
Um segundo QA Engineer, lendo apenas a string do `it(...)` (sem o corpo do teste), consegue reproduzir o mesmo teste. Se a descrição exige leitura do corpo para entender o que está sendo testado, ela **falha** este critério.
> Verificação: dois revisores leem a descrição e escrevem o teste independentemente — se chegam ao mesmo resultado, passa.
