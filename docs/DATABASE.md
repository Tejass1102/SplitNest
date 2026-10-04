# DATABASE.md — Data Model

PostgreSQL, fully relational. All monetary columns use `NUMERIC(12,2)` (maps to `BigDecimal` in Java).

## 1. Entities Overview

- **User**
- **Group**
- **GroupMember** (join table: User ↔ Group, with role)
- **Category**
- **Expense** (personal expense OR group expense — see note below)
- **ExpenseSplit** (per-participant share of a group expense)
- **Settlement**

> Design decision: a single `Expense` table serves both personal and group expenses, distinguished by a nullable `group_id`. Personal expenses have `group_id = NULL` and no associated `ExpenseSplit` rows. Group expenses have a `group_id` and one or more `ExpenseSplit` rows. This avoids duplicating the expense model. (Mark as DECISION REQUIRED if you'd rather split these into two tables.)

## 2. Entity Definitions

### User
| Column | Type | Notes |
|---|---|---|
| id | UUID / BIGSERIAL | PK |
| name | VARCHAR | not null |
| email | VARCHAR | unique, not null |
| password_hash | VARCHAR | not null |
| created_at | TIMESTAMP | default now() |
| updated_at | TIMESTAMP | |

### Group
| Column | Type | Notes |
|---|---|---|
| id | BIGSERIAL | PK |
| name | VARCHAR | not null |
| created_by | FK → User.id | |
| created_at | TIMESTAMP | |

### GroupMember
| Column | Type | Notes |
|---|---|---|
| id | BIGSERIAL | PK |
| group_id | FK → Group.id | not null |
| user_id | FK → User.id | not null |
| role | ENUM('ADMIN','MEMBER') | not null |
| joined_at | TIMESTAMP | |
| UNIQUE(group_id, user_id) | | prevents duplicate membership |

### Category
| Column | Type | Notes |
|---|---|---|
| id | BIGSERIAL | PK |
| name | VARCHAR | e.g. Food, Travel, Rent, Other |
| is_default | BOOLEAN | seeded system categories vs future user-defined ones |

### Expense
| Column | Type | Notes |
|---|---|---|
| id | BIGSERIAL | PK |
| user_id | FK → User.id | owner (personal) or creator (group) |
| group_id | FK → Group.id, NULLABLE | null = personal expense |
| paid_by | FK → User.id | who paid (relevant for group expenses) |
| category_id | FK → Category.id | |
| amount | NUMERIC(12,2) | not null, > 0 |
| description | VARCHAR | |
| payment_method | ENUM | Cash, UPI, Credit Card, Debit Card, Bank Transfer, Other |
| split_type | ENUM('EQUAL','EXACT','PERCENTAGE'), NULLABLE | null for personal expenses |
| expense_date | DATE | not null |
| created_at | TIMESTAMP | |
| updated_at | TIMESTAMP | |

### ExpenseSplit
| Column | Type | Notes |
|---|---|---|
| id | BIGSERIAL | PK |
| expense_id | FK → Expense.id | not null, cascade delete |
| user_id | FK → User.id | participant |
| share_amount | NUMERIC(12,2) | not null — this participant's owed amount |
| share_percentage | NUMERIC(5,2), NULLABLE | only populated for PERCENTAGE split type |

### Settlement
| Column | Type | Notes |
|---|---|---|
| id | BIGSERIAL | PK |
| group_id | FK → Group.id | not null |
| from_user_id | FK → User.id | who paid |
| to_user_id | FK → User.id | who received |
| amount | NUMERIC(12,2) | not null, > 0 |
| settled_at | TIMESTAMP | not null |
| created_at | TIMESTAMP | |

## 3. Relationships
- User ↔ Group: many-to-many via GroupMember
- Group → Expense: one-to-many (nullable group_id distinguishes personal vs group)
- Expense → ExpenseSplit: one-to-many, cascade delete
- User → Expense: one-to-many (as owner and as payer)
- Group → Settlement: one-to-many
- User → Settlement: many-to-many via from_user_id / to_user_id

## 4. Cascade Rules
- Deleting an `Expense` cascades to delete its `ExpenseSplit` rows.
- Deleting a `Group` — DECISION REQUIRED: cascade-delete all group expenses/settlements, or block deletion if unsettled balances exist? Recommend: block deletion unless all balances are zero.
- Removing a `GroupMember` with a nonzero balance — DECISION REQUIRED: block removal, or archive their balance? Recommend: block removal until settled.

## 5. Indexes
- `Expense(user_id)`, `Expense(group_id)`, `Expense(expense_date)`
- `ExpenseSplit(expense_id)`, `ExpenseSplit(user_id)`
- `GroupMember(group_id)`, `GroupMember(user_id)`
- `Settlement(group_id)`

## 6. Money Precision
- All monetary columns: `NUMERIC(12,2)`.
- Application layer uses `BigDecimal` exclusively — never `double`/`float`.
- See ARCHITECTURE.md §5 for rounding strategy on splits.
