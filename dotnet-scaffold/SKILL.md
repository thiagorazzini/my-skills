---
name: dotnet-scaffold
description: >
  Interactive .NET project scaffolder with Docker infrastructure. Use this skill whenever the user wants to
  create a new .NET project, scaffold a .NET API, set up a .NET boilerplate, bootstrap a backend with Docker,
  or start a new C# web API project. Also triggers when the user mentions needing PostgreSQL + Redis + Docker
  setup for .NET, or wants to scaffold infrastructure with docker-compose for a .NET backend. Even if they
  just say "create the project" or "set up the backend" in a repo that has .NET in its CLAUDE.md, use this skill.
---

# dotnet-scaffold

An interactive skill that scaffolds a production-ready .NET Web API project with full Docker infrastructure,
testing setup, monitoring, and operational scripts — all configured to the user's preferences.

The skill works as a guided wizard: it asks questions, collects choices, then generates everything at once.
This approach matters because many decisions are interconnected (e.g., cache strategy affects docker-compose,
.NET packages, and configuration files simultaneously).

---

## Workflow Overview

The skill follows two phases:

1. **Discovery Phase** — Ask the user a series of questions to understand their preferences
2. **Scaffold Phase** — Generate all files based on collected answers

Never start generating files until all questions are answered. Present a summary of choices before scaffolding.

---

## Phase 1: Discovery

Ask these questions **one at a time**, waiting for each answer before proceeding. Group related questions
when it feels natural (e.g., cache questions together), but don't overwhelm with a wall of text.

### Q1: .NET Version

Ask which .NET version to use. Only accept **7, 8, 9, or 10**. Default suggestion: **.NET 10** (latest).

Example:
> Which .NET version would you like? (7 / 8 / 9 / 10)
> Default: 10

### Q2: Instruction File

Ask whether project documentation and AI instructions should go in `CLAUDE.md` or `AGENTS.md`.
Explain briefly: CLAUDE.md is for Claude Code / Claude CLI users, AGENTS.md is for Cursor, Windsurf,
or other IDE agents.

> Where should I document the project setup and AI instructions?
> - **CLAUDE.md** (Claude Code / CLI)
> - **AGENTS.md** (Cursor, Windsurf, other IDEs)

### Q3: Docker Check

Before asking about infrastructure, check if Docker is installed:

```bash
docker --version 2>/dev/null
docker compose version 2>/dev/null
```

