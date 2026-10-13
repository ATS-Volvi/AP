# Architecture & Domain Decision Register

## Purpose & Governance
In accordance with Section 8 of the project specifications, open architectural decisions and policy forks must **never be silently resolved**. This register captures all 8 open points identified in `AP_Entry_Point_Catalogue_v0.1.pdf` (Page 56) and the Domain Map, establishing clearly labelled **provisional policies** to enable safe development while awaiting client governance sign-off.

---

### DEC-01: AI Autonomy Levels & Human Fallbacks
- **Status**: `PROVISIONAL — Baseline Confirmed for Implementation`
- **Referenced Scenarios**: 02.05, 02.10, 03.01, 03.03, 03.04, 03.06, 06.09, 07.01, 08.05
- **Affected Entry Points**: EP-INT-02, EP-INT-05, EP-CAP-01, EP-MAT-01, EP-EXC-05, EP-APR-01

**Context & Catalogue Directive**:
The catalogue proposes four decision tiers across 149 processing steps: CODE (115 steps), AI SUGGESTS (5 steps), AI DECIDES (2 steps), and PERSON (27 steps). Confirming these closes the autonomy open item for AP v1.

**Options Evaluated**:
Option A: Adopt proposed hybrid levels with confidence thresholds (Recommended).
Option B: Full manual review for all AI steps (disables AI DECIDES).
Option C: Increased AI autonomy (allow AI approval below €500).

**Provisional Policy Implemented**:
**Option A (Proposed Catalogue Baseline)**: AI DECIDES is restricted strictly to document classification (EP-INT-02) and field extraction (EP-CAP-01) above 85% confidence. Document text is treated as data only. Any confidence below 85% creates a human review task. AI NEVER approves invoices, releases holds, or touches money/tax.

---

### DEC-02: Scope of ERP Posting Acknowledgements & Refusals in v1
- **Status**: `PROVISIONAL — Implemented via Mock Adapter`
- **Referenced Scenarios**: 08.01, 09.05, 11.01
- **Affected Entry Points**: EP-PST-01, EP-PST-02, EP-PST-03

**Context & Catalogue Directive**:
The catalogue notes on Page 56: 'EP-PST-02 and EP-PST-03 exist only if posting is in v1. Otherwise EP-PST-01 is the last step and the adapter is outside v1.'

**Options Evaluated**:
Option A: Implement mock ERP adapter with asynchronous ACK (EP-PST-02) and REFUSED (EP-PST-03) callbacks in v1.
Option B: Scope v1 strictly up to EP-PST-01 (Ready to post) and omit downstream posting lifecycle.

**Provisional Policy Implemented**:
**Option A**: Implement full lifecycle (EP-PST-01 -> EP-PST-02 / EP-PST-03) using a robust mock ERP adapter interface. Transition T09 marks `Ready to post`, T10 marks `Posted` on ACK, and T10b reverts to `Approved` on refusal. This ensures complete end-to-end auditability and transition engine completeness.

---

### DEC-03: Partial Invoice Approval and Line-Level Posting
- **Status**: `PROVISIONAL — Whole-Invoice Freeze Active`
- **Referenced Scenarios**: 07.06 (The undisputed portion is approved; the rest is held)
- **Affected Entry Points**: EP-EXC-01, EP-EXC-04, EP-EXC-06, EP-APR-03, EP-APR-04

**Context & Catalogue Directive**:
Catalogue note: 'Line-level holds freeze the whole invoice for now. Allowing partial processing would add an entry point to split the invoice into postable and held parts.'

**Options Evaluated**:
Option A: Strict whole-invoice freeze while any line hold is active (Default state model).
Option B: Invoice splitting (split into original held invoice and a postable child invoice).
Option C: Line-level approval and multi-stage posting.

**Provisional Policy Implemented**:
**Option A (Whole-Invoice Freeze with Configurable Split Flag)**: Any active hold on an invoice or invoice line blocks forward transition (T06/T07/T09). In Scenario 07.06, the AP Controller can formally split the line into a separate credit/dispute case or resolve the dispute before forward movement.

---

### DEC-04: Automated Rejection of Exact Duplicates
- **Status**: `PROVISIONAL — Safe Auto-Reject with Near-Match Review`
- **Referenced Scenarios**: 02.08, 05.03
- **Affected Entry Points**: EP-CAP-04, EP-ELG-01

