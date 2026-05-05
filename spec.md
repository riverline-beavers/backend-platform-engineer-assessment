# Backend Engineering Take-Home: Secure Multi-Tenant Data Platform

## 1. Context

You're joining a fintech platform that operates AI-powered debt counseling agents. These agents conduct thousands of conversations daily with borrowers across multiple banking partners (NBFCs). Each banking partner's data — borrower records, conversation transcripts, payment histories — must be completely isolated from every other partner's data.

The platform currently runs on a **shared database architecture**: all partners' data lives in one MongoDB database, separated only by a `clientId` field on each document. This architecture has already caused security incidents (you'll find evidence in the seed data). It doesn't meet the compliance bar required for financial data at scale.

Your job: **design and build the replacement** — a secure, multi-tenant data platform from scratch.

---

## 2. The Current System (what exists today)

You are NOT building on top of the current system. You are replacing it.

What exists today:

```
┌─────────────────────────────────────────────────┐
│            Shared MongoDB Database              │
│                                                 │
│  All tenants in one database.                   │
│  Separation by clientId field on every document.│
│  No database-level isolation.                   │
│  No PII protection.                             │
│  No proper access control.                      │
└─────────────────────────────────────────────────┘
```

**Known problems:**
- A single query bug leaks data across tenants
- No database-level isolation guarantees
- PII stored in plaintext across all collections
- Access control is binary (authenticated = full access)
- No audit trail
- Evidence of past cross-tenant data access exists in the access logs

You receive the **seed data** — a dump of the current shared database. Your system must ingest this data into whatever architecture you design.

---

## 3. Requirements

Build a system that satisfies ALL of the following. How you architect it is your decision — document and defend your choices.

### 3.1 Tenant Isolation

- Each tenant's data must live in its own database. No shared data layer between tenants.
- A request authenticated as Tenant A must never be able to read, write, or infer the existence of Tenant B's data.
- Tenant context must be correct in ALL execution paths — not just HTTP request handlers. Background jobs, scheduled tasks, and webhook receivers all process tenant-specific data and must operate in the correct tenant context.

### 3.2 PII Tokenization

All personally identifiable information must be tokenized:

- **Reversible**: Authorized roles can retrieve the original value.
- **Format-preserving**: A tokenized phone number must still look like a phone number (10 digits). A tokenized Aadhaar must still be 12 digits.
- **Consistent**: The same value must always produce the same token within a tenant. If a borrower's phone appears in multiple places, the token must be identical everywhere.
- **Tenant-scoped**: Tokenization is per-tenant. Same phone in Tenant A and Tenant B produces different tokens.

**What counts as PII:**
- Phone number
- Aadhaar number
- PAN number
- Email address
- Bank account number
- Full name

### 3.3 Access Control (RBAC)

Four roles:

| Role | Scope | PII Access |
|------|-------|------------|
| `admin` | All data across all tenants | Full PII (detokenized) |
| `debt-counselor` | **Only borrowers assigned to them** | Phone + name visible. Aadhaar, PAN, bank account masked. |
| `engineer` | Read access to all tenants (debugging/support) | All PII masked. System data (audit logs, metrics) visible. |
| `client-viewer` | Own tenant only, read-only | All PII masked. Aggregate data only. |

**Scope enforcement for debt-counselors:**

In the seed data, borrowers are distributed across counselors via the `assignedTo` field. A debt-counselor must ONLY be able to query, view, and act on borrowers assigned to them. This scoping must be enforced at the data layer — a counselor's query must never touch records that aren't theirs. Attempting to access another counselor's borrower must be denied, not just filtered from results.

### 3.4 Audit Trail

Every data access must be logged:

- **Who**: User ID, role, tenant
- **What**: Endpoint, method, resource ID(s) accessed
- **When**: Timestamp (UTC)
- **Masking level**: What level of data the user actually saw (full PII, partial, masked)
- **Outcome**: Success/failure, response code

Rules:
1. Audit logs must never contain PII.
2. Audit logs are append-only. No deletion, no modification.
3. Tenant A's audit logs must not be visible to Tenant B.
4. Failed access attempts (wrong tenant, wrong scope, permission denied) must be logged with the same detail as successful ones.

