# Atividade Avaliativa 3 — Auditoria de Testes de Software e Plano de Evolução da Qualidade

**Disciplina:** Engenharia de Software (COMP0503)

**Instituição:** UFS

**Turma / semestre:** T02 — 2026.2

**Data de elaboração deste documento:** 30/06/2026

**Projeto analisado:** **DeepResearch** (Tongyi DeepResearch)

**Repositório oficial:** https://github.com/Alibaba-NLP/DeepResearch

**Repositório de trabalho da equipe (artefatos da auditoria):** https://github.com/athena272/ES_2026-1_DeepResearch_Atividade3

**Atividades anteriores:** A1 — https://github.com/athena272/ES_2026-2_DeepResearch · A2 — https://github.com/athena272/ES_2026-2_DeepResearch_Atividade2

**Vídeo de auditoria técnica:** _(link a definir)_

---

## Identificação da equipe (Equipe 1)

| # | Nome | Matrícula | Contribuição individual (Atividade 3) |
|---|------|-----------|----------------------------------------|
| 01 | Guilherme Rosário Alves | 202100022784 | **Parte 4** — Eixo B: qualidade técnica dos testes; dossiê de evidências reproduzíveis (`assets/parte4/`); consolidação deste relatório. |
| 02 | Alícia Vitória Sousa Santos | 202300027015 | _a definir_ |
| 03 | Uilson Alves dos Santos Neto | 201900115954 | _a definir_ |
| 04 | Brenno Phelipe Silva dos Santos | 202400050750 | **Parte 5** — Eixo C: lacunas e riscos em módulos críticos (`tool_*`, `react_agent.py`, regressões, MTBD). |
| 05 | Davi Emanuel de Menezes Costa | 202300027178 | **Parte 3** — Eixo C: dependências externas (LLM/API) sem isolamento para teste; mapa de acoplamento e recomendações de abstração. |
| 06 | Alisson Francisco dos Santos | 202300083248 | **Parte 2** — Eixo A: CI/CD, automação, cobertura e documentação para rodar testes. |

_Parte 1 (estrutura e tipos de teste) e Parte 6 (plano de evolução) ainda em elaboração._

---

## 1. Introdução e contextualização do sistema

### 1.1 Objetivo

Este relatório apresenta a **Atividade 3** da Equipe 1: uma **auditoria da estratégia de testes** do projeto open source **DeepResearch**, aprofundando achados das **Atividades 1 e 2** com foco em existência, qualidade, cobertura e relevância dos testes automatizados e/ou manuais, identificação de riscos e lacunas, e proposta de evolução da qualidade.

A investigação segue quatro eixos. **Cada item** é apresentado no formato: **(a) Evidência · (b) Diagnóstico · (c) Risco · (d) Recomendação**.

### 1.2 Contextualização do sistema

O **DeepResearch** é um agente de pesquisa profunda baseado em LLM, desenvolvido pela Alibaba-NLP. O monorepo concentra o núcleo de inferência em `inference/` (agente ReAct, ferramentas de busca, visita e execução Python), scripts de avaliação em `evaluation/` e subprojetos em `WebAgent/`. Nas Atividades 1 e 2, a equipe já havia mapeado rastreabilidade fraca, gate de PRs frágil ("aprovado seco"), violações SOLID/GoF e dívida técnica no `inference/react_agent.py`. Esta auditoria verifica se o projeto possui rede de segurança automatizada capaz de sustentar evolução segura — e conclui que **não possui**.

### 1.3 Achado transversal (síntese antecipada)

O projeto **não possui suíte de testes automatizada de software**. O que existe são scripts de benchmark, avaliadores de acurácia por LLM-como-juiz e funções com prefixo `test_` que são execuções manuais. Não há CI de qualidade, cobertura, mocks ou documentação de testes. A qualidade depende exclusivamente de revisão humana inconsistente.

---

