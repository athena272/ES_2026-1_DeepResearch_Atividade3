# Parte 6 — Eixo D: Plano de evolução da qualidade

> Parte mais importante da nota. Base: achados das Partes 1–5.

**Responsável:** Alícia Vitória Sousa Santos

**Projeto analisado:** [Alibaba-NLP/DeepResearch](https://github.com/Alibaba-NLP/DeepResearch)

---

## Contexto do plano

As Partes 1–5 convergem num diagnóstico único: o DeepResearch **não possui suíte de testes de software**, **não tem CI de qualidade**, **depende de APIs externas sem isolamento** e concentra risco no `inference/react_agent.py` e nas ferramentas `tool_*`. O plano abaixo é **priorizado, incremental e tecnicamente fundamentado** nos achados — não é uma lista genérica de boas práticas.

---

## 1. Priorização (Top 3 prioridades de teste)

### Prioridade 1 — Pipeline mínimo de CI com smoke test e lint

**(a) Evidência base:** Partes 1 e 2 — ausência de `.github/workflows/` para qualidade; única Action é deploy de página; PRs mergeados sem verificação (A2, "aprovado seco").

**(b) Objetivo:** Estabelecer a primeira barreira automatizada antes de qualquer merge na `main`.

**(c) Escopo:** Criar `.github/workflows/ci.yml` com: (1) `pip install -r requirements.txt`; (2) smoke test — importação de `inference.react_agent` e módulos críticos; (3) `ruff check inference/` (erros críticos apenas).

**(d) Indicador de sucesso:** Todo PR bloqueado se smoke test ou lint falhar; tempo de execução < 5 minutos.

**(e) Risco se não fizer:** **Alto** — regressões silenciosas continuam (Partes 2, 4, 5).

---

### Prioridade 2 — Abstrações e fakes para LLM e APIs externas

**(a) Evidência base:** Partes 3 e 5 — ~26 arquivos com `OpenAI(...)` direto; `tool_search.py`, `tool_visit.py`, `tool_python.py` sem injeção de dependência.

**(b) Objetivo:** Tornar o núcleo testável sem rede, custo ou tokens.

**(c) Escopo:** Interface `LLMClient` + `FakeLLMClient`; adaptadores `SearchProvider`, `PageReader`, `CodeExecutor` injetáveis nas tools; configuração centralizada de variáveis de ambiente.

**(d) Indicador de sucesso:** Pelo menos um teste unitário por adaptador usando fake, executado no CI, sem chamada HTTP real.

**(e) Risco se não fizer:** **Alto** — impossível escrever testes rápidos; refatoração da A2 permanece inviável.

---

### Prioridade 3 — Characterization tests no `react_agent.py`

**(a) Evidência base:** Partes 1, 4 e 5 — god module sem cobertura; parsing de `<tool_call>`, retries em `call_server` e loop ReAct são regras críticas sem asserção.

**(b) Objetivo:** Congelar o comportamento atual do orquestrador antes de qualquer refatoração estrutural (A2, SRP/DRY).

**(c) Escopo:** 5–10 casos golden em `tests/fixtures/` (entrada de mensagem → parsing esperado de tool_call; payload inválido → exceção; retry respeita `max_tries`) com LLM mockado.

**(d) Indicador de sucesso:** Suíte roda em < 30 s localmente; CI falha se comportamento congelado mudar.

**(e) Risco se não fizer:** **Alto** — refatoração do pior módulo vira aposta (Parte 5).

---

## 2. Camadas de foco (unidade, integração, contratos, regressão)

| Fase | Camada | O quê testar | Quando |
|------|--------|--------------|--------|
| **Fase 1** (semanas 1–2) | **Unidade** | Parsing `<tool_call>`, montagem de payload, retries, validação de entrada das tools | Após Prioridade 2 (fakes disponíveis) |
| **Fase 2** (semanas 3–4) | **Integração** | Agente + tool fake + LLM fake (fluxo ReAct simplificado, 1–2 turnos) | Após characterization tests |
| **Fase 3** (semanas 5–6) | **Contratos** | Interface dos adaptadores (LLM, Search, Visit) — resposta esperada vs. contrato quebrado | Paralelo à Fase 2 |
| **Fase 4** (mês 2+) | **Regressão** | Golden files versionados; CI bloqueia merge se regressão | Quando suíte unitária > 20 testes |
| **Contínuo** | **Benchmark** | Manter `evaluation/` para acurácia do **modelo** (não substituir por teste de software) | Offline / releases |

**Princípio:** investir primeiro onde o custo de falha é maior (`inference/`), depois expandir para subprojetos `WebAgent/*`.

---

## 3. Implantação gradual (sem interromper o desenvolvimento ativo)

### Etapa 0 — Preparação (sem alterar código de produção)

- Adicionar `requirements-dev.txt` com `pytest`, `pytest-cov`, `pytest-mock`, `responses`, `ruff`.
- Criar pasta `tests/` vazia com `conftest.py` base.
- Criar `CONTRIBUTING.md` com seção "Running tests" (Parte 2).

### Etapa 1 — CI mínimo (Prioridade 1)

- Workflow que só importa módulos e roda lint — **zero testes de negócio ainda**.
- Não exige que mantenedores externos parem desenvolvimento; só adiciona gate.

### Etapa 2 — Primeiros testes (Prioridade 2 + 3)

- Introduzir fakes em módulo separado (`inference/adapters/` ou `tests/fakes/`).
- PRs **novos** devem incluir teste quando tocarem `inference/`; código legado ganha testes incrementalmente (não big-bang).

### Etapa 3 — Cobertura e política

- Ativar `pytest-cov` no CI; meta inicial **40% em `inference/`** (Parte 2).
- Regra: PR não pode **reduzir** cobertura global.

### Etapa 4 — Integração e regressão

- Expandir para testes agente + tools com fakes.
- Golden files para parsing e fluxos críticos.

**Estratégia de convivência:** benchmarks em `evaluation/` continuam rodando offline para papers/releases; CI diário roda apenas suíte rápida (< 5 min).

---

## 4. Automação (ferramentas, pipeline e política mínima de execução)

### Ferramentas

| Ferramenta | Função |
|------------|--------|
| `pytest` | Framework de testes |
| `pytest-mock` / `unittest.mock` | Dublês para LLM e HTTP |
| `responses` ou `httpx.MockTransport` | Simular APIs Serper/Jina |
| `pytest-cov` | Medição de cobertura |
| `ruff` | Lint rápido no CI |
| GitHub Actions | Pipeline em PR/push |

### Pipeline proposto (`.github/workflows/ci.yml`)

```
on: [push, pull_request] → branches: [main]
jobs:
  quality:
    - checkout
    - setup-python
    - pip install -r requirements.txt -r requirements-dev.txt
    - ruff check inference/
    - pytest tests/ --cov=inference --cov-report=xml --cov-fail-under=0  # subir limiar progressivamente
    - (opcional) upload coverage → Codecov
```

### Política mínima de execução

1. **Todo PR** para `main` deve passar no CI antes do merge.
2. **Branch protection:** exigir status check `quality` + pelo menos 1 revisão com comentário (endereça A2, "aprovado seco").
3. **Código novo** em `inference/` deve incluir teste unitário correspondente.
4. **Benchmarks** (`evaluation/`) não rodam no CI padrão — apenas em workflow manual (`workflow_dispatch`) ou release.

---

## 5. Critérios de sucesso (entrada e saída de maturidade)

### Critérios de entrada (para iniciar o plano)

- [x] Diagnóstico completo das Partes 1–5 (este repositório).
- [ ] Acordo da equipe sobre Prioridades 1–3.
- [ ] `requirements-dev.txt` e pasta `tests/` criados.

### Critérios de saída — Nível G (maturidade mínima alcançável em 6–8 semanas)

| Critério | Meta | Como medir |
|----------|------|------------|
| CI ativo | Workflow verde em PRs | Aba Actions do fork/PR |
| Smoke + lint | 100% dos PRs verificados | Branch protection ativa |
| Testes unitários | ≥ 15 testes em `tests/` | `pytest --collect-only` |
| Cobertura `inference/` | ≥ 40% | `pytest-cov` report |
| Fakes operacionais | LLM + 1 tool com fake | Testes passam sem rede |
| Documentação | `CONTRIBUTING.md` com "Running tests" | Arquivo no repo |
| Characterization | ≥ 5 casos golden em `react_agent` | Testes no CI |

### Critérios de saída — Nível intermediário (3–6 meses)

- Cobertura `inference/` ≥ 70%.
- Testes de integração agente + tools com fakes.
- Nenhum PR reduz cobertura global.
- Badge de CI e cobertura no README.

### O que **não** conta como avanço (evitar falsa maturidade)

- Renomear `test.sh` sem criar testes reais.
- Rodar `evaluate_*_official.py` no CI como "teste" (mede modelo, não código — Parte 4).
- Adicionar `assert` em código de produção e chamar de teste.
- Badge de CI que só faz echo/hello-world.

---

## Síntese do plano

| Prioridade | Ação | Camada | Prazo sugerido |
|------------|------|--------|----------------|
| 1 | CI smoke + lint | Automação | Semana 1 |
| 2 | Abstrações LLM/API + fakes | Unidade | Semanas 2–3 |
| 3 | Characterization tests `react_agent.py` | Regressão | Semanas 3–4 |
| 4 | Testes unitários tools (`tool_*`) | Unidade | Semanas 4–5 |
| 5 | Cobertura 40% + CONTRIBUTING.md | Processo | Semana 6 |
| 6 | Integração agente + tools (fakes) | Integração | Mês 2 |

**Mensagem final:** o DeepResearch não precisa de uma suíte massiva overnight — precisa de **um gate automatizado mínimo**, **pontos de isolamento** e **testes que protejam o núcleo** antes de repetir os erros de qualidade já diagnosticados nas Atividades 1, 2 e 3.
