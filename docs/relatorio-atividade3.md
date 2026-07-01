# Atividade Avaliativa 3 — Auditoria de Testes de Software e Plano de Evolução da Qualidade

**Disciplina:** Engenharia de Software (COMP0503)

**Instituição:** UFS

**Turma / semestre:** T02 — 2026.2

**Data de elaboração deste documento:** 30/06/2026

**Projeto analisado:** **DeepResearch** (Tongyi DeepResearch)

**Repositório oficial:** https://github.com/Alibaba-NLP/DeepResearch

**Repositório de trabalho da equipe (artefatos da auditoria):** https://github.com/athena272/ES_2026-1_DeepResearch_Atividade3

**Atividades anteriores:** A1 — https://github.com/athena272/ES_2026-2_DeepResearch · A2 — https://github.com/athena272/ES_2026-2_DeepResearch_Atividade2

**Vídeo de auditoria técnica:** _(link a definir — inserir URL do YouTube/Drive antes da entrega)_

---

## Identificação da equipe (Equipe 1)

| # | Nome | Matrícula | Contribuição individual (Atividade 3) |
|---|------|-----------|----------------------------------------|
| 01 | Guilherme Rosário Alves | 202100022784 | **Parte 4** — Eixo B: qualidade técnica dos testes; dossiê de evidências (`assets/parte4/`); consolidação e finalização deste relatório. |
| 02 | Alícia Vitória Sousa Santos | 202300027015 | **Parte 6** — Eixo D: plano de evolução da qualidade (Top 3, camadas, automação, critérios de sucesso). |
| 03 | Uilson Alves dos Santos Neto | 201900115954 | **Parte 1** — Eixo A: estrutura dedicada e tipos de teste (unitário, integração, e2e, regressão, manual, benchmark). |
| 04 | Brenno Phelipe Silva dos Santos | 202400050750 | **Parte 5** — Eixo C: lacunas e riscos em módulos críticos (`tool_*`, `react_agent.py`, regressões, MTBD). |
| 05 | Davi Emanuel de Menezes Costa | 202300027178 | **Parte 3** — Eixo C: dependências externas (LLM/API) sem isolamento para teste; mapa de acoplamento. |
| 06 | Alisson Francisco dos Santos | 202300083248 | **Parte 2** — Eixo A: CI/CD, automação, cobertura e documentação para rodar testes. |

_Revisão conjunta do documento e do repositório antes da entrega em PDF._

---

## 1. Introdução e contextualização do sistema

### 1.1 Objetivo

Este relatório apresenta a **Atividade 3** da Equipe 1: uma **auditoria da estratégia de testes** do projeto open source **DeepResearch**, aprofundando achados das **Atividades 1 e 2** com foco em existência, qualidade, cobertura e relevância dos testes automatizados e/ou manuais, identificação de riscos e lacunas, e **plano concreto de evolução da qualidade**.

A investigação segue quatro eixos. **Cada item** é apresentado no formato: **(a) Evidência · (b) Diagnóstico · (c) Risco · (d) Recomendação**.

### 1.2 Contextualização do sistema

O **DeepResearch** é um agente de pesquisa profunda baseado em LLM, desenvolvido pela Alibaba-NLP. O monorepo concentra o núcleo de inferência em `inference/` (agente ReAct, ferramentas de busca, visita e execução Python), scripts de avaliação em `evaluation/` e subprojetos em `WebAgent/`. Nas Atividades 1 e 2, a equipe já havia mapeado rastreabilidade fraca, gate de PRs frágil ("aprovado seco"), violações SOLID/GoF e dívida técnica no `inference/react_agent.py`. Esta auditoria verifica se o projeto possui rede de segurança automatizada capaz de sustentar evolução segura — e conclui que **não possui**.

### 1.3 Achado transversal

O projeto **não possui suíte de testes automatizada de software**. O que existe são scripts de benchmark, avaliadores de acurácia por LLM-como-juiz e funções com prefixo `test_` que são execuções manuais. Não há CI de qualidade, cobertura, mocks ou documentação de testes. A qualidade depende exclusivamente de revisão humana inconsistente — cenário que as seis partes desta auditoria confirmam de ângulos complementares.

---

## 2. Eixo A — Estratégia atual de testes

### 2.1 Estrutura e tipos de teste _(Parte 1 — Uilson Alves dos Santos Neto)_

Documento completo: [parte1-estrategia-estrutura.md](parte1-estrategia-estrutura.md)

#### 2.1.1 Estrutura dedicada a testes

**(a) Evidência:** Não existe pasta `tests/`, `test/` ou `__tests__/`; nenhum `test_*.py` ou `*_test.py`; nenhum `pytest`/`unittest`, `pytest.ini`, `conftest.py` ou `.github/workflows/`. O único arquivo com "test" no nome é `WebAgent/WebSailor/src/scripts/test.sh`, que executa benchmark GAIA. Em vez de suíte, existem `evaluation/` (benchmarks) e `eval_data/` (`.jsonl` para inferência manual).

