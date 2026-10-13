# REST API Contracts & OpenAPI Specification

## 1. Protocol Conventions & Standards
- **Base URL**: `/api/v1`
- **Content-Type**: `application/json` (or `multipart/form-data` for file intake)
- **Authentication**: Bearer JWT in `Authorization: Bearer <token>`. Actor identity and legal entities are resolved from the session context.
- **Idempotency**: All mutating `POST`/`PUT`/`PATCH` endpoints require `Idempotency-Key: <UUID>` header.
- **Optimistic Concurrency**: Resource update commands accept `If-Match: <version>` or `expected_version` in the JSON body.
- **Error Format**: RFC 7807 Problem Details compliant JSON.

```json
{
  "type": "https://ap.abc-bv.nl/errors/GUARD_FAILED",
  "title": "Guard Failed",
  "status": 422,
  "detail": "Active hold 'TRUST_REVIEW' prevents transition to 'Screened'",
  "code": "GUARD_FAILED",
  "rule": "NO_ACTIVE_HOLD_ALLOWS_FORWARD_TRANSITION",
  "invalidParams": []
}
```

## 2. API Routes Summary (All 27 Commands & 8 Queries)

| Entry Point ID | Method & Route | Actor Role | Summary |
| :--- | :--- | :--- | :--- |
| `EP-APR-02` | `PUT /ap/invoices/{id}/coding` | AP Staff / Approver | ConfirmCoding |
| `EP-APR-04` | `POST /ap/approvals/tasks/{id}:decide` | AP Staff / Approver | DecideApprovalTask |
| `EP-APR-05` | `POST /ap/approvals/tasks/{id}:reassign` | AP Staff / Approver | ReassignApprovalTask |
| `EP-APR-06` | `POST /ap/invoices/{id}:mark-urgent` | AP Staff / Approver | MarkUrgent |
| `EP-APR-08` | `POST /ap/invoices/{id}:resubmit` | AP Staff / Approver | ResubmitForApproval |
| `EP-CAP-03` | `PATCH /ap/invoices/{id}/capture` | AP Staff / Approver | EnterOrCorrectCapturedFields |
| `EP-CFG-01` | `POST /ap/config/versions` | AP Staff / Approver | PublishConfigurationVersion |
| `EP-ELG-02` | `POST /ap/invoices/{id}:decide-proforma` | AP Staff / Approver | DecideProforma |
| `EP-ELG-03` | `POST /ap/invoices/{id}:supersede` | AP Staff / Approver | SupersedeInvoice |
| `EP-EXC-01` | `POST /ap/invoices/{id}/holds` | AP Staff / Approver | PlaceHold |
| `EP-EXC-02` | `POST /ap/invoices/{id}/holds/{holdId}:release` | AP Staff / Approver | ReleaseHold |
| `EP-EXC-03` | `POST /ap/exceptions` | AP Staff / Approver | OpenException |
| `EP-EXC-04` | `POST /ap/exceptions/{id}:assign` | AP Staff / Approver | AssignException |
| `EP-EXC-05` | `POST /ap/exceptions/{id}:contact` | AP Staff / Approver | ContactParty |
| `EP-EXC-06` | `POST /ap/exceptions/{id}:resolve` | AP Staff / Approver | ResolveException |
| `EP-EXC-08` | `POST /ap/exceptions/{id}:escalate` | AP Staff / Approver | EscalateException |
| `EP-EXC-09` | `POST /ap/invoices/{id}:reject` | AP Staff / Approver | RejectInvoice |
| `EP-INT-01` | `POST /ap/documents` | AP Staff / Approver | RegisterInboundDocument |
| `EP-INT-03` | `POST /ap/documents/{id}:classify` | AP Staff / Approver | ConfirmClassification |
| `EP-INT-06` | `POST /ap/documents/{id}:return` | AP Staff / Approver | ReturnDocumentToSender |
| `EP-INT-07` | `POST /ap/documents/{id}:file` | AP Staff / Approver | FileNonInvoiceDocument |
| `EP-MAT-02` | `POST /ap/invoices/{id}/lines:pair` | AP Staff / Approver | ConfirmLinePairing |
| `EP-MAT-03` | `POST /ap/invoices/{id}/matches/{matchId}:accept` | AP Staff / Approver | AcceptVariance |
| `EP-MAT-05` | `POST /ap/invoices/{id}:rematch` | AP Staff / Approver | RematchInvoice |
| `EP-MAT-06` | `POST /ap/invoices/{id}:link-credit-note` | AP Staff / Approver | LinkCreditNote |
| `EP-QRY-01` | `GET /ap/work-queue` | AP Staff / Approver | ListWorkQueue |
| `EP-QRY-02` | `GET /ap/invoices/{id}` | AP Staff / Approver | GetInvoiceDetail |
| `EP-QRY-03` | `GET /ap/exceptions` | AP Staff / Approver | ListExceptions |
| `EP-QRY-04` | `GET /ap/approvals/mine` | AP Staff / Approver | ListMyApprovals |
| `EP-QRY-05` | `GET /ap/invoices/{id}/audit-trail` | AP Staff / Approver | GetAuditTrail |
| `EP-QRY-06` | `GET /ap/reports/aging` | AP Staff / Approver | GetApAging |
| `EP-QRY-07` | `GET /ap/invoices` | AP Staff / Approver | SearchInvoices |
| `EP-QRY-08` | `GET /ap/documents` | AP Staff / Approver | ListIntakeQueue |
| `EP-REF-07` | `POST /ap/master-data-requests` | AP Staff / Approver | RaiseMasterDataRequest |
| `EP-SCR-02` | `POST /ap/invoices/{id}/holds/{holdId}:verify` | AP Staff / Approver | RecordOutOfBandVerification |

