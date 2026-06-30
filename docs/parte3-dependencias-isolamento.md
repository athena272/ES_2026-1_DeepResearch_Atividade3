# Parte 3 — Eixo C: Dependências externas (LLM/API) sem isolamento para teste

> Formato obrigatório por item: **(a) Evidência · (b) Diagnóstico · (c) Risco · (d) Recomendação**

## Responsável

Davi EManuel de Menezes Costa

## 1. Dependência direta do OpenAI SDK e endpoints fixos

### (a) Evidência

No arquivo `inference/react_agent.py`, a classe `MultiTurnReactAgent` instancia o cliente OpenAI diretamente no método `call_server`, com URL e chave fixas:

```python
def call_server(self, msgs, planning_port, max_tries=10):
    openai_api_key = "EMPTY"
    openai_api_base = f"http://127.0.0.1:{planning_port}/v1"
    client = OpenAI(
        api_key=openai_api_key,
        base_url=openai_api_base,
        timeout=600.0,
    )
    # ...
    chat_response = client.chat.completions.create(
        model=self.model,
        messages=msgs,
        # ...
    )
```

O mesmo padrão aparece em `WebAgent/WebWalker/src/agent.py`:

```python
self.client = OpenAI(
    api_key=llm['api_key'],
    base_url=llm['model_server'],
)
```

A Atividade 2 já havia documentado que **26 arquivos** no repositório contêm chamadas diretas a `OpenAI(` ou `AsyncOpenAI(`.

### (b) Diagnóstico

O sistema não possui nenhuma abstração para o cliente LLM. Cada módulo que precisa de inferência cria seu próprio cliente OpenAI, com URLs e chaves hardcoded ou lidas de variáveis de ambiente de forma descentralizada. Não há uma interface `LLMClient` que permita substituir a implementação real por um mock ou fake em testes. A troca de provedor (ex.: de OpenAI para OpenRouter ou modelo local) exige alteração manual em dezenas de arquivos.

### (c) Risco: **Alto**

- **Testabilidade**: impossível testar o agente sem uma chamada de rede real a um endpoint LLM.
- **Custo e latência**: testes que dependem de chamadas reais são lentos, caros e sujeitos a falhas de rede.
- **Vendor lock-in**: o sistema fica refém do SDK e dos endpoints da OpenAI, inviabilizando a migração para outros provedores sem reescrita extensiva.

### (d) Recomendação

- Criar uma interface `LLMClient` com métodos `generate()` e `generate_stream()`.
- Implementar um `OpenAICompatibleClient` que encapsule todas as chamadas ao SDK.
- Proibir a instanciação direta de `OpenAI` fora do módulo de adaptadores.
- Para testes, implementar um `FakeLLMClient` que retorna respostas pré-definidas ou configuráveis, permitindo testar o fluxo do agente sem rede.

---

## 2. Dependências de APIs externas (Search, Visit, Python)

### (a) Evidência

**Ferramenta Search** (`inference/tool_search.py`):

- Utiliza a chave `SERPER_KEY` do ambiente e faz requisições HTTP diretas para `google.serper.dev`.
- Não há qualquer mecanismo de injeção de dependência para o cliente HTTP.

```python
SERPER_KEY = os.environ.get('SERPER_KEY_ID')
conn = http.client.HTTPSConnection("google.serper.dev")
headers = { 'X-API-KEY': SERPER_KEY, ... }
conn.request("POST", "/search", payload, headers)
```

**Ferramenta Visit** (`inference/tool_visit.py`):

- Depende de `JINA_API_KEYS` e faz requisições para serviços de extração de conteúdo.
- Importa e usa `OpenAI` para sumarização.

**Ferramenta PythonInterpreter** (`inference/tool_python.py`):

- Utiliza `SANDBOX_FUSION_ENDPOINTS` (variável de ambiente) para apontar para um serviço de sandbox.
- Faz chamadas a `run_code()` que dependem de um endpoint externo.

### (b) Diagnóstico

Cada ferramenta carrega suas próprias credenciais e configurações de endpoint diretamente do ambiente, sem passar por um ponto central de configuração. Não há injeção de dependência: as ferramentas instanciam seus próprios clientes HTTP e leem variáveis de ambiente no momento da importação ou da chamada. Isso torna impossível:

- Testar unitariamente cada ferramenta sem chamadas reais a serviços pagos.
- Simular falhas de rede ou respostas específicas para validar comportamentos de erro.
- Substituir o serviço real por um fake durante a execução de testes de integração.

### (c) Risco: **Alto**

- **Custo financeiro**: cada execução de teste pode gerar custos reais nas APIs (Serper, Jina, OpenAI).
- **Não determinismo**: testes dependem de disponibilidade e latência de serviços externos.
- **Fragilidade**: alterações nas APIs externas quebram os testes sem que o código do projeto tenha mudado.
- **Dívida técnica**: a ausência de isolamento dificulta a refatoração segura das ferramentas, conforme apontado na Atividade 2 para o `MultiTurnReactAgent`.

