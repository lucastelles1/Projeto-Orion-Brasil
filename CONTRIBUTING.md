# 🤝 Guia de Contribuição — Orion Brasil

Obrigado por considerar contribuir com o **Projeto Orion Brasil**! Este documento explica como configurar o ambiente, os padrões que seguimos e como submeter suas contribuições.

---

## 📋 Índice

1. [Código de Conduta](#-código-de-conduta)
2. [Configuração do Ambiente](#-configuração-do-ambiente)
3. [Estrutura de Branches](#-estrutura-de-branches)
4. [Convenção de Commits](#-convenção-de-commits)
5. [Padrões de Código](#-padrões-de-código)
6. [Como Escrever um Novo Scraper](#-como-escrever-um-novo-scraper)
7. [Testes](#-testes)
8. [Pull Requests](#-pull-requests)

---

## 📜 Código de Conduta

Ao participar deste projeto, você concorda com o nosso [Código de Conduta](CODE_OF_CONDUCT.md). Leia-o antes de contribuir.

---

## ⚙️ Configuração do Ambiente

### Pré-requisitos

- **Python 3.11+**
- **Docker 24+** com Docker Compose v2
- **Git**
- Uma API Key do **Anthropic (Claude)** ou **Google (Gemini)**

### Passos

```bash
# 1. Fork e clone o repositório
git clone https://github.com/lucastelles1/Projeto-Orion-Brasil.git
cd Projeto-Orion-Brasil

# 2. Crie um ambiente virtual
python -m venv .venv
source .venv/bin/activate  # Linux/macOS
# .venv\Scripts\activate   # Windows

# 3. Instale as dependências de desenvolvimento
pip install -e ".[dev]"

# 4. Configure as variáveis de ambiente
cp .env.example .env
# Edite o .env com suas API Keys

# 5. Suba a infraestrutura (Neo4j, PostgreSQL, etc.)
docker-compose up -d

# 6. Verifique que tudo está funcionando
pytest --quick
```

---

## 🌿 Estrutura de Branches

Adotamos o seguinte padrão de nomenclatura:

| Prefixo | Uso | Exemplo |
|---|---|---|
| `feature/` | Nova funcionalidade | `feature/scraper-ibama` |
| `fix/` | Correção de bug | `fix/neo4j-connection-timeout` |
| `docs/` | Documentação | `docs/api-endpoints` |
| `refactor/` | Refatoração sem mudança de comportamento | `refactor/entity-resolver` |
| `test/` | Adição ou correção de testes | `test/scraper-tse-unit` |

**Regras:**
- Sempre crie branches a partir da `main` atualizada.
- Use **kebab-case** nos nomes.
- Branches devem ser curtas e focadas em uma única mudança.

---

## 📝 Convenção de Commits

Seguimos o padrão **[Conventional Commits](https://www.conventionalcommits.org/pt-br/)**:

```
<tipo>(escopo opcional): descrição curta

[corpo opcional]

[rodapé opcional]
```

### Tipos permitidos

| Tipo | Descrição |
|---|---|
| `feat` | Nova funcionalidade |
| `fix` | Correção de bug |
| `docs` | Alteração em documentação |
| `style` | Formatação (sem mudança de lógica) |
| `refactor` | Refatoração de código |
| `test` | Adição ou alteração de testes |
| `chore` | Tarefas de manutenção (CI, deps, configs) |
| `perf` | Melhoria de performance |

### Exemplos

```
feat(scraper): adiciona módulo de ingestão do IBAMA
fix(graph): corrige query Cypher para resolução de entidades duplicadas
docs(readme): adiciona seção de pré-requisitos
test(etl): adiciona testes unitários para validação Pydantic
```

---

## 🎨 Padrões de Código

### Python

- **Formatter:** [Black](https://black.readthedocs.io/) (linha máx: 88 caracteres)
- **Linter:** [Ruff](https://docs.astral.sh/ruff/)
- **Tipagem:** Obrigatória em funções públicas (use `mypy --strict` como referência)
- **Docstrings:** Google-style em todas as classes e funções públicas

```python
def resolver_entidade(nome: str, cpf: str | None = None) -> EntidadeResolvida:
    """Resolve uma entidade cruzando múltiplas bases de dados.

    Args:
        nome: Nome completo da entidade.
        cpf: CPF opcional para resolução direta.

    Returns:
        Entidade resolvida com referências cruzadas.

    Raises:
        EntidadeNaoEncontradaError: Se nenhuma base retornar resultados.
    """
```

### Verificação local

```bash
# Rodar todas as verificações de uma vez
make lint    # Ruff + Black --check + mypy
make format  # Black + Ruff --fix
```

---

## 🕷️ Como Escrever um Novo Scraper

A arquitetura de plugins permite adicionar novas fontes de dados sem tocar no core. Siga estes passos:

### 1. Crie o módulo

```
src/scrapers/
├── tse/           # 🆕 Seu novo módulo
├── receita/       # 🆕 Seu novo módulo
└── ibama/         # 🆕 Seu novo módulo
    ├── __init__.py
    ├── scraper.py      # Lógica de extração
    ├── models.py       # Schemas Pydantic
    ├── transforms.py   # Transformações para o grafo
    └── tests/
        ├── test_scraper.py
        └── fixtures/   # Dados de exemplo para testes
```

### 2. Implemente a interface base

```python
from orion.scrapers.base import BaseScraper, ScraperResult

class IbamaScraper(BaseScraper):
    """Scraper para multas ambientais do IBAMA."""

    name = "ibama"
    source_url = "https://dados.ibama.gov.br/"

    async def extract(self) -> list[ScraperResult]:
        """Extrai dados brutos da fonte."""
        ...

    async def transform(self, raw: list[ScraperResult]) -> list[GraphEntity]:
        """Transforma dados brutos em entidades do grafo."""
        ...
```

### 3. Registre no Dagster

Adicione o asset correspondente em `src/pipelines/assets.py`.

### 4. Escreva testes

- Mínimo: testes unitários para `extract()` e `transform()`.
- Use fixtures com dados reais anonimizados — **nunca** faça requests HTTP em testes unitários.

---

## 🧪 Testes

```bash
# Rodar todos os testes
pytest

# Apenas testes rápidos (sem Docker/DB)
pytest --quick

# Com cobertura
pytest --cov=src --cov-report=html

# Apenas um módulo
pytest src/scrapers/tse/tests/
```

**Regras:**
- Todo PR **deve** manter a cobertura acima de **80%** nas linhas alteradas.
- Testes de integração (que dependem de Neo4j/PostgreSQL) devem ser marcados com `@pytest.mark.integration`.
- Use `pytest-asyncio` para testes assíncronos.

---

## 🔀 Pull Requests

### Antes de abrir um PR

1. ✅ Seus testes passam localmente (`pytest`)
2. ✅ O linter não reclama (`make lint`)
3. ✅ Você escreveu/atualizou testes para suas mudanças
4. ✅ Você atualizou a documentação se necessário

### Template do PR

Ao abrir um PR, preencha o template abaixo:

```markdown
## Descrição
<!-- O que este PR faz? Por que é necessário? -->

## Tipo de mudança
- [ ] `feat`: Nova funcionalidade
- [ ] `fix`: Correção de bug
- [ ] `docs`: Documentação
- [ ] `refactor`: Refatoração
- [ ] `test`: Testes

## Como testar
<!-- Passos para o reviewer validar sua mudança -->

## Checklist
- [ ] Testes passando (`pytest`)
- [ ] Linter limpo (`make lint`)
- [ ] Cobertura ≥ 80% nas linhas alteradas
- [ ] Documentação atualizada (se aplicável)

## Screenshots (se aplicável)
<!-- Cole screenshots de grafos, logs ou dashboards -->
```

### Processo de review

- Todo PR precisa de **pelo menos 1 aprovação** antes do merge.
- Mantenedores podem solicitar alterações — não leve para o lado pessoal! 😊
- Usamos **Squash and Merge** para manter o histórico limpo.

---

## ❓ Dúvidas?

Abra uma [Discussion](https://github.com/lucastelles1/Projeto-Orion-Brasil/discussions) no GitHub. Evite usar Issues para perguntas — Issues são para bugs e feature requests.

---

<div align="center">

**Obrigado por ajudar a tornar os dados públicos mais transparentes! 🇧🇷**

</div>
