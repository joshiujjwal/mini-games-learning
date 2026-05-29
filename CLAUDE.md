# CLAUDE.md — MiniGamesLearning

Context for Claude (and other AI agents) working in this repo.
Keep this file updated when you discover non-obvious conventions.

---

## Quick Start

```bash
# Install all deps (monorepo workspaces)
npm install

# Start dev (client + server in parallel)
# TODO: npm run dev

# Run all tests
# TODO: npm test

# Unit tests only
# TODO: npm run test:unit

# E2E tests
# TODO: npm run test:e2e

# Lint + type check
# TODO: npm run lint
# TODO: npm run typecheck

# DB migration
# TODO: npx prisma migrate dev --name <migration-name>

# DB seed
# TODO: npx prisma db seed

# Build for production
# TODO: npm run build
```

> ⚠️ Update the TODO commands above once package.json scripts are wired in Phase 0.

---

## Workflow (Follow This Every Session)

1. **Read TODO.md first** — find the current phase and next unchecked task
2. **Run tests** before touching any code: `npm test` — know the baseline
3. **Write failing tests first** (red) — commit with message `test: add failing tests for X`
4. **Implement minimum code to pass** (green) — commit `feat: implement X`
5. **Refactor if needed** — don't change tests during refactor
6. **Update this file** if you learn something non-obvious
7. **Check the task off** in TODO.md before moving on

---

## Directory Map

```
src/client/                  React frontend (Vite)
  components/games/          One file per game type (QuizGame, MatchingGame, etc.)
  components/ui/             Shared primitives (Button, Card, Timer, ProgressBar)
  components/layout/         Nav, PageWrapper, Sidebar
  pages/                     Route-level components (mapped 1:1 to React Router routes)
  hooks/                     Custom hooks — prefix: use* (useAuth, useGameSession, useScore)
  context/                   React context providers (AuthContext, GameContext)
  utils/                     Pure functions — no side effects, 100% unit-testable

src/server/
  routes/                    Express Router definitions — thin, delegate to controllers
  controllers/               Request/response handling — call services, format response
  services/                  Business logic — GameGeneratorService, AuthService, ScoreService
  middleware/                authMiddleware, roleGuard, rateLimiter, errorHandler
  db/migrations/             Prisma migration files — never hand-edit
  db/seeds/                  Seed scripts for dev/test data

tests/unit/client/           Vitest tests for React components and hooks
tests/unit/server/           Vitest tests for services, controllers (mocked deps)
tests/integration/           Supertest API tests (real DB, mocked OpenAI)
tests/e2e/                   Playwright tests (full browser, full stack)
```

---

## Key Conventions

### TypeScript
- Strict mode ON — no `any`, no `// @ts-ignore`
- Prefer `type` over `interface` for data shapes; use `interface` only when extending
- All API request/response types defined in `src/shared/types/` and imported by both client and server

### React
- Functional components only — no class components
- Custom hooks for all async data fetching — no raw `fetch` in components
- Game components receive their content as typed props — no direct API calls inside game components
- Error and loading states required for every hook that fetches data

### Server
- Controllers are thin: validate → call service → format response
- Services own business logic and call external dependencies (DB, OpenAI)
- Never call OpenAI directly from a controller or route
- All DB access goes through Prisma — no raw SQL except in migrations

### AI / OpenAI
- **Never call OpenAI in unit tests** — mock the client (`vi.mock`)
- Use `response_format: { type: "json_object" }` for all generation calls
- Always validate GPT responses with Zod before using them
- The prompt lives in `src/server/services/prompts/` — one file per game type
- Log generation failures with full prompt + response for debugging (sanitise user data first)

### Testing
- Unit test files co-located with source: `GameGeneratorService.ts` → `GameGeneratorService.test.ts`
- Integration tests in `tests/integration/` — named `<resource>.api.test.ts`
- E2E tests in `tests/e2e/` — named `<flow>.spec.ts`
- No test should make real external HTTP calls — stub/mock all
- Minimum coverage targets (enforced in CI): 80% lines for services, 70% for controllers

### Environment Variables
- Documented in `.env.example` — every var must have a comment explaining its purpose
- Never log env var values
- Use `zod` to validate env vars at server startup (`src/server/config/env.ts`)

---

## Non-Obvious Gotchas

- GPT-4o sometimes returns valid JSON but with different field names than requested — Zod schema must be strict (`z.object().strict()`)
- Prisma `Json` fields are untyped at the DB level — always cast/validate after reading
- The `gradeLevel` field is a free-text string (not enum) to allow flexibility — validate on the client instead
- React game components use CSS animations — test with `reduced-motion` media query support
- Rate limiting uses in-memory store by default — swap for Redis before scaling horizontally

---

## Environment Variables (see .env.example)

| Variable | Purpose |
|---|---|
| `DATABASE_URL` | PostgreSQL connection string |
| `OPENAI_API_KEY` | GPT-4o API key |
| `JWT_SECRET` | HS256 secret for access tokens |
| `JWT_REFRESH_SECRET` | HS256 secret for refresh tokens |
| `SESSION_SECRET` | Express session secret |
| `PORT` | Server port (default: 3001) |
| `CLIENT_URL` | Frontend origin for CORS (default: http://localhost:5173) |
| `NODE_ENV` | `development` | `test` | `production` |
