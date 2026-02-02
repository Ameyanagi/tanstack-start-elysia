# Stack Details Reference

All configuration file contents for the init-tanstack-elysia skill.

---

## Biome Configuration (Frontend)

`frontend/biome.json`:

```json
{
  "$schema": "https://biomejs.dev/schemas/2.0.0/schema.json",
  "assist": { "actions": { "source": { "organizeImports": "on" } } },
  "vcs": {
    "enabled": true,
    "clientKind": "git",
    "useIgnoreFile": true,
    "defaultBranch": "main"
  },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "correctness": {
        "noUnusedImports": "error",
        "useExhaustiveDependencies": "warn"
      },
      "style": {
        "noNonNullAssertion": "off"
      },
      "a11y": {
        "noLabelWithoutControl": "warn",
        "useSemanticElements": "warn",
        "useKeyWithClickEvents": "warn",
        "noRedundantAlt": "warn"
      },
      "suspicious": {
        "noArrayIndexKey": "off"
      }
    }
  },
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 100
  },
  "javascript": {
    "formatter": {
      "quoteStyle": "single",
      "semicolons": "asNeeded"
    }
  },
  "css": {
    "parser": {
      "tailwindDirectives": true
    }
  },
  "files": {
    "includes": [
      "**",
      "!**/node_modules",
      "!**/.output",
      "!**/.tanstack",
      "!**/dist",
      "!**/src/routeTree.gen.ts"
    ]
  }
}
```

---

## Biome Configuration (Backend)

`backend/biome.json`:

```json
{
  "$schema": "https://biomejs.dev/schemas/2.0.0/schema.json",
  "assist": { "actions": { "source": { "organizeImports": "on" } } },
  "vcs": {
    "enabled": true,
    "clientKind": "git",
    "useIgnoreFile": true,
    "defaultBranch": "main"
  },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true,
      "correctness": {
        "noUnusedImports": "error"
      },
      "style": {
        "noNonNullAssertion": "off"
      }
    }
  },
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 100
  },
  "javascript": {
    "formatter": {
      "quoteStyle": "single",
      "semicolons": "asNeeded"
    }
  },
  "files": {
    "includes": [
      "**",
      "!**/node_modules",
      "!**/drizzle",
      "!**/dist"
    ]
  }
}
```

---

## Root Package JSON

`package.json` (project root):

```json
{
  "name": "project-root",
  "private": true,
  "workspaces": ["frontend", "backend"],
  "scripts": {
    "dev": "just dev",
    "check": "just check",
    "test": "just test"
  }
}
```

After creating this file, run `bun install` to link the workspaces.

---

## Backend Package JSON

`backend/package.json`:

```json
{
  "name": "@repo/backend",
  "private": true,
  "version": "0.1.0",
  "type": "module",
  "scripts": {
    "dev": "bun --watch src/index.ts",
    "start": "bun src/index.ts",
    "check": "biome check .",
    "check:fix": "biome check --write .",
    "typecheck": "tsc --noEmit",
    "test": "bun test",
    "db:push": "bunx drizzle-kit push",
    "db:migrate": "bunx drizzle-kit migrate",
    "db:generate": "bunx drizzle-kit generate",
    "db:studio": "bunx drizzle-kit studio",
    "gen:openapi": "bun run scripts/generate-openapi.ts",
    "gen:auth-schema": "bunx @better-auth/cli generate --config ./src/auth.ts --output ./src/db/auth-schema.ts"
  },
  "dependencies": {
    "better-auth": "latest",
    "drizzle-orm": "latest",
    "elysia": "latest",
    "@elysiajs/cors": "latest",
    "@elysiajs/swagger": "latest",
    "pg": "latest"
  },
  "devDependencies": {
    "@biomejs/biome": "latest",
    "@types/pg": "latest",
    "drizzle-kit": "latest",
    "typescript": "latest"
  }
}
```

Note: After creating this file, run `bun install` from the project root (not from the backend directory) to properly resolve workspace dependencies.

---

## Backend TSConfig

`backend/tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "types": ["bun-types"],
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*.ts"],
  "exclude": ["node_modules", "dist", "drizzle", "scripts", "tests"]
}
```

---

## Drizzle Client

