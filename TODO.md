# MiniGamesLearning — Task Breakdown

## How to Use This File

Each task is self-contained and independently verifiable. Follow this workflow per task:

1. **Write tests FIRST** — red phase (failing tests define the contract)
2. **Implement until tests pass** — green phase (minimum code to pass)
3. **Refactor if needed** — don't touch working tests during refactor
4. **Review the diff manually** — catch what CI misses
5. **Commit with a descriptive message** (what + why)
6. **Update CLAUDE.md / AGENTS.md** if you learned something non-obvious (compound loop)

**Evidence gates**: ✅ means the phase is LOCKED until all items have passing tests + human review sign-off.

---

## Phase 0: Foundation ⬜

> Goal: working repo with CI green, lint clean, and a smoke test passing.

- [ ] Init `package.json` with workspaces (client + server + shared)
- [ ] Configure TypeScript (`tsconfig.json` for root, client, server)
- [ ] Install and configure ESLint + Prettier (enforce on commit via lint-staged + husky)
- [ ] Set up Vitest for unit tests — write first smoke test (`1 + 1 === 2`)
- [ ] Set up Playwright for e2e — write first smoke test (page title check)
- [ ] Create `.env.example` with all required env vars documented
- [ ] Set up Prisma with PostgreSQL connection
- [ ] Write GitHub Actions CI workflow (`.github/workflows/ci.yml`):
  - lint → type-check → unit tests → integration tests
- [ ] Verify CI passes on a test push
- [ ] Review all AI config files (CLAUDE.md, AGENTS.md, copilot-instructions.md) — update with real commands once stack is wired

**Evidence gate** ✅: CI green screenshot + smoke test output in PR description

---

## Phase 1: Database Schema & Core Models ⬜

> Goal: Prisma schema defined, migrations run, seed data works.

- [ ] Write tests for Prisma schema validation (check required fields, relations)
- [ ] Define `User` model (id, email, passwordHash, role: TEACHER | LEARNER, createdAt)
- [ ] Define `Topic` model (id, title, subject, gradeLevel, createdBy → User)
- [ ] Define `GameSession` model (id, topicId, gameType, generatedContent JSONB, createdAt)
- [ ] Define `Score` model (id, sessionId, userId, points, completedAt)
- [ ] Run `prisma migrate dev --name init`
- [ ] Write seed script: 2 users (teacher + learner), 3 topics, 1 session per topic
- [ ] Write integration tests for seed data integrity
- [ ] Verify `prisma studio` shows correct data

**Evidence gate** ✅: Migration diff + seed output in PR. No orphaned relations.

---

## Phase 2: Authentication ⬜

> Goal: register, login, logout with JWT. Protected routes work.

- [ ] Write tests for `AuthService`: register hashes password, login returns JWT, invalid creds rejected
- [ ] Implement `POST /api/auth/register` — validate email/password, bcrypt hash, return JWT
- [ ] Implement `POST /api/auth/login` — compare hash, return JWT + refresh token
- [ ] Implement `POST /api/auth/logout` — invalidate refresh token
- [ ] Write `authMiddleware` — verify JWT, attach `req.user`
- [ ] Write tests for `authMiddleware`: valid token passes, expired token 401, missing token 401
- [ ] Write integration tests: full register → login → access protected route flow
- [ ] Add role guard middleware (TEACHER vs LEARNER)

**Evidence gate** ✅: All auth tests green. Postman/curl trace of full auth flow in PR.

---

## Phase 3: AI Game Generation Engine ⬜

> Goal: given a topic string + game type, GPT-4o returns valid structured game content.

- [ ] Write tests for `GameGeneratorService` with mocked OpenAI responses (never call real API in unit tests)
- [ ] Define TypeScript interfaces for each game type's content schema:
  - `QuizContent` — questions[], each with options[], correctIndex, explanation
  - `WordPuzzleContent` — words[], clues[], gridLayout
  - `MatchingContent` — pairs[] of { term, definition }
  - `FlashcardContent` — cards[] of { front, back, hint? }
  - `TrueFalseContent` — statements[] of { text, isTrue, explanation }
- [ ] Implement `GameGeneratorService.generate(topic, gameType, difficulty)`:
  - Build prompt with schema enforcement (JSON mode)
  - Call GPT-4o with `response_format: { type: "json_object" }`
  - Validate response against Zod schema before returning
  - Throw structured error if validation fails
- [ ] Write Zod validation schemas for each game content type
- [ ] Implement `POST /api/games/generate` — auth required, save to `GameSession`, return content
- [ ] Write integration tests for `/api/games/generate` with mocked OpenAI
- [ ] Add retry logic (max 2 retries) on malformed AI response

**Evidence gate** ✅: Unit tests green with mocked GPT. Manual test of 1 real generation per game type logged in PR.

---

## Phase 4: Game API & Topic Management ⬜

> Goal: full CRUD for topics, sessions retrieval, scoring endpoints.

- [ ] Write tests for `TopicController`: create, list, get by id, delete (owner only)
- [ ] Implement `POST /api/topics` — TEACHER role only
- [ ] Implement `GET /api/topics` — paginated, filterable by subject/gradeLevel
- [ ] Implement `GET /api/topics/:id`
- [ ] Implement `DELETE /api/topics/:id` — owner or admin only
- [ ] Write tests for `ScoreController`: submit score, leaderboard by topic
- [ ] Implement `POST /api/scores` — auth required, validate session ownership
- [ ] Implement `GET /api/scores/leaderboard/:topicId` — top 10, with usernames
- [ ] Implement `GET /api/sessions/:id` — return game session content + metadata