**Context & Catalogue Directive**:
Catalogue note: 'EP-CAP-04 adds exception type DUPLICATE_REVIEW and auto-rejects exact duplicates. Confirm that exact duplicates may be rejected without a person.'

**Options Evaluated**:
Option A: Automated rejection (T11) for exact hash/fingerprint duplicates, opening DUPLICATE_REVIEW exception only for suspected/near duplicates.
Option B: Mandatory AP Clerk confirmation for all duplicate detections.

**Provisional Policy Implemented**:
**Option A**: Exact 100% cryptographic file hash duplicates on live documents are auto-transitioned to `Rejected` (T11 with reason `EXACT_DUPLICATE`). Near matches (same vendor + invoice number + amount with different files) raise a `DUPLICATE_REVIEW` hold requiring Clerk review.

---

### DEC-05: Configuration Publication Dual-Control Authorization
- **Status**: `PROVISIONAL — Dual Control Guard Active`
- **Referenced Scenarios**: Act 12, Configuration Management
- **Affected Entry Points**: EP-CFG-01

**Context & Catalogue Directive**:
Catalogue note: 'EP-CFG-01 requires a second person to publish configuration. Confirm who that is per client.'

**Options Evaluated**:
Option A: Finance Director must approve configurations authored by AP Controller.
Option B: Any two AP Controllers (Peer dual-control).
Option C: Single AP Controller without dual-control.

**Provisional Policy Implemented**:
**Option A (Segregation of Duties)**: Configuration draft authored by AP Controller must be approved by Finance Director (or distinct Senior Controller). Strict `author_id != approver_id` guard enforced.

---

### DEC-06: Exception Deadlines, SLA Timers, and Escalation Chains
- **Status**: `PROVISIONAL — Configurable SLA Active`
- **Referenced Scenarios**: 07.04, 07.05
- **Affected Entry Points**: EP-EXC-07, EP-EXC-08

**Context & Catalogue Directive**:
Catalogue note: 'EP-EXC-07 needs default due times per exception type and an escalation chain per client.'

**Options Evaluated**:
Option A: Configurable SLA table in ConfigurationVersion with default 5-business-day timer for price mismatch and 3-business-day timer for missing receipt.
Option B: Hardcoded 48-hour deadline for all exceptions.

**Provisional Policy Implemented**:
**Option A**: Default deadlines: `PRICE_MISMATCH`: 5 days (escalates to Buyer Manager); `MISSING_RECEIPT`: 3 days (escalates to Warehouse Lead); `DUPLICATE_REVIEW`: 2 days (escalates to AP Controller); `TAX_MISMATCH`: 2 days (escalates to Tax Lead).

---

### DEC-07: Domain Ownership of Vendor Disputes and Quality Claims
- **Status**: `PROVISIONAL — Dual Inbound Support`
- **Referenced Scenarios**: 05.05, 07.07, 09.05
- **Affected Entry Points**: EP-REF-06, EP-EXC-01

**Context & Catalogue Directive**:
Catalogue note: 'EP-REF-06 assumes another domain publishes these events. If AP owns them, they become commands instead.'

**Options Evaluated**:
Option A: Event handler consumes domain events from external Procurement/ERP, with an internal AP Command fallback for manual logging.
Option B: Pure event handler with no manual input.

**Provisional Policy Implemented**:
**Option A**: AP agent handles `ap.vendor.dispute_or_claim_changed.v1` as an event handler (EP-REF-06) while exposing `EP-EXC-01` (PlaceHold) for AP staff to manually hold invoices when notified informally.

---

### DEC-08: Policy on Auto-Accepting AI Coding Proposals
- **Status**: `PROVISIONAL — Mandatory Human Confirmation`
- **Referenced Scenarios**: 08.05, 08.07, 12.05
- **Affected Entry Points**: EP-APR-01, EP-APR-02

**Context & Catalogue Directive**:
Catalogue note: 'EP-APR-01 never auto-accepts in this draft. AI suggests GL and cost object from history, with confidence.'

**Options Evaluated**:
Option A: Never auto-accept; AP Clerk or Approver must always confirm coding in EP-APR-02.
Option B: Auto-accept recurring utilities/rent coding above 95% confidence.

**Provisional Policy Implemented**:
**Option A (Catalogue Strict Mandate)**: EP-APR-01 outputs proposal into `coding_splits` with status `proposed`. Forward progression to approval (EP-APR-03) requires explicit confirmation (EP-APR-02).

---