`backend/src/db/index.ts`:

```typescript
import { drizzle } from 'drizzle-orm/node-postgres'
import { Pool } from 'pg'
import * as schema from './schema'

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
})

export const db = drizzle(pool, { schema })
```

---

## Drizzle Config

`backend/drizzle.config.ts`:

```typescript
import { defineConfig } from 'drizzle-kit'

export default defineConfig({
  out: './drizzle',
  schema: './src/db/schema.ts',
  dialect: 'postgresql',
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
})
```

---

## App Schema

`backend/src/db/schema.ts`:

After generating the Better Auth schema with the CLI (Step 5), this file should re-export the auth tables and add any app-specific tables.

```typescript
// Re-export all Better Auth tables
export * from './auth-schema'

// Add app-specific tables below
// import { pgTable, text, timestamp, uuid } from 'drizzle-orm/pg-core'
//
// export const posts = pgTable('posts', {
//   id: uuid('id').defaultRandom().primaryKey(),
//   title: text('title').notNull(),
//   content: text('content'),
//   createdAt: timestamp('created_at').defaultNow().notNull(),
// })
```

If the Better Auth CLI fails or is unavailable, manually create `backend/src/db/auth-schema.ts` with the following minimal schema:

```typescript
import { pgTable, text, timestamp, boolean } from 'drizzle-orm/pg-core'

export const user = pgTable('user', {
  id: text('id').primaryKey(),
  name: text('name').notNull(),
  email: text('email').notNull().unique(),
  emailVerified: boolean('email_verified').notNull().default(false),
  image: text('image'),
  username: text('username').unique(),
  displayName: text('display_name'),
  createdAt: timestamp('created_at').notNull().defaultNow(),
  updatedAt: timestamp('updated_at').notNull().defaultNow(),
})

export const session = pgTable('session', {
  id: text('id').primaryKey(),
  expiresAt: timestamp('expires_at').notNull(),
  token: text('token').notNull().unique(),
  ipAddress: text('ip_address'),
  userAgent: text('user_agent'),
  userId: text('user_id')
    .notNull()
    .references(() => user.id, { onDelete: 'cascade' }),
  createdAt: timestamp('created_at').notNull().defaultNow(),
  updatedAt: timestamp('updated_at').notNull().defaultNow(),
})

export const account = pgTable('account', {
  id: text('id').primaryKey(),
  accountId: text('account_id').notNull(),
  providerId: text('provider_id').notNull(),
  userId: text('user_id')
    .notNull()
    .references(() => user.id, { onDelete: 'cascade' }),
  accessToken: text('access_token'),
  refreshToken: text('refresh_token'),
  idToken: text('id_token'),
  accessTokenExpiresAt: timestamp('access_token_expires_at'),
  refreshTokenExpiresAt: timestamp('refresh_token_expires_at'),
  scope: text('scope'),
  password: text('password'),
  createdAt: timestamp('created_at').notNull().defaultNow(),
  updatedAt: timestamp('updated_at').notNull().defaultNow(),
})

export const verification = pgTable('verification', {
  id: text('id').primaryKey(),
  identifier: text('identifier').notNull(),
  value: text('value').notNull(),
  expiresAt: timestamp('expires_at').notNull(),
  createdAt: timestamp('created_at').notNull().defaultNow(),
  updatedAt: timestamp('updated_at').notNull().defaultNow(),
})
```

---

## Better Auth Server Config

`backend/src/auth.ts`:

```typescript
import { betterAuth } from 'better-auth'
import { drizzleAdapter } from 'better-auth/adapters/drizzle'
import { username } from 'better-auth/plugins'
import { db } from './db'

export const auth = betterAuth({
  database: drizzleAdapter(db, {
    provider: 'pg',
  }),
  emailAndPassword: {
    enabled: true,
  },
  plugins: [username()],
  trustedOrigins: [process.env.FRONTEND_URL ?? 'http://localhost:3000'],
})
```

---

## Better Auth Client

`frontend/src/lib/auth-client.ts`:

