# CLAUDE.md — WAMY NGO Finance Module

**Module:** finance  
**Parent project:** [WAMY NGO ERP](../CLAUDE.md)  
**Domain:** Double-entry accounting, donor fund management, financial reporting  
**Security:** Spring Security JWT (see parent CLAUDE.md for role definitions)

---

## 1. Module Overview

The Finance module is the accounting core of the WAMY NGO ERP. It implements:

- A **double-entry bookkeeping** engine (every debit has a matching credit).
- A **chart of accounts** structured for NGO fund accounting (assets, liabilities, income, expenditure, net assets).
- **10 financial reports** generated from the ledger.
- **Donor fund tracking** — expenditure linked back to the grant that funded it.
- **Budget vs Actual** comparison per project and cost centre.
- **Approval workflows** before journal entries are posted to the ledger.

---

## 2. Package Structure

```
src/main/java/com/wamy/finance/
├── controller/
│   ├── AccountController.java
│   ├── JournalEntryController.java
│   ├── LedgerController.java
│   ├── BudgetController.java
│   └── ReportController.java
├── service/
│   ├── AccountService.java / AccountServiceImpl.java
│   ├── JournalEntryService.java / JournalEntryServiceImpl.java
│   ├── LedgerService.java / LedgerServiceImpl.java
│   ├── BudgetService.java / BudgetServiceImpl.java
│   └── ReportService.java / ReportServiceImpl.java
├── repository/
│   ├── AccountRepository.java
│   ├── JournalEntryRepository.java
│   ├── JournalEntryLineRepository.java
│   ├── LedgerEntryRepository.java
│   └── BudgetRepository.java
├── entity/
│   ├── Account.java
│   ├── JournalEntry.java
│   ├── JournalEntryLine.java
│   ├── LedgerEntry.java
│   └── Budget.java
├── dto/
│   ├── request/
│   └── response/
├── mapper/
└── exception/
    ├── DoubleEntryViolationException.java
    └── BudgetExceededException.java
```

---

## 3. Core Accounting Rules (Enforce in Code)

### Double-Entry Invariant

Every `JournalEntry` must balance: `sum(debit lines) == sum(credit lines)`.

Validate in `JournalEntryServiceImpl.post()` before persisting:

```java
BigDecimal totalDebit  = lines.stream()
    .filter(l -> l.getType() == EntryType.DEBIT)
    .map(JournalEntryLine::getAmount)
    .reduce(BigDecimal.ZERO, BigDecimal::add);

BigDecimal totalCredit = lines.stream()
    .filter(l -> l.getType() == EntryType.CREDIT)
    .map(JournalEntryLine::getAmount)
    .reduce(BigDecimal.ZERO, BigDecimal::add);

if (totalDebit.compareTo(totalCredit) != 0) {
    throw new DoubleEntryViolationException(
        "Journal entry does not balance: debit=%s credit=%s".formatted(totalDebit, totalCredit));
}
```

**Never skip this check**, even for system-generated entries.

### Account Types and Normal Balances

| Account Type | Normal Balance | Debit Effect | Credit Effect |
|--------------|---------------|--------------|---------------|
| Asset | Debit | Increase | Decrease |
| Liability | Credit | Decrease | Increase |
| Income | Credit | Decrease | Increase |
| Expenditure | Debit | Increase | Decrease |
| Net Assets / Fund | Credit | Decrease | Increase |

### Posting Workflow

```
DRAFT → PENDING_APPROVAL → APPROVED → POSTED
                         ↘ REJECTED
```

- Only `POSTED` entries appear in the ledger and reports.
- Only users with `ROLE_FINANCE` or `ROLE_ADMIN` may approve.
- Posted entries are **immutable** — reverse with a new counter-entry; never edit a posted entry.

---

## 4. Entity Design

### Account

```java
@Entity @Table(name = "accounts")
public class Account extends BaseEntity {
    private String code;                 // e.g. "1001", "4002"
    private String name;
    private AccountType type;            // ASSET, LIABILITY, INCOME, EXPENDITURE, NET_ASSETS
    private Long parentAccountId;        // nullable — null means root account
    private boolean isControlAccount;    // control accounts cannot receive direct postings
    private Long projectId;              // nullable — null means shared across all projects
}
```

