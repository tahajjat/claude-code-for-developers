# CLAUDE.md — WAMY NGO ERP

**Project:** wamy-ngo  
**Organisation:** WAMY (World Assembly of Muslim Youth) — regional NGO  
**Model:** Multi-module ERP, project-based accounting  
**Stack:** Spring Boot 3.2 · React 19 · MySQL 8 · GitHub Actions · Robi Cloud

---

## 1. Project Overview

WAMY NGO is a **multi-module ERP** built for a non-governmental organisation that manages donor funding, project-based expenditure tracking, HR, procurement, and financial reporting. All financial activity is tied to a specific **project** and **cost centre**, not just an organisation-wide ledger.

Key business constraints:
- Funds arrive as **donor grants** tagged to a specific project and purpose.
- Every expense must be **approved** before it moves to accounting.
- Reports are submitted to donors periodically — accuracy is non-negotiable.
- The system must support **multiple concurrent projects** with separate budgets.

---

## 2. Tech Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Backend framework | Spring Boot | 3.2 |
| Language | Java | 21 |
| Frontend | React | 19 |
| Build tool (BE) | Maven | 3.9+ |
| Build tool (FE) | Vite | 5+ |
| Database | MySQL | 8.0 |
| ORM | Spring Data JPA / Hibernate | 6.x |
| Security | Spring Security + JWT | - |
| API style | REST (JSON) | - |
| CI/CD | GitHub Actions | - |
| Hosting | Robi Cloud (Bangladesh) | - |
| Containerisation | Docker + Docker Compose | - |

---

## 3. Module Map

```
wamy-ngo/
├── backend/                    ← Spring Boot monolith (modular packages)
│   └── src/main/java/com/wamy/
│       ├── finance/            ← Double-entry accounting, reports
│       ├── hr/                 ← Staff, payroll, attendance
│       ├── procurement/        ← Purchase requests, vendors, GRN
│       ├── project/            ← Project master, budget allocation
│       └── shared/             ← BaseEntity, common DTOs, security config
├── frontend/                   ← React 19 SPA
│   └── src/
│       ├── modules/finance/
│       ├── modules/hr/
│       ├── modules/procurement/
│       └── modules/project/
├── db/                         ← Flyway migrations (V{n}__{description}.sql)
└── docker-compose.yml
```

---

## 4. Domain Model Fundamentals

### Multi-Project Accounting

Every financial entity carries:

```java
private Long projectId;          // which project this belongs to
private Long costCentreId;       // which cost centre within the project
private Long donorId;            // which donor funded this (nullable for unrestricted)
```

**Never aggregate across projects without an explicit project filter.** Mixing project funds is a compliance violation.

### Base Entity

```java
@MappedSuperclass
public abstract class BaseEntity {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private boolean isDeleted = false;          // soft delete only
    private LocalDateTime createdAt;
    private String createdBy;
    private LocalDateTime updatedAt;
    private String updatedBy;
}
```

All tables inherit this. Hard deletes are forbidden.

---

## 5. Naming Conventions

| Artifact | Pattern | Example |
|----------|---------|---------|
| Entity | `PascalCase` noun | `JournalEntry` |
| Repository | `{Entity}Repository` | `JournalEntryRepository` |
| Service | `{Entity}Service` | `JournalEntryService` |
| Service impl | `{Entity}ServiceImpl` | `JournalEntryServiceImpl` |
| Controller | `{Entity}Controller` | `JournalEntryController` |
| Request DTO | `{Action}{Entity}Request` | `CreateJournalEntryRequest` |
| Response DTO | `{Entity}Response` | `JournalEntryResponse` |
| Mapper | `{Entity}Mapper` | `JournalEntryMapper` |
| DB table | `snake_case` plural | `journal_entries` |
| DB column | `snake_case` | `cost_centre_id` |
| React component | `PascalCase.tsx` | `JournalEntryForm.tsx` |
| React hook | `use{Feature}.ts` | `useJournalEntry.ts` |
| API endpoint | `/api/v1/{resource}` | `/api/v1/journal-entries` |

---

## 6. Security Model

- JWT issued by the backend on login, stored in `httpOnly` cookie on the frontend.
- Every request carries the JWT — the backend validates it via `JwtAuthenticationFilter`.
- Roles: `ROLE_ADMIN`, `ROLE_FINANCE`, `ROLE_PROJECT_MANAGER`, `ROLE_VIEWER`.
- Method-level security via `@PreAuthorize` — never rely on URL patterns alone.
- Multi-tenancy is **project-scoped**, not organisation-scoped: a user may have different roles on different projects.

---

## 7. API Conventions

- All endpoints versioned under `/api/v1/`.
- Paginated list endpoints accept `page`, `size`, `sort` query params (Spring Pageable).
- All responses wrapped in a standard envelope:

```json
{
  "success": true,
  "data": { ... },
  "message": "Created successfully",
  "timestamp": "2026-06-12T10:00:00Z"
}
```

- Validation errors return `400` with `errors` array inside the envelope.
- Business rule violations return `422` (Unprocessable Entity).

---

## 8. Database Migrations

All schema changes go through **Flyway**:

```
db/migrations/
  V1__init_schema.sql
  V2__add_finance_tables.sql
  V3__add_project_budget.sql
```

- Never alter an already-applied migration file — always add a new version.
- Migration scripts use `snake_case` table and column names.
- Every table must have `created_at`, `created_by`, `updated_at`, `updated_by`, `is_deleted` columns (inherited from base migration pattern).

---

## 9. GitHub Actions CI/CD

```
.github/workflows/
  build.yml        ← compile + unit tests on every push
  deploy-staging.yml  ← build Docker image → push to Robi Cloud registry → deploy
  deploy-prod.yml     ← triggered on release tag (v*.*.*)
```

Robi Cloud uses a Kubernetes-compatible container runtime. Deployment uses `kubectl apply` with environment-specific `values/` overrides.

---

## 10. Critical Anti-Patterns — NEVER Do These

```
✗ Hard delete any record                → set is_deleted = true only
✗ Mix project funds in a single query   → always filter by project_id
✗ Put business logic in controllers     → controllers call service methods only
✗ Read userId/projectId from request body → read from SecurityContext / JWT claims
✗ Alter an applied Flyway migration     → always create a new migration version
✗ AllowAll CORS in production           → restrict to known frontend domain
✗ Log sensitive data (passwords, tokens) → scrub before logging
✗ Return entity classes directly from controllers → always use response DTOs
```

---

## 11. Module-Specific Context Files

| Module | Context file |
|--------|-------------|
| Finance | [finance-module/CLAUDE.md](finance-module/CLAUDE.md) |
| HR | *(not yet created)* |
| Procurement | *(not yet created)* |
| Project | *(not yet created)* |