## 2. Eixo A — Estratégia atual de testes

### 2.1 Estrutura e tipos de teste _(Parte 1 — pendente)_

A **Parte 1** (estrutura dedicada e tipagem de testes) ainda será entregue. Até lá, achados convergentes das demais partes indicam:

- Não há pasta `tests/` nem convenção `test_*.py` / `*_test.py` para testes de software.
- O único arquivo com "test" no nome é `WebAgent/WebSailor/src/scripts/test.sh`, que executa benchmark (dataset GAIA), não teste automatizado.
- Os scripts em `evaluation/` medem acurácia do modelo (benchmark acadêmico), não corretude do código.

Documento previsto: [parte1-estrategia-estrutura.md](parte1-estrategia-estrutura.md).

### 2.2 CI/CD, automação e cobertura _(Parte 2 — Alisson Francisco dos Santos)_

Documento completo: [parte2-cicd-cobertura.md](parte2-cicd-cobertura.md)

#### 2.2.1 Integração com CI/CD

**(a) Evidência:** `git ls-files ".github/*"` retorna vazio no clone local. Na aba Actions do nota-se apenas o workflow `pages-build-deployment` (deploy de site estático na branch `webpage`), sem etapas de instalação de dependências, testes ou lint.

**(b) Diagnóstico:** Não há integração contínua de qualidade de código. Nenhum PR na `main` é verificado automaticamente antes do merge.

**(c) Risco:** **Alto** — regressões podem ser introduzidas silenciosamente.

**(d) Recomendação:** Criar `.github/workflows/ci.yml` com instalação de dependências, smoke test (importação de `inference.react_agent`) e lint (`ruff`) em PRs e pushes para `main`.

#### 2.2.2 Cobertura, badges e ferramentas de apoio

**(a) Evidência:** Ausência de `codecov.yml`, `.coveragerc`, `tox.ini`, `pyproject.toml`, `.ruff.toml`, `mypy.ini`. README exibe apenas badges de modelo/paper/blogs — nenhum de CI ou cobertura.

**(b) Diagnóstico:** Nenhum contrato de qualidade foi estabelecido; não há o que medir porque não há testes nem pipeline.

**(c) Risco:** **Alto** — dívida técnica acumula silenciosamente.

**(d) Recomendação:** Integrar cobertura ao CI quando a suíte existir; badge público; política progressiva no `CONTRIBUTING.md` (ex.: meta inicial 40% em `inference/`).

#### 2.2.3 Instruções para rodar testes

**(a) Evidência:** README sem menção a `pytest`, `test`, `coverage` ou `unittest`. Não existe `CONTRIBUTING.md`, `TESTING.md` ou `DEVELOPMENT.md`.

**(b) Diagnóstico:** Colaboradores não têm caminho documentado para validar alterações localmente.

**(c) Risco:** **Médio** — toda detecção de regressão recai sobre revisão manual.

**(d) Recomendação:** Criar `CONTRIBUTING.md` com seção "Running tests" (dependências de dev, comando `pytest`, política mínima em PRs).

**Síntese — Eixo A (Parte 2):**

| # | Item | Veredito | Risco |
|---|------|----------|-------|
| 1 | CI em PRs/pushes | Inexistente (só deploy de página) | Alto |
| 2 | Cobertura, badges, ferramentas | Inexistentes | Alto |
| 3 | Documentação para rodar testes | Inexistente | Médio |

**Veredito parcial do Eixo A:** zero automação de qualidade; gate humano frágil (confirma A2 — Parte 2, GPR "aprovado seco").

---

## 3. Eixo B — Qualidade técnica dos testes

_(Parte 4 — Guilherme Rosário Alves. Documento completo: [parte4-qualidade-tecnica.md](parte4-qualidade-tecnica.md) · evidências: [assets/parte4/](../assets/parte4/README.md))_

