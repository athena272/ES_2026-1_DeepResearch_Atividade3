# Atividade Avaliativa 3 — Auditoria de Testes de Software e Plano de Evolução da Qualidade

**Disciplina:** Engenharia de Software (COMP0503)

**Instituição:** UFS

**Projeto analisado:** DeepResearch — https://github.com/Alibaba-NLP/DeepResearch

**Atividades anteriores:** A1 — https://github.com/athena272/ES_2026-2_DeepResearch · A2 — https://github.com/athena272/ES_2026-2_DeepResearch_Atividade2

**Vídeo de auditoria técnica:** _(link a definir)_

---

## Identificação da equipe (Equipe 1)

| # | Nome | Matrícula | Contribuição individual (Atividade 3) |
|---|------|-----------|----------------------------------------|
| 01 | Guilherme Rosário Alves | 202100022784 | **Parte 4** — Eixo B: qualidade técnica dos testes; dossiê de evidências reproduzíveis (`assets/parte4/`). |
| 02 | Alícia Vitória Sousa Santos | 202300027015 | _a definir_ |
| 03 | Uilson Alves dos Santos Neto | 201900115954 | _a definir_ |
| 04 | Brenno Phelipe Silva dos Santos | 202400050750 | _a definir_ |
| 05 | Davi Emanuel de Menezes Costa | 202300027178 | _a definir_ |
| 06 | Alisson Francisco dos Santos | 202300083248 | _a definir_ |

---

## 1. Introdução e contextualização do sistema

## 2. Eixo A — Estratégia atual de testes

_(Partes 1 e 2)_

## 3. Eixo B — Qualidade técnica dos testes

_(Parte 4 — responsável: Guilherme Rosário Alves. Documento completo: [parte4-qualidade-tecnica.md](parte4-qualidade-tecnica.md) · evidências: [assets/parte4/](../assets/parte4/README.md))_

**Achado central.** O `DeepResearch` não possui suíte de testes automatizada (sem `pytest`/`unittest`/`conftest.py`, sem CI em `.github/` — ver dossiê de evidências E1–E7). Logo, este eixo submete os artefatos que apenas carregam o rótulo "test" aos cinco critérios de qualidade e mostra que nenhum cumpre a função de um teste de software. Isso desfaz a confusão entre **existência de arquivos com nome de teste** e **qualidade efetiva**.

Artefatos avaliados:

1. `WebAgent/WebSailor/src/scripts/test.sh` — apesar do nome, executa um benchmark de inferência (GAIA).
2. `test_llm_call` / `test_audio_output_call` em `.../gpt4o/openai_style_api_client.py` (L116-175) — scripts manuais via `__main__`, chamam a API de LLM ao vivo, sem `assert`, com `print()`.
3. `evaluation/evaluate_deepsearch_official.py` e `evaluate_hle_official.py` — avaliadores de acurácia por LLM-como-juiz (não-determinísticos, dependentes de API externa).

Síntese por critério:

| # | Critério (Eixo B) | Veredito | Risco |
|---|-------------------|----------|-------|
| 4.1 | Nomes claros / comportamento observável | Nomes enganosos (execuções, não testes) | Médio |
| 4.2 | Isolamento, independência, reprodutibilidade | Dependem de API/rede/chaves; não-determinístico | Alto |
| 4.3 | Mocks/stubs/fixtures/dados de teste | Sem dublês; "dados" são datasets de benchmark | Alto |
| 4.4 | Validação de regras vs. superficial | Sem asserções; avaliam o modelo, não o código | Alto |
| 4.5 | Redundantes, frágeis, difíceis de manter | Frágeis; código morto e acoplamento a host fixo | Médio |

**Veredito geral:** não há qualidade técnica de testes a medir porque não há testes de software — apenas scripts de execução e avaliadores de modelo.

## 4. Eixo C — Lacunas, riscos e falhas potenciais

_(Partes 3 e 5)_

## 5. Eixo D — Plano de evolução da qualidade

_(Parte 6)_

## 6. Síntese final e prioridades de evolução

## 7. Contribuição dos integrantes
