# State Machine & Business Logic Engine

## 1. Dual-Entity Lifecycle Architecture

The platform strictly avoids conflating inbound raw documents with commercial accounting invoices. Two formal state machines govern the intake and accounting pipelines.

```mermaid
stateDiagram-v2
    [*] --> Inbound_Received: EP-INT-01
    Inbound_Received --> Inbound_Classified: EP-INT-02 / EP-INT-03
    Inbound_Classified --> Inbound_Duplicate: EP-INT-04 (Duplicate match)
    Inbound_Classified --> Inbound_Returned: EP-INT-06 (Wrong entity)
    Inbound_Classified --> Inbound_Filed: EP-INT-07 (Delivery note/statement)
    Inbound_Classified --> Inbound_Converted: EP-INT-08 (Creates Invoice)

    state "Invoice Pipeline" as IP {
        [*] --> Received: T01 (EP-INT-08)
        Received --> Captured: T02 (EP-CAP-02)
        Captured --> Screened: T03 (EP-SCR-01)
        Screened --> Eligible: T04 (EP-ELG-01)
        Eligible --> Matched: T05 (EP-MAT-01 / EP-MAT-03)
        Matched --> Awaiting_Approval: T06 (EP-APR-03)
        Awaiting_Approval --> Approved: T07 (EP-APR-04)
        Awaiting_Approval --> Matched: T08 (EP-APR-04 Rejected/Rematch)
        Approved --> Ready_To_Post: T09 (EP-PST-01)
        Ready_To_Post --> Posted: T10 (EP-PST-02 ERP Ack)
        Ready_To_Post --> Approved: T10b (EP-PST-03 ERP Refusal / Hold)

        Screened --> Closed: T14 (EP-ELG-02 Proforma / Quote)
        
        Received --> Rejected: T11 (EP-EXC-09)
        Captured --> Rejected: T11 (EP-CAP-04 Exact duplicate)
        Screened --> Rejected: T11 (EP-ELG-01 Paid duplicate)
        Eligible --> Rejected: T11 (EP-EXC-09)
        Matched --> Rejected: T11 (EP-EXC-09)

        Approved --> Matched: T12 (EP-APR-07 Material change)
        Approved --> Eligible: T12b (EP-APR-07 Material line change)

        Received --> Superseded: T13 (EP-ELG-03 Correction re-issue)
        Captured --> Superseded: T13
        Screened --> Superseded: T13
        Eligible --> Superseded: T13
    }
```

---

## 2. Invoice Named Transitions Reference Table

| Transition ID | From State | To State | Triggering Entry Point | Mandatory Guards & Business Rules |
| :--- | :--- | :--- | :--- | :--- |
| **T01** | `(new)` | `Received` | `EP-INT-08` | Inbound document classified as commercial invoice, legal entity identified, not duplicate. |
| **T02** | `Received` | `Captured` | `EP-CAP-02` | Mandatory fields present; line items net amount sum equals header net amount; currency valid. |
| **T03** | `Captured` | `Screened` | `EP-SCR-01` | Vendor active; bank IBAN matches master file; VAT number passes VIES check; no trust review hold. |
| **T04** | `Screened` | `Eligible` | `EP-ELG-01` | No active duplicate invoice; vendor not in insolvency/stop-pay; active contract/PO found if required. |
| **T05** | `Eligible` | `Matched` | `EP-MAT-01`, `EP-MAT-03` | Price and quantity variances within configured tolerance thresholds, or explicitly accepted by Controller/Buyer. |
| **T06** | `Matched` | `Awaiting approval` | `EP-APR-03` | All lines assigned GL account and cost centre/project; approval tasks generated for cost centre owners. |
| **T07** | `Awaiting approval`| `Approved` | `EP-APR-04` | All required approvers (Tier 1 Budget Owner, Tier 2 Finance Director if > threshold) have approved. |
| **T08** | `Awaiting approval`| `Matched` | `EP-APR-04` | Approver rejects or requests re-evaluation; resets approval tasks for resubmission. |
| **T09** | `Approved` | `Ready to post` | `EP-PST-01` | Accounting period open; no active dispute or legal hold placed post-approval. |
| **T10** | `Ready to post` | `Posted` | `EP-PST-02` | ERP integration acknowledges successful posting with ERP voucher ID. |
| **T10b** | `Ready to post` | `Approved` | `EP-PST-03`, `EP-EXC-01`| ERP posting refused or active hold placed while waiting to post; halts release. |
| **T11** | `any non-terminal` | `Rejected` | `EP-CAP-04`, `EP-ELG-01`, `EP-EXC-09` | Exact duplicate, previously paid invoice, or formal commercial rejection by Controller. |
| **T12** | `Approved` / `Ready to post` | `Matched` | `EP-APR-07` | Material change in cost centre / coding occurs; voids approvals and requires re-approval. |
| **T12b**| `Approved` / `Ready to post` | `Eligible` | `EP-APR-07` | Material change in quantities, prices, or line amounts occurs; voids match and approvals. |
| **T13** | `any non-terminal` | `Superseded` | `EP-ELG-03` | Vendor issues corrected replacement invoice; old invoice marked superseded and linked both ways. |
| **T14** | `Screened` | `Closed` | `EP-ELG-02` | Proforma invoice decided as advance payment or quote only; closed without normal AP posting. |

---

## 3. Orthogonal Hold Governance Model

**Core Rule**: No forward transition may execute while any hold is active on an invoice or any of its lines.

```mermaid
graph LR
    subgraph Invoice State
        Screened --> Eligible --> Matched --> AwaitingApproval --> Approved --> ReadyToPost --> Posted
    end

    subgraph Orthogonal Holds Engine
        Hold[ACTIVE HOLD ENFORCEMENT]
        Hold -. BLOCKS .-> Eligible
        Hold -. BLOCKS .-> Matched
        Hold -. BLOCKS .-> AwaitingApproval
        Hold -. BLOCKS .-> Approved
        Hold -. BLOCKS .-> ReadyToPost
        Hold -. BLOCKS .-> Posted
    end
```

### Active Hold Types and Blocking Logic
1. `CAPTURE_REVIEW`: Placed when OCR extraction confidence is low or totals disagree. Blocks T02.
2. `TRUST_REVIEW`: Placed when vendor bank details changed recently, personal email used, or out-of-band callback pending. Blocks T03.
3. `DUPLICATE_REVIEW`: Placed on suspected near-duplicate invoice. Blocks T04.
4. `VARIANCE_REVIEW`: Placed when price or quantity exceeds tolerance limit. Blocks T05.
5. `AWAITING_RECEIPT`: Placed when goods receipt note has not yet been logged in warehouse. Blocks T05.
6. `BUDGET_REVIEW`: Placed when cost-centre budget is exhausted. Blocks T06/T07.
7. `DISPUTE_HOLD`: Placed when commercial dispute or vendor claim is open. Blocks T09/T10.
8. `POST_APPROVAL_REVIEW`: Placed when material data changes occur prior to posting. Reverts via T10b.
9. `MANUAL_HOLD`: Placed by AP Controller for ad-hoc compliance or credit check. Blocks all forward steps.
