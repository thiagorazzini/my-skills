# dotnet-scaffold

An interactive skill that scaffolds production-ready .NET Web API projects with full Docker infrastructure, testing setup, monitoring, and operational scripts — all configured to the user's preferences.

---

## What it does

Works as a **guided wizard in two phases**:

1. **Discovery** — asks the user a series of questions to understand their preferences
2. **Scaffold** — generates all files at once based on the collected answers

This approach ensures consistency: many decisions are interconnected (e.g., cache strategy affects `docker-compose.yml`, NuGet packages, and configuration files simultaneously).

### What gets generated

| Artifact | Details |
|---|---|
| Solution structure | Clean Architecture: `Api`, `Domain`, `Application`, `Infrastructure` |
| Unit tests | xUnit + FakeItEasy + Coverlet + ReportGenerator |
| Integration tests | Jest (Node.js) with HTTP client and automatic setup/teardown |
| Docker Compose | PostgreSQL, Redis, RabbitMQ, ElasticSearch, Grafana + Loki |
| Operational scripts | `init.sh`, `down.sh`, `check.sh`, `watch.sh`, `test.sh` |
| tmux scripts | `tmux-dev.sh` (multi-window) and `tmux-stop.sh` |
| Health check endpoint | `/health` checking DB, Redis and other services |
| Instructions file | `CLAUDE.md` or `AGENTS.md` with full project documentation |

---

## How to use

Just tell Claude you want to create a new .NET project. The skill is triggered automatically by phrases like:

- "create a new .NET project"
- "scaffold a C# API"
- "I want a .NET boilerplate with Docker"
- "set up the backend with PostgreSQL and Redis"
- "set up the backend" (in a repo that has .NET in CLAUDE.md)

### Questions asked during Discovery

The skill asks **one at a time**:

1. **.NET Version** — 7, 8, 9, or 10 (default: 10)
2. **Instructions file** — `CLAUDE.md` (Claude Code/CLI) or `AGENTS.md` (Cursor, Windsurf)
3. **Docker** — checks if it's installed; offers to install it if not
4. **Database** — PostgreSQL (default: yes)
5. **Cache** — Hybrid (MemoryCache + Redis), Redis only, or MemoryCache only
6. **Additional infrastructure** — RabbitMQ, ElasticSearch, Grafana + Loki (multi-select)
7. **API documentation** — Scalar (default) or Swagger
8. **Social login** — Google, Apple, Facebook, etc. (affects NuGet packages and auth config)

After all answers, presents a **summary of choices** and asks for confirmation before generating any files.

---

## Generated structure

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
└── CLAUDE.md (or AGENTS.md)
```

---

## Generated operational scripts

| Script | What it does |
|---|---|
| `scripts/init.sh` | Full setup: copies `.env`, restores packages, starts Docker, runs migrations, seeds and checks health |
| `scripts/down.sh` | Stops everything; flags `--force`, `--clean`, `--force --clean` |
| `scripts/check.sh` | Checks status (OK / PENDING / DEGRADED); flag `--fix` for auto-repair |
| `scripts/watch.sh` | Tails logs; flag `--ps` to monitor processes |
| `scripts/test.sh` | Runs unit tests with coverage and generates HTML report |
| `scripts/tmux-dev.sh` | Opens tmux session with 4 windows: server, docker logs, health monitor, free terminal |
| `scripts/tmux-stop.sh` | Terminates the tmux session |

---

## Internal references

- [references/docker-compose-templates.md](references/docker-compose-templates.md) — compose snippets for each service
- [references/tmux-setup.md](references/tmux-setup.md) — tmux script templates