## 3. Detailed Request / Response Contracts (Key Entry Points)

### EP-INT-01: RegisterInboundDocument
`POST /ap/documents`
```json
// Request Headers:
// Idempotency-Key: 7b2c5e12-4f32-4d1a-8c9f-3d4e5a6b7c8d
// Content-Type: multipart/form-data
// Body fields:
// channel: "email" | "portal_edi" | "paper" | "photo"
// sender: "billing@solarsupplier.de"
// subject: "Invoice INV-2026-9042"
// received_at: "2026-10-13T10:00:00Z"
// files: [ <binary PDF file> ]
// legal_entity_hint: "ABC_NL_01" (optional)

// 201 Created Response:
{
  "document_id": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
  "fingerprint": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
  "status": "received",
  "received_at": "2026-10-13T10:00:00Z",
  "version": 1
}
```

### EP-INT-03: ConfirmClassification
`POST /ap/documents/{id}:classify`
```json
// Request Headers:
// Idempotency-Key: 123e4567-e89b-12d3-a456-426614174000
// Body:
{
  "classification": "commercial_invoice", // proforma, credit_note, delivery_note, statement
  "expected_version": 1
}

// 200 OK Response:
{
  "document_id": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
  "status": "classified",
  "document_class": "commercial_invoice",
  "version": 2
}
```

### EP-MAT-03: AcceptVariance
`POST /ap/invoices/{id}/lines/{lineId}:accept-variance`
```json
// Request Headers:
// Idempotency-Key: c9b1deb4-3b7d-4bad-9bdd-2b0d7b3dcb6e
// Body:
{
  "reason": "APPROVED_PRICE_VARIANCE_UNDER_THRESHOLD",
  "notes": "Approved by Buyer as per updated freight quotation",
  "expected_version": 4
}

// 200 OK Response:
{
  "invoice_id": "4a1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6a",
  "line_id": "5b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6b",
  "match_status": "variance_accepted",
  "invoice_status": "Matched",
  "version": 5
}
```

### EP-APR-04: DecideApprovalTask
`POST /ap/invoices/{id}/approval-tasks/{taskId}:decide`
```json
// Request Headers:
// Idempotency-Key: a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d
// Body:
{
  "decision": "approve", // "approve" | "reject"
  "reason": "Verified against Q4 Project Sunflow Capex budget",
  "expected_version": 6
}

// 200 OK Response:
{
  "task_id": "8c1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6f",
  "decision": "approved",
  "invoice_status": "Approved", // or "Matched" if rejected
  "decided_at": "2026-10-13T12:00:00Z",
  "version": 7
}
```

### EP-CFG-01: PublishConfigurationVersion
`POST /ap/config/versions`
```json
// Request Headers:
// Idempotency-Key: f1e2d3c4-b5a6-9788-1234-56789abcdef0
// Body:
{
  "config_area": "tolerances", // confidence_thresholds, hold_permissions, exception_timers, rounding_tolerance, trust_windows
  "values": {
    "price_tolerance_percentage": 2.0,
    "price_tolerance_absolute_eur": 50.00,
    "quantity_tolerance_percentage": 0.0
  },
  "effective_from": "2026-11-01T00:00:00Z",
  "approver_ref": "fd_user_02" // Segregation of duties: must differ from author
}

// 201 Created Response:
{
  "version_id": "3b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6c",
  "config_area": "tolerances",
  "status": "published",
  "effective_from": "2026-11-01T00:00:00Z",
  "version": 1
}
```
