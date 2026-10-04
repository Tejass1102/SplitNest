# ROADMAP.md — Development Roadmap (PERN Stack)

## Working Rules (read this first, every session)
1. Work on **one phase at a time** — do not jump ahead or implement multiple phases in one pass.
2. Read `PRD.md`, `ARCHITECTURE.md`, and `DATABASE.md` before starting a new phase.
3. Do not introduce new technologies/dependencies not already listed in `ARCHITECTURE.md` without flagging it first.
4. Do not modify files unrelated to the current phase.
5. After implementing a phase: run the build/tests, fix errors, then check off the phase below.
6. If an ambiguity comes up (e.g., "what happens when X"), state the ambiguity and your chosen resolution before implementing — don't decide silently.

---

## Phases

- [ ] **Phase 0 — Planning & Architecture**
  Generate/review PRD, ARCHITECTURE, DATABASE, ROADMAP docs. Resolve open decisions.

- [ ] **Phase 1 — Repo & Dev Environment Setup**
  Git init, `backend/` + `frontend/` folders, Docker Compose for Postgres, `.env` conventions.
  **DoD:** `docker compose up` gives a working Postgres instance.

- [ ] **Phase 2 — Express Backend Foundation**
  `npm init`, Express setup, base folder structure (routes/controllers/services/middleware/validators), health-check endpoint.
  **DoD:** Backend starts with `npm run dev`, `/health` returns 200.

- [ ] **Phase 3 — PostgreSQL Schema & Prisma Models**
  Define all entities from DATABASE.md in `schema.prisma`, run initial migration.
  **DoD:** Schema created correctly in Postgres; Prisma Client generates without errors; constraints enforced.

- [ ] **Phase 4 — Authentication & Security**
  Register/login routes, bcrypt hashing, JWT issuing + verification middleware, get current user, update profile, change password.
  **DoD:** Can register, log in, access a protected route with a token; unauthorized requests rejected.

- [ ] **Phase 5 — Personal Expense Management (Backend)**
  CRUD routes, filtering, sorting, pagination, Zod validation.
  **DoD:** Full CRUD + filters verified via Postman; service-layer tests pass.

- [ ] **Phase 6 — React Frontend Foundation**
  Vite + TS setup, routing, Axios/React Query, Tailwind, auth context, login/register pages.
  **DoD:** Can register/login from the UI; protected routes redirect correctly when logged out.

- [ ] **Phase 7 — Personal Expense UI**
  Expense list, add/edit/delete forms, filter/sort UI, loading/error/empty states.
  **DoD:** Full personal expense flow usable end-to-end in the browser.

- [ ] **Phase 8 — Group Management**
  Backend + frontend: create/view group, add/remove members, admin/member roles.
  **DoD:** Can create a group, add existing users, remove members, respecting role permissions.

- [ ] **Phase 9 — Group Expense Management**
  Backend + frontend: CRUD for group expenses.
  **DoD:** Group expenses fully CRUD-able from the UI.

- [ ] **Phase 10 — Expense Splitting Engine**
  Equal / exact / percentage split logic + validation; split type selector UI.
  **DoD:** All 3 split types produce correct split records; invalid splits rejected with clear errors; tests cover rounding edge cases.

- [ ] **Phase 11 — Balance Calculation Engine**
  Per-user net balance within a group, across multiple expenses/payers; correct on edit/delete.
  **DoD:** Balance output verified against hand-calculated test scenarios.

- [ ] **Phase 12 — Settlement System**
  Record settlement (partial/full), settlement history, balance updates post-settlement.
  **DoD:** Settling a debt updates balances immediately and correctly; history persists.

- [ ] **Phase 13 — Dashboards & Analytics**
  Aggregation routes + Recharts UI for personal and group dashboards.
  **DoD:** Both dashboards render correct data matching underlying numbers.

- [ ] **Phase 14 — Validation, Error Handling & UX Polish**
  Centralized Express error middleware, consistent error shape, unified frontend error/loading/empty states.
  **DoD:** No unhandled/raw errors surfaced to the user.

- [ ] **Phase 15 — Testing & Bug Fixing**
  Jest + Supertest tests for splitting, balances, settlements, auth/authorization. Fix found bugs.
  **DoD:** Critical business logic has test coverage; test suite passes.

- [ ] **Phase 16 — Dockerization**
  Dockerfiles for backend/frontend, full docker-compose (backend + frontend + Postgres).
  **DoD:** `docker compose up` runs the entire app with one command.

- [ ] **Phase 17 — Deployment**
  Deploy backend, frontend, database to chosen providers; production env vars configured.
  **DoD:** App accessible via public URL, fully functional end-to-end.

---

## Suggested Git Branch Mapping
| Branch | Phases |
|---|---|
| `feature/auth` | 4 |
| `feature/personal-expenses` | 5, 7 |
| `feature/groups` | 8 |
| `feature/expense-splitting` | 9, 10 |
| `feature/balances` | 11 |
| `feature/settlements` | 12 |
| `feature/dashboard` | 13 |

## Open Decisions to Resolve (see DATABASE.md)
- Group deletion behavior when balances are unsettled
- Member removal behavior when balance is nonzero
- Prisma vs Sequelize (recommended: Prisma)
