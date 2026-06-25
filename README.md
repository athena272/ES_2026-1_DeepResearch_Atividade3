# ES_2026-1_DeepResearch_Atividade3

Repositório de trabalho da **Equipe 1** para a **Atividade 3** (Auditoria de Testes de Software e Plano de Evolução da Qualidade) — projeto analisado: [Alibaba-NLP/DeepResearch](https://github.com/Alibaba-NLP/DeepResearch).

Atividades anteriores (contexto):
- Atividade 1: [ES_2026-2_DeepResearch](https://github.com/athena272/ES_2026-2_DeepResearch)
- Atividade 2: [ES_2026-2_DeepResearch_Atividade2](https://github.com/athena272/ES_2026-2_DeepResearch_Atividade2)

**Relatório para PDF:** [docs/relatorio-atividade3.md](docs/relatorio-atividade3.md) — documento consolidado das seis partes (copiar e converter para PDF).

**Vídeo de auditoria técnica:** _(link a definir — YouTube/Drive, 7 a 15 min)_

---

## Objetivo

Realizar um diagnóstico da estratégia de testes do projeto **DeepResearch**, investigando a existência, qualidade, cobertura e relevância dos testes automatizados e/ou manuais, identificando riscos e lacunas, e propondo um plano concreto de evolução da qualidade. A análise mantém a continuidade das Atividades 1 e 2: os achados anteriores (rastreabilidade fraca, fragilidade do gate de PRs, violações SOLID/GoF e dívida técnica no núcleo `inference/`) são usados como pistas para orientar a auditoria de testes.

A investigação segue quatro eixos, e **cada item** é apresentado no formato: **(a) Evidência · (b) Diagnóstico · (c) Risco · (d) Recomendação**.

- **Eixo A** — Estratégia atual de testes
- **Eixo B** — Qualidade técnica dos testes
- **Eixo C** — Lacunas, riscos e falhas potenciais
- **Eixo D** — Plano de evolução da qualidade

---

## Índice das partes (Atividade 3)

| Parte | Eixo | Tema | Responsável | Documento |
|-------|------|------|-------------|-----------|
| 1 | A | Estrutura e tipos de teste | _a definir_ | [docs/parte1-estrategia-estrutura.md](docs/parte1-estrategia-estrutura.md) |
| 2 | A | CI/CD, automação e cobertura | _a definir_ | [docs/parte2-cicd-cobertura.md](docs/parte2-cicd-cobertura.md) |
| 3 | C | Dependências externas (LLM/API) sem isolamento para teste | _a definir_ | [docs/parte3-dependencias-isolamento.md](docs/parte3-dependencias-isolamento.md) |
| 4 | B | Qualidade técnica dos testes | Guilherme Rosário Alves | [docs/parte4-qualidade-tecnica.md](docs/parte4-qualidade-tecnica.md) · [evidências](assets/parte4/README.md) |
| 5 | C | Lacunas e riscos em módulos críticos | _a definir_ | [docs/parte5-lacunas-riscos.md](docs/parte5-lacunas-riscos.md) |
| 6 | D | Plano de evolução da qualidade | _a definir_ | [docs/parte6-plano-evolucao.md](docs/parte6-plano-evolucao.md) |

> Demais atribuições serão definidas pela equipe. Parte 4 já desenvolvida (com dossiê de evidências em [assets/parte4/](assets/parte4/README.md)).

### Equipe 1

- Guilherme Rosário Alves
- Alícia Vitória Sousa Santos
- Uilson Alves dos Santos Neto
- Brenno Phelipe Silva dos Santos
- Davi Emanuel de Menezes Costa
- Alisson Francisco dos Santos

---

## Estrutura do repositório

- `docs/` — relatório consolidado e documentos por parte.
- `assets/` — evidências visuais (prints do GitHub, pipelines, trechos de configuração) organizadas por parte.

> A Seção de Contribuição dos Integrantes é obrigatória no relatório final, indicando a participação de cada membro.
