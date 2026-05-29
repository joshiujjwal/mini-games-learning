# AGENTS.md — MiniGamesLearning

Instructions for OpenAI Codex, GitHub Copilot Workspace, and other AI coding agents working on this project.

---

## Setup

```bash
# 1. Install dependencies
npm install

# 2. Set up environment
cp .env.example .env
# Fill in: DATABASE_URL, OPENAI_API_KEY, JWT_SECRET, JWT_REFRESH_SECRET

# 3. Set up database
# TODO: npx prisma migrate dev
# TODO: npx prisma db seed

# 4. Start dev servers
# TODO: npm run dev
```

---

## Before You Write Any Code

1. Read `TODO.md` — identify the current phase and the specific unchecked task
2. Run existing tests: `npm test` — understand the current baseline
3. Check `docs/spec.md` for the feature spec and data model
4. Do NOT start implementing before tests exist for the feature

---

## Red/Green TDD — Non-Negotiable

Every feature follows this order:

```
write failing test → commit (red)
→ implement minimum code
→ tests pass → commit (green)
→ refactor if needed
→ update CLAUDE.md/AGENTS.md if non-obvious
```

Never write implementation code that doesn't have a corresponding test. Never delete a test to make CI pass.

---

## Code Style — TypeScript

```typescript
// ✅ Explicit return types on exported functions
export function calculateScore(correct: number, total: number, timeBonus: number): number {
  return Math.round((correct / total) * 100 + timeBonus);
}

// ✅ Zod for all external data validation (API input, AI responses, env vars)
import { z } from 'zod';
const QuizContentSchema = z.object({
  questions: z.array(QuestionSchema).min(1).max(20),
});

// ✅ Type-only imports where no runtime value is needed
import type { GameSession } from '@prisma/client';

// ❌ Never use `any`
// ❌ Never use `// @ts-ignore`
// ❌ Never `console.log` in production code — use the logger (Pino/Winston)
```

---

## Code Style — React

```tsx
// ✅ Named exports for components
export function QuizGame({ content, onComplete }: QuizGameProps) { ... }

// ✅ Props interfaces defined in the same file
type QuizGameProps = {
  content: QuizContent;
  onComplete: (score: number) => void;
};

// ✅ Custom hooks for data fetching
function useGameSession(id: string) {
  // fetch, return { data, isLoading, error }
}

// ❌ No direct fetch() inside game components — use hooks
// ❌ No class components
// ❌ No default exports for components (makes refactoring harder)
```

---

## Code Style — Server / Express

```typescript
// ✅ Controllers: validate → call service → respond
export async function generateGame(req: Request, res: Response) {
  const input = GenerateGameSchema.parse(req.body); // throws on bad input
  const session = await gameGeneratorService.generate(input);
  res.json({ data: session });
}

// ✅ Services: own business logic, call Prisma + OpenAI
// ✅ Routes: mount controllers, apply middleware
// ❌ Never call OpenAI directly from a route or controller
// ❌ Never access req.body without validating with Zod first
```

---

## Testing

```bash
# Run all tests
# TODO: npm test

# Unit tests only (Vitest)
# TODO: npm run test:unit

# Integration tests (Supertest + real DB)
# TODO: npm run test:integration

# E2E tests (Playwright)
# TODO: npm run test:e2e

# Watch mode
# TODO: npm run test:watch
```

### Test rules
- Mock OpenAI in all unit and integration tests (`vi.mock`)
- Use a separate `DATABASE_URL` for test database (set in `.env.test`)
- Clean DB between integration test suites (`beforeEach` truncate or transaction rollback)
- Every game component must have a test that simulates full play-through
- Minimum thresholds (enforced): services ≥ 80% line coverage, controllers ≥ 70%

---

## PR Instructions

- **Title format**: `type(scope): description` — e.g. `feat(games): add word puzzle component`
- **Types**: `feat` | `fix` | `test` | `refactor` | `docs` | `chore`
- **PR body must include**:
  1. What was changed and why
  2. Test output (copy-paste or screenshot of `npm test` passing)
  3. For UI changes: screenshot or Loom of the feature working
  4. For AI generation changes: example of generated content for 1 topic
- **Evidence gates**: do not merge a phase until all evidence in TODO.md is provided
- Keep PRs under 400 lines changed — split larger work into sequential PRs
- Do not refactor unrelated code in a feature PR

---

## Boundaries — What NOT to Do

- Do NOT remove or skip existing tests — fix the underlying code
- Do NOT call real OpenAI API in tests — always mock
- Do NOT add dependencies without checking if an existing one covers the need
- Do NOT change the Prisma schema without creating a migration
- Do NOT merge to `main` without CI passing
- Do NOT hardcode API keys, secrets, or URLs — use env vars
- Do NOT refactor files unrelated to the current task
