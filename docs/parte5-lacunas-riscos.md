# Parte 5 — Eixo C: Lacunas e riscos em módulos críticos

**Responsável:** Brenno Phelipe

Este documento cruza as funcionalidades críticas mapeadas na Atividade 1 e a dívida técnica/arquitetural levantada na Atividade 2 com a total ausência de testes de software. O objetivo é identificar o que não está coberto e evidenciar os riscos gerados para o negócio no contexto do ecossistema atual do repositório. A investigação obedece ao padrão exigido: Evidência, Diagnóstico, Risco e Recomendação para cada lacuna.  

---

## 1. Quais funcionalidades críticas (mapeadas na A1) estão sem testes?

**(a) Evidência:** As ferramentas que permitem ao agente interagir com a web e o sistema operacional (Requisitos Funcionais mapeados na A1, como `tool_search.py`, `tool_visit.py` e `tool_python.py`) não possuem qualquer arquivo de teste correspondente. A varredura não encontrou nenhuma suíte automatizada focada no comportamento dessas ferramentas isoladas.  

**(b) Diagnóstico:** As capacidades fundamentais que dão nome e utilidade ao "DeepResearch" operam sem qualquer verificação contínua de corretude. O projeto assume que a pesquisa web e a extração de páginas funcionarão perfeitamente baseando-se apenas em experimentações manuais.

**(c) Risco:** **Alto.** Funcionalidades de núcleo (*core features*) podem quebrar sem aviso prévio devido a alterações em arquivos adjacentes ou atualizações de dependências.

**(d) Recomendação:** Criar testes de unidade específicos para cada classe que herda de `BaseTool`, validando se as entradas e saídas esperadas (*parsing* de HTML, sanitização de inputs) ocorrem corretamente antes de enviar dados ao orquestrador.

---

## 2. Quais módulos (com dívida técnica apontada na A2) apresentam maior risco caso sejam alterados?

**(a) Evidência:** O arquivo `inference/react_agent.py`, apontado na Atividade 2 como o "God Module" do sistema, viola o Princípio da Responsabilidade Única (SRP) e o princípio DRY por misturar chamadas de rede HTTP, *parsing* manual de strings (`split('<tool_call>')`) e o próprio loop de raciocínio da inteligência artificial. Nenhum desses comportamentos é coberto por testes.  

**(b) Diagnóstico:** Sendo o módulo mais acoplado e complexo da arquitetura, sua manutenção é um "voo às cegas". Sem testes para garantir o comportamento atual, qualquer tentativa de aplicar a refatoração sugerida na A2 (como extrair o cliente de rede e o parser) tem alta probabilidade de injetar regressões letais na lógica de negócio principal.  

**(c) Risco:** **Alto.** O orquestrador é o coração do sistema; um erro nesse módulo paralisa toda a aplicação, impactando diretamente o usuário final e aumentando o custo de manutenção da equipe.  

**(d) Recomendação:** Antes de tentar desmembrar o "God Module", a equipe deve escrever **Testes de Caracterização** (*Characterization Tests*). Estes testes servem unicamente para congelar e descrever o comportamento do código legado no estado atual, criando uma barreira de segurança vitalícia para a futura refatoração estrutural.

---

## 3. Há dependências externas sem estratégia de isolamento para teste?

**(a) Evidência:** A análise da Atividade 2 detectou 26 arquivos instanciando clientes da OpenAI diretamente (ex: `client = OpenAI(...)`), configurando forte acoplamento e *vendor lock-in*. Os testes executados através dos scripts de benchmark chamam os provedores reais (via rede). Não há uso de bibliotecas de simulação (como `unittest.mock`) ou *stubs* para as APIs da OpenAI ou provedores de pesquisa.  

**(b) Diagnóstico:** O sistema testa diretamente "em produção". A ausência de isolamento significa que a equipe não consegue testar como o sistema reage a falhas da API da OpenAI (como *Timeout* ou HTTP 500) sem de fato sofrer essas indisponibilidades no mundo real.

**(c) Risco:** **Alto.** Gera desperdício financeiro através do consumo desnecessário de tokens (cobrados em faturas de API) para validações básicas de software. Além disso, causa não-determinismo: a "validação" pode falhar por uma oscilação na rede externa, e não por um defeito no código.  

**(d) Recomendação:** Implementar dublês de teste (*Mocks/Fakes*) para todas as interfaces de comunicação externa (LLM e APIs de busca). Usar `responses` ou injeção de dependência via adaptador (proposto na A2) para simular retornos de erro da rede e verificar se o sistema se recupera adequadamente sem gastar tokens reais.  

---

## 4. O projeto está protegido contra regressões em fluxos centrais?

**(a) Evidência:** A governança de código mapeada na Parte 2 da Atividade 2 revela uma cultura de PRs mergeados com "zero comentários" de revisão ("Aprovado Seco") e a ausência absoluta de automação CI/CD (pasta `.github/workflows` inexistente).  

**(b) Diagnóstico:** A total ausência de Integração Contínua (CI) alinhada com a falta de *branch protection rules* significa que não existe um *Quality Gate* ou rede de segurança antes de aceitar novo código na `main`.  

**(c) Risco:** **Alto.** O ambiente não está protegido contra regressões severas. Qualquer contribuidor (interno ou externo) pode introduzir um código quebrado na linha principal, invalidando a ferramenta imediatamente para toda a comunidade.  

**(d) Recomendação:** Implantar obrigatoriamente um *workflow* no GitHub Actions que execute um **Teste de Fumaça (Smoke Test)** automatizado e independente de rede a cada novo *Pull Request*, bloqueando o *merge* caso as funcionalidades mínimas falhem.  

---

## 5. Se um bug severo surgisse hoje, a suíte atual ajudaria a encontrá-lo rapidamente?

**(a) Evidência:** A "suíte" existente é composta exclusivamente por avaliadores de acurácia baseados em *benchmarks* extensos (ex: `evaluate_deepsearch_official.py`), os quais invocam Modelos de Linguagem para atuar como juízes (*LLM as a Judge*).  

**(b) Diagnóstico:** Não existe um loop de *feedback* rápido (*Fast Feedback Loop*). Esses scripts demoram horas e servem para avaliar a inteligência do modelo de IA, não para verificar se uma simples concatenação de *string* quebrou na camada de software. Se um bug clássico de engenharia de software surgir, o desenvolvedor só perceberá quando o pesado script acadêmico falhar no meio do processo ou estourar no console final.  

**(c) Risco:** **Alto.** O Tempo Médio para Detecção (MTBD - *Mean Time to Detect*) é irrazoável. Os desenvolvedores perdem tempo de trabalho aguardando uma avaliação lenta e não determinística apenas para descobrir um erro banal de programação.  

**(d) Recomendação:** Desacoplar a avaliação do modelo da avaliação do software. Manter os scripts baseados em LLM restritos aos relatórios acadêmicos / *benchmarks* e introduzir uma suíte de testes de unidade leve com `pytest` (que rode em milissegundos localmente) dedicada a apontar exclusivamente falhas de lógica no código.