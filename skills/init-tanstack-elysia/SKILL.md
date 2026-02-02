---
name: init-tanstack-elysia
description: Initialize a full-stack monorepo with TanStack Start + shadcn/ui frontend (Bun, Biome, tsc) and ElysiaJS backend with Better Auth (username + password), Drizzle ORM (PostgreSQL), and Eden Treaty for end-to-end type safety. Includes a justfile task runner and Docker production setup. Use when creating a new full-stack project, scaffolding a monorepo, or setting up TanStack Start with ElysiaJS.
allowed-tools: Bash Read Write
---

# Initialize TanStack Start + ElysiaJS Monorepo

Create a production-ready Bun workspace monorepo with a TanStack Start frontend and ElysiaJS backend.

For detailed configuration file contents, see [references/stack-details.md](references/stack-details.md).

## Prerequisites

Verify these tools are installed before proceeding:

```bash
bun --version      # >= 1.2
just --version     # >= 1.0
jj --version       # >= 0.25
docker --version   # >= 24.0 (optional, for production builds)
```

If any required tool is missing (bun, just, jj), stop and inform the user. Docker is optional — warn if missing but continue.

## Step 1: Initialize Version Control

```bash
jj git init --colocate
```

Create `.jj/.gitignore` with content `/*` so git ignores jj metadata.

## Step 2: Scaffold the Frontend

Run the shadcn create command to scaffold a TanStack Start project:

```bash
bunx --bun shadcn@latest create \
  --preset "https://ui.shadcn.com/init?base=base&style=mira&baseColor=neutral&theme=indigo&iconLibrary=lucide&font=geist&menuAccent=subtle&menuColor=default&radius=medium&template=start&rtl=false" \
  --template start \
  frontend
```

After scaffolding completes:

1. Remove the nested git repository created by shadcn (the project uses the root-level jj/git repo):
   ```bash
   rm -rf frontend/.git
   ```

2. Remove ESLint configuration if present (the project uses Biome instead):
   - Delete `eslint.config.js` or `.eslintrc.*` if they exist
   - Remove `eslint`, `@eslint/*`, and `prettier` packages from `package.json` devDependencies
   - Remove any eslint/prettier scripts from `package.json`

3. Install Biome and additional dependencies:
   ```bash
   cd frontend && bun add -d @biomejs/biome vitest jsdom
   cd frontend && bun add @elysiajs/eden @tanstack/react-query better-auth
   ```

4. Create `frontend/biome.json` with the configuration from stack-details.md section "Biome Configuration (Frontend)".

5. Auto-fix the scaffolded code to conform to Biome rules (formatting, imports):
   ```bash
   cd frontend && bunx biome check --write --unsafe .
   ```

6. Update the `name` field in `frontend/package.json` to `"@repo/frontend"`.

7. Create `frontend/src/lib/auth-client.ts` with the Better Auth client from stack-details.md section "Better Auth Client".

8. Create `frontend/src/lib/api.ts` with the Eden Treaty client from stack-details.md section "Eden Treaty Client".

9. Create `frontend/vitest.config.ts` and `frontend/src/__tests__/smoke.test.ts` from stack-details.md section "Frontend Test Configuration".

10. Update `frontend/tsconfig.json`:
    - Add `"@repo/backend/*": ["../backend/*"]` to `compilerOptions.paths` (needed for Eden Treaty type imports)
    - Remove `eslint.config.js` and `prettier.config.js` from the `include` array if present

11. Merge these scripts into `frontend/package.json` (preserve existing scripts from scaffolding):
    ```json
    {
      "scripts": {
        "check": "biome check .",
        "check:fix": "biome check --write .",
        "typecheck": "tsc --noEmit",
        "test": "vitest run"
      }
    }
    ```

## Step 3: Create Root Workspace

1. Create `package.json` in the project root from stack-details.md section "Root Package JSON".

2. Run `bun install` from the project root to link workspaces:
   ```bash
   bun install
   ```

## Step 4: Scaffold the Backend

1. Create `backend/package.json` from stack-details.md section "Backend Package JSON".

2. Install backend dependencies:
   ```bash
   cd backend && bun install
   ```

3. Create the backend application structure:
   ```
   backend/
   ├── src/
   │   ├── app.ts            # Elysia app (without listen)
   │   ├── index.ts           # Entry point (app.listen)
   │   ├── auth.ts            # Better Auth config
   │   ├── db/
   │   │   ├── index.ts       # Drizzle client
   │   │   └── schema.ts      # App-specific schema
   │   └── routes/
   │       └── health.ts      # Health check endpoint
   ├── scripts/
   │   ├── generate-openapi.ts
   │   └── validate-openapi.ts
   └── tests/
       └── health.test.ts
   ```

4. Write `backend/src/db/index.ts` with the Drizzle client from stack-details.md section "Drizzle Client".

5. Write `backend/src/auth.ts` with the Better Auth config from stack-details.md section "Better Auth Server Config".

6. Write `backend/src/routes/health.ts` with the health endpoint from stack-details.md section "Health Route".

7. Write `backend/src/app.ts` with the Elysia app from stack-details.md section "Elysia App".

8. Write `backend/src/index.ts` with the entry point from stack-details.md section "Backend Entry Point".

9. Write `backend/drizzle.config.ts` from stack-details.md section "Drizzle Config".

10. Create `backend/biome.json` from stack-details.md section "Biome Configuration (Backend)".

11. Create `backend/tsconfig.json` from stack-details.md section "Backend TSConfig".