### 3.5 Migration

Your system must include tooling that migrates the provided seed data into your architecture:

- The migration must preserve all data — no silent drops, no corruption.
- Migration must support **zero-downtime operation**. The system must be usable during migration. How you achieve this is your design decision.
- Migration must be **reversible** — if something goes wrong, you can revert without data loss.
- Migration tooling must complete for the provided seed data within **60 seconds**.

### 3.6 Resilience

- If one tenant's database becomes unreachable, requests for other tenants must continue to succeed.
- A single-tenant failure must not cascade.
- Error responses from the failing tenant must not leak information about other tenants or the system's internal state.

### 3.7 Async Processing

Your system must handle tenant-aware processing in at least these contexts:

| Context | Example | Challenge |
|---------|---------|-----------|
| Background jobs | Generate a compliance report for a tenant | Job must execute in the correct tenant's database context |
| Scheduled tasks | Daily check for overdue payments across all tenants | Must iterate tenants correctly — not run in one global context |
| Webhook receivers | Payment gateway confirms a payment | Tenant identity comes from the payment reference, not auth headers |

---

## 4. API Contract

Your system must expose the following endpoints. You design the schema, request/response shapes, and error handling.

| Method | Path | Description | Notes |
|--------|------|-------------|-------|
| `POST` | `/borrowers` | Create a borrower record | |
| `GET` | `/borrowers` | List borrowers | Scoped by role (counselor sees only assigned) |
| `GET` | `/borrowers/:id` | Get borrower detail | PII masking varies by role |
| `PUT` | `/borrowers/:id` | Update borrower record | |
| `GET` | `/conversations/:borrowerId` | Get conversation history | |
| `POST` | `/conversations/:borrowerId/messages` | Send a message | Must respect quiet hours (see Section 6) |
| `POST` | `/payments` | Record a payment | |
| `GET` | `/payments/:borrowerId` | Get payment history for a borrower | |
| `GET` | `/reports/compliance` | Generate compliance report | Async — triggers background job |
| `POST` | `/webhooks/payment-gateway` | Receive payment confirmation | Tenant from payment reference, not auth |
| `GET` | `/audit-logs` | Query audit trail | Admin and engineer only |
| `GET` | `/health` | System health | |

---

## 5. Compliance Rules

These rules govern how the platform handles borrower data. They are abstracted from financial data protection regulations.

### Data Residency
- All borrower data must remain within the platform's infrastructure. No external API calls that transmit PII.
- Tokenization keys must be stored separately from the data they protect.

### Data Retention
- Active borrower records: retained for the duration of the lending relationship.
- Closed accounts: all PII must be purged within 90 days of closure. Non-PII metadata may be retained.
- Audit logs: retained for 7 years minimum.

### Communication Timing
- No outbound messages to a borrower between **8 PM and 8 AM** local time.
- If a borrower sends a message during restricted hours, the system may respond.
- All communications must be logged in the audit trail.

### Data Minimization
- Internal tools and dashboards must default to the minimum masking level required for the viewer's role.
- Debug logging must never contain PII, even in development environments.

