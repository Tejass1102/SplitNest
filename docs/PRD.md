# PRD.md — Product Requirements Document

## Project Name: SplitNest

## 1. Problem Statement
People need a simple way to track their personal spending and to split shared expenses (trips, roommates, group outings) with friends — without manually calculating who owes whom. Existing solutions are either too simple (no group splitting) or too complex/bloated for casual use.

## 2. Goals
- Let a user track personal expenses with categories and basic analytics.
- Let a user create groups and split shared expenses (equal, exact, percentage).
- Automatically calculate and display accurate balances (who owes whom, how much).
- Let users record settlements and see updated balances.
- Keep the system simple, correct, and easy to extend later.

## 3. Non-Goals (explicitly out of scope for MVP)
- No multi-currency support
- No receipt upload / OCR
- No push or email notifications
- No recurring expenses
- No debt-simplification/optimization algorithm
- No OAuth / social login
- No mobile app
- No microservices, Kafka, Redis, or GraphQL

## 4. Target Users
College students, roommates, friend groups, couples, and small teams who want lightweight personal + shared expense tracking.

## 5. Core Features (MVP)

### 5.1 Authentication
- Register, login, JWT-based sessions
- Get current user, update profile, change password

### 5.2 Personal Expenses
- Create / edit / delete / view personal expenses
- Fields: amount, category, date, description, payment method
- Filter by category, date range, payment method; sort; basic search

### 5.3 Personal Dashboard
- Total spend, current vs previous month
- Spending by category (chart)
- Monthly trend (chart)
- Recent expenses

### 5.4 Groups
- Create group, name it
- Add / remove members (existing registered users only — no invite links in MVP)
- View group, members, leave group
- Admin vs member roles (admin can remove members / delete group)

### 5.5 Group Expenses
- Add expense to a group: description, amount, paid-by, date, split type, participants
- Edit / delete / view group expenses

### 5.6 Expense Splitting
- **Equal split** — divide evenly among selected participants
- **Exact amount split** — manually specified per-person amounts, must sum to total
- **Percentage split** — per-person percentages, must sum to 100%
- All amounts stored as `BigDecimal`; rounding remainder is added to the first participant in the split order (documented in ARCHITECTURE.md)

### 5.7 Balance Engine
- Compute each member's net balance within a group (owed / owes)
- Correctly handle multiple expenses, multiple payers, edits, and deletions
- Deterministic and unit-testable

### 5.8 Settlements
- Record a settlement (from, to, amount, date)
- Support partial and full settlement
- View settlement history
- Balances update immediately after settlement

### 5.9 Group Dashboard
- Total group spend, your net balance, who owes you, who you owe, recent activity

## 6. Future Features (not in MVP)
- Receipt upload / OCR
- Notifications (email/push)
- Recurring expenses
- Debt simplification / optimized settle-up
- Group invite links / email invitations
- Export to CSV/PDF
- Multi-currency
- Budgets / spending limits
- Google OAuth
- Dark mode / PWA

## 7. Acceptance Criteria (MVP-level)
- A user can register, log in, and manage personal expenses end to end.
- A user can create a group, add members, and add a shared expense with any of the 3 split types.
- Balances shown for a group are mathematically correct for multiple expenses and multiple payers.
- Recording a settlement updates balances correctly and is reflected in history.
- All financial calculations use `BigDecimal` — no floating-point arithmetic anywhere in money-related code.