**Achado central.** O `DeepResearch` não possui suíte de testes automatizada (sem `pytest`/`unittest`/`conftest.py`, sem CI em `.github/` — ver dossiê E1–E7). Este eixo submete os artefatos que carregam o rótulo "test" aos cinco critérios de qualidade e demonstra que **nenhum cumpre a função de um teste de software**.

Artefatos avaliados:

1. `WebAgent/WebSailor/src/scripts/test.sh` — benchmark de inferência (GAIA), não teste.
2. `test_llm_call` / `test_audio_output_call` em `.../gpt4o/openai_style_api_client.py` (L116-175) — scripts manuais, LLM ao vivo, sem `assert`.
3. `evaluation/evaluate_deepsearch_official.py` e `evaluate_hle_official.py` — avaliadores por LLM-como-juiz (não-determinísticos).

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

#### 4.1.1 Acoplamento direto ao OpenAI SDK

**(a) Evidência:** `MultiTurnReactAgent.call_server` em `inference/react_agent.py` instancia `OpenAI(...)` com URL e chave fixas. Padrão repetido em ~26 arquivos (A2).

**(b) Diagnóstico:** Sem abstração `LLMClient`; impossível substituir por mock/fake em testes.

**(c) Risco:** **Alto** — testabilidade, custo, latência e vendor lock-in.

**(d) Recomendação:** Interface `LLMClient` + `OpenAICompatibleClient` + `FakeLLMClient` para testes.

#### 4.1.2 APIs externas (Search, Visit, Python)

**(a) Evidência:** `tool_search.py` (Serper), `tool_visit.py` (Jina + OpenAI), `tool_python.py` (Sandbox Fusion) leem credenciais do ambiente e fazem HTTP direto, sem injeção de dependência.

**(b) Diagnóstico:** Impossível testar unitariamente sem chamadas pagas e não determinísticas.

**(c) Risco:** **Alto** — custo financeiro, fragilidade ante mudanças de API.

**(d) Recomendação:** Camada de adaptadores injetáveis; fakes em testes; configuração centralizada.

#### 4.1.3 Ausência de infraestrutura de teste

**(a) Evidência:** Sem `tests/`, sem fixtures/mocks, sem menção a testes no README (confirma Partes 2 e 4).

**(b) Diagnóstico:** Validação exclusivamente manual ou via benchmarks completos.

**(c) Risco:** **Alto** — qualquer refatoração é aposta; diagnóstico de bugs severos é lento.

**(d) Recomendação:** `pytest` + `pytest-mock` + `responses`; política de testes em PRs via GitHub Actions.

**Tabela — dependências externas:**

| Dependência | Local | Isolamento? | Risco |
|-------------|-------|-------------|-------|
| OpenAI / LLM | `react_agent.py`, `agent.py`, `tool_visit.py` | Não | Alto |
| Serper | `tool_search.py` | Não | Alto |
| Jina | `tool_visit.py` | Não | Alto |
| Sandbox Fusion | `tool_python.py` | Não | Alto |
| Variáveis de ambiente | Vários arquivos | Não | Médio |

### 4.2 Lacunas e riscos em módulos críticos _(Parte 5 — Brenno Phelipe Silva dos Santos)_

Documento completo: [parte5-lacunas-riscos.md](parte5-lacunas-riscos.md)

| # | Lacuna investigada | Diagnóstico resumido | Risco | Recomendação-chave |
|---|-------------------|----------------------|-------|-------------------|
| 1 | Funcionalidades críticas (A1) sem testes | `tool_search`, `tool_visit`, `tool_python` sem cobertura | Alto | Testes unitários por `BaseTool` |
| 2 | Módulos com dívida técnica (A2) | `react_agent.py` ("god module") sem testes de caracterização | Alto | Characterization tests antes de refatorar |
| 3 | Dependências sem isolamento | 26 arquivos com `OpenAI(...)` direto; benchmarks em produção | Alto | Mocks/fakes + adaptadores (A2) |
| 4 | Proteção contra regressões | Sem CI + "aprovado seco" (A2) | Alto | Smoke test em PR via GitHub Actions |
| 5 | Detecção rápida de bugs | "Suíte" = benchmarks lentos com LLM-juiz | Alto | `pytest` local em milissegundos |