### JournalEntry

```java
@Entity @Table(name = "journal_entries")
public class JournalEntry extends BaseEntity {
    private String referenceNumber;       // auto-generated, unique
    private LocalDate entryDate;
    private String description;
    private JournalEntryStatus status;    // DRAFT, PENDING_APPROVAL, APPROVED, POSTED, REJECTED
    private Long projectId;
    private Long donorId;                 // nullable
    private List<JournalEntryLine> lines; // must balance
    private String approvedBy;
    private LocalDateTime approvedAt;
}
```

### JournalEntryLine

```java
@Entity @Table(name = "journal_entry_lines")
public class JournalEntryLine extends BaseEntity {
    private Long journalEntryId;
    private Long accountId;
    private EntryType type;               // DEBIT or CREDIT
    private BigDecimal amount;            // always positive; type determines direction
    private String narration;
    private Long costCentreId;
}
```

---

## 5. The 10 Financial Reports

| # | Report Name | Description |
|---|-------------|-------------|
| 1 | Trial Balance | All accounts with debit/credit totals; confirms books balance |
| 2 | Income & Expenditure Statement | Revenue vs expenses for a period (NGO equivalent of P&L) |
| 3 | Balance Sheet | Assets, liabilities, and net assets at a point in time |
| 4 | Cash Flow Statement | Operating, investing, financing activities |
| 5 | Budget vs Actual | Budgeted vs actual spend per project/cost centre |
| 6 | Donor Fund Report | Receipts and disbursements per donor grant |
| 7 | Project Expenditure Report | All expenses per project with account breakdown |
| 8 | General Ledger | All posted entries per account for a period |
| 9 | Accounts Payable Ageing | Outstanding payables grouped by overdue days |
| 10 | Bank Reconciliation Statement | Bank statement vs book balance |

All reports are generated by `ReportService` using JPQL or native queries. Reports return a `ReportResult` DTO — never return raw entity lists.

---

## 6. Security Constraints

| Operation | Required Role |
|-----------|--------------|
| View reports | `ROLE_VIEWER`, `ROLE_FINANCE`, `ROLE_ADMIN` |
| Create / edit draft journal entries | `ROLE_FINANCE`, `ROLE_ADMIN` |
| Approve journal entries | `ROLE_FINANCE`, `ROLE_ADMIN` |
| Post to ledger | `ROLE_FINANCE`, `ROLE_ADMIN` |
| Manage chart of accounts | `ROLE_ADMIN` |
| Manage budgets | `ROLE_PROJECT_MANAGER`, `ROLE_ADMIN` |

Always use `@PreAuthorize` on service methods, not just controllers.

---

## 7. Monetary Precision

- All monetary values stored as `DECIMAL(19,4)` in MySQL.
- Java type: `BigDecimal` — **never use `double` or `float`** for money.
- Rounding: `RoundingMode.HALF_UP` when rounding is unavoidable.
- Currency: BDT (Bangladeshi Taka) by default; `currencyCode` column if multi-currency is added later.

---

## 8. Critical Anti-Patterns — Finance Module

```
✗ Post a journal entry that does not balance      → throw DoubleEntryViolationException
✗ Edit or delete a POSTED journal entry           → reverse with a counter-entry
✗ Use double/float for monetary amounts           → always BigDecimal
✗ Aggregate across projects without project filter → compliance violation
✗ Return entity objects from ReportService        → always use ReportResult DTOs
✗ Allow DRAFT entries to appear in reports        → filter status = POSTED only
✗ Skip approval step                             → DRAFT must reach POSTED via workflow
```

---

## 9. Reference Implementation Pattern

When adding a new financial feature, pattern-match against `JournalEntryServiceImpl`:

1. Validate input (FluentValidation-style `@Valid` + custom service-level checks).
2. Check double-entry balance for any entry that touches the ledger.
3. Persist with status `DRAFT`.
4. Return response DTO — never the entity.
5. Separate approval endpoint moves status → `PENDING_APPROVAL` → `APPROVED` → `POSTED`.

Report features pattern-match against `ReportService.generateTrialBalance()`.