### Breach Detection
- If cross-tenant data access is detected (a user from Tenant A accessing Tenant B's data), the system must:
  1. Immediately revoke the session
  2. Log the incident with full detail
  3. Flag for manual review

---

## 6. Constraints

| Constraint | Value | Rationale |
|------------|-------|-----------|
| **Single command startup** | `docker compose up` must bring up your ENTIRE system — databases, queues, API, everything. No manual steps. | If we can't run it, we can't evaluate it. |
| **All infrastructure local** | No Firebase, Supabase, managed databases, cloud services, or anything requiring internet/API keys. Everything runs in Docker. | We evaluate your engineering, not your ability to configure a managed service. |
| **No auth/RBAC frameworks** | No casbin, accesscontrol, casl, passport strategies, Auth0, Clerk, Firebase Auth. Build your own. Crypto primitives (jsonwebtoken, bcrypt, crypto) are fine. | The auth and RBAC system IS what we're evaluating. Using a pre-built one skips the test. |
| **Database-scoped isolation** | Separate MongoDB databases per tenant. Not row-level filtering on a shared DB. | Application-level filtering is what the broken current system does. You're building the replacement. |
| **Explainability** | If you use a library for a core requirement, explain how it works in your architecture doc. | If you can't explain it, you didn't build it. |
| Max concurrent MongoDB connections | 10 total | Forces real connection management, not "new connection per request" |
| Migration time | < 60 seconds for seed data | Forces efficient batch processing |
| Timeline | **3 days** from receipt | |
| Language | Node.js / TypeScript | |
| Database | MongoDB | |
| Queue | BullMQ (Redis-backed) | For background job processing |

**See [CONSTRAINTS.md](CONSTRAINTS.md) for detailed rules on what's allowed and what's not.**

---

## 7. What You Receive

| Material | Description |
|----------|-------------|
| This spec document | Requirements, API contract, compliance rules |
| `seed-data/` | ~2,000 records across 3 tenants in the current shared-DB format. Collections: borrowers, conversations, payments, users, access_logs, clients |
| `docs/compliance-rules.md` | Section 5 in standalone format |
| `docs/domain-glossary.md` | Debt collection terminology (DPD, POS, TOS, settlement, escalation, etc.) |

There is no scaffold. There is no starter code. There are no provided test scripts. You design and build the system — and prove it works — from scratch.

---

## 8. What You Submit

### Mandatory deliverables

1. **Public GitHub repository** with your complete solution

2. **Single-command setup** — `docker compose up` brings up everything. No exceptions.

3. **Architecture document** (2-3 pages max) covering:
   - Your database schema design with rationale
   - How tenant isolation works in your system
   - How PII tokenization is implemented
   - How tenant context propagates through async paths
   - Any tradeoffs you made and why

4. **Migration tooling** — commands/scripts that ingest the seed data into your architecture

5. **Test suite** — your own tests proving your system works under adversarial conditions:
   - Tenant isolation under concurrent load (can data ever leak between tenants?)
   - Scope enforcement (can a counselor access another counselor's borrowers?)
   - PII tokenization correctness and consistency
   - Role-based masking (does each role see only what it should?)
   - Resilience (what happens when a tenant's database goes down?)
   
   The depth and design of your test suite is itself an evaluation signal.

6. **Decision journal** — a raw markdown file documenting:
   - At least 3 architectural decisions with alternatives you considered
   - At least 2 moments where you were wrong, stuck, or changed direction
   - At least 1 thing you intentionally did NOT build and why
   - This is not a polished document. If every entry reads like a blog post, we'll assume it was generated.

7. **5-minute Loom recording** — walk through your system: architecture, key decisions, what you'd do differently with more time

### Submission format

Email your GitHub repo link and Loom recording link to the provided address by the deadline.

---

## 9. Evaluation

You will be evaluated on:

- **Architecture & schema design** — is your data model well-reasoned? Does it support the requirements efficiently?
- **Tenant isolation correctness** — can data ever leak across tenants, in any execution path?
- **PII protection quality** — is tokenization consistent, reversible, and correctly scoped?
- **Access control rigor** — does scope enforcement work at the data layer? Can a counselor ever see another counselor's accounts?
- **Migration approach** — is it safe? Reversible? Performant?
- **Security mindset** — did you notice problems in the seed data? Does your system prevent the failures that the old system allowed?
- **Engineering judgment** — the decisions you made, the tradeoffs you navigated, and how you defended them
- **Production readiness** — failure handling, observability, audit completeness

**We value depth over breadth.** A thorough, well-architected solution covering 70% of requirements is better than a shallow pass at 100%.

---

## 10. Questions & Ambiguity

Parts of this spec are intentionally underspecified. When you encounter ambiguity:

1. Make a reasonable assumption
2. Document it in your decision journal
3. Proceed

If something is genuinely blocking (corrupt seed data, unclear deliverable format), email us.

Candidates who ask thoughtful clarifying questions before submitting are viewed positively.