**(b) Diagnóstico:** O repositório não adota convenção dedicada a testes de software; a verificação está deslocada para avaliação de modelo e execução manual.

**(c) Risco:** **Alto** — alterações em `react_agent.py`, tools ou clientes LLM entram sem detecção automatizada.

**(d) Recomendação:** Criar `tests/` com smoke tests; workflow CI com `pytest`; renomear/documentar `test.sh` como benchmark.

#### 2.1.2 Tipos de teste encontrados

| Tipo | Encontrado? | Evidência resumida |
|------|-------------|-------------------|
| Unitário | Não | Sem `test_*.py` nem framework |
| Integração | Não | Sem pass/fail automatizado agente + tool + LLM |
| Sistema | Não | Sem critérios no código por subsistema |
| E2E | Parcial | Inferência real (`run_multi_react.py`, `eval.sh`), sem asserções |
| Regressão | Não | Sem golden files, snapshots ou CI bloqueante |
| Manual | Sim | README, `run_demo.sh`, `if __name__ == "__main__"` em tools |
| Benchmark | Sim (complementar) | `evaluation/evaluate_*_official.py` — mede modelo, não código |

**(b) Diagnóstico:** Unitário, integração, sistema e regressão ausentes; E2E existe como experimento; manual é o mecanismo dominante; benchmark gera falsa sensação de cobertura.

**(c) Risco:** **Alto** (evolução de código); **médio** (só hiperparâmetros de modelo).

**(d) Recomendação:** Testes para parsing `<tool_call>`, retries e contrato das tools; golden files em `tests/fixtures/`; manter benchmarks separados do gate de qualidade de código.

### 2.2 CI/CD, automação e cobertura _(Parte 2 — Alisson Francisco dos Santos)_

Documento completo: [parte2-cicd-cobertura.md](parte2-cicd-cobertura.md)

#### 2.2.1 Integração com CI/CD

**(a) Evidência:** `git ls-files ".github/*"` vazio; aba Actions mostra apenas `pages-build-deployment` (deploy estático), sem testes ou lint.

**(b) Diagnóstico:** Nenhum PR é verificado automaticamente antes do merge.

**(c) Risco:** **Alto**

**(d) Recomendação:** `.github/workflows/ci.yml` com dependências, smoke test e `ruff`.

#### 2.2.2 Cobertura, badges e ferramentas

**(a) Evidência:** Sem `codecov`, `.coveragerc`, `tox`, `pyproject.toml`, lint config; README sem badges de CI/cobertura.

**(b) Diagnóstico:** Nenhum contrato de qualidade estabelecido.

**(c) Risco:** **Alto**

**(d) Recomendação:** Cobertura no CI quando suíte existir; badge; meta 40% em `inference/`.

#### 2.2.3 Instruções para rodar testes

**(a) Evidência:** README sem menção a testes; sem `CONTRIBUTING.md`.

**(b) Diagnóstico:** Colaboradores sem caminho documentado para validar alterações.

**(c) Risco:** **Médio**

**(d) Recomendação:** `CONTRIBUTING.md` com seção "Running tests".

**Síntese — Eixo A:**

| # | Item | Veredito | Risco |
|---|------|----------|-------|
| 1 | Estrutura e tipos | Sem suíte; só benchmark/manual | Alto |
| 2 | CI em PRs/pushes | Inexistente (só deploy de página) | Alto |
| 3 | Cobertura, badges, ferramentas | Inexistentes | Alto |
| 4 | Documentação para rodar testes | Inexistente | Médio |

**Veredito geral do Eixo A:** zero automação de qualidade de código; validação deslocada para benchmark de modelo e execução manual; gate humano frágil (confirma A2 — GPR "aprovado seco").

---

## 3. Eixo B — Qualidade técnica dos testes

_(Parte 4 — Guilherme Rosário Alves. Documento: [parte4-qualidade-tecnica.md](parte4-qualidade-tecnica.md) · evidências: [assets/parte4/](../assets/parte4/README.md))_

**Achado central.** O `DeepResearch` não possui suíte de testes automatizada. Este eixo submete os artefatos com rótulo "test" aos cinco critérios de qualidade e demonstra que **nenhum cumpre a função de um teste de software**.

Artefatos avaliados:

1. `WebAgent/WebSailor/src/scripts/test.sh` — benchmark, não teste.
2. `test_llm_call` / `test_audio_output_call` (L116-175) — scripts manuais, LLM ao vivo, sem `assert`.
3. `evaluation/evaluate_*_official.py` — LLM-como-juiz (não-determinístico).

