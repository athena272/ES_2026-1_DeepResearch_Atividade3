# Parte 1 — Eixo A: Estrutura e tipos de teste

**Formato obrigatório por item:** (a) Evidência · (b) Diagnóstico · (c) Risco · (d) Recomendação

**Responsável:** Uilson Alves dos Santos Neto

**Repositório:** https://github.com/Alibaba-NLP/DeepResearch  
**Âmbito:** `inference/`, `WebAgent/`, `evaluation/`

---

## 1. Estrutura dedicada a testes

### (a) Evidência

Pesquisa estática no clone: **não existe** pasta `tests/`, `test/` nem `__tests__/`; **não há** ficheiros `test_*.py` ou `*_test.py`; **não há** `pytest` / `unittest` em `*.py`, nem `pytest.ini`, `conftest.py` ou `.github/workflows/` (CI).

O único ficheiro com “test” no nome é `WebAgent/WebSailor/src/scripts/test.sh`, que o README descreve como script de **avaliação**, não de testes:

```bash
# WebAgent/WebSailor/src/scripts/test.sh
bash run.sh websailor_3b gaia output_path
```

O que **existe** em vez de suíte de testes é a pasta `evaluation/` (benchmarks) e `eval_data/` com exemplos `.jsonl` para inferência manual (`inference/eval_data/example.jsonl`, etc.).

### (b) Diagnóstico

O repositório **não adota convenção dedicada a testes de software**. A verificação está deslocada para **avaliação de modelo** (`evaluation/evaluate_*_official.py`, `evaluate.py` por agente) e para **execução manual** documentada em READMEs e *shell scripts*. Isto confirma na prática a fragilidade que a **A2** aponta no *gate* de qualidade: não há barreira automatizada antes de *merge*.

### (c) Risco

**Alto.** Alterações em `inference/react_agent.py`, *tools* ou clientes LLM podem entrar sem nenhum teste que as detecte. A validação depende de correr inferência ou benchmarks caros (API, GPU), o que desincentiva verificação frequente.

### (d) Recomendação

- Criar pasta `tests/` na raiz ou em `inference/` com pelo menos *smoke tests* do agente e das *tools* críticas.
- Adicionar workflow CI mínimo (ex.: `pytest` em PR) para fechar o *gate* que a A2 identifica como ausente.
- Renomear ou documentar claramente `test.sh` como `eval.sh` / “benchmark”, para não confundir com testes unitários.

---

## 2. Tipos de teste encontrados (unitário, integração, sistema, e2e, regressão, manual)

### (a) Evidência

| Tipo | Encontrado? | Evidência no repo |
|------|-------------|-------------------|
| **Unitário** | Não | Nenhum `test_*.py`, `assert` em suíte, nem `unittest`/`pytest`. |
| **Integração** | Não | Nenhum teste que ligue agente + *tool* + cliente LLM com pass/fail automatizado. |
| **Sistema** | Não | Nenhum teste de subsistema (`inference/`, um agente `WebAgent/*`) com critérios no código. |
| **E2E** | Parcial | Fluxos completos existem como **inferência** (`run_multi_react.py`, `agent_eval.py`, `eval.sh`), mas **sem asserções** de regressão. |
| **Regressão** | Não | Sem *golden files*, *snapshots* nem CI que bloqueie por falha. |
| **Manual** | Sim | `README.md`, `run_demo.sh`, `app.py` (Streamlit); blocos `if __name__ == "__main__"` em *tools* (ex.: `WebAgent/WebResummer/src/tool_search.py`). |

**Benchmark** (complementar, não é teste de software): `evaluation/evaluate_hle_official.py` usa *judge* LLM sobre `.jsonl`; por agente há `evaluate.py` ou `scripts_eval/agent_eval.py`.

```python
# evaluation/evaluate_hle_official.py
JUDGE_MODEL = "openai/o3-mini"
API_KEY = os.getenv("API_KEY", "")
```

```python
# WebAgent/WebResummer/src/tool_search.py — smoke manual
if __name__ == "__main__":
    search = Search()
    result = search.call({"query": [...]})
    print(result)
```

### (b) Diagnóstico

- **Unitário, integração, sistema e regressão:** ausentes. O núcleo ReAct (`inference/react_agent.py`) e as *tools* (`inference/tool_*.py`) **não** estão protegidos por testes — ligação directa com **A1** (requisitos centrais sem cobertura automatizada).
- **E2E:** o projeto corre pipelines reais (modelo + ferramentas + APIs), mas trata-os como **experimento/benchmark**, não como teste repetível com resultado esperado fixo.
- **Manual:** é o mecanismo dominante de “ver se funciona” antes de publicar ou após mudança local.
- **Benchmark:** mede **qualidade do modelo** em datasets públicos; **não** substitui teste de software (ex.: um refactor que parte o parsing de `<tool_call>` pode passar despercebido até alguém correr inferência à mão).

### (c) Risco

**Alto** para evolução do código; **médio** se a equipa só alterar hiperparâmetros ou configs de modelo (aí o benchmark ajuda). A mistura entre “avaliar modelo” e “testar software” gera falsa sensação de cobertura: correr `evaluate_hle_official.py` não prova que `react_agent.py` continua correcto após um *patch*.

### (d) Recomendação

- **Prioridade 1 — unitário/integração:** testes para parsing de `<tool_call>`, *retries* em `call_server` e contrato das *tools* base (`inference/tool_visit.py`, `tool_search.py`, …).
- **Prioridade 2 — regressão:** poucos casos *golden* (entrada → saída esperada) versionados em `tests/fixtures/`, executados no CI.
- **Manter benchmarks** em `evaluation/` para A1 no eixo “desempenho em dataset”, mas **não** contar com eles como único *gate* de qualidade de código (A2).
- Formalizar o que hoje é manual: checklist no README de inferência ou script `scripts/smoke_infer.sh` com saída comparável.