```typescript
import { createAuthClient } from 'better-auth/react'
import { usernameClient } from 'better-auth/client/plugins'

export const authClient = createAuthClient({
  baseURL: import.meta.env.VITE_BACKEND_URL ?? 'http://localhost:3001',
  plugins: [usernameClient()],
})

export const { signIn, signUp, signOut, useSession } = authClient
```

---

## Eden Treaty Client

`frontend/src/lib/api.ts`:

```typescript
import { treaty } from '@elysiajs/eden'
import type { App } from '@repo/backend/src/app'

export const api = treaty<App>(import.meta.env.VITE_BACKEND_URL ?? 'http://localhost:3001')
```

Note: This import relies on Bun workspaces resolving `@repo/backend`. The `type` import ensures no runtime backend code is bundled into the frontend. If TypeScript cannot resolve the path, add a path alias in the frontend's tsconfig.json:

```json
{
  "compilerOptions": {
    "paths": {
      "@repo/backend/*": ["../backend/*"]
    }
  }
}
```

---

## Health Route

`backend/src/routes/health.ts`:

```typescript
import { Elysia, t } from 'elysia'

export const healthRoutes = new Elysia().get(
  '/health',
  () => ({ status: 'healthy' }),
  {
    response: t.Object({
      status: t.String(),
    }),
    detail: {
      tags: ['Health'],
      summary: 'Health check',
    },
  },
)
```

---

## Elysia App

`backend/src/app.ts`:

This file defines the Elysia app without calling `.listen()` so it can be imported by scripts (OpenAPI generation, tests) and by the frontend (Eden Treaty types).

```typescript
import { Elysia, t } from 'elysia'
import { cors } from '@elysiajs/cors'
import { swagger } from '@elysiajs/swagger'
import { auth } from './auth'
import { healthRoutes } from './routes/health'

const origins = (process.env.CORS_ORIGINS ?? 'http://localhost:3000')
  .split(',')
  .map((o) => o.trim())
  .filter(Boolean)

export const app = new Elysia()
  .use(
    cors({
      origin: origins,
      methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'OPTIONS'],
      credentials: true,
      allowedHeaders: ['Content-Type', 'Authorization'],
    }),
  )
  .use(
    swagger({
      path: '/docs',
      documentation: {
        info: {
          title: 'Backend API',
          version: '0.1.0',
        },
        tags: [
          { name: 'Health', description: 'Health check endpoints' },
          { name: 'Better Auth', description: 'Authentication endpoints' },
        ],
      },
    }),
  )
  .mount(auth.handler)
  .use(healthRoutes)
  .get(
    '/',
    () => ({ message: 'API is running' }),
    {
      response: t.Object({
        message: t.String(),
      }),
      detail: {
        tags: ['Health'],
        summary: 'Root endpoint',
      },
    },
  )

export type App = typeof app
```

---

## Backend Entry Point

`backend/src/index.ts`:

```typescript
import { app } from './app'

const port = process.env.BACKEND_PORT ?? 3001

app.listen(port)

console.log(`Backend running at http://localhost:${port}`)
console.log(`API docs at http://localhost:${port}/docs`)
```

---

## OpenAPI Generation Script

`backend/scripts/generate-openapi.ts`:

```typescript
import { app } from '../src/app'

const response = await app.handle(new Request('http://localhost/docs/json'))
const spec = await response.json()

await Bun.write('openapi.json', JSON.stringify(spec, null, 2) + '\n')
console.log('OpenAPI schema written to openapi.json')
```

---

## OpenAPI Validation Script

`backend/scripts/validate-openapi.ts`:

```typescript
import { app } from '../src/app'

const response = await app.handle(new Request('http://localhost/docs/json'))
const current = JSON.stringify(await response.json(), null, 2) + '\n'

const file = Bun.file('openapi.json')
if (!(await file.exists())) {
  console.error('ERROR: openapi.json does not exist. Run "just gen-api" to generate it.')
  process.exit(1)
}

const existing = await file.text()

if (current !== existing) {
  console.error('ERROR: openapi.json is out of date. Run "just gen-api" to update.')
  process.exit(1)
}

console.log('OpenAPI schema is up to date.')
```

---

## Backend Tests

`backend/tests/health.test.ts`:

```typescript
import { describe, expect, it } from 'bun:test'
import { app } from '../src/app'