| # | Critério (Eixo B) | Veredito | Risco |
|---|-------------------|----------|-------|
| 4.1 | Nomes claros / comportamento observável | Nomes enganosos | Médio |
| 4.2 | Isolamento, independência, reprodutibilidade | API/rede/chaves; não-determinístico | Alto |
| 4.3 | Mocks/stubs/fixtures/dados de teste | Sem dublês; dados são benchmarks | Alto |
| 4.4 | Validação de regras vs. superficial | Sem asserções; avaliam o modelo | Alto |
| 4.5 | Redundantes, frágeis, difíceis de manter | Código morto; host fixo | Médio |

**Veredito geral:** não há qualidade técnica de testes a medir — apenas scripts de execução e avaliadores de modelo.

---

## 4. Eixo C — Lacunas, riscos e falhas potenciais

### 4.1 Dependências externas sem isolamento _(Parte 3 — Davi Emanuel de Menezes Costa)_

Documento completo: [parte3-dependencias-isolamento.md](parte3-dependencias-isolamento.md)

| Dependência | Local | Isolamento? | Risco |
|-------------|-------|-------------|-------|
| OpenAI / LLM | `react_agent.py`, `agent.py`, `tool_visit.py` | Não | Alto |
| Serper | `tool_search.py` | Não | Alto |
| Jina | `tool_visit.py` | Não | Alto |
| Sandbox Fusion | `tool_python.py` | Não | Alto |
| Variáveis de ambiente | Vários arquivos | Não | Médio |

**Diagnóstico resumido:** ~26 arquivos instanciam `OpenAI(...)` diretamente; tools fazem HTTP sem injeção de dependência; impossível testar sem rede, custo e tokens. Confirma DIP/vendor lock-in da A2 sob a ótica de testabilidade.

**Recomendações-chave:** `LLMClient` + fakes; adaptadores injetáveis; `pytest` + `pytest-mock` + `responses`; política de testes em PRs.

### 4.2 Lacunas e riscos em módulos críticos _(Parte 5 — Brenno Phelipe Silva dos Santos)_

Documento completo: [parte5-lacunas-riscos.md](parte5-lacunas-riscos.md)

| # | Lacuna | Diagnóstico resumido | Risco | Recomendação-chave |
|---|--------|----------------------|-------|-------------------|
| 1 | Funcionalidades críticas (A1) sem testes | `tool_search`, `tool_visit`, `tool_python` descobertos | Alto | Testes por `BaseTool` |
| 2 | Dívida técnica (A2) | `react_agent.py` sem characterization tests | Alto | Congelar comportamento antes de refatorar |
| 3 | Dependências sem isolamento | Benchmarks em produção; sem mocks | Alto | Fakes + adaptadores |
| 4 | Proteção contra regressões | Sem CI + "aprovado seco" | Alto | Smoke test em PR |
| 5 | Detecção rápida de bugs | Suíte = benchmarks lentos com LLM-juiz | Alto | `pytest` local em ms |

**Veredito geral do Eixo C:** sistema sem rede de segurança automatizada; refatorações da A2 inviáveis sem testes prévios.

---

## 5. Eixo D — Plano de evolução da qualidade

_(Parte 6 — Alícia Vitória Sousa Santos. Documento completo: [parte6-plano-evolucao.md](parte6-plano-evolucao.md))_

Plano priorizado, incremental e fundamentado nos achados das Partes 1–5.

### 5.1 Top 3 prioridades

| # | Prioridade | Objetivo | Indicador de sucesso |
|---|------------|----------|----------------------|
| 1 | **CI mínimo** (smoke + lint) | Primeira barreira automatizada em PRs | PR bloqueado se falhar; execução < 5 min |
| 2 | **Abstrações e fakes** (LLM/API) | Testabilidade sem rede/custo | ≥ 1 teste unitário por adaptador no CI |
| 3 | **Characterization tests** (`react_agent.py`) | Proteger núcleo antes de refatorar | 5–10 casos golden; suíte < 30 s |

### 5.2 Camadas de foco

1. **Unidade** (semanas 1–2): parsing `<tool_call>`, payload, retries, tools.
2. **Integração** (semanas 3–4): agente + tool fake + LLM fake.
3. **Contratos** (semanas 5–6): interfaces dos adaptadores.
4. **Regressão** (mês 2+): golden files; CI bloqueante.
5. **Benchmark** (contínuo, offline): manter `evaluation/` para acurácia do **modelo**, separado do gate de **código**.

### 5.3 Implantação gradual

- **Etapa 0:** `requirements-dev.txt`, pasta `tests/`, `CONTRIBUTING.md`.
- **Etapa 1:** CI que só importa módulos + lint (Prioridade 1).
- **Etapa 2:** Fakes + primeiros testes (Prioridades 2 e 3); PRs novos em `inference/` exigem teste.
- **Etapa 3:** Cobertura 40% em `inference/`; PR não reduz cobertura global.
- **Etapa 4:** Integração agente + tools; golden files.

