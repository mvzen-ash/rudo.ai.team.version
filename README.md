# Rudo.ai — Team Version

> AI-driven security assessment platform that helps organizations evaluate the security of their systems through a set of specialized scanning agents, managed from a single dashboard.

![CI](https://github.com/mvzen-ash/rudo.ai.team.version/actions/workflows/ci_checks.yml/badge.svg)
**Repo:** https://github.com/mvzen-ash/rudo.ai.team.version

## What it is

One dashboard, four specialized scanning agents, and an AI brain that plans each scan, analyzes the results, and answers questions through an in-app assistant. Built for an IT employee at any organization — small company to bank — who needs a full, prioritized security report without being a pentest expert.

| Agent | Does |
|---|---|
| **Network** | Maps the network, runs known attack checks, visualizes it for the specialist |
| **Web App** | Same idea, scoped to the company's own web application |
| **Advanced** | Deeper scan profile with more features |
| **Vulnerability Searching** | Pulls CVE/severity from trusted sources, produces the technical report |

## Tech stack

| Layer | Choice |
|---|---|
| Frontend | React 19 + Vite, built in Docker, served by Nginx, installable as a PWA |
| Backend | C# .NET 8 + ASP.NET Core |
| Database | SQL Server 2022 |
| Background jobs | Redis-backed Hangfire |
| AI | Ollama (llama3) on the host machine, Groq API as cloud fallback |
| Web server | Nginx — reverse proxy, static files, WebSocket, PWA headers |
| Orchestration | Docker Compose |
| CI/CD | GitHub Actions + Postman/Newman API checks |

## Network

Port 80 (Nginx) → serves frontend + proxies /api/* to backend
Port 5000 (ASP.NET Core) → internal only
Port 1433 (SQL Server) → internal only
Port 6379 (Redis) → internal only
Port 11434 (Ollama) → on the HOST machine, outside Docker


## Repo structure

rudo.ai.team.version/
├── app/ # Frontend (React + Vite)
├── backend/
│ └── Rudo.Api/ # ASP.NET Core app, agents, AI brain
├── tests/ # Postman collection
├── docs/
├── scripts/
│ ├── run_platform.sh / .bat
│ └── setup_local_ollama.sh / .bat
├── .github/workflows/ci_checks.yml
├── docker-compose.yml
├── docker-compose.dev.yml
├── nginx.conf
├── .env.example
├── .gitignore
└── README.md


## Team roles

| Role | Owns |
|---|---|
| UI/UX Designer | `app/src/` (design) |
| Frontend | `app/src/` (implementation) |
| Backend & DB | `backend/Rudo.Api/` (core, EF Core schema) |
| DevOps | `docker-compose*.yml`, `nginx.conf`, `.github/`, `scripts/`, `.env.example`, `.gitignore` |
| Security Engineer(s) | `backend/Rudo.Api/` (agents, AI, hardening) |

## Getting started

```bash
git clone https://github.com/mvzen-ash/rudo.ai.team.version.git
cd rudo.ai.team.version
cp .env.example .env            # fill in secrets
./scripts/setup_local_ollama.sh # pulls llama3 via Ollama on the host
docker compose up --build -d
# open http://localhost
```

Dev mode (hot-reload):
```bash
docker compose -f docker-compose.yml -f docker-compose.dev.yml up
```

## Branch strategy

- `main` — protected, PR + review + green CI required
- `develop` — integration branch, all feature branches merge here first
- `feature/ui-ux`, `feature/frontend`, `feature/backend-db`, `feature/devops`, `feature/security`

## CI

`.github/workflows/ci_checks.yml` runs on every push/PR to `develop`/`main`:
1. `frontend-check` — lint + build the React app
2. `backend-check` — build + test the ASP.NET Core app
3. `api-contract-check` — runs the Postman collection against the live API via Newman
4. `docker-build-check` — full `docker compose up` + `curl /api/health`

## Golden rules (zero-conflict)

1. DevOps never touches `app/src/` or `backend/Rudo.Api/` core+security code.
2. Coordinate before changing `.csproj` (Backend) or `package.json` (Frontend).
3. Each role works only on its own feature branch.
