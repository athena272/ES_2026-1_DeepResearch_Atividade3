# Evidências — Parte 4 (Eixo B: Qualidade técnica dos testes)

Dossiê de evidências **reais e reproduzíveis** que sustentam a Parte 4. Cada item traz o **comando exato**, a **saída real** e o **permalink** no GitHub.

- Projeto analisado: [Alibaba-NLP/DeepResearch](https://github.com/Alibaba-NLP/DeepResearch)
- Commit auditado: `f72f75d8c3eb842f2bbbab096a12206ff66e270f` (branch `main`)
- Como reproduzir: clonar o repositório, fazer `git checkout f72f75d` e rodar os comandos abaixo na raiz.

> **Sobre os prints:** as imagens em PNG não foram geradas automaticamente (um print fabricado não seria evidência válida). Para os slides/PDF, rode os comandos abaixo no seu terminal e capture a tela — a saída será idêntica à registrada aqui. Salve as imagens nesta pasta como `e1-arquivos-test.png`, `e2-frameworks.png`, etc.

---

## E1 — Não há suíte de testes; só um script de benchmark com nome "test"

**Comando:**

```bash
git ls-files "*test*"
```

**Saída real:**

```
WebAgent/WebSailor/src/scripts/test.sh
```

O único arquivo "test" do repositório é um shell script que dispara um benchmark, não um teste:

```bash
cd src
# bash run.sh <model_path> <dataset> <output_path>
bash run.sh websailor_3b gaia output_path
```

Permalink: https://github.com/Alibaba-NLP/DeepResearch/blob/f72f75d8c3eb842f2bbbab096a12206ff66e270f/WebAgent/WebSailor/src/scripts/test.sh

---

## E2 — Nenhum framework/configuração de teste

**Comando:**

```bash
git ls-files "*conftest*" "*pytest.ini*" "*tox.ini*" "pyproject.toml" "setup.cfg"
```

**Saída real:** _(vazio — nenhum arquivo encontrado)_

---

## E3 — Nenhum pipeline de CI

**Comando:**

```bash
git ls-files ".github/*"
```

**Saída real:** _(vazio — não existe pasta `.github/`, logo não há execução automática de testes em PRs/pushes)_

---

## E4 — As únicas funções "test_" são scripts manuais de chamada de LLM

**Comando:**

```bash
git grep -n "def test_" -- "*.py"
```

**Saída real:**

```
WebAgent/WebWatcher/infer/vl_search_r1/qwen-agent-o1_search/qwen_agent/tools/gpt4o/openai_style_api_client.py:116:def test_llm_call(prompt, call_llm='gpt-4o', evaluate=False):
WebAgent/WebWatcher/infer/vl_search_r1/qwen-agent-o1_search/qwen_agent/tools/gpt4o/openai_style_api_client.py:149:def test_audio_output_call(args):
```

Essas funções são executadas via `if __name__ == '__main__'`, chamam a API de LLM ao vivo e **não fazem asserção** (o segundo só dá `print`).

Permalink: https://github.com/Alibaba-NLP/DeepResearch/blob/f72f75d8c3eb842f2bbbab096a12206ff66e270f/WebAgent/WebWatcher/infer/vl_search_r1/qwen-agent-o1_search/qwen_agent/tools/gpt4o/openai_style_api_client.py#L116-L175

---

## E5 — Nenhum import de framework de teste em código Python

**Comando:**

```bash
git grep -n -E "^\s*(import pytest|import unittest|from unittest)" -- "*.py"
```

**Saída real:** _(vazio — nenhum)_

---

## E6 — A única asserção é guarda de runtime no código de produção (não é teste)

**Comando:**

```bash
git grep -n "assert " -- "*/gpt4o/openai_style_api_client.py"
```

**Saída real:**

```
.../gpt4o/openai_style_api_client.py:87:        assert 'model' in payload
.../gpt4o/openai_style_api_client.py:108:                # assert message_tokens == ret_json['usage']['prompt_tokens']
```

A linha 87 é uma checagem dentro do método `call()` em produção; a linha 108 é uma asserção **comentada** (código morto). Nenhuma das duas é um teste.

Permalink: https://github.com/Alibaba-NLP/DeepResearch/blob/f72f75d8c3eb842f2bbbab096a12206ff66e270f/WebAgent/WebWatcher/infer/vl_search_r1/qwen-agent-o1_search/qwen_agent/tools/gpt4o/openai_style_api_client.py#L87

---

## E7 — O que existe em `evaluation/` é avaliação de acurácia por LLM-juiz (não teste de software)

Os scripts `evaluate_deepsearch_official.py` e `evaluate_hle_official.py` instanciam um cliente OpenAI e usam o modelo como **juiz** da resposta final (campo `"correct": "yes"|"no"`), dependendo de API externa e sendo não-determinísticos.

Permalinks:
- https://github.com/Alibaba-NLP/DeepResearch/blob/f72f75d8c3eb842f2bbbab096a12206ff66e270f/evaluation/evaluate_deepsearch_official.py
- https://github.com/Alibaba-NLP/DeepResearch/blob/f72f75d8c3eb842f2bbbab096a12206ff66e270f/evaluation/evaluate_hle_official.py

---

## Resumo das evidências

| ID | O que prova | Veredito |
|----|-------------|----------|
| E1 | Único "test" é benchmark (`test.sh`) | Nome enganoso |
| E2 | Sem pytest/unittest/conftest/tox | Sem framework de teste |
| E3 | Sem `.github/` | Sem CI executando testes |
| E4 | `test_*` chamam LLM ao vivo, sem assert | Não são testes |
| E5 | Sem imports de framework em `.py` | Confirma E2 |
| E6 | Única asserção é guarda de runtime / código morto | Não é teste |
| E7 | `evaluation/` usa LLM-juiz | Avalia modelo, não código |
