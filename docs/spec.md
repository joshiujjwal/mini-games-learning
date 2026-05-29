# MiniGamesLearning — Feature Specification

**Version**: 0.1.0 (pre-implementation)
**Author**: joshiujjwal
**Status**: Draft

---

## 1. Overview

### Problem Statement

Traditional educational tools (textbooks, slideshows, static quizzes) are passive and low-engagement. Teachers spend hours creating custom exercises. Learners disengage quickly when content feels like work, not play.

### Solution

MiniGamesLearning lets teachers or learners type a topic (e.g. *"mitosis"*, *"French Revolution"*, *"JavaScript closures"*) and instantly receive 5 types of auto-generated, playable mini games powered by GPT-4o. No content creation effort required. Games are saved to a session so progress and scores can be tracked over time.

### Success Criteria

- A new topic + game can be generated in under 10 seconds
- Generated content is factually coherent for the given topic (manual review gate)
- All 5 game types are playable on desktop and mobile
- Scores are persisted and visible on a leaderboard per topic

---

## 2. Functional Requirements

### 2.1 User Management
- [ ] Users can register with email + password
- [ ] Users can log in and receive a JWT (15-min access + 7-day refresh)
- [ ] Users have a role: `TEACHER` or `LEARNER`
- [ ] Only `TEACHER` role can create/delete topics
- [ ] All authenticated users can generate games and submit scores

### 2.2 Topic Management
- [ ] A topic has: `title`, `subject` (e.g. Biology), `gradeLevel` (K-12/College/Adult), `description`
- [ ] Teachers can create, list, update, and delete their own topics
- [ ] Learners can browse and search all topics
- [ ] Topics are filterable by `subject` and `gradeLevel`

### 2.3 Game Generation
- [ ] A user selects a topic + game type + difficulty level (Easy / Medium / Hard)
- [ ] The system calls GPT-4o with a structured prompt to generate game content
- [ ] GPT-4o responses are validated against a Zod schema before being accepted
- [ ] Invalid responses trigger up to 2 automatic retries with a clarifying prompt
- [ ] Generated content is stored in a `GameSession` record (JSONB)
- [ ] Each generation has a unique session ID used for play and scoring

### 2.4 Game Types

#### Quiz Battle
- 10 questions, each with 4 multiple-choice options
- One correct answer per question
- Explanation shown after answer selected
- 10 seconds per question (configurable)
- Score = correct answers × (time_remaining_bonus)

#### Word Puzzle
- 8 fill-in-the-blank sentences using key vocabulary from the topic
- Case-insensitive matching
- Hint: first letter revealed after 15 seconds
- Score = correct fills with fewest hints used

#### Matching Game
- 8 term-definition pairs
- Drag-and-drop interface
- Wrong match causes a brief shake animation (no penalty, just visual feedback)
- Score = matches completed × time bonus

#### Flashcard Battle
- 12 cards (front: term, back: definition/explanation)
- User self-rates: "Got it" / "Almost" / "Nope"
- "Nope" cards cycle back into the deck
- Session ends when all cards rated "Got it"
- Score = total attempts taken (lower = better)

#### True/False Blitz
- 15 statements, 8 seconds per statement
- Timer visible, ticking
- Explanation shown for wrong answers
- Score = correct answers / total × 100

### 2.5 Scoring & Leaderboard
- [ ] Each completed game submits a score to the server
- [ ] Leaderboard shows top 10 scores per topic + game type
- [ ] User can view their own score history

### 2.6 Session History
- [ ] Each user can see their past game sessions
- [ ] Sessions show: topic, game type, score, date, "Play Again" link

---

## 3. Non-Functional Requirements

- [ ] API response time < 300ms for all non-AI endpoints (p95)
- [ ] AI generation < 10 seconds p90
- [ ] Frontend bundle < 300KB gzipped (initial load)
- [ ] WCAG 2.1 AA accessibility compliance for all game UIs
- [ ] Postgres row-level data isolation (users can only see their own scores/sessions unless leaderboard)
- [ ] All secrets in env vars — never in code or logs
- [ ] Rate limit: 10 AI generation requests per user per hour
- [ ] Input sanitisation to prevent prompt injection in topic/subject fields

---

## 4. Data Model