describe('health', () => {
  it('should return healthy', async () => {
    const response = await app.handle(new Request('http://localhost/health'))
    expect(response.status).toBe(200)
    const body = await response.json()
    expect(body.status).toBe('healthy')
  })

  it('should return root message', async () => {
    const response = await app.handle(new Request('http://localhost/'))
    expect(response.status).toBe(200)
    const body = await response.json()
    expect(body.message).toBe('API is running')
  })
})
```

---

## Frontend Test Configuration

`frontend/vitest.config.ts`:

```typescript
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    globals: true,
    environment: 'jsdom',
  },
})
```

`frontend/src/__tests__/smoke.test.ts`:

```typescript
import { describe, expect, it } from 'vitest'

describe('smoke test', () => {
  it('should pass', () => {
    expect(true).toBe(true)
  })
})
```

---

## Justfile

`justfile` (project root):

```justfile
set dotenv-load

# Default: show available commands
default:
    @just --list

# Run dev database, frontend, and backend in parallel
dev: dev-db
    just dev-backend & just dev-frontend & wait

# Run frontend dev server
dev-frontend:
    cd frontend && bun run dev

# Run backend dev server with watch mode
dev-backend:
    cd backend && bun --watch src/index.ts

# Start dev PostgreSQL via Docker
dev-db:
    docker compose -f docker-compose.dev.yml up -d
    @echo "PostgreSQL running on port ${DB_PORT:-5432}"

# Stop dev PostgreSQL
stop-db:
    docker compose -f docker-compose.dev.yml down

# Run all checks: lint, format, typecheck, OpenAPI validation
check: check-frontend check-backend check-openapi

# Check frontend: biome + tsc
check-frontend:
    cd frontend && bun run check
    cd frontend && bun run typecheck

# Check backend: biome + tsc
check-backend:
    cd backend && bun run check
    cd backend && bun run typecheck

# Validate OpenAPI schema is up to date
check-openapi:
    cd backend && bun run scripts/validate-openapi.ts

# Fix lint and format issues
fix: fix-frontend fix-backend

# Fix frontend issues
fix-frontend:
    cd frontend && bun run check:fix

# Fix backend issues
fix-backend:
    cd backend && bun run check:fix

# Generate OpenAPI schema
gen-api:
    cd backend && bun run scripts/generate-openapi.ts

# Generate Better Auth database schema
gen-auth-schema:
    cd backend && bun run gen:auth-schema

# Run all tests
test: test-frontend test-backend

# Run frontend tests
test-frontend:
    cd frontend && bun run test

# Run backend tests
test-backend:
    cd backend && bun test

# Push Drizzle schema to database
db-push:
    cd backend && bunx drizzle-kit push

# Generate Drizzle migration
db-migrate:
    cd backend && bunx drizzle-kit generate

# Apply Drizzle migrations
db-migrate-run:
    cd backend && bunx drizzle-kit migrate

# Open Drizzle Studio
db-studio:
    cd backend && bunx drizzle-kit studio

# Build Docker production images
docker-build:
    docker compose build

# Run production stack with Docker
docker-up:
    docker compose up -d

# Stop production stack
docker-down:
    docker compose down

# Clean generated files
clean:
    rm -rf frontend/.output frontend/.tanstack frontend/node_modules
    rm -rf backend/node_modules backend/dist backend/drizzle backend/openapi.json
    rm -rf node_modules
```

---

## Docker Compose Dev

`docker-compose.dev.yml` (project root):

This file provides only PostgreSQL for local development. Frontend and backend run natively via `just dev`.

```yaml
services:
  postgres:
    image: postgres:17-alpine
    ports:
      - "${DB_PORT:-5432}:5432"
    environment:
      POSTGRES_USER: ${DB_USER:-postgres}
      POSTGRES_PASSWORD: ${DB_PASSWORD:-postgres}
      POSTGRES_DB: ${DB_NAME:-devdb}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER:-postgres}"]
      interval: 5s
      timeout: 3s
      retries: 5

volumes:
  postgres_data:
```

---

## Frontend Dockerfile

`frontend/Dockerfile`:

```dockerfile
FROM oven/bun:1 AS base
WORKDIR /app