If Docker is **not installed**, inform the user and ask:
> Docker doesn't seem to be installed. Would you like me to install it?
> (I'll use the official Docker Engine install script for your OS)

If they say yes, install Docker using the official convenience script for Linux or guide them for macOS/Windows.

### Q4: Database

Ask if they want **PostgreSQL** as their database (this is the expected default for this project).

> Use PostgreSQL as the primary database? (Y/n)

### Q5: Cache Strategy

Ask about caching approach:

> Which caching strategy would you like?
> 1. **Hybrid** (MemoryCache + Redis) — recommended for session + distributed cache
> 2. **Redis only** — all caching goes through Redis
> 3. **MemoryCache only** — in-process only, no distributed cache

Default: **Hybrid** (this matches the HomeShield HMS spec in CLAUDE.md).

### Q6: Message Broker & Observability

Ask about optional infrastructure components. Present as a checklist:

> Which additional infrastructure do you need? (comma-separated, or "none")
> 1. **RabbitMQ** — message broker for async processing
> 2. **ElasticSearch** — full-text search and log indexing
> 3. **Grafana + Loki** — observability (logs, metrics, dashboards)
>
> Example: `1,3` or `all` or `none`

### Q7: API Documentation

> Which API documentation tool?
> - **Scalar** (default, modern UI)
> - **Swagger** (classic Swashbuckle)

Default: **Scalar**.

### Q8: Social Login

> Will the project need social login (Google, Apple, Facebook, etc.)?
> If yes, which providers?

This affects NuGet packages and auth configuration.

### Summary

After all questions, present a summary table:

```
Your choices:
  .NET Version:      10
  Instruction File:  CLAUDE.md
  Database:          PostgreSQL
  Cache:             Hybrid (MemoryCache + Redis)
  Message Broker:    RabbitMQ
  Search:            ElasticSearch
  Observability:     Grafana + Loki
  API Docs:          Scalar
  Social Login:      Google, Apple
```

Ask: **"Does this look right? Should I proceed with scaffolding?"**

---

## Phase 2: Scaffold

Once confirmed, generate everything below. Use the project name from the current directory or ask if unclear.

### 2.1 .NET Project Structure

Create the solution using `dotnet new`:

```bash
dotnet new sln -n <ProjectName>
dotnet new webapi -n <ProjectName>.Api -o src/<ProjectName>.Api
dotnet new classlib -n <ProjectName>.Domain -o src/<ProjectName>.Domain
dotnet new classlib -n <ProjectName>.Infrastructure -o src/<ProjectName>.Infrastructure
dotnet new classlib -n <ProjectName>.Application -o src/<ProjectName>.Application
dotnet new xunit -n <ProjectName>.UnitTests -o tests/<ProjectName>.UnitTests
```

Add all projects to the solution. Set up project references following Clean Architecture:
- Api → Application
- Application → Domain
- Infrastructure → Application, Domain
- Api → Infrastructure

#### NuGet Packages

**Always install:**
- `Microsoft.EntityFrameworkCore` + provider for chosen DB
- `Microsoft.Extensions.Caching.StackExchangeRedis` (if Redis chosen)
- `Microsoft.Extensions.Caching.Memory` (if MemoryCache chosen)

**API Docs:**
- Scalar: `Scalar.AspNetCore`
- Swagger: `Swashbuckle.AspNetCore`

**Testing (UnitTests project):**
- `xunit` + `xunit.runner.visualstudio`
- `FakeItEasy`
- `coverlet.msbuild` + `coverlet.collector`
- `ReportGenerator` (as dotnet tool)

**Social Login (if selected):**
- `Microsoft.AspNetCore.Authentication.Google`
- `Microsoft.AspNetCore.Authentication.Facebook`
- `AspNet.Security.OAuth.Apple` (for Apple Sign-In)

**Conditional:**
- `RabbitMQ.Client` (if RabbitMQ selected)
- `NEST` or `Elastic.Clients.Elasticsearch` (if ElasticSearch selected)
- `Serilog.Sinks.Grafana.Loki` (if Grafana+Loki selected)

### 2.2 Docker Compose

Create `docker-compose.yml` at the project root. All infrastructure runs in a single cluster.

Read `references/docker-compose-templates.md` for the compose snippets for each service.

The compose file should include:
- A custom Docker network (`homeshield-net` or similar)
- Named volumes for data persistence
- Health checks on every service
- Environment variables loaded from `.env`

### 2.3 Operational Scripts

Create all scripts in `/scripts/` with `chmod +x`:

#### `scripts/init.sh`
1. Copy `.env.example` → `.env` if not exists
2. Run `dotnet restore`
3. Start Docker Compose services
4. Wait for DB health check
5. Run EF Core migrations (`dotnet ef database update`)
6. Seed admin user (basic seed)
7. Start the .NET server in background
8. Run HTTP health check against `/health`
9. Report status

#### `scripts/down.sh`
- No flags: `dotnet` process graceful shutdown + `docker compose stop`
- `--force`: `kill -9` on dotnet, playwright, jest, next-server orphan processes
- `--clean`: Stop + remove `.env.local`, `bin/`, `obj/`, `node_modules/` (for integration tests)
- `--force --clean`: Both

#### `scripts/check.sh`
- No flags: Show status (OK / PENDING / DEGRADED) by checking:
  - dotnet process running?
  - Docker containers healthy?
  - HTTP GET `/health` returns 200?
- `--fix`: Auto-fix by killing orphans, restarting degraded services

#### `scripts/watch.sh`
- No flags: `tail -f` on server logs
- `--ps`: Show running processes related to the project (refreshes every 1s via `watch`)

### 2.4 tmux Scripts

Read `references/tmux-setup.md` for the tmux script templates.

Create `scripts/tmux-dev.sh`:
- Window 1: .NET server logs (`dotnet watch run`)
- Window 2: Docker container logs (`docker compose logs -f`)
- Window 3: Health check loop (`watch -n 5 scripts/check.sh`)
- Window 4: Free terminal for commands

Create `scripts/tmux-stop.sh`:
- Kill the tmux session gracefully

### 2.5 Integration Tests (Jest)

Create `integration-tests/` directory at root with:

```
integration-tests/
├── package.json
├── jest.config.js
├── .env.test
├── setup.js           # Global setup (start server, wait for health)
├── teardown.js        # Global teardown (stop server)
├── helpers/
│   └── api-client.js  # Axios wrapper with base URL + auth
└── __tests__/
    └── health.test.js # Smoke test: GET /health → 200
```

**package.json** should include:
- `jest` + `@types/jest`
- `axios` for HTTP requests
- `dotenv` for env loading
- Scripts: `test`, `test:watch`, `test:ci`

### 2.6 Unit Test Configuration

In the xUnit test project, configure:
- `coverlet.msbuild` in the `.csproj` for code coverage
- Add `dotnet tool install -g dotnet-reportgenerator-globaltool` to `init.sh`
- Create `scripts/test.sh` that runs:
  ```bash
  dotnet test --collect:"XPlat Code Coverage"
  reportgenerator -reports:**/coverage.cobertura.xml -targetdir:coverage-report
  ```

### 2.7 Environment Files

Create `.env.example` with all required environment variables (DB connection string, Redis URL,
RabbitMQ URL, etc.) — values set to `CHANGE_ME` or localhost defaults.

Create `.gitignore` additions for `.env`, coverage reports, Docker volumes.

### 2.8 Instruction File (CLAUDE.md or AGENTS.md)

Write comprehensive instructions in the chosen file. Document:

1. **Project overview** — what this project is, the tech stack
2. **All user choices** — cache strategy, infra components, API docs tool, etc.
3. **How to start** — `scripts/init.sh`
4. **How to stop** — `scripts/down.sh` with flag explanations
5. **How to check status** — `scripts/check.sh`
6. **How to monitor** — `scripts/watch.sh` and `scripts/tmux-dev.sh`
7. **Testing**:
   - Unit tests: `dotnet test` with xUnit, FakeItEasy, Coverlet
   - Integration tests: `cd integration-tests && npm test` with Jest
   - Coverage: `scripts/test.sh`
8. **Jest setup guide** — What dependencies to install, how to run, how to write tests,
   example test structure
9. **Docker services** — what's running, ports, how to access
10. **Social login setup** (if chosen) — what env vars to configure, OAuth app setup links

### 2.9 Health Check Endpoint

Add a `/health` endpoint to the .NET API that checks:
- Database connectivity
- Redis connectivity (if used)
- Returns JSON with component statuses

---

## Important Notes

- Always use `AskUserQuestion` tool for the discovery questions — never assume answers
- Generate all files in a single pass after confirmation to avoid partial states
- Make all shell scripts executable with `chmod +x`
- Use `LF` line endings in all `.sh` files (not CRLF)
- Test that `dotnet restore` succeeds after generating the project
- If any `dotnet new` command fails, check if the SDK version is installed and inform the user