### (d) Recomendação

- Criar uma camada de adaptadores para cada serviço externo (ex.: `SearchProvider`, `PageReader`, `CodeExecutor`).
- Injetar esses adaptadores nas ferramentas via construtor, em vez de instanciá-los internamente.
- Para testes, implementar fakes que retornam dados estáticos ou simulam comportamentos de erro.
- Centralizar a leitura de variáveis de ambiente em um módulo de configuração único, que pode ser sobrescrito em testes.

---

## 3. Ausência de estratégia de isolamento para testes

### (a) Evidência

- Não há diretório `tests/` na raiz do repositório, nem convenção de arquivos `test_*.py` ou `*_test.py`.
- Não há menção a testes automatizados no `README.md` ou em qualquer documentação interna.
- Não há arquivos de fixture, mock ou stub em todo o código analisado.
- A Atividade 2 já havia documentado a ausência de CI para testes e o "gate de qualidade" baseado em revisões secas, sem verificação automatizada.

### (b) Diagnóstico

O projeto não possui qualquer infraestrutura de teste automatizado. Não há:

- Testes unitários para as classes centrais (`MultiTurnReactAgent`, ferramentas).
- Testes de integração para verificar a interação com serviços externos.
- Testes de regressão para proteger fluxos centrais.
- Mocks ou stubs que permitam isolar dependências externas.

Isso significa que toda a validação do sistema é feita de forma manual ou por meio de scripts de avaliação que executam o sistema completo contra datasets de benchmark, sem qualquer garantia de que as unidades funcionam isoladamente.

### (c) Risco: **Crítico**

- **Sem testes, qualquer refatoração é uma aposta.** O sistema não tem rede de segurança para detectar regressões.
- **Bug severo = diagnóstico lento.** Sem testes automatizados, encontrar a causa raiz de um bug exige depuração manual extensa.
- **Dependências externas mudam.** Se a API do Serper ou da Jina mudar, o sistema pode quebrar em produção sem que ninguém saiba até o momento do uso.
- **Dívida técnica acumulada.** O alto acoplamento identificado na Atividade 2 não pode ser resolvido com segurança sem testes que garantam que o comportamento permanece o mesmo.

### (d) Recomendação

**Implantação gradual:**

1. Começar com testes unitários para as ferramentas que não dependem de rede (ex.: parsing, formatação).
2. Em seguida, introduzir testes com fakes para as ferramentas que chamam APIs externas.
3. Por fim, implementar testes de integração com um ambiente de staging que use versões mockadas dos serviços.

**Ferramentas sugeridas:**

- `pytest` como framework de testes.
- `pytest-mock` para criar mocks de clientes HTTP e SDKs.
- `responses` ou `httpx.MockTransport` para simular respostas HTTP.
- `unittest.mock` para substituir chamadas a OpenAI por objetos fake.

**Política mínima:**

- Todo novo código deve vir acompanhado de testes unitários.
- Pull requests devem passar por uma suíte de testes antes do merge (a ser implementada via GitHub Actions).

---

## 4. Síntese: impacto na testabilidade e relação com achados da A2

| Dependência externa | Local no código | Estratégia de isolamento? | Risco |
|---|---|---|---|
| OpenAI / LLM | `react_agent.py`, `agent.py`, `tool_visit.py` | ❌ Nenhuma | Alto |
| Serper (busca) | `tool_search.py` | ❌ Nenhuma | Alto |
| Jina (leitura de páginas) | `tool_visit.py` | ❌ Nenhuma | Alto |
| Sandbox Fusion (execução Python) | `tool_python.py` | ❌ Nenhuma | Alto |
| Variáveis de ambiente | Distribuídas em vários arquivos | ❌ Nenhuma | Médio |

**Conexão com a Atividade 2:** o diagnóstico de DIP (Inversão de Dependência) e vendor lock-in feito na A2 se confirma agora pela ótica da testabilidade: a falta de abstrações não apenas prende o sistema a provedores específicos, mas também inviabiliza testes isolados. Cada dependência externa é um ponto de falha que torna os testes lentos, caros e não determinísticos. O "god module" `MultiTurnReactAgent` concentra múltiplas responsabilidades, incluindo a criação de clientes HTTP e o parsing de respostas, o que agrava ainda mais a dificuldade de testar cada parte separadamente.

---

## Recomendações finais (priorizadas)

1. **Criar abstrações para LLM e serviços externos** (prioridade máxima) — isso ataca a raiz do problema de acoplamento e testabilidade.
2. **Implementar fakes para testes** — permitir que os testes rodem sem rede, custo ou latência.
3. **Estabelecer uma política de testes** — definir que todo PR deve incluir testes e que a suíte deve passar antes do merge.
