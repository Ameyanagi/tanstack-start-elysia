# init-tanstack-elysia

An [Agent Skill](https://agentskills.io) for scaffolding a full-stack monorepo with **TanStack Start + shadcn/ui** (frontend) and **ElysiaJS** (backend). Agent Skills are reusable automation recipes for AI coding agents like [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — invoke the skill and the agent builds the entire project for you.

## Install

```bash
npx skills add Ameyanagi/tanstack-start-elysia
```

Or install for Claude Code specifically:

```bash
npx skills add Ameyanagi/tanstack-start-elysia -a claude-code
```

## Usage

Once installed, invoke the skill in your agent:

```
/init-tanstack-elysia
```

The skill scaffolds a complete monorepo with:

- **Frontend**: TanStack Start + shadcn/ui (Mira style, Indigo theme) + Biome + tsc
- **Backend**: ElysiaJS + Better Auth (username + password) + Drizzle ORM (PostgreSQL)
- **API Client**: Eden Treaty for end-to-end type safety (no codegen)
- **Task Runner**: justfile with `dev`, `check`, `fix`, `test`, `gen-api` commands
- **Production**: Docker multi-stage builds + docker-compose

## How It Works

The skill performs these steps automatically:

1. **Version Control** — initializes [Jujutsu](https://jj-vcs.github.io/jj/) colocated with Git
2. **Frontend** — scaffolds TanStack Start via `shadcn create`, replaces ESLint/Prettier with Biome, adds Eden Treaty client and Better Auth client SDK
3. **Backend** — creates an ElysiaJS app with Better Auth (username + password), Drizzle ORM, Swagger docs, and CORS
4. **Database** — generates Better Auth schema, pushes to PostgreSQL via Drizzle Kit
5. **OpenAPI** — generates and validates OpenAPI spec from the Elysia app
6. **Tooling** — creates justfile, Docker configs, CI workflow, editor settings
7. **Verification** — runs `just check` (Biome + tsc + OpenAPI validation) and `just test`

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
│   │   │   ├── auth-client.ts  # Better Auth client SDK
│   │   │   └── api.ts          # Eden Treaty client
│   │   ├── components/ui/      # shadcn components
│   │   └── __tests__/
│   ├── biome.json
│   ├── vitest.config.ts
│   ├── tsconfig.json
│   ├── Dockerfile
│   └── package.json
├── backend/
│   ├── src/
│   │   ├── app.ts            # Elysia app (exports type for Eden)
│   │   ├── index.ts          # Entry point
│   │   ├── auth.ts           # Better Auth config
│   │   ├── db/               # Drizzle schema + client
│   │   └── routes/
│   ├── scripts/              # OpenAPI generation + validation
│   ├── tests/
│   ├── drizzle.config.ts
│   ├── biome.json
│   ├── tsconfig.json
│   ├── openapi.json          # Generated OpenAPI spec
│   ├── Dockerfile
│   └── package.json
├── docker-compose.yml        # Production
├── docker-compose.dev.yml    # Dev PostgreSQL
├── .vscode/                  # Editor settings + extensions
├── .github/workflows/ci.yml  # GitHub Actions CI
├── .editorconfig
├── .env.example
├── .env
└── .gitignore
```

## Prerequisites

**Required:**

- [Bun](https://bun.sh) >= 1.2 — JavaScript runtime and package manager
- [just](https://github.com/casey/just) >= 1.0 — task runner (replaces Make)
- [jj](https://jj-vcs.github.io/jj/) >= 0.25 — version control (colocated with Git)

**Optional:**

- [Docker](https://www.docker.com/) >= 24.0 — required for local PostgreSQL and production builds. The skill warns if missing but continues.

## Next Steps

After the skill completes:

1. Review `.env` and update `BETTER_AUTH_SECRET` with a strong random value
2. Run `just dev` to start PostgreSQL, frontend, and backend
3. Access the app:
   - Frontend: http://localhost:3000
   - Backend API: http://localhost:3001
   - Swagger docs: http://localhost:3001/docs
4. Run `just db-studio` to browse the database

## Tooling Overview

| Concern    | Frontend                  | Backend                  |
| ---------- | ------------------------- | ------------------------ |
| Runtime    | Bun                       | Bun                      |
| Framework  | TanStack Start            | ElysiaJS                 |
| Auth       | Better Auth client SDK    | Better Auth server       |
| DB         | —                         | Drizzle ORM + PostgreSQL |
| Migrations | —                         | Drizzle Kit              |
| API Client | Eden Treaty               | —                        |
| Lint       | Biome                     | Biome                    |
| Format     | Biome                     | Biome                    |
| Typecheck  | tsc                       | tsc                      |
| Test       | Vitest                    | Bun test                 |

## Documentation

- [SKILL.md](skills/init-tanstack-elysia/SKILL.md) — full automation steps
- [references/stack-details.md](skills/init-tanstack-elysia/references/stack-details.md) — all generated configuration file contents
- [TanStack Start](https://tanstack.com/start) | [shadcn/ui](https://ui.shadcn.com/) | [ElysiaJS](https://elysiajs.com/) | [Better Auth](https://www.better-auth.com/) | [Drizzle ORM](https://orm.drizzle.team/) | [Eden Treaty](https://elysiajs.com/eden/overview)

## License

[MIT](LICENSE)
