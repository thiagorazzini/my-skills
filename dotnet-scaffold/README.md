# dotnet-scaffold

Skill interativa para scaffolding de projetos .NET Web API prontos para produção, com infraestrutura Docker completa, testes, monitoramento e scripts operacionais — tudo configurado de acordo com as preferências do usuário.

---

## O que ela faz

Funciona como um **wizard guiado em duas fases**:

1. **Discovery** — faz perguntas ao usuário para entender suas escolhas
2. **Scaffold** — gera todos os arquivos de uma vez, com base nas respostas

Essa abordagem garante consistência: muitas decisões são interconectadas (ex: estratégia de cache afeta `docker-compose.yml`, pacotes NuGet e arquivos de configuração simultaneamente).

### O que é gerado

| Artefato | Detalhes |
|---|---|
| Estrutura da solution | Clean Architecture: `Api`, `Domain`, `Application`, `Infrastructure` |
| Testes unitários | xUnit + FakeItEasy + Coverlet + ReportGenerator |
| Testes de integração | Jest (Node.js) com cliente HTTP e setup/teardown automáticos |
| Docker Compose | PostgreSQL, Redis, RabbitMQ, ElasticSearch, Grafana + Loki |
| Scripts operacionais | `init.sh`, `down.sh`, `check.sh`, `watch.sh`, `test.sh` |
| Scripts tmux | `tmux-dev.sh` (multi-janela) e `tmux-stop.sh` |
| Health check endpoint | `/health` verificando DB, Redis e demais serviços |
| Arquivo de instruções | `CLAUDE.md` ou `AGENTS.md` com documentação completa do projeto |

---

## Como usar

Basta dizer ao Claude que quer criar um novo projeto .NET. A skill é ativada automaticamente por frases como:

- "cria um novo projeto .NET"
- "scaffold de uma API em C#"
- "quero um boilerplate .NET com Docker"
- "configura o backend com PostgreSQL e Redis"
- "set up the backend" (em repositório com .NET no CLAUDE.md)

### Perguntas feitas durante o Discovery

A skill pergunta **uma de cada vez**:

1. **Versão do .NET** — 7, 8, 9 ou 10 (padrão: 10)
2. **Arquivo de instruções** — `CLAUDE.md` (Claude Code/CLI) ou `AGENTS.md` (Cursor, Windsurf)
3. **Docker** — verifica se está instalado; oferece instalação caso não esteja
4. **Banco de dados** — PostgreSQL (padrão: sim)
5. **Cache** — Hybrid (MemoryCache + Redis), Redis only, ou MemoryCache only
6. **Infraestrutura adicional** — RabbitMQ, ElasticSearch, Grafana + Loki (seleção múltipla)
7. **Documentação de API** — Scalar (padrão) ou Swagger
8. **Login social** — Google, Apple, Facebook, etc. (afeta pacotes NuGet e configuração de auth)

Após todas as respostas, apresenta um **resumo das escolhas** e pede confirmação antes de gerar qualquer arquivo.

---

## Estrutura gerada

```
<ProjectName>/
├── src/
│   ├── <ProjectName>.Api/
│   ├── <ProjectName>.Application/
│   ├── <ProjectName>.Domain/
│   └── <ProjectName>.Infrastructure/
├── tests/
│   └── <ProjectName>.UnitTests/
├── integration-tests/
│   ├── __tests__/
│   ├── helpers/
│   ├── setup.js
│   ├── teardown.js
│   └── package.json
├── scripts/
│   ├── init.sh
│   ├── down.sh
│   ├── check.sh
│   ├── watch.sh
│   ├── test.sh
│   ├── tmux-dev.sh
│   └── tmux-stop.sh
├── docker-compose.yml
├── .env.example
├── .gitignore
└── CLAUDE.md (ou AGENTS.md)
```

---

## Scripts operacionais gerados

| Script | O que faz |
|---|---|
| `scripts/init.sh` | Setup completo: copia `.env`, restaura pacotes, sobe Docker, roda migrations, seed e verifica health |
| `scripts/down.sh` | Para tudo; flags `--force`, `--clean`, `--force --clean` |
| `scripts/check.sh` | Verifica status (OK / PENDING / DEGRADED); flag `--fix` para auto-reparo |
| `scripts/watch.sh` | Tail de logs; flag `--ps` para monitorar processos |
| `scripts/test.sh` | Roda testes unitários com cobertura e gera relatório HTML |
| `scripts/tmux-dev.sh` | Abre sessão tmux com 4 janelas: server, docker logs, health monitor, terminal livre |
| `scripts/tmux-stop.sh` | Encerra a sessão tmux |

---

## Referências internas

- [references/docker-compose-templates.md](references/docker-compose-templates.md) — snippets de compose para cada serviço
- [references/tmux-setup.md](references/tmux-setup.md) — templates dos scripts tmux