```prisma
model User {
  id           String        @id @default(cuid())
  email        String        @unique
  passwordHash String
  role         Role          @default(LEARNER)
  createdAt    DateTime      @default(now())
  topics       Topic[]
  scores       Score[]
  sessions     GameSession[]
}

enum Role {
  TEACHER
  LEARNER
}

model Topic {
  id          String        @id @default(cuid())
  title       String
  subject     String
  gradeLevel  String
  description String?
  createdBy   User          @relation(fields: [userId], references: [id])
  userId      String
  createdAt   DateTime      @default(now())
  sessions    GameSession[]
}

model GameSession {
  id               String   @id @default(cuid())
  topic            Topic    @relation(fields: [topicId], references: [id])
  topicId          String
  generatedBy      User     @relation(fields: [userId], references: [id])
  userId           String
  gameType         GameType
  difficulty       Difficulty
  generatedContent Json     // validated against game-type-specific Zod schema
  createdAt        DateTime @default(now())
  scores           Score[]
}

enum GameType {
  QUIZ
  WORD_PUZZLE
  MATCHING
  FLASHCARD
  TRUE_FALSE
}

enum Difficulty {
  EASY
  MEDIUM
  HARD
}

model Score {
  id          String      @id @default(cuid())
  session     GameSession @relation(fields: [sessionId], references: [id])
  sessionId   String
  user        User        @relation(fields: [userId], references: [id])
  userId      String
  points      Int
  completedAt DateTime    @default(now())
}
```

---

## 5. API Design

### Auth
| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/api/auth/register` | None | Create account |
| POST | `/api/auth/login` | None | Return JWT pair |
| POST | `/api/auth/refresh` | Refresh token | New access JWT |
| POST | `/api/auth/logout` | Access JWT | Invalidate refresh |

### Topics
| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/api/topics` | TEACHER | Create topic |
| GET | `/api/topics` | Any | List (paginated, filterable) |
| GET | `/api/topics/:id` | Any | Get topic detail |
| DELETE | `/api/topics/:id` | TEACHER (owner) | Delete topic |

### Games
| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/api/games/generate` | Any | Generate game session for topic |
| GET | `/api/sessions/:id` | Owner | Get session content |
| GET | `/api/sessions` | Any | List user's sessions (paginated) |

### Scores
| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/api/scores` | Any | Submit score for session |
| GET | `/api/scores/leaderboard/:topicId` | Any | Top 10 by topic+gameType |
| GET | `/api/scores/me` | Any | User's own score history |

### Request/Response conventions
- All responses: `{ data, meta? }` on success, `{ error, code, message }` on failure
- Pagination: `?page=1&limit=20` → `meta: { total, page, limit, hasMore }`
- Auth header: `Authorization: Bearer <token>`

---

## 6. AI Prompt Design

### Generation prompt structure

```
System: You are an educational game designer. Generate {gameType} content for the topic "{topic}" at {difficulty} level for {gradeLevel} students.
Output ONLY valid JSON matching the provided schema. Do not include explanations outside the JSON.

User: Generate a {gameType} game about "{topicTitle}". Subject: {subject}. 
Difficulty: {difficulty}. Number of items: {count}.
Return JSON conforming to this schema:
{zodSchemaAsJsonSchema}
```

### Prompt injection defence
- Topic and subject fields are stripped of: backticks, `<`, `>`, `{`, `}`, quotes
- Max topic length: 200 chars
- Max subject length: 100 chars

---

## 7. Test Plan

### Unit Tests (Vitest)
- `GameGeneratorService.generate()` with mocked OpenAI — all 5 game types
- `GameGeneratorService` Zod validation rejects malformed responses
- `AuthService.register()` hashes password, rejects duplicate email
- `AuthService.login()` returns JWT on valid credentials, throws on bad creds
- `authMiddleware` — valid token passes, expired 401, malformed 401
- Each React game component renders correctly with fixture content
- `useAuth` hook: login sets user, logout clears, token persists in localStorage
- `ScoreCalculator` utility: correct scoring formula per game type

### Integration Tests (Supertest)
- `POST /api/auth/register` → login → access protected route
- `POST /api/topics` — TEACHER creates, LEARNER gets 403
- `POST /api/games/generate` with mocked OpenAI — returns session id
- `POST /api/scores` → leaderboard updated

### E2E Tests (Playwright)
- Register as learner → browse topics → generate quiz → play to completion → score appears on leaderboard
- Teacher registers → creates topic → generates matching game → plays it
- Rate limit: 11th generation request in 1 hour returns 429

### Edge Cases
- GPT-4o returns malformed JSON → retries → error response after max retries
- Topic with special characters (emoji, HTML) → sanitised before prompt
- Score submission for already-scored session → 409 conflict
- Leaderboard with 0 scores → empty array, not error
- Session played by different user from generator → allowed (public sessions)

---

## 8. Open Questions

- [ ] Should generated games be public (shareable link) or private by default?
- [ ] Do we want a "classroom code" feature so teachers can assign a game to a group?
- [ ] Should we cache GPT responses for identical topic+type+difficulty combos? (cost saving)
- [ ] What's the max number of items per game? (10 quiz questions vs 15 T/F vs 8 matching)
- [ ] Do learners need accounts, or can anonymous play be supported?
- [ ] Should scores on the leaderboard show real names or usernames?
