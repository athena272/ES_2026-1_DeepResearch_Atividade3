# Parte 4 — Eixo B: Qualidade técnica dos testes

> Formato obrigatório por item: **(a) Evidência · (b) Diagnóstico · (c) Risco · (d) Recomendação**

**Responsável:** Guilherme Rosário Alves

**Projeto analisado:** [Alibaba-NLP/DeepResearch](https://github.com/Alibaba-NLP/DeepResearch)

---

## Nota de método (como a evidência foi coletada)

A varredura foi feita sobre o clone local do monorepo, buscando qualquer artefato que pudesse constituir teste:

- Busca por arquivos de teste por convenção: `**/test_*.py`, `**/*_test.py`, `**/*test*` → **nenhum** arquivo de teste Python; o único resultado foi um script de shell (`WebAgent/WebSailor/src/scripts/test.sh`).
- Busca por frameworks/configuração de teste: `pytest.ini`, `tox.ini`, `conftest.py`, `setup.cfg`, `pyproject.toml` → **nenhum** encontrado.
- Busca textual por `import pytest`, `import unittest`, `from unittest`, `def test_` → as ocorrências de `def test_` aparecem **apenas** em um cliente de API (`openai_style_api_client.py`), não em arquivos de teste.
- Busca por pipeline de CI: pasta `.github/` → **inexistente**.

Conclusão da varredura: **o projeto não possui uma suíte de testes automatizada**.

---

## Achado central

Como não há suíte de testes, este eixo **não compara "testes bons vs. ruins"**. Em vez disso, submete os poucos **artefatos que carregam o rótulo "test"** aos cinco critérios de qualidade técnica exigidos no enunciado e demonstra que **nenhum deles cumpre a função de um teste de software**. Em outras palavras: a existência de arquivos/funções com a palavra "test" no nome **não** representa garantia de qualidade — é justamente a confusão que a rubrica pede para desfazer.

Os três artefatos avaliados são:

1. `WebAgent/WebSailor/src/scripts/test.sh` — apesar do nome, executa um **benchmark de inferência** sobre o dataset GAIA.
2. `test_llm_call` e `test_audio_output_call` em `.../tools/gpt4o/openai_style_api_client.py` — **scripts manuais** executados via `__main__`, que chamam a API de LLM ao vivo.
3. `evaluation/evaluate_deepsearch_official.py` e `evaluation/evaluate_hle_official.py` — **avaliadores de acurácia** que usam um LLM como juiz.

---

## 4.1 Os testes têm nomes claros e descrevem o comportamento observável?

### (a) Evidência

`WebAgent/WebSailor/src/scripts/test.sh` ([link](https://github.com/Alibaba-NLP/DeepResearch/blob/main/WebAgent/WebSailor/src/scripts/test.sh)) tem nome de teste, mas o corpo apenas dispara um benchmark:

```bash
cd src
# bash run.sh <model_path> <dataset> <output_path>
bash run.sh websailor_3b gaia output_path
```

No cliente de API (`openai_style_api_client.py`, [link](https://github.com/Alibaba-NLP/DeepResearch/blob/main/WebAgent/WebWatcher/infer/vl_search_r1/qwen-agent-o1_search/qwen_agent/tools/gpt4o/openai_style_api_client.py)), as funções `test_llm_call` e `test_audio_output_call` (linhas 116-175) não descrevem nenhum comportamento esperado — são apenas invocações da API.

### (b) Diagnóstico

O prefixo `test`/`test_` é usado como sinônimo de "experimentar/rodar manualmente", e não no sentido de verificação automatizada. O nome induz ao erro: um avaliador apressado contaria `test.sh` como "o projeto tem testes". Não há, em nenhum dos artefatos, um nome que expresse a forma `dado_X_quando_Y_entao_Z` (comportamento observável).

### (c) Risco

**Médio** — A nomenclatura enganosa cria uma falsa sensação de cobertura e dificulta auditorias e o onboarding de novos mantenedores, que precisam abrir cada arquivo para descobrir que não há verificação real.

### (d) Recomendação

Reservar o prefixo `test_` exclusivamente para testes automatizados sob `tests/` e renomear os scripts atuais para refletir a intenção real (ex.: `run_benchmark_gaia.sh`, `manual_smoke_llm.py`).

---

## 4.2 Os testes são isolados, independentes e reprodutíveis?

### (a) Evidência

`test_llm_call` cria um cliente real e dispara uma requisição HTTP para o provedor de LLM, dependendo de chaves de ambiente e da rede:

```python
def test_llm_call(prompt, call_llm='gpt-4o', evaluate=False):
    ...
    api = OpenAIAPIClient(timeout=timeout)
    response = api.call(**kwargs)
    return response
```

O avaliador `evaluation/evaluate_deepsearch_official.py` ([link](https://github.com/Alibaba-NLP/DeepResearch/blob/main/evaluation/evaluate_deepsearch_official.py)) instancia um cliente OpenAI a partir de variáveis de ambiente e usa o modelo como juiz:

```python
os.environ["OPENAI_API_KEY"] = os.getenv("OPENAI_API_KEY","")
...
def get_client():
    if not hasattr(thread_local, 'client'):
        thread_local.client = OpenAI(api_key=API_KEY, base_url=BASE_URL)
    return thread_local.client
```

### (b) Diagnóstico

Nenhum artefato é isolado: todos dependem de serviços externos (API de LLM), credenciais válidas, acesso de rede e do estado de um modelo remoto. Como o juiz é um LLM, o resultado é **não-determinístico** — a mesma entrada pode produzir vereditos diferentes. Não há, portanto, reprodutibilidade nem independência entre execuções.

### (c) Risco

**Alto** — Sem execução isolada e determinística, é impossível ter um gate de qualidade confiável. Qualquer "verificação" custa tokens, exige segredos e pode falhar por motivos alheios ao código (cota, indisponibilidade do provedor), o que inviabiliza rodar em CI.

### (d) Recomendação

Separar testes de unidade (puros, sem rede) das avaliações de modelo. Para a lógica de software (parsing, montagem de payload, tratamento de erro), escrever testes determinísticos com `pytest` e dublês; reservar a avaliação por LLM como etapa offline/manual, claramente rotulada como benchmark.

---

## 4.3 Há uso adequado de mocks, stubs, fixtures ou dados de teste?

### (a) Evidência

Não há mocks/stubs em nenhum dos artefatos: `test_llm_call` chama o provedor real e `evaluate_deepsearch_official.py` chama o juiz real. Os únicos "dados" associados a essas rotinas são **datasets de benchmark** (ex.: `WebAgent/WebWatcher/infer/vl_search_r1/eval_data/hle.jsonl`, `.../simplevqa.jsonl`), e não fixtures de teste com entrada/saída esperada.

### (b) Diagnóstico

O projeto confunde "dados de teste" (fixtures controladas para verificar comportamento) com "dados de avaliação" (amostras grandes para medir acurácia do modelo). Sem dublês para isolar as fronteiras (HTTP, LLM, ferramentas), nenhuma camada de software pode ser testada sem acionar a infraestrutura completa.

### (c) Risco

**Alto** — A ausência de pontos de isolamento (mocks/stubs) é o que tecnicamente **impede** a criação de testes rápidos e baratos. É a raiz da intestabilidade: mesmo quem quisesse escrever testes esbarraria no acoplamento direto às dependências externas.

### (d) Recomendação

Introduzir `pytest` com fixtures e `unittest.mock`/`responses` para simular o cliente HTTP/LLM, criando dados de teste pequenos e versionados em `tests/fixtures/`. Isso depende da inversão de dependência discutida na Parte 3 (Eixo C).

---

## 4.4 Os testes validam regras importantes ou apenas detalhes superficiais?

### (a) Evidência

`test_llm_call` e `test_audio_output_call` **não fazem nenhuma asserção**; o segundo apenas imprime o retorno:

```python
def test_audio_output_call(args):
    ...
    ret_json = api.call(**kwargs)
    print(truncate_long_strings(ret_json))
```

A única asserção próxima está dentro do **código de produção** (`call`, linha 87), como guarda de runtime, não como teste:

```python
assert 'model' in payload
```

Os avaliadores em `evaluation/` verificam se a **resposta final do modelo** está correta (campo `"correct": "yes"|"no"` julgado por LLM) — ou seja, medem acurácia do produto de IA, não a corretude das regras do software.

### (b) Diagnóstico

Não há validação de regras de negócio do sistema (ex.: política de retry, parsing de `finish_reason`, montagem do payload, tratamento de `APIException`). As rotinas `test_*` validam **zero** comportamento; os avaliadores validam o **modelo**, não o código. Detalhes críticos do fluxo do agente ficam completamente descobertos.

### (c) Risco

**Alto** — Regras importantes (retries, limites de tokens, tratamento de erros de API, formatação de mensagens) podem quebrar silenciosamente. Como nada as verifica, regressões só apareceriam em produção/inferência, encarecendo o diagnóstico.

### (d) Recomendação

Escrever testes de unidade para as regras de maior valor do `OpenAIAPIClient` e do loop do agente: validar que payload inválido é rejeitado, que `finish_reason` inesperado levanta `APIException`, e que o mecanismo de retry respeita `max_try`/`retry_sleep` — tudo com o provedor mockado.

---

## 4.5 Existem testes redundantes, frágeis ou difíceis de manter?

### (a) Evidência

Além de dependerem de serviços externos, os artefatos carregam **código morto/comentado** e acoplamento a endpoints específicos, sinais de manutenção frágil:

```python
# message_tokens = OpenAIAPIClient.num_tokens_from_messages(payload['messages'], payload['model'])
...
# assert message_tokens == ret_json['usage']['prompt_tokens']
```

```python
if self.call_url.startswith('http://47.88.8.18') and payload['model'].startswith('claude'):
    ret_json = openai_ret_wrapper(ret_json, 'mit', 'claude')
```

### (b) Diagnóstico

As rotinas `test_*` são frágeis por construção: quebram com qualquer instabilidade de rede, mudança de contrato do provedor ou rotação de credenciais, sem indicar se o defeito está no código. O acoplamento a IP/host fixo e a presença de asserções comentadas mostram que essas funções foram usadas como rascunho de depuração, não como teste mantível.

### (c) Risco

**Médio** — Testes frágeis, quando existem, costumam ser desativados ou ignorados (tornam-se ruído). No estado atual, o custo de manter esses artefatos como "testes" supera o benefício, reforçando a cultura de não testar.

### (d) Recomendação

Descartar os trechos de depuração comentados, extrair a configuração de endpoint para fora do código e substituir os scripts manuais por testes determinísticos e pequenos, que falhem apenas quando o comportamento do software mudar.

---

## Síntese — critério x veredito x risco

| # | Critério (Eixo B) | Veredito | Risco |
|---|-------------------|----------|-------|
| 4.1 | Nomes claros / comportamento observável | Nomes enganosos (`test.sh`, `test_*` são execuções, não testes) | Médio |
| 4.2 | Isolamento, independência, reprodutibilidade | Dependem de API/rede/chaves; não-determinístico | Alto |
| 4.3 | Mocks/stubs/fixtures/dados de teste | Sem dublês; "dados" são datasets de benchmark | Alto |
| 4.4 | Validação de regras vs. superficial | Sem asserções; avaliam o modelo, não o código | Alto |
| 4.5 | Redundantes, frágeis, difíceis de manter | Frágeis; código morto e acoplamento a host fixo | Médio |

**Veredito geral do Eixo B:** não há qualidade técnica de testes a medir porque não há testes de software — apenas scripts de execução e avaliadores de modelo. A "quantidade" aparente de artefatos com nome de teste é zero em termos de **qualidade efetiva**.

---

## Ligação com as Atividades 1 e 2

- **A2 — Parte 4 (SOLID: SRP e DRY em `inference/react_agent.py`):** o núcleo de baixa coesão e com duplicação identificado na A2 é exatamente o que torna o código difícil de testar. A ausência de testes confirmada aqui é a consequência prática daquela dívida estrutural.
- **A2 — Parte 3 (DIP / acoplamento a provedores LLM):** a falta de inversão de dependência explica por que não há mocks (4.3): as fronteiras externas não são injetáveis, então nada pode ser isolado. A Parte 3 desta A3 aprofunda esse ponto sob a ótica de testabilidade.
- **A2 — Parte 2 (GPR: "aprovado seco" e ausência de CI):** sem `branch protection` e sem pipeline (`.github/` inexistente), nunca houve um gate que exigisse testes. O comportamento observável de PRs explica a ausência estrutural de testes que este eixo documenta.
- **A1 (requisitos centrais):** as funcionalidades críticas mapeadas na A1 seguem sem qualquer rede de proteção automatizada — risco retomado e priorizado na Parte 5 (lacunas/riscos) e na Parte 6 (plano de evolução).

---

## Pontos para o vídeo (apoio à apresentação)

- Mostrar a busca falhando: nenhum `test_*.py`, nenhum `conftest.py`, nenhum `.github/`.
- Abrir `test.sh` e revelar que é um benchmark, não um teste.
- Abrir `test_llm_call` e destacar: chama LLM ao vivo, sem `assert`, só `print`.
- Fechar com a síntese: "ter arquivo com nome de teste não é ter teste".

> Espaço para evidências visuais: se a equipe quiser anexar prints (resultados das buscas, trechos no GitHub), criar `assets/parte4/` e referenciar aqui.