Benchmarks acadêmicos continuam offline; CI diário roda suíte rápida (< 5 min).

### 5.4 Automação

**Ferramentas:** `pytest`, `pytest-mock`, `responses`, `pytest-cov`, `ruff`, GitHub Actions.

**Política mínima:**

1. Todo PR para `main` passa no CI antes do merge.
2. Branch protection: status check `quality` + revisão com comentário (endereça "aprovado seco" da A2).
3. Código novo em `inference/` inclui teste unitário.
4. Benchmarks não rodam no CI padrão (apenas `workflow_dispatch`/release).

### 5.5 Critérios de sucesso (Nível G — 6–8 semanas)

| Critério | Meta |
|----------|------|
| CI ativo | Workflow verde em PRs |
| Smoke + lint | 100% dos PRs verificados |
| Testes unitários | ≥ 15 testes em `tests/` |
| Cobertura `inference/` | ≥ 40% |
| Fakes operacionais | LLM + 1 tool sem rede |
| Documentação | `CONTRIBUTING.md` com "Running tests" |
| Characterization | ≥ 5 casos golden em `react_agent` |

**O que não conta como avanço:** renomear `test.sh` sem testes reais; rodar `evaluate_*` no CI como teste de software; badge de CI hello-world.

---

## 6. Síntese final e prioridades de evolução

### 6.1 Diagnóstico consolidado

| Eixo | Achado principal | Risco dominante |
|------|------------------|-----------------|
| A — Estratégia | Sem estrutura de testes, sem CI de qualidade, sem docs | Alto |
| B — Qualidade técnica | Artefatos "test" não são testes; zero asserções | Alto |
| C — Lacunas/riscos | Core sem cobertura; acoplamento a APIs externas | Alto |
| D — Evolução | Plano incremental: CI → fakes → characterization → cobertura | — |

### 6.2 Conclusão

A auditoria das seis partes confirma que o **DeepResearch opera sem qualquer garantia automatizada de qualidade de software**. O projeto mistura benchmark de modelo, execução manual e nomenclatura enganosa ("test") com testes reais — gerando falsa sensação de cobertura. Isso transforma a dívida arquitetural mapeada nas Atividades 1 e 2 (SRP/DRY/DIP, gate de PR frágil, ausência de `CONTRIBUTING.md`) em **risco operacional imediato**: qualquer alteração no núcleo `inference/` é uma aposta.

O plano de evolução (Eixo D) propõe um caminho **realista e incremental**: começar por um CI mínimo (custo < 1 dia), introduzir abstrações testáveis (pré-requisito técnico), congelar o comportamento do `react_agent.py` com characterization tests, e só então expandir cobertura e integração — sem interromper o desenvolvimento ativo nem confundir benchmark acadêmico com teste de software.

### 6.3 Continuidade com A1 e A2

| Achado A1/A2 | Confirmação A3 |
|--------------|----------------|
| Ausência de CI (A1 V&V) | Parte 2: zero workflow de qualidade |
| Sem `CONTRIBUTING.md` (A1 GQA) | Partes 1 e 2: sem docs de teste |
| "Aprovado seco" (A2 GPR) | Partes 2 e 5: sem gate automatizado |
| SRP/DRY/DIP em `inference/` (A2 SOLID) | Partes 3, 4, 5: impossível testar/refatorar |
| Plano de resgate MPS.BR G (A2) | Parte 6: roadmap concreto de testes |

---

## 7. Contribuição dos integrantes

| Integrante | Parte | Entrega |
|------------|-------|---------|
| Uilson Alves dos Santos Neto | 1 | Eixo A: estrutura dedicada e tipos de teste |
| Alisson Francisco dos Santos | 2 | Eixo A: CI/CD, cobertura, documentação |
| Davi Emanuel de Menezes Costa | 3 | Eixo C: dependências externas e isolamento |
| Guilherme Rosário Alves | 4 | Eixo B: qualidade técnica + dossiê `assets/parte4/` + consolidação/finalização deste relatório |
| Brenno Phelipe Silva dos Santos | 5 | Eixo C: lacunas e riscos em módulos críticos |
| Alícia Vitória Sousa Santos | 6 | Eixo D: plano de evolução da qualidade |

**Documentos individuais:** [parte1](parte1-estrategia-estrutura.md) · [parte2](parte2-cicd-cobertura.md) · [parte3](parte3-dependencias-isolamento.md) · [parte4](parte4-qualidade-tecnica.md) · [parte5](parte5-lacunas-riscos.md) · [parte6](parte6-plano-evolucao.md)

**Evidências visuais:** [assets/parte4/](../assets/parte4/README.md)

_Revisão conjunta do documento e do repositório antes da entrega em PDF._