12. Write `backend/scripts/generate-openapi.ts` from stack-details.md section "OpenAPI Generation Script".

13. Write `backend/scripts/validate-openapi.ts` from stack-details.md section "OpenAPI Validation Script".

14. Create `backend/.gitignore` from stack-details.md section "Backend Gitignore".

15. Write `backend/tests/health.test.ts` from stack-details.md section "Backend Tests".

16. Auto-fix the backend code to conform to Biome rules (formatting, imports):
    ```bash
    cd backend && bunx biome check --write --unsafe .
    ```

## Step 5: Generate Better Auth Schema & Initial Migration

1. Generate the Better Auth database schema:
   ```bash
   cd backend && bunx @better-auth/cli generate --config ./src/auth.ts --output ./src/db/auth-schema.ts
   ```

2. Import the generated auth schema in `backend/src/db/schema.ts`. See stack-details.md section "App Schema" for the pattern.

3. Start the dev PostgreSQL database:
   ```bash
   docker compose -f docker-compose.dev.yml up -d
   ```

4. Wait for PostgreSQL to be ready, then push the schema to the database:
   ```bash
   cd backend && bunx drizzle-kit push
   ```

## Step 6: Generate Initial OpenAPI Schema

```bash
cd backend && bun run scripts/generate-openapi.ts
```

## Step 7: Create the Root Justfile

Create `justfile` in the project root with the content from stack-details.md section "Justfile".

Key recipes:
- `set dotenv-load` to read `.env`
- `dev` spawns dev database, frontend, and backend in parallel
- `dev-frontend` / `dev-backend` / `dev-db` for individual services
- `check` runs all linters, formatters, type checkers, and OpenAPI validation
- `fix` auto-fixes lint and format issues
- `gen-api` generates OpenAPI schema
- `db-push` / `db-migrate` / `db-studio` for database management

## Step 8: Create Docker Configuration

1. Create `docker-compose.dev.yml` from stack-details.md section "Docker Compose Dev".
2. Create `frontend/Dockerfile` from stack-details.md section "Frontend Dockerfile".
3. Create `backend/Dockerfile` from stack-details.md section "Backend Dockerfile".
4. Create `docker-compose.yml` in the project root from stack-details.md section "Docker Compose Production".
5. Create `frontend/.dockerignore` and `backend/.dockerignore` from stack-details.md section "Dockerignore Files".

## Step 9: Create Root Configuration Files

1. Create `.env.example` from stack-details.md section "Environment Template".
2. Copy `.env.example` to `.env`:
   ```bash
   cp .env.example .env
   ```
3. Create `.gitignore` from stack-details.md section "Gitignore".
4. Create `.editorconfig` from stack-details.md section "EditorConfig".
5. Create `.vscode/extensions.json` from stack-details.md section "VS Code Extensions".
6. Create `.vscode/settings.json` from stack-details.md section "VS Code Settings".
7. Create `.github/workflows/ci.yml` from stack-details.md section "GitHub Actions CI".
8. Create `README.md` from stack-details.md section "Project README".

## Step 10: Run Install and Verify

1. Install all workspace dependencies:
   ```bash
   bun install
   ```

2. Run checks:
   ```bash
   just check
   ```

3. Run tests:
   ```bash
   just test
   ```

4. Run dev servers:
   ```bash
   just dev
   ```
   Verify:
   - Frontend at http://localhost:3000
   - Backend at http://localhost:3001
   - API docs at http://localhost:3001/docs

5. Stop the dev servers after verification.

## Final Project Structure

```
project/
├── package.json              # Root Bun workspace
├── bun.lock
├── justfile
├── frontend/
│   ├── package.json          # @repo/frontend
│   ├── app/                  # TanStack Start app directory
│   │   └── routes/
│   ├── src/
│   │   ├── lib/
│   │   │   ├── auth-client.ts  # Better Auth client SDK
│   │   │   ├── api.ts          # Eden Treaty client
│   │   │   └── utils.ts
│   │   ├── components/
│   │   │   └── ui/             # shadcn components
│   │   └── __tests__/
│   ├── biome.json
│   ├── vitest.config.ts
│   ├── tsconfig.json
│   ├── Dockerfile
│   └── .dockerignore
├── backend/
│   ├── package.json          # @repo/backend
│   ├── src/
│   │   ├── app.ts            # Elysia app definition
│   │   ├── index.ts          # Entry point (listen)
│   │   ├── auth.ts           # Better Auth config
│   │   ├── db/
│   │   │   ├── index.ts      # Drizzle client
│   │   │   ├── schema.ts     # App schema
│   │   │   └── auth-schema.ts # Generated by Better Auth CLI
│   │   └── routes/
│   │       └── health.ts
│   ├── scripts/
│   │   ├── generate-openapi.ts
│   │   └── validate-openapi.ts
│   ├── tests/
│   │   └── health.test.ts
│   ├── drizzle/              # Migrations
│   ├── drizzle.config.ts
│   ├── biome.json
│   ├── tsconfig.json
│   ├── openapi.json          # Generated OpenAPI spec
│   ├── Dockerfile
│   └── .dockerignore
├── docker-compose.yml        # Production
├── docker-compose.dev.yml    # Dev (PostgreSQL)
├── .vscode/
│   ├── extensions.json
│   └── settings.json
├── .github/
│   └── workflows/
│       └── ci.yml
├── .editorconfig
├── .env.example
├── .env
├── .gitignore
└── README.md
```

Report the final status to the user, including any warnings or errors encountered during setup.