# Install dependencies
FROM base AS deps
COPY package.json bun.lock* ./
RUN bun install --frozen-lockfile

# Build the application
FROM deps AS build
COPY . .
RUN bun run build

# Production image
FROM oven/bun:1-slim AS runtime
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY --from=build /app/.output ./.output
COPY --from=build /app/package.json ./package.json
ENV NODE_ENV=production
EXPOSE 3000
CMD ["bun", ".output/server/index.mjs"]
```

---

## Backend Dockerfile

`backend/Dockerfile`:

```dockerfile
FROM oven/bun:1 AS base
WORKDIR /app

# Install dependencies
FROM base AS deps
COPY package.json bun.lock* ./
RUN bun install --frozen-lockfile --production

# Production image
FROM oven/bun:1-slim AS runtime
WORKDIR /app

RUN groupadd --system app && useradd --system --gid app app

COPY --from=deps --chown=app:app /app/node_modules ./node_modules
COPY --chown=app:app src ./src
COPY --chown=app:app package.json ./

ENV NODE_ENV=production
USER app
EXPOSE 3001
CMD ["bun", "src/index.ts"]
```

---

## Docker Compose Production

`docker-compose.yml` (project root):

```yaml
services:
  postgres:
    image: postgres:17-alpine
    ports:
      - "${DB_PORT:-5432}:5432"
    environment:
      POSTGRES_USER: ${DB_USER:-postgres}
      POSTGRES_PASSWORD: ${DB_PASSWORD:-postgres}
      POSTGRES_DB: ${DB_NAME:-devdb}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER:-postgres}"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 10s
    restart: unless-stopped

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "${BACKEND_PORT:-3001}:3001"
    env_file:
      - .env
    depends_on:
      postgres:
        condition: service_healthy
    restart: unless-stopped

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "${FRONTEND_PORT:-3000}:3000"
    environment:
      - NODE_ENV=production
    depends_on:
      backend:
        condition: service_started
    restart: unless-stopped

volumes:
  postgres_data:
```

---

## Dockerignore Files

`frontend/.dockerignore`:

```
node_modules
.output
.tanstack
.git
*.md
biome.json
vitest.config.ts
src/__tests__
```

`backend/.dockerignore`:

```
node_modules
dist
.git
*.md
biome.json
drizzle
openapi.json
scripts
tests
```

---

## Environment Template

`.env.example`:

```bash
# Frontend
FRONTEND_PORT=3000
VITE_BACKEND_URL=http://localhost:3001

# Backend
BACKEND_PORT=3001

# CORS (comma-separated origins)
CORS_ORIGINS=http://localhost:3000
FRONTEND_URL=http://localhost:3000

# Database
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/devdb
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=postgres
DB_NAME=devdb

# Better Auth
BETTER_AUTH_SECRET=change-me-in-production
```

---

## Gitignore

`.gitignore`:

```gitignore
# Dependencies
node_modules/

# Build output
.output/
.tanstack/
dist/

# Generated
backend/openapi.json
backend/src/db/auth-schema.ts
backend/drizzle/

# Environment
.env
.env.local
.env.*.local

# IDE
.idea/
*.swp
*.swo
*~

# OS
.DS_Store
Thumbs.db

# Caches
.turbo/
```

---

## EditorConfig

`.editorconfig` (project root):

```ini
root = true

[*]
indent_style = space
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
charset = utf-8

[*.{js,ts,tsx,jsx,json,css,yml,yaml}]
indent_size = 2

[*.md]
trim_trailing_whitespace = false

