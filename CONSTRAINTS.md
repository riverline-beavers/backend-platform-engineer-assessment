# Technical Constraints

## All infrastructure runs locally

Your entire system must run via `docker compose up` on a local machine. No cloud services, no managed backends, no external dependencies that require internet access or API keys.

**Not allowed:**
- Firebase, Supabase, PlanetScale, MongoDB Atlas, or any managed database service
- Cloud functions, serverless platforms, or hosted compute
- Any service that requires an account, API key, or internet connection to function

**Allowed:**
- MongoDB running in a Docker container
- Redis running in a Docker container
- Any service you can run locally in Docker

## No pre-built authentication or authorization frameworks

You are building the auth and RBAC system. Do not use libraries that solve this for you.

**Not allowed:**
- Firebase Auth, Auth0, Clerk, Supabase Auth, or any managed identity provider
- `casbin`, `accesscontrol`, `casl`, `permit.io`, or any policy/authorization engine
- `passport.js` with pre-built strategies that handle the full auth flow
- Any "multi-tenant" npm package that provides tenant isolation out of the box

**Allowed:**
- `jsonwebtoken` — for signing and verifying JWTs (this is a crypto primitive, not an auth framework)
- `bcrypt` or `argon2` — for password hashing
- Node.js native `crypto` module — for encryption, HMAC, key derivation
- Any cryptographic primitive library (e.g., format-preserving encryption algorithms)

The distinction: **primitives** (sign a token, hash a password, encrypt a value) are fine. **Frameworks** (manage sessions, enforce policies, handle OAuth flows) are not.

## No managed abstractions that hide the core logic

The purpose of this assessment is to see how YOU design and build these systems. If a library does the thinking for you, you haven't demonstrated the skill.

**Rule of thumb:** If removing the library would require you to redesign your architecture, it's doing too much. If removing it would just mean writing 20 lines of the same logic by hand, it's fine.

**Examples:**
- Using Mongoose for MongoDB access → fine (it's a query interface, not a design decision)
- Using a "multi-tenant Mongoose plugin" that auto-scopes queries → not fine (that's the core thing we're testing)
- Using BullMQ for job queues → fine (it's infrastructure)
- Using a library that auto-propagates tenant context through queues → not fine (that's what you need to build)

## Tenant isolation must be database-scoped

Tenant isolation must be enforced at the database level — each tenant gets their own database. This is a hard requirement, not a suggestion.

**Not acceptable:**
- A single shared database with a `tenantId` filter on every query (this is what the broken current system does — you're replacing it, not replicating it)
- Application-level row filtering where a query bug could leak data
- "Virtual" isolation through views or collection prefixes within one database

**Required:**
- Separate MongoDB databases per tenant
- A request for Tenant A physically cannot query Tenant B's database
- Isolation is architectural, not just logic

## Explainability requirement

For anything you submit, you must be able to explain how it works. This applies to:

- **Libraries:** If you use a third-party package for a core requirement (tokenization, encryption, context propagation), explain in your architecture doc how it works under the hood and why you chose it over alternatives.
- **Architecture decisions:** Don't just describe WHAT your system does — explain WHY you designed it that way and what alternatives you considered.
- **Code:** Your code should be readable without comments. If a reviewer can't understand what a module does by reading it, that's a problem.

The Loom recording and decision journal are where explainability is evaluated. If you can't walk through your own system and explain the design choices, that signals you didn't build it.
