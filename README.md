# Backend Platform Engineer — Take-Home Assessment

> **Disclaimer:** This is a simulated assessment. All data is synthetic. Nothing here reflects Riverline's actual infrastructure or security posture. See [DISCLAIMER.md](DISCLAIMER.md) for full details.

## What This Is

A take-home assessment for the Backend Engineer — Platform & Security role. You'll design and build a secure, multi-tenant data platform for a debt collection system.

**Read `spec.md` first.** It contains everything: context, requirements, API contract, compliance rules, constraints, and deliverables.

## Repository Structure

```
├── spec.md                    ← START HERE. The full specification.
├── seed-data/
│   ├── borrowers.json         ← ~2,000 borrower records (3 tenants, shared-DB format)
│   ├── conversations.json     ← Conversation transcripts
│   ├── payments.json          ← Payment history
│   ├── users.json             ← System users (admins, counselors, engineers, viewers)
│   ├── access_logs.json       ← 30 days of access logs
│   └── clients.json           ← Tenant registry (3 banking partners)
├── docs/
│   ├── compliance-rules.md    ← Data protection rules
│   └── domain-glossary.md     ← Debt collection terminology
└── README.md                  ← You are here
```

## Quick Start

1. Read `spec.md` thoroughly
2. Explore the seed data — understand what you're working with
3. Build your system
4. Migrate the seed data into your architecture
5. Write tests that prove your system works
6. Submit

## Constraints

- **Timeline:** 3 days from receipt
- **Stack:** Node.js / TypeScript / MongoDB / BullMQ (Redis)
- **Setup:** `docker compose up` must bring up your entire system. No manual steps.
- **All infrastructure runs locally.** No Firebase, Supabase, managed databases, or cloud services.
- **No pre-built auth/RBAC frameworks.** No casbin, accesscontrol, passport strategies, Auth0, Clerk. You build it. Crypto primitives (jsonwebtoken, bcrypt) are fine.
- **Tenant isolation must be database-scoped.** Separate databases per tenant — not row-level filtering on a shared DB.
- **Explainability.** If you can't explain how something in your system works, don't use it.

**Read [CONSTRAINTS.md](CONSTRAINTS.md) for the full rules on what's allowed and what's not.**

## Deliverables

1. Public GitHub repo
2. Single-command Docker Compose setup
3. Architecture document (2-3 pages)
4. Migration tooling
5. Your own test suite
6. Decision journal
7. 5-minute Loom recording

Full details in `spec.md`, Section 8.

## Questions?

If something is ambiguous, make a reasonable assumption and document it. If something is genuinely blocking (corrupt data, unclear deliverable format), email us.
