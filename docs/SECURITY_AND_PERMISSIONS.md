# Security, Permissions & Segregation of Duties (SoD)

## 1. Authentication & Actor Derivation

1. **Session-Derived Actor**: The authenticated user ID, roles, and accessible legal entity memberships are derived **strictly from the cryptographically validated session JWT** (`req.user`). The system rejects any request payload attempting to specify or override the acting actor ID.
2. **Token Format**: Standard RFC 7519 JWT signed with RS256 containing:
   - `sub`: Unique internal user UUID
   - `roles`: Array of role names (e.g. `["AP_CLERK", "AP_CONTROLLER"]`)
   - `legal_entities`: Array of authorized ABC BV legal entities (e.g. `["ABC_NL_01", "ABC_NL_02"]`)
   - `cost_centres`: Array of cost centre codes authorized for budget approval

---

## 2. Role-Based Access Control (RBAC) Matrix

The ten actors map to role definitions enforced at the API controller and Domain command levels:

| Subsystem Group | Entry Points | AP Clerk | AP Controller | Buyer / Procurement | Warehouse | Budget Owner | Finance Director | Auditor |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Intake (EP-INT)** | EP-INT-01, 03, 06, 07 | **EXEC** | **EXEC** | - | - | - | - | READ |
| **Capture (EP-CAP)** | EP-CAP-03 | **EXEC** | **EXEC** | - | - | - | - | READ |
| **Screening (EP-SCR)**| EP-SCR-02 | **EXEC** | **EXEC** | - | - | - | - | READ |
| **Eligibility (EP-ELG)**| EP-ELG-02, 03 | READ | **EXEC** | - | - | - | **EXEC** | READ |
| **Matching (EP-MAT)**| EP-MAT-02, 03, 05, 06 | READ | **EXEC** | **EXEC** | - | - | - | READ |
| **Exceptions (EP-EXC)**| EP-EXC-01, 02, 03, 04, 05, 06, 08, 09 | EXEC | **EXEC** | EXEC (assigned) | EXEC (assigned) | - | **EXEC** | READ |
| **Approvals (EP-APR)**| EP-APR-02, 04, 05, 06, 08 | READ | EXEC | - | - | **EXEC (own CC)** | **EXEC (> threshold)**| READ |
| **Master Data (EP-REF)**| EP-REF-07 | EXEC | **EXEC** | EXEC | - | - | - | READ |
| **Configuration (EP-CFG)**| EP-CFG-01 | - | **AUTHOR** | - | - | - | **APPROVE** | READ |
| **Queries (EP-QRY)** | EP-QRY-01 to 08 | Own scope | All scope | Assigned scope | Warehouse scope | Own CC scope | All scope | All (Read Only) |

---

## 3. Mandatory Segregation of Duties (SoD) Invariants

The platform enforces four critical segregation of duties rules at the database transaction level:

### SoD-01: Dual-Control Configuration Publication (`EP-CFG-01`)
- **Rule**: An author of a configuration draft cannot approve or publish that draft.
- **Guard**: `author_actor_id != approver_actor_id`.
- **Enforcement**: Enforced inside `EP-CFG-01` before committing the configuration version.

### SoD-02: Purchase Order Creator vs. Invoice Approver (`EP-APR-04`)
- **Rule**: The individual buyer who raised and issued the Purchase Order cannot act as the final invoice approval authority for that invoice.
- **Guard**: `po.created_by_actor_id != approval_task.assigned_actor_id`.
- **Enforcement**: If the budget owner is also the PO creator, the approval task automatically escalates to the Finance Director or departmental peer.

### SoD-03: Vendor Bank Account Modifier vs. Payment Releaser (`EP-REF-01`, `EP-PST-01`)
- **Rule**: The user who updated or verified a vendor's IBAN/bank details cannot authorize the payment release for an invoice paid to that new bank account within 30 days.
- **Guard**: Verified against `vendor_bank_details.verified_by_actor_id != invoice.release_actor_id`.

### SoD-04: Self-Approval Prevention (`EP-APR-04`)
- **Rule**: An AP Clerk who manually corrected fields or entered an invoice cannot approve the invoice.