**Veredito geral do Eixo C:** o sistema opera sem rede de segurança automatizada. Módulos centrais e dependências externas concentram risco **alto**; refatorações propostas na A2 são inviáveis sem testes prévios.

---

## 5. Eixo D — Plano de evolução da qualidade

_(Parte 6 — pendente. Documento previsto: [parte6-plano-evolucao.md](parte6-plano-evolucao.md))_

Com base nos achados das Partes 2–5, a equipe deverá consolidar:

1. **Top 3 prioridades** (ex.: CI mínimo, abstrações LLM/API, characterization tests em `react_agent.py`).
2. **Camadas de foco** (unidade → integração com fakes → contratos → regressão).
3. **Implantação gradual** sem travar desenvolvimento ativo.
4. **Automação** (pytest, GitHub Actions, política mínima em PRs).
5. **Critérios de sucesso** (entrada/saída de maturidade).

---

## 6. Síntese final e prioridades de evolução

### 6.1 Diagnóstico consolidado

| Eixo | Achado principal | Risco dominante |
|------|------------------|-----------------|
| A — Estratégia | Sem CI de qualidade, sem cobertura, sem docs de teste | Alto |
| B — Qualidade técnica | Artefatos "test" não são testes; zero asserções | Alto |
| C — Lacunas/riscos | Core sem cobertura; acoplamento a APIs externas | Alto |
| D — Evolução | _(a consolidar na Parte 6)_ | — |

### 6.2 Prioridades emergentes (das partes entregues)

1. **Pipeline mínimo de CI** — smoke test + lint em PRs (Parte 2).
2. **Abstrações e fakes para LLM/APIs** — pré-requisito para qualquer teste isolado (Partes 3 e 5).
3. **Characterization tests no `react_agent.py`** — barreira antes de refatoração (Parte 5).
4. **Suíte pytest local rápida** — desacoplar validação de software de benchmarks com LLM-juiz (Partes 4 e 5).
5. **`CONTRIBUTING.md` com "Running tests"** — formalizar contrato de contribuição (Parte 2).

### 6.3 Continuidade com A1 e A2

Os achados desta auditoria **confirmam e aprofundam** as Atividades anteriores: ausência de CI (A1 V&V), falta de `CONTRIBUTING.md` (A1 GQA), "aprovado seco" e dependência de herói (A2 GPR), violações SRP/DRY e DIP no núcleo `inference/` (A2 SOLID). A diferença desta etapa é demonstrar que **não existe camada técnica de testes** capaz de sustentar as refatorações já propostas — transformando dívida arquitetural em **risco operacional imediato**.

---

## 7. Contribuição dos integrantes

| Integrante | Parte | Entrega |
|------------|-------|---------|
| Guilherme Rosário Alves | 4 | Eixo B completo + dossiê `assets/parte4/` + consolidação deste relatório |
| Alisson Francisco dos Santos | 2 | Eixo A (CI/CD, cobertura, documentação) |
| Davi Emanuel de Menezes Costa | 3 | Eixo C (dependências externas e isolamento) |
| Brenno Phelipe Silva dos Santos | 5 | Eixo C (lacunas e riscos em módulos críticos) |
| _Pendente_ | 1 | Eixo A (estrutura e tipos de teste) |
| _Pendente_ | 6 | Eixo D (plano de evolução da qualidade) |
| Alícia Vitória Sousa Santos | — | _a definir_ |
| Uilson Alves dos Santos Neto | — | _a definir_ |

_Revisão conjunta do documento e do repositório antes da entrega em PDF._
