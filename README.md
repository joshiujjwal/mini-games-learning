# MiniGamesLearning 🎮📚

> AI-powered mini games platform for educational learning — teachers and learners provide a topic, and the platform auto-generates engaging mini games to reinforce knowledge in a fun way.

![Status](https://img.shields.io/badge/status-🚧%20Early%20Development-orange)
![Stack](https://img.shields.io/badge/stack-TypeScript%20%7C%20React%20%7C%20Node.js%20%7C%20PostgreSQL-blue)
![License](https://img.shields.io/badge/license-MIT-green)

---

## What It Does

Teachers or learners enter a topic/subject (e.g. "photosynthesis", "World War II", "Python loops") and the platform uses **OpenAI GPT-4o** to automatically generate a set of engaging mini games:

| Game Type | Description |
|---|---|
| **Quiz Battle** | Multiple-choice questions with instant feedback and scoring |
| **Word Puzzle** | Fill-in-the-blank and crossword-style vocabulary games |
| **Matching Game** | Drag-and-drop term-to-definition matching |
| **Flashcard Battle** | Spaced-repetition flashcards with flip animations |
| **True/False Blitz** | Speed-based true/false rounds with a timer |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | TypeScript + React 18 + Vite |
| Backend | Node.js + Express + TypeScript |
| Database | PostgreSQL (via Prisma ORM) |
| AI | OpenAI GPT-4o (game content generation) |
| Testing | Vitest (unit) + Supertest (API) + Playwright (e2e) |
| Linting | ESLint + Prettier |
| CI | GitHub Actions |

---

## Getting Started

### Prerequisites

- Node.js ≥ 20
- PostgreSQL ≥ 15
- OpenAI API key

### Clone & Install

```bash
git clone https://github.com/joshiujjwal/mini-games-learning.git
cd mini-games-learning
npm install
```

### Environment Setup

```bash
cp .env.example .env
# Fill in DATABASE_URL, OPENAI_API_KEY, SESSION_SECRET
```

### Database Setup

```bash
# TODO: npx prisma migrate dev
# TODO: npx prisma db seed
```

### Development

```bash
# TODO: npm run dev        # starts both client (Vite) and server (tsx watch)
# TODO: npm run dev:client # Vite dev server only
# TODO: npm run dev:server # Express server only
```

### Run Tests

```bash
# TODO: npm test           # run all tests
# TODO: npm run test:unit  # unit tests only
# TODO: npm run test:e2e   # Playwright e2e
```

---

## Project Structure

```
mini-games-learning/
├── src/
│   ├── client/                    # React frontend
│   │   ├── components/
│   │   │   ├── games/             # Individual game components
│   │   │   ├── ui/                # Shared UI primitives
│   │   │   └── layout/            # Page layouts, nav
│   │   ├── pages/                 # Route-level page components
│   │   ├── hooks/                 # Custom React hooks
│   │   ├── context/               # React context providers
│   │   └── utils/                 # Frontend utilities
│   └── server/
│       ├── routes/                # Express route definitions
│       ├── controllers/           # Request handlers
│       ├── services/              # Business logic (AI generation, scoring)
│       ├── middleware/            # Auth, validation, error handling
│       └── db/
│           ├── migrations/        # Prisma migrations
│           └── seeds/             # Seed data
├── tests/
│   ├── unit/client/               # Component + hook tests (Vitest)
│   ├── unit/server/               # Service + controller tests
│   ├── integration/               # API integration tests (Supertest)
│   └── e2e/                       # End-to-end tests (Playwright)
├── docs/
│   ├── spec.md                    # Feature specification
│   └── adr/                       # Architecture Decision Records
├── .github/
│   ├── copilot-instructions.md    # GitHub Copilot context
│   ├── instructions/              # Path-specific AI instructions
│   ├── skills/                    # Agent skills
│   └── workflows/                 # CI/CD pipelines
├── CLAUDE.md                      # Claude AI context
├── AGENTS.md                      # OpenAI Codex / agent context
└── TODO.md                        # Evidence-gated task breakdown
```

---

## Contributing

1. **Read TODO.md** before starting any work — each task is a self-contained unit
2. **Write tests first** (red phase), then implement (green phase)
3. **Run the full test suite** before opening a PR — no red tests allowed
4. **Evidence in PRs**: include test output screenshot or CI link showing green
5. **Small, focused PRs**: one feature or fix per PR
6. **Update CLAUDE.md / AGENTS.md** if you discover something non-obvious about the codebase
7. Never delete a test without explicit approval and a documented reason

---

## License

MIT © joshiujjwal
