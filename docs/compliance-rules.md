# Compliance Rules

These rules govern how the platform handles borrower data. They are abstracted from financial data protection regulations applicable to the platform's operating jurisdiction.

---

## 1. Data Residency

- All borrower data must remain within the platform's infrastructure. No external API calls that transmit PII.
- Tokenization keys must be stored separately from the data they protect.
- Backups containing PII must follow the same isolation guarantees as live data.

## 2. Data Retention

| Data Type | Retention Period |
|-----------|----------------|
| Active borrower records | Duration of lending relationship |
| Closed accounts (PII) | Purge within 90 days of closure |
| Closed accounts (non-PII metadata) | May be retained indefinitely |
| Audit logs | 7 years minimum |
| Conversation transcripts | 5 years from last interaction |
| Payment records | 7 years |

## 3. Communication Timing (Quiet Hours)

- No outbound messages to a borrower between **8:00 PM and 8:00 AM** local time.
- If a borrower initiates a message during restricted hours, the system may respond.
- All communications (including those during quiet hours) must be logged in the audit trail.
- Quiet hours violations must be flagged in compliance reports.

## 4. Data Minimization

- Collect and store only the PII necessary for the lending relationship.
- Internal tools and dashboards must default to the minimum masking level required for the viewer's role.
- Debug logging must never contain PII, even in development environments.
- API responses must not include fields the requesting role is not authorized to see. Do not return masked placeholders for unauthorized fields — omit them entirely.

## 5. Breach Detection & Response

If cross-tenant data access is detected (a user from Tenant A accessing Tenant B's data):

1. **Immediately** revoke the user's session.
2. **Log** the incident with full detail (who, what, when, how).
3. **Flag** the incident for manual review — do not auto-resolve.
4. **Notify** the admin of the affected tenant.

Cross-tenant access includes:
- Direct data access (reading borrower records)
- Indirect inference (response time differences that reveal existence)
- Metadata leakage (error messages containing cross-tenant identifiers)

## 6. PII in Logs

- Application logs (stdout, debug, error) must NEVER contain PII.
- Audit logs record WHAT was accessed but not the DATA itself.
- If an endpoint URL contains PII (e.g., borrower phone in a path parameter), the audit record must redact it.
- Stack traces in error responses must not expose internal file paths or PII.

## 7. Consent & Right to Erasure

- Borrowers may request deletion of their data.
- Upon a valid erasure request: all PII must be purged. Audit records of the relationship may be retained (with PII redacted).
- Tokenization mappings for the erased borrower must be destroyed (making tokens permanently irreversible).

## 8. Access Control Principles

- **Least privilege**: Every role has the minimum access required for its function.
- **Deny by default**: Unrecognized roles or missing auth must be denied, not granted default access.
- **Scope isolation**: A debt counselor accessing borrowers outside their assignment is a security event, not just a permission error.
- **Audit everything**: Both successful and failed access attempts are logged with equal detail.
