# SplitNest

A full-stack expense management app combining personal expense tracking with Splitwise-style group expense splitting.

## Features
- Personal expense tracking with categories, filtering, and a spending dashboard
- Group creation and member management
- Group expense splitting — equal, exact amount, and percentage-based
- Automatic balance calculation (who owes whom)
- Settlement recording with partial/full payment support and history

See [`docs/PRD.md`](docs/PRD.md) for full feature scope and non-goals.

## Tech Stack (PERN)

**Frontend:** React, Vite, TypeScript, Tailwind CSS, React Router, React Hook Form, Zod, Recharts
**Backend:** Node.js, Express.js, Prisma ORM, JWT, bcrypt
**Database:** PostgreSQL
**Dev tools:** Docker, Docker Compose, Postman

See [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) for full architecture details.

## Project Structure
```
├── docs/           # PRD, architecture, database schema, roadmap
├── backend/        # Express + Prisma + PostgreSQL API
├── frontend/       # React + Vite + TypeScript client
└── docker-compose.yml
```

## Local Development Setup

### Prerequisites
- Node.js (LTS version)
- Docker Desktop
- Git

### 1. Clone the repo
```bash
git clone <your-repo-url>
cd your-repo
```

### 2. Start PostgreSQL
```bash
docker compose up -d
```

### 3. Backend setup
```bash
cd backend
npm install
cp .env.example .env   # fill in DATABASE_URL (postgresql://splitnest_user:splitnest_pass@localhost:5432/splitnest), JWT_SECRET
npx prisma migrate dev
npm run dev
```
Backend runs at `http://localhost:8080` (adjust if configured differently).

### 4. Frontend setup
```bash
cd frontend
npm install
npm run dev
```
Frontend runs at `http://localhost:5173` (default Vite port).

## Testing
```bash
cd backend
npm test
```

## Roadmap
Development is tracked phase-by-phase in [`docs/ROADMAP.md`](docs/ROADMAP.md).

## Documentation
- [`docs/PRD.md`](docs/PRD.md) — product requirements and scope
- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — system architecture and conventions
- [`docs/DATABASE.md`](docs/DATABASE.md) — data model and schema
- [`docs/ROADMAP.md`](docs/ROADMAP.md) — phased development plan
