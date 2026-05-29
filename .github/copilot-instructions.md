# GitHub Copilot Instructions — MiniGamesLearning

## Project Context

AI-powered educational mini games platform. Users enter a topic; GPT-4o generates structured game content (quizzes, word puzzles, matching games, flashcards, true/false) which is rendered as interactive browser games.

**Stack**: TypeScript (strict) · React 18 + Vite · Node.js + Express · PostgreSQL + Prisma ORM · OpenAI GPT-4o · Vitest · Playwright

---

## TypeScript Conventions

- Strict mode always — no `any`, no `@ts-ignore`
- Explicit return types on all exported functions
- Use `type` for data shapes, `interface` only when extending
- Validate all external data with Zod (API inputs, AI responses, env vars)
- Use `import type` for type-only imports
- Shared types live in `src/shared/types/` — import in both client and server

---

## React Conventions

- Functional components, named exports only
- Props type defined inline in the component file as `type XxxProps = {...}`
- All data fetching in custom hooks (`src/client/hooks/`)
- Game components are pure: receive content via props, emit score via `onComplete` callback
- Every component needs loading state (`isLoading`) and error state (`error`) handling
- Support `prefers-reduced-motion` for CSS animations in game components

---

## Server Conventions

- Route → Controller → Service layering is strict — no business logic in routes
- `GameGeneratorService` is the only place that calls OpenAI
- All Prisma access through the singleton `prisma` client from `src/server/db/client.ts`
- Express error handler at the end of middleware chain converts thrown errors to `{ error, code, message }`
- Rate limiting middleware applied to `/api/games/generate` route

---

## AI Generation Conventions

- All prompts defined in `src/server/services/prompts/` — never inline
- Always use `response_format: { type: "json_object" }` with GPT-4o
- Validate GPT response with Zod `.strict()` before returning
- Retry up to 2 times on validation failure with a clarifying follow-up message
- Strip special chars from user-supplied topic/subject before injecting into prompt

---

## Testing Conventions

- Write tests BEFORE implementation (red/green TDD)
- Test files co-located with source: `MyService.ts` → `MyService.test.ts`
- Mock OpenAI with `vi.mock` — never call real API in tests
- Integration tests use a separate test database
- Game component tests must simulate a complete play-through interaction
- Never delete a test to make CI pass — fix the implementation instead

---

## What Copilot Should NOT Do

- Do not suggest `any` types — suggest the proper type or `unknown` with a guard
- Do not suggest `console.log` in server code — suggest the project logger instead
- Do not inline prompt strings — suggest moving them to `src/server/services/prompts/`
- Do not suggest skipping Zod validation on AI responses
- Do not refactor files outside the current task scope
- Do not suggest hardcoded secrets — always suggest env vars
- Do not remove error or loading state handling from components