**Evidence gate** ✅: All controller tests green. API tested via Supertest integration suite.

---

## Phase 5: React Frontend — Core Shell ⬜

> Goal: Vite app running, routing, auth context, and topic browsing working.

- [ ] Set up React Router v6 with routes: `/`, `/login`, `/register`, `/dashboard`, `/topics`, `/play/:sessionId`
- [ ] Implement `AuthContext` + `useAuth` hook — stores JWT, exposes login/logout/user
- [ ] Write tests for `useAuth` hook: login sets user, logout clears, persists on refresh (localStorage)
- [ ] Build `LoginPage` and `RegisterPage` with form validation
- [ ] Write component tests for login form: validation errors shown, submit calls API
- [ ] Build `DashboardPage`: shows recent sessions, quick-start by topic
- [ ] Build `TopicsPage`: list topics with search/filter, "Generate Game" button per topic
- [ ] Build `TopicCard` component with tests (renders title, subject, grade level)
- [ ] Add loading and error states to all data-fetching hooks

**Evidence gate** ✅: Component tests green. Recorded Loom/screenshot of navigating auth + topics in PR.

---

## Phase 6: React Frontend — Game Components ⬜

> Goal: all 5 game types playable end-to-end in the browser.

- [ ] Write tests for each game component (pure rendering + interaction):
  - `QuizGame` — renders question, selects option, shows feedback, advances
  - `WordPuzzleGame` — renders clues, accepts input, validates answer
  - `MatchingGame` — drag-and-drop pairs, validates all matched
  - `FlashcardGame` — flips card on click, navigates forward/back, marks known
  - `TrueFalseGame` — renders statement, accepts T/F tap, shows score timer
- [ ] Implement each game component (use existing content schema from Phase 3)
- [ ] Build `GameRouter` — reads `gameType` from session, renders correct component
- [ ] Build `ScoreSummary` component — shows final score, correct/wrong breakdown, "Play Again" / "Try Different Game"
- [ ] Add keyboard navigation support (a11y)
- [ ] Add score submission on game completion → `POST /api/scores`
- [ ] Write e2e Playwright test: generate a quiz → play to completion → score saved

**Evidence gate** ✅: All game component tests green. Playwright e2e passing. Video of all 5 games played in PR.

---

## Phase 7: Game Generation UX ⬜

> Goal: smooth topic → game-type selection → AI generation → play flow.

- [ ] Build `GenerateGameModal` — topic input, game type selector, difficulty selector, "Generate" CTA
- [ ] Write tests for `GenerateGameModal`: validates empty topic, shows loading during generation, error state on failure
- [ ] Add streaming/progress UX during AI generation (show "Generating your quiz..." with spinner)
- [ ] Build `GameTypePicker` component — visual card picker for the 5 game types
- [ ] Add `POST /api/games/generate` call wired to frontend, redirect to `/play/:sessionId` on success
- [ ] Handle generation errors gracefully (show retry option)
- [ ] Add topic autocomplete suggestions from existing topics

**Evidence gate** ✅: Modal tests green. Full generate-to-play flow captured in Playwright test.

---

## Phase 8: Polish & Harden ⬜

> Goal: production-ready quality — no unhandled errors, accessible, responsive.

- [ ] Global error boundary in React — catches render errors, shows friendly fallback
- [ ] API error handling middleware — consistent `{ error, code, message }` shape
- [ ] Rate limiting on `/api/games/generate` (10 req/hour per user via `express-rate-limit`)
- [ ] Input sanitisation on all API endpoints (prevent prompt injection)
- [ ] Responsive design audit — test at 375px, 768px, 1280px
- [ ] Accessibility audit — WCAG AA for all game components (focus management, aria labels)
- [ ] Add request logging (Morgan) and structured error logging (Winston or Pino)
- [ ] Write load test for `/api/games/generate` (k6 or Artillery — 10 concurrent users)
- [ ] Security review: helmet, CORS config, no secrets in logs

**Evidence gate** ✅: Lighthouse score ≥90 (perf + a11y). Rate limit test passing. Load test results in PR.

---

## Phase 9: Ship ⬜

> Goal: deployed and accessible at a real URL.

- [ ] Choose deployment target (Railway / Render / Fly.io for server; Vercel for client)
- [ ] Write `Dockerfile` for server (multi-stage build)
- [ ] Add `deploy` GitHub Actions workflow (on push to `main`)
- [ ] Configure environment variables in deployment target
- [ ] Run `prisma migrate deploy` in production
- [ ] Set up database backups (daily)
- [ ] Write smoke test that hits the production URL
- [ ] Update README with live URL and deployment instructions

**Evidence gate** ✅: Live URL in PR. Smoke test green against production.

---

## Parking Lot 🅿️

> Ideas deferred — don't implement until Phases 0–9 are complete.

- Multiplayer/classroom mode (teacher generates, whole class plays simultaneously)
- Voice-mode flashcards (text-to-speech via OpenAI Audio)
- PDF/document upload as topic source (parse and generate games from notes)
- Progress tracking dashboard with spaced repetition scheduling
- Public game gallery (community-shared topics)
- Mobile app (React Native)

---

## Lessons Learned 📝

> Update this section as you work. Short, sharp observations only.

- _Add entries here as you discover non-obvious things about the codebase or AI generation._