[justfile]
indent_size = 4
```

---

## VS Code Extensions

`.vscode/extensions.json`:

```json
{
  "recommendations": [
    "biomejs.biome",
    "bradlc.vscode-tailwindcss",
    "martinvoelk.jj"
  ]
}
```

---

## VS Code Settings

`.vscode/settings.json`:

```json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "biomejs.biome",
  "[markdown]": {
    "editor.defaultFormatter": null
  },
  "typescript.tsdk": "node_modules/typescript/lib"
}
```

---

## GitHub Actions CI

`.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  check:
    name: Lint & Typecheck
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v2
      - run: bun install --frozen-lockfile

      - name: Check frontend (biome + tsc)
        run: |
          cd frontend && bun run check
          cd frontend && bun run typecheck

      - name: Check backend (biome + tsc)
        run: |
          cd backend && bun run check
          cd backend && bun run typecheck

      - name: Validate OpenAPI schema
        run: cd backend && bun run scripts/validate-openapi.ts

  test:
    name: Tests
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:17-alpine
        env:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: testdb
        ports:
          - 5432:5432
        options: >-
          --health-cmd "pg_isready -U postgres"
          --health-interval 5s
          --health-timeout 3s
          --health-retries 5
    env:
      DATABASE_URL: postgresql://postgres:postgres@localhost:5432/testdb
    steps:
      - uses: actions/checkout@v4
      - uses: oven-sh/setup-bun@v2
      - run: bun install --frozen-lockfile

      - name: Run frontend tests
        run: cd frontend && bun run test

      - name: Run backend tests
        run: cd backend && bun test
```

---

## Project README

`README.md` (in the generated project root):

````markdown
# Project Name

Full-stack monorepo: TanStack Start + shadcn/ui (frontend) and ElysiaJS (backend) with Better Auth and PostgreSQL.

## Prerequisites

- [Bun](https://bun.sh) >= 1.2
- [just](https://github.com/casey/just) >= 1.0
- [jj](https://jj-vcs.github.io/jj/) >= 0.25 (colocated with git)
- [Docker](https://www.docker.com/) >= 24.0

## Quick Start

```bash
cp .env.example .env
just dev
```

- Frontend: http://localhost:3000
- Backend: http://localhost:3001
- API Docs: http://localhost:3001/docs

## Commands

```bash
just dev              # Start database + frontend + backend
just dev-frontend     # Frontend only
just dev-backend      # Backend only
just dev-db           # Start PostgreSQL only
just stop-db          # Stop PostgreSQL
just check            # Lint + format + typecheck + OpenAPI validation
just test             # Run all tests
just fix              # Auto-fix lint and format issues
just gen-api          # Generate OpenAPI schema
just gen-auth-schema  # Generate Better Auth DB schema
just db-push          # Push Drizzle schema to database
just db-migrate       # Generate Drizzle migration
just db-migrate-run   # Apply Drizzle migrations
just db-studio        # Open Drizzle Studio
just docker-build     # Build Docker images
just docker-up        # Start production stack
just docker-down      # Stop production stack
just clean            # Remove generated files
```

## Stack

| Layer   | Technology                                           |
| ------- | ---------------------------------------------------- |
| Frontend| TanStack Start, React, shadcn/ui (Mira), Tailwind   |
| Backend | ElysiaJS, Bun                                        |
| Auth    | Better Auth (username + password)                    |
| DB      | PostgreSQL, Drizzle ORM                              |
| API     | Eden Treaty (end-to-end type safety)                 |
| Lint    | Biome (frontend + backend)                           |
| Types   | tsc (frontend + backend)                             |
| Package | Bun workspaces                                       |
| Tasks   | just                                                 |
| VCS     | jj (colocated with git)                              |
| CI      | GitHub Actions                                       |
| Deploy  | Docker + docker compose                              |

## Project Structure

```
├── frontend/          # TanStack Start + shadcn/ui
├── backend/           # ElysiaJS + Better Auth + Drizzle
├── justfile           # Task runner
├── docker-compose.yml # Production deployment
├── docker-compose.dev.yml # Dev PostgreSQL
├── .env.example       # Environment template
└── .gitignore
```

## API Client

The frontend uses [Eden Treaty](https://elysiajs.com/eden/overview) for end-to-end type-safe API calls directly from ElysiaJS types. No code generation step needed.

```typescript
import { api } from '~/lib/api'

// Fully typed - types inferred from ElysiaJS route definitions
const { data } = await api.health.get()
```

## Authentication

Authentication is handled by [Better Auth](https://www.better-auth.com/) with username + password support.

```typescript
import { signIn, signUp, useSession } from '~/lib/auth-client'

// Sign up
await signUp.username({ username: 'alice', password: 'secret', name: 'Alice' })

// Sign in
await signIn.username({ username: 'alice', password: 'secret' })

// Check session (React hook)
const { data: session } = useSession()
```
````
