# Domain Glossary — Debt Collection

This glossary covers terminology used in the seed data and throughout the platform.

---

## Loan & Account Terms

| Term | Full Form | Meaning |
|------|-----------|---------|
| **DPD** | Days Past Due | Number of days since the borrower missed a payment. Higher DPD = more serious default. |
| **DPD Bucket** | — | Grouping of accounts by DPD range: 30-60, 60-90, 90-180, 180+ |
| **POS** | Principal Outstanding | The total amount still owed (principal + accrued interest) |
| **TOS** | Total Outstanding for Settlement | The amount the lender will accept to close the account. Always less than POS. |
| **Settlement Floor** | — | The absolute minimum amount the lender will accept. Going below this requires escalation. |
| **NPA** | Non-Performing Asset | A loan where repayment is overdue by 90+ days. Regulatory classification. |
| **Write-off** | — | When the lender removes the loan from their books as unrecoverable. Borrower still owes the money. |

## Collection Process

| Term | Meaning |
|------|---------|
| **Soft collection** | Early-stage outreach (30-60 DPD). Reminders, payment links, gentle nudges. |
| **Hard collection** | Late-stage (90+ DPD). Negotiation, settlement offers, legal notices. |
| **Escalation** | Moving a case to a more senior agent or legal team. Triggered by borrower distress, threats, or repeated non-response. |
| **Settlement** | One-time payment (typically 30-70% of POS) that closes the account permanently. |
| **Restructuring** | Converting outstanding to a new EMI plan with revised terms. |
| **Disposition** | The outcome of a collection call/interaction. Examples: "Promise to Pay", "Refused", "Not Reachable", "Settled". |
| **PTP** | Promise to Pay. Borrower verbally commits to paying by a specific date. |
| **DNC** | Do Not Contact. Borrower has explicitly requested no further communication. Legal obligation to comply. |

## Compliance & Regulatory

| Term | Meaning |
|------|---------|
| **NBFC** | Non-Banking Financial Company. Licensed lenders that aren't traditional banks. Riverline's clients are NBFCs. |
| **RBI** | Reserve Bank of India. The central bank that regulates all lending in India. |
| **DPDPA** | Digital Personal Data Protection Act. India's data privacy law (2023). Governs PII handling. |
| **Quiet Hours** | 8 PM to 8 AM — no outbound borrower communication allowed during this window. |
| **ZCM** | Zero Contact Mode. When the system has exhausted all channels and the borrower is completely unresponsive. Account enters dormancy. |
| **Hardship** | When a borrower demonstrates genuine inability to pay (job loss, medical emergency). Triggers special handling — reduced payments, hold on collections. |
| **DRA** | Debt Recovery Agent. The entity performing collections on behalf of the lender. Must be registered with the NBFC. |

## Platform-Specific

| Term | Meaning |
|------|---------|
| **Tenant** | A banking partner (NBFC or cooperative bank) whose borrower data is managed on the platform. |
| **Client** | Same as tenant — used interchangeably. The `clientId` field in the data refers to the tenant. |
| **Counselor** | A debt counselor — the human (or AI) agent who communicates with borrowers. |
| **Assigned accounts** | Borrowers allocated to a specific counselor. Each borrower has exactly one assigned counselor at a time. |
| **Channel** | The communication medium — WhatsApp, SMS, voice call, or in-app chat. |

## Data Fields in Seed Data

| Field | Found In | Meaning |
|-------|----------|---------|
| `clientId` | All collections | Identifies which tenant owns this record |
| `assignedTo` | borrowers | The userId of the debt-counselor assigned to this borrower |
| `dpdBucket` | borrowers | Current DPD classification |
| `outstandingAmount` | borrowers | Total amount owed (equals POS) |
| `posAmount` | borrowers | Principal Outstanding — same as outstandingAmount |
| `tosAmount` | borrowers | Settlement amount the lender will accept |
| `settlementFloor` | borrowers | Minimum acceptable settlement |
| `reference` | payments | Internal payment tracking ID. Format: `PAY-{clientId prefix}-{uuid}` |
| `gatewayReference` | payments | External payment gateway's reference ID |
| `createdBy` | borrowers | The userId of who created this record in the system |
