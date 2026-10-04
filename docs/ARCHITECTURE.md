# ARCHITECTURE.md — System Architecture

## 1. Guiding Principles
- Modular monolith. No microservices, no Kafka, no Redis, no GraphQL, no Kubernetes.
- Business logic lives in the service/controller layer, never duplicated across route handlers.
- APIs never expose raw database rows directly where it can be avoided — use response shaping/DTO-like objects for sensitive or complex entities (e.g., never return password hashes).
- All money values use PostgreSQL `NUMERIC` + a precise decimal library in JS (never native floating-point math for money).
- Prefer maintainability over cleverness. Avoid premature optimization.
- Authorization is enforced server-side, always — never trust frontend validation alone.

## 2. Tech Stack (PERN)

### Backend
- Node.js (LTS version)
- Express.js
- Prisma ORM (recommended over Sequelize — better TypeScript support, cleaner migrations, type-safe queries)
- `jsonwebtoken` + `bcrypt` for authentication
- `zod` for request validation (shared validation logic style with frontend, which also uses Zod)
- Jest + Supertest for testing

### Database
- PostgreSQL
- Prisma Migrate for schema migrations

### Frontend
- React + Vite
- TypeScript
- React Router
- Axios or React Query for data fetching/caching
- Tailwind CSS
- Recharts (dashboard charts)
- React Hook Form + Zod (forms + validation)
- No Redux — component state / context is sufficient at this scale

### Dev Tools
- Git, Docker, Docker Compose, Postman

## 3. Backend Layering

```
Route  →  Controller  →  Service  →  Prisma (DB access)  →  PostgreSQL
```

- **Routes**: Express router definitions, map HTTP verb + path to controller function
- **Controllers**: thin — parse request, call service, shape response, no business logic
- **Services**: all business logic (splitting, balance calculation, validation beyond basic shape-checking)
- **Prisma layer**: schema-defined models, used directly by services (Prisma replaces a separate repository layer)
- **Middleware**: JWT auth verification, centralized error handler, request validation (Zod schemas)
- **Validators**: Zod schemas per route, shared where possible with frontend form validation

Suggested folder structure inside `backend/src/`:
```
src/
  routes/
  controllers/
  services/
  middleware/
  validators/
  utils/
  prisma/        -- schema.prisma, migrations/
  config/
```

Use Prisma's `$transaction` for any multi-step write (e.g., creating an expense + its splits together) to guarantee atomicity.

## 4. Frontend Structure

```
src/
  components/   -- reusable UI pieces
  pages/        -- route-level views
  layouts/      -- shared page shells
  services/     -- API call wrappers (axios)
  hooks/        -- custom React hooks
  context/      -- auth context, etc.
  utils/        -- helpers
  types/        -- TS types/interfaces
  routes/       -- route definitions
```

## 5. Money & Rounding Strategy
- All monetary columns in Postgres: `NUMERIC(12,2)`.
- In Node/JS, never use native `number` for money arithmetic (floating-point precision issues). Use a decimal library — `decimal.js` or Prisma's built-in `Decimal` type (Prisma returns `NUMERIC` columns as `Decimal` objects automatically) for all calculations.
- Equal split: divide total by participant count; if not evenly divisible, the remainder (in smallest currency unit) is added to the **first participant** in the group's member order.
- Exact split: sum of parts must equal total exactly, validated server-side before persisting.
- Percentage split: percentages must sum to exactly 100.00, validated server-side.
- Rounding: round half up, applied consistently via the decimal library — never manual float rounding.

## 6. Authentication & Security
- Passwords hashed with bcrypt, never stored in plaintext.
- JWT issued on login, sent as `Authorization: Bearer <token>`.
- Secrets/config via environment variables (`.env`, never committed) — never hardcoded.
- Every group/expense/settlement route verifies the requesting user is authorized for that resource (e.g., is a member of the group) via middleware before the controller logic runs.

## 7. Error Handling
Consistent error response shape (example):
```json
{
  "timestamp": "2026-08-31T10:00:00Z",
  "status": 400,
  "error": "VALIDATION_ERROR",
  "message": "Split amounts must sum to the total expense amount",
  "path": "/api/groups/3/expenses"
}
```
Handled centrally via Express error-handling middleware — no raw stack traces returned to the client.

## 8. Deployment Readiness (not implemented until Phase 17)
- Frontend: Vercel or Netlify
- Backend: Render or Railway
- Database: Neon or Supabase (managed Postgres)
- Architecture should not block any of the above — no local-filesystem dependencies, config via env vars.
