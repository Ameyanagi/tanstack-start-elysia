# init-tanstack-elysia

An [Agent Skill](https://agentskills.io) that initializes a full-stack monorepo with **TanStack Start + shadcn/ui** (frontend) and **ElysiaJS** (backend).

## Install

```bash
npx skills add <owner>/tanstack-start-elysia
```

Or install for a specific agent:

```bash
npx skills add <owner>/tanstack-start-elysia -a claude-code
```

## Usage

Once installed, invoke the skill:

```
/init-tanstack-elysia
```

The skill will scaffold a complete monorepo with:

- **Frontend**: TanStack Start + shadcn/ui (Mira style, Indigo theme) + Biome + tsc
- **Backend**: ElysiaJS + Better Auth (username + password) + Drizzle ORM (PostgreSQL)
- **API Client**: Eden Treaty for end-to-end type safety (no codegen)
- **Task Runner**: justfile with `dev`, `check`, `fix`, `test`, `gen-api` commands
- **Production**: Docker multi-stage builds + docker-compose

## Generated Project Structure

```
project/
├── package.json              # Root Bun workspace
├── bun.lock
├── justfile
├── frontend/
│   ├── app/                  # TanStack Start routes
│   ├── src/
│   │   ├── lib/
│   │   │   ├── auth-client.ts  # Better Auth client
│   │   │   └── api.ts          # Eden Treaty client
│   │   └── components/ui/      # shadcn components
│   ├── biome.json
│   ├── Dockerfile
│   └── package.json
├── backend/
│   ├── src/
│   │   ├── app.ts            # Elysia app
│   │   ├── auth.ts           # Better Auth config
│   │   ├── db/               # Drizzle schema + client
│   │   └── routes/
│   ├── drizzle.config.ts
│   ├── biome.json
│   ├── Dockerfile
│   └── package.json
├── docker-compose.yml        # Production
├── docker-compose.dev.yml    # Dev PostgreSQL
├── .env.example
└── .gitignore
```

## Prerequisites

The following tools must be installed before running the skill:

- [Bun](https://bun.sh) >= 1.2
- [just](https://github.com/casey/just) >= 1.0
- [jj](https://jj-vcs.github.io/jj/) >= 0.25
- [Docker](https://www.docker.com/) >= 24.0

## Tooling Overview

| Concern    | Frontend                  | Backend                  |
| ---------- | ------------------------- | ------------------------ |
| Runtime    | Bun                       | Bun                      |
| Framework  | TanStack Start            | ElysiaJS                 |
| Auth       | Better Auth client SDK    | Better Auth server       |
| DB         | —                         | Drizzle ORM + PostgreSQL |
| API Client | Eden Treaty               | —                        |
| Lint       | Biome                     | Biome                    |
| Format     | Biome                     | Biome                    |
| Typecheck  | tsc                       | tsc                      |
| Test       | Vitest                    | Bun test                 |

## License

MIT
