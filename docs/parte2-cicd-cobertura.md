# Parte 2 — Eixo A: CI/CD, automação e cobertura

> Formato obrigatório por item: **(a) Evidência · (b) Diagnóstico · (c) Risco · (d) Recomendação**

**Responsável:** Alisson Francisco dos Santos

**Projeto analisado:** [Alibaba-NLP/DeepResearch](https://github.com/Alibaba-NLP/DeepResearch)

---

## Nota de método

A varredura foi feita sobre o clone local do repositório e complementada com consulta direta ao GitHub remoto, cobrindo três frentes:

- Busca por pipelines de CI: pasta `.github/workflows/` e arquivos `*.yml`/`*.yaml` na raiz e subdiretórios → **nenhum workflow de qualidade encontrado**.
- Busca por ferramentas de cobertura e análise estática: `codecov.yml`, `.coveragerc`, `tox.ini`, `pyproject.toml`, `.ruff.toml`, `.flake8`, `mypy.ini` → **nenhum encontrado**.
- Leitura do `README.md` e busca por documentação de desenvolvimento (`CONTRIBUTING.md`, `TESTING.md`, `DEVELOPMENT.md`) → **nenhuma instrução sobre testes encontrada**.

---

## 1. Integração com CI/CD (workflows, execução em PRs/pushes)

### (a) Evidência

**Comando (filesystem):**

```bash
git ls-files ".github/*"
```

**Saída:** *(vazio — a pasta `.github/` não existe no repositório)*

**Confirmação via GitHub remoto (aba Actions):**

A página `https://github.com/Alibaba-NLP/DeepResearch/actions` lista **apenas dois runs** de um único workflow:

```
Workflows
└── pages-build-deployment  (pages/pages-build-deployment)
```

Ambos os runs são do tipo "pages build and deployment", executados na branch `webpage` (deploy de site estático). Não existe nenhum workflow nomeado `ci.yml`, `test.yml`, `lint.yml`, `python-test.yml` ou qualquer variante. Os detalhes dos runs confirmam que as únicas etapas executadas são checkout e publicação de página — sem instalação de dependências Python, sem testes, sem lint.

### (b) Diagnóstico

O repositório **não possui nenhuma integração contínua de qualidade de código**. A única Action registrada (`pages-build-deployment`) é gerada automaticamente pelo GitHub ao ativar GitHub Pages e serve exclusivamente à página de divulgação; ela não executa nada relacionado ao código Python do projeto.

A ausência de `.github/workflows/` confirma que **nenhum workflow foi escrito pela equipe do DeepResearch**. Isso significa que a cada PR mergeado na `main` **nenhuma verificação automatizada é executada**. O gate de integração é exclusivamente humano e não foi documentado.

### (c) Risco

**Alto.** A ausência de CI/CD elimina a camada mais básica de proteção contra regressões. Mudanças que quebram funcionalidades centrais podem ser introduzidas e mergeadas silenciosamente, só sendo descobertas em produção ou durante execuções manuais de benchmark — o que atrasa a detecção e encarece o diagnóstico.

### (d) Recomendação

Criar um workflow de integração contínua no caminho `.github/workflows/ci.yml`, configurado para ser acionado em *pushes* para a branch `main` e em *pull requests* direcionados a ela. Esse pipeline deve contemplar, de forma progressiva, três etapas mínimas de validação:

1. **Instalação de dependências:** restaurar o ambiente a partir do arquivo `requirements.txt`, garantindo reprodutibilidade e detectando eventuais conflitos de versão.

2. **Teste de fumaça (*smoke test*):** verificar a integridade dos módulos centrais do agente por meio da importação dos principais *entry points* (`inference.react_agent` e `inference.prompt`). Essa etapa identifica erros de sintaxe, dependências faltantes ou quebras de interface entre módulos.

3. **Análise estática (*lint*):** aplicar uma ferramenta de verificação de estilo e boas práticas (ex.: `ruff`) sobre o diretório `inference/`, com foco inicial em erros críticos de sintaxe e definições não resolvidas, evitando sobrecarregar o pipeline com questões puramente estilísticas.

Essa implementação representa uma **rede de segurança mínima**, com custo de implantação inferior a um dia de trabalho, e ataca diretamente a lacuna mais crítica identificada: a ausência total de verificação automatizada antes da integração. O pipeline deve evoluir naturalmente com a adição de uma etapa de execução de testes (`pytest`) à medida que a suíte de testes for sendo construída.

---

## 2. Cobertura, badges e ferramentas de apoio à qualidade

### (a) Evidência

**Busca por configuração de cobertura e análise estática:**

```bash
git ls-files "codecov*" ".coveragerc" "tox.ini" "pyproject.toml" "setup.cfg"
```

**Saída:** *(vazio — nenhum arquivo encontrado)*

```bash
git ls-files ".ruff.toml" ".flake8" "mypy.ini" ".pylintrc" "sonar-project.properties"
```

**Saída:** *(vazio — nenhum arquivo encontrado)*

**Badges no README.md:**

O README exibe exclusivamente badges de modelo, paper, blogs e conquistas:

```
[![MODELS]  [![PAPER]  [![BLOG]  [ModelScope]  [HuggingFace] ...
```

Não há badges de:
- Cobertura de código (`codecov.io`, `coveralls.io`)
- Status de CI (`GitHub Actions — build passing`)
- Qualidade de código (`codeclimate`, `sonarcloud`)
- Segurança de dependências (`snyk`, `dependabot`)

**Busca na aba "Security and quality" do GitHub:**

A seção não registra nenhuma ferramenta de análise estática ou scanner de vulnerabilidades configurada pelo mantenedor.

### (b) Diagnóstico

O projeto **não possui nenhuma ferramenta de apoio à qualidade de código** configurada: sem medição de cobertura, sem análise estática, sem scanner de segurança de dependências. A ausência de badges de cobertura e build é consequência direta da ausência de testes e CI. Os badges do README comunicam exclusivamente modelo, paper, blogs e conquistas (github trending, paper, models). Não há o que medir, nem pipeline para gerar relatórios.

### (c) Risco

**Alto.** A ausência de medição de cobertura não é apenas uma lacuna de visibilidade — é um sinal de que **nenhum contrato de qualidade foi estabelecido**. Projetos que nunca mediram cobertura tendem a acumular dívida técnica silenciosamente até que uma refatoração ou novo requisito revele que partes críticas do código nunca foram exercitadas.

### (d) Recomendação

Integrar a medição de cobertura de testes ao pipeline de CI como uma evolução natural, ativando-a **assim que os primeiros testes automatizados forem escritos**. Essa integração deve incluir:

- A geração de relatórios de cobertura em formato padronizado durante a execução da suíte de testes.

- O envio automático desses relatórios a um serviço de monitoramento (ex.: Codecov ou Coveralls), que disponibilizará um *badge* público de cobertura para inserção no `README.md`.

- O estabelecimento, no `CONTRIBUTING.md`, de uma política progressiva de cobertura mínima: uma meta inicial realista (ex.: 40% para os diretórios `inference/` e `evaluation/`) e uma meta de médio prazo (ex.: 70% em seis meses), com a regra de que **nenhum *pull request* pode reduzir a cobertura global** abaixo do limiar vigente.

Essa abordagem transforma a cobertura em um **contrato de qualidade público e auditável**, criando incentivos para que novos códigos venham acompanhados de testes e para que a dívida técnica existente seja gradualmente reduzida.
 
---

## 3. Instruções para rodar os testes (README/docs/scripts)

### (a) Evidência

**Busca no README.md:**

```bash
grep -n "pytest\|test\|coverage\|unittest\|CI\|continuous" README.md
```

**Saída:** *(vazio — nenhuma ocorrência)*

Não existe seção intitulada "Testing", "Development", "Running tests" ou qualquer variante. 
A documentação adicional (FAQ, Tech Blog) também não aborda testes de software.

**Busca por arquivos de documentação de desenvolvimento:**

```bash
find . -name "CONTRIBUTING.md" -o -name "DEVELOPMENT.md" -o -name "TESTING.md"
```

**Saída:** *(vazio)* — Conforme já identificado na A1 (seção 7.1, Eixo GQA), `CONTRIBUTING.md` não existe no repositório.

### (b) Diagnóstico

Não há caminho documentado para um colaborador entender se o projeto tem testes, como executá-los ou qual é o critério de aceitação para um PR. Esse vazio é coerente com a ausência de suíte de testes: literalmente não há nada a documentar.

### (c) Risco

**Médio.** Novos contribuidores não têm um ponto de partida para validar suas alterações. Mudanças são submetidas sem verificação local, o que transfere toda a responsabilidade de detecção de regressões para revisão manual de código.

### (d) Recomendação

Criar o arquivo `CONTRIBUTING.md` (conforme já proposto no plano de melhoria da Atividade 1, Seção 8) com uma seção dedicada a "Running tests", contendo:

1. Instruções para instalação das dependências de desenvolvimento (ex.: `pytest`, `pytest-cov`, `ruff`).

2. O comando para execução da suíte de testes localmente.

3. A política mínima exigida em *pull requests* (ex.: novos módulos devem incluir testes; a cobertura global não pode regredir).

Mesmo que a suíte ainda seja pequena no início, documentar o processo estabelece o hábito e formaliza o contrato de contribuição, sinalizando para a comunidade que o projeto valoriza e aceita contribuições de qualidade.

---

## Síntese — Eixo A (CI/CD e Automação)

| # | Item investigado | Veredito | Risco |
|---|------------------|----------|-------|
| 1 | CI em PRs/pushes | Inexistente — única Action é deploy de página estática | **Alto** |
| 2 | Cobertura, badges, ferramentas de qualidade | Inexistentes — sem codecov, lint config, análise estática | **Alto** |
| 3 | Documentação para rodar testes | Inexistente — README sem seção de testes; sem CONTRIBUTING.md | **Médio** |

**Veredito geral:** o projeto possui **zero automação de qualidade de código**. A única GitHub Action existente serve à página web do projeto. Nenhum PR é verificado automaticamente antes de ser mergeado; nenhuma métrica de cobertura é coletada; nenhuma ferramenta de análise estática está configurada; não há instrução de como rodar testes porque não há testes a rodar. O resultado é que **a qualidade do código depende inteiramente e de forma inconsistente da boa vontade de revisores humanos** — cenário que a A2 — Parte 2 (GPR: gate de qualidade "aprovado seco") já demonstrou ser frágil.

---

## Ligação com as Atividades 1 e 2

- **Atividade 1 — Eixo V&V (Seção 6.2):** a equipe identificou a ausência de workflows de integração contínua voltados à validação do software. A auditoria atual confirma essa situação: a única GitHub Action existente está relacionada exclusivamente ao deploy da página do projeto, sem qualquer etapa de validação do código.
- **Atividade 1 — Eixo GQA (Seção 7.1):** foi apontada a inexistência de um `CONTRIBUTING.md`, indicando que os procedimentos de contribuição e validação eram implícitos. Esta auditoria verifica que essa ausência também se reflete na inexistência de documentação para execução de verificações de qualidade.
- **Atividade 1 — Seção 8 (Plano de Melhoria):** entre as ações priorizadas pela equipe já constavam a criação de um `CONTRIBUTING.md` para padronizar o processo de contribuição e a implantação de um pipeline mínimo de CI. A presente auditoria complementa essas recomendações com a sugestão de incluir uma seção dedicada a "Running tests" nesse documento.
- **Atividade 2 — Parte 2 (GPR):** o relatório revelou o fenômeno do "aprovado seco" — PRs mergeados sem revisão substancial, com dependência de herói e gargalos de integração. Esta auditoria demonstra a causa técnica desse cenário: a ausência de CI/CD elimina qualquer verificação automática antes da integração, tornando o processo inteiramente dependente de revisão manual.
- **Atividade 2 — Parte 6 (Plano de Resgate):** o roadmap proposto pela equipe reforçou a necessidade de institucionalizar práticas de qualidade, incluindo a formalização do processo de contribuição e a implantação de um pipeline mínimo de CI. Os resultados desta auditoria fornecem evidências objetivas de que essas ações permanecem prioritárias e são pré-requisitos para elevar a maturidade do processo de desenvolvimento.
