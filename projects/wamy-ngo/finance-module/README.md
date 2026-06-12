# WAMY NGO — Finance Module

![Status](https://img.shields.io/badge/status-in%20progress-yellow)
![Stack](https://img.shields.io/badge/stack-Spring%20Boot%203.2%20%7C%20React%2019%20%7C%20MySQL%208-blue)
![License](https://img.shields.io/badge/license-proprietary-lightgrey)

Double-entry accounting engine and financial reporting suite for the WAMY NGO ERP.

---

## Table of Contents

- [Overview](#overview)
- [Requirements Summary](#requirements-summary)
  - [Core Accounting Engine](#core-accounting-engine)
  - [Financial Reports](#financial-reports)
  - [Security and Access Control](#security-and-access-control)
  - [Non-Functional Requirements](#non-functional-requirements)
- [Module Status](#module-status)
- [Folder Guide](#folder-guide)
- [Key Concepts](#key-concepts)
- [Getting Started](#getting-started)
- [Related Files](#related-files)

---

## Overview

The Finance module is the accounting backbone of the WAMY NGO ERP. It manages:

- A **chart of accounts** structured for NGO fund accounting.
- A **double-entry journal** where every posted entry is balanced and immutable.
- **Donor fund tracking** — each expenditure traces back to the grant that funded it.
- **Budget control** — raises warnings or blocks when project spending exceeds budget.
- **10 financial reports** covering everything from a trial balance to bank reconciliation.

All financial data is **project-scoped** — funds for Project A cannot be mixed with funds for Project B.

---

## Requirements Summary

### Core Accounting Engine

| # | Requirement | Priority |
|---|-------------|----------|
| F-01 | Maintain a chart of accounts with hierarchical account codes | Must |
| F-02 | Record journal entries with two or more balanced debit/credit lines | Must |
| F-03 | Enforce double-entry invariant: sum(debits) == sum(credits) on every entry | Must |
| F-04 | Approval workflow: DRAFT → PENDING_APPROVAL → APPROVED → POSTED | Must |
| F-05 | Posted entries are immutable — reversals only via counter-entries | Must |
| F-06 | Tag every entry with project ID and optionally donor ID and cost centre | Must |
| F-07 | Budget allocation per project/cost centre with variance calculation | Must |
| F-08 | Alert or block posting when expenditure would exceed approved budget | Should |
| F-09 | Support multi-period reporting with configurable fiscal year start | Should |
| F-10 | Audit trail for all entity changes (who changed what and when) | Must |

### Financial Reports

| # | Report | Status |
|---|--------|--------|
| R-01 | Trial Balance | Planned |
| R-02 | Income & Expenditure Statement | Planned |
| R-03 | Balance Sheet | Planned |
| R-04 | Cash Flow Statement | Planned |
| R-05 | Budget vs Actual | Planned |
| R-06 | Donor Fund Report | Planned |
| R-07 | Project Expenditure Report | Planned |
| R-08 | General Ledger | Planned |
| R-09 | Accounts Payable Ageing | Planned |
| R-10 | Bank Reconciliation Statement | Planned |

All reports must be filterable by project, date range, and cost centre. Reports return data as JSON (for UI rendering) and must support PDF export.

### Security and Access Control

| Role | Permissions |
|------|------------|
| `ROLE_VIEWER` | View reports, view journal entries |
| `ROLE_FINANCE` | All viewer permissions + create/edit/approve/post journal entries |
| `ROLE_PROJECT_MANAGER` | View reports + manage project budgets |
| `ROLE_ADMIN` | Full access including chart of accounts management |

JWT-based authentication via Spring Security. Tokens issued at login, validated on every request.

### Non-Functional Requirements

| Requirement | Target |
|-------------|--------|
| Report generation time | < 3 seconds for date ranges up to 12 months |
| Monetary precision | `DECIMAL(19,4)` — no floating-point types |
| Concurrency | Optimistic locking on journal entries to prevent double-posting |
| Data retention | Soft deletes only — no hard deletes on any financial record |
| Audit | Every entity mutation logged with user ID and timestamp |

---

## Module Status

| Area | Status | Notes |
|------|--------|-------|
| Chart of accounts API | Planned | Entity and repository designed |
| Journal entry engine | Planned | Double-entry validation designed |
| Approval workflow | Planned | State machine defined |
| Reports (all 10) | Planned | Query logic not yet implemented |
| Frontend components | Planned | Module scaffolded in React |
| Flyway migrations | Planned | Schema pending |
| Unit tests | Planned | No tests written yet |

---

## Folder Guide

```
finance-module/
├── CLAUDE.md                  ← AI context: rules, entity design, anti-patterns
├── README.md                  ← This file
│
└── (source lives in the backend/frontend monorepo above this folder)
    backend/src/main/java/com/wamy/finance/
    ├── controller/            ← REST endpoints (thin — delegate to services)
    ├── service/               ← Business logic, double-entry validation, report generation
    ├── repository/            ← Spring Data JPA interfaces
    ├── entity/                ← JPA entities (extend BaseEntity)
    ├── dto/
    │   ├── request/           ← Inbound DTOs ({Action}{Entity}Request)
    │   └── response/          ← Outbound DTOs ({Entity}Response, ReportResult)
    ├── mapper/                ← MapStruct mappers
    └── exception/             ← Domain exceptions (DoubleEntryViolationException, etc.)

    frontend/src/modules/finance/
    ├── components/            ← Reusable UI pieces
    ├── pages/                 ← Route-level components
    ├── hooks/                 ← Data-fetching hooks (useJournalEntry, useReport, etc.)
    └── api/                   ← Axios API client wrappers
```

---

## Key Concepts

**Double-Entry Accounting:** Every financial transaction is recorded as at least two lines — a debit on one account and an equal credit on another. The books are always in balance.

**Chart of Accounts:** A numbered hierarchy of account categories (Assets 1xxx, Liabilities 2xxx, Net Assets 3xxx, Income 4xxx, Expenditure 5xxx). Account codes are assigned per NGO accounting standards.

**Project-Based Accounting:** NGO funds are restricted — money granted for Project A must be spent on Project A. The finance module enforces this by requiring a `projectId` on every transaction.

**Donor Fund Report:** Donors receive periodic reports showing how their grant was spent. This report is generated directly from the tagged journal entries, making it auditable.

**Budget vs Actual:** Each project has an approved budget per cost centre. The module compares actual expenditure (from posted journal entries) against the budget and flags variances.

---

## Getting Started

The finance module is part of the WAMY NGO backend monolith. To run it locally:

```bash
# 1. Start MySQL 8 and the backend
cd wamy-ngo/
docker-compose up -d mysql
./mvnw spring-boot:run -pl backend

# 2. Start the frontend
cd frontend/
npm install
npm run dev
```

The finance API is available at `http://localhost:8080/api/v1/` (accounts, journal-entries, ledger, budgets, reports).

Swagger UI: `http://localhost:8080/swagger-ui.html`

---

## Related Files

- [CLAUDE.md](CLAUDE.md) — AI context for this module (entity design, accounting rules, anti-patterns)
- [../CLAUDE.md](../CLAUDE.md) — WAMY NGO project-level context (stack, security, naming conventions)
- [../../CLAUDE.md](../../CLAUDE.md) — Root repo context (how `CLAUDE.md` files are layered)
