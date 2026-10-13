# Technical Entry Point Register (All 60 Entry Points)

## Executive Summary

This document is the authoritative engineering contract implementing all 60 entry points specified in `AP_Entry_Point_Catalogue_v0.1.pdf`.

- **Total Entry Points**: 60
- **Commands (State modifying)**: 27
- **Event Handlers (Reactive processing)**: 25
- **Queries (Read models & screens)**: 8
- **Proposed Entries**: 16 (marked with open points or provisional flags)

## Complete Index of Entry Points

| ID | Name | Kind | Route / Trigger | State Change | Scenarios | Status |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **EP-APR-01** | ProposeCoding | `EVENT HANDLER` | ap.invoice.matched.v1... | None | 08.05, 08.07 | Specified |
| **EP-APR-02** | ConfirmCoding | `COMMAND` | `PUT /ap/invoices/{id}/coding` | None directly | 08.05, 08.07, 12.05 | Specified |
| **EP-APR-03** | SubmitForApproval | `EVENT HANDLER` | ap.invoice.coding_confirmed.v1... | Invoice T06: Matched → Awaiting app... | 08.01, 08.02, 08.05, 08.0 | Specified |
| **EP-APR-04** | DecideApprovalTask | `COMMAND` | `POST /ap/approvals/tasks/{id}:decide` | Invoice T07: Awaiting approval → Ap... | 08.01 – 08.04, 08.06 | Specified |
| **EP-APR-05** | ReassignApprovalTask | `COMMAND / EVENT HANDLER` | `POST /ap/approvals/tasks/{id}:reassign` | Approval Task: pending → reassigned... | 08.03 | Specified |
| **EP-APR-06** | MarkUrgent | `COMMAND` | `POST /ap/invoices/{id}:mark-urgent` | None | 08.06, 04.04 | Specified |
| **EP-APR-07** | VoidApprovalOnMaterialChange | `EVENT HANDLER` | ap.invoice.data_changed.v1 whe... | Invoice T12 or T12b | Material-change rule | Specified |
| **EP-APR-08** | ResubmitForApproval | `COMMAND` | `POST /ap/invoices/{id}:resubmit` | Leads to T06 | 08.04 | Specified |
| **EP-CAP-01** | ExtractInvoiceData | `EVENT HANDLER` | ap.invoice.received.v1... | None | 02.02, 02.09, 03.01, 03.0 | Specified |
| **EP-CAP-02** | ValidateCapture | `EVENT HANDLER` | ap.invoice.extracted.v1, or af... | Invoice T02: Received → Captured (w... | 03.01, 03.03, 03.07 | Specified |
| **EP-CAP-03** | EnterOrCorrectCapturedFields | `COMMAND` | `PATCH /ap/invoices/{id}/capture` | None directly (EP-CAP-02 may then a... | 02.04, 03.03, 03.04, 03.0 | Specified |
| **EP-CAP-04** | CheckInvoiceNumberDuplicate | `EVENT HANDLER` | ap.invoice.captured.v1... | Invoice T11: Captured → Rejected (e... | 02.08, 05.03 | Specified |
| **EP-CFG-01** | PublishConfigurationVersion | `COMMAND` | `POST /ap/config/versions` | Configuration version: draft → publ... | Configuration (per client | Specified |
| **EP-ELG-01** | CheckEligibility | `EVENT HANDLER` | ap.invoice.screened.v1, or EP-... | Invoice T04: Screened → Eligible (o... | 05.02 – 05.06, 05.08 | Specified |
| **EP-ELG-02** | DecideProforma | `COMMAND` | `POST /ap/invoices/{id}:decide-proforma` | Invoice T14: Screened → Closed; Adv... | 02.05, 05.01 | Specified |
| **EP-ELG-03** | SupersedeInvoice | `COMMAND` | `POST /ap/invoices/{id}:supersede` | Invoice T13: any non-terminal → Sup... | 04.06, 04.07, 04.09, 05.0 | Specified |
| **EP-EXC-01** | PlaceHold | `COMMAND` | `POST /ap/invoices/{id}/holds` | None, or T10b: Ready to post → Appr... | 05.05, 05.08, 07.02, 07.0 | Specified |
| **EP-EXC-02** | ReleaseHold | `COMMAND` | `POST /ap/invoices/{id}/holds/{holdId}:release` | None directly | 04.10, 05.02, 05.05, 05.0 | Specified |
| **EP-EXC-03** | OpenException | `COMMAND` | `POST /ap/exceptions` | Exception: (new) → open | 07.01, 07.04 | Specified |
| **EP-EXC-04** | AssignException | `COMMAND` | `POST /ap/exceptions/{id}:assign` | Exception: open → in progress | 07.01 – 07.07 | Specified |
| **EP-EXC-05** | ContactParty | `COMMAND` | `POST /ap/exceptions/{id}:contact` | Exception: in progress → waiting | 03.03, 03.07, 04.05, 04.0 | Specified |
| **EP-EXC-06** | ResolveException | `COMMAND` | `POST /ap/exceptions/{id}:resolve` | Exception: → resolved | 05.02, 07.01 – 07.07, 08. | Specified |
| **EP-EXC-07** | EscalateOverdueExceptions | `EVENT HANDLER` | Schedule (every 15 minutes, co... | None (status unchanged) | 07.04, 07.05 | Specified |
| **EP-EXC-08** | EscalateException | `COMMAND` | `POST /ap/exceptions/{id}:escalate` | None | 07.05 | Specified |
| **EP-EXC-09** | RejectInvoice | `COMMAND` | `POST /ap/invoices/{id}:reject` | Invoice T11: any non-terminal → Rej... | 02.10, 05.02, 05.04, 05.0 | Specified |
| **EP-INT-01** | RegisterInboundDocument | `COMMAND` | `POST /ap/documents` | Inbound Document: (new) → received | 02.01, 02.02, 02.03, 02.0 | Specified |
| **EP-INT-02** | ClassifyDocument | `EVENT HANDLER` | ap.document.received.v1... | Inbound Document: received → classi... | 02.05, 02.06, 02.07, 12.0 | Specified |
| **EP-INT-03** | ConfirmClassification | `COMMAND` | `POST /ap/documents/{id}:classify` | Inbound Document: received → classi... | 02.05, 02.06, 02.07 | Specified |
| **EP-INT-04** | CheckDocumentDuplicate | `EVENT HANDLER` | ap.document.classified.v1... | Inbound Document: received/classifi... | 02.08 | Specified |
| **EP-INT-05** | IdentifyLegalEntity | `EVENT HANDLER` | ap.document.classified.v1 (aft... | None (sets legal_entity_id) | 01.07, 02.10 | Specified |
| **EP-INT-06** | ReturnDocumentToSender | `COMMAND` | `POST /ap/documents/{id}:return` | Inbound Document: classified → retu... | 02.10 | Specified |
| **EP-INT-07** | FileNonInvoiceDocument | `COMMAND` | `POST /ap/documents/{id}:file` | Inbound Document: classified → file... | 02.06, 02.07 | Specified |
| **EP-INT-08** | CreateInvoiceFromDocument | `EVENT HANDLER` | ap.document.entity_identified.... | Invoice T01: (new) → Received; Inbo... | 02.01 – 02.05, 02.09, 04. | Specified |
| **EP-MAT-01** | RunMatch | `EVENT HANDLER` | ap.invoice.eligible.v1; EP-MAT... | Invoice T05: Eligible → Matched (al... | 06.01 – 06.12, 10.05 | Specified |
| **EP-MAT-02** | ConfirmLinePairing | `COMMAND` | `POST /ap/invoices/{id}/lines:pair` | None directly | 06.09 | Specified |
| **EP-MAT-03** | AcceptVariance | `COMMAND` | `POST /ap/invoices/{id}/matches/{matchId}:accept` | Invoice T05: Eligible → Matched | 06.02, 06.07, 06.11, 07.0 | Specified |
| **EP-MAT-04** | ReleaseAwaitingReceipt | `EVENT HANDLER` | GoodsReceiptRecorded (referenc... | None directly | 06.06, 06.08, 07.04 | Specified |
| **EP-MAT-05** | RematchInvoice | `COMMAND` | `POST /ap/invoices/{id}:rematch` | Possibly T05 | 06.11, 07.01 | Specified |
| **EP-MAT-06** | LinkCreditNote | `COMMAND` | `POST /ap/invoices/{id}:link-credit-note` | Credit note T05: Eligible → Matched | 07.02, 10.05 | Specified |
| **EP-PRC-01** | AdvanceInvoice | `EVENT HANDLER` | ap.invoice.hold_released.v1, a... | Whichever transition the called ent... | All v1 acts | Specified |
| **EP-PST-01** | ReleaseForPosting | `EVENT HANDLER` | ap.invoice.approved.v1, or EP-... | Invoice T09: Approved → Ready to po... | 08.01, 09.05, 11.01 | Specified |
| **EP-PST-02** | HandlePostingAcknowledged | `EVENT HANDLER` | erp.posting.accepted.v1 (from ... | Invoice T10: Ready to post → Posted | Open point: posting in v1 | Specified |
| **EP-PST-03** | HandlePostingRefused | `EVENT HANDLER` | erp.posting.refused.v1 (from t... | Invoice T10b: Ready to post → Appro... | Open point: posting in v1 | Specified |
| **EP-QRY-01** | ListWorkQueue | `QUERY` | `GET /ap/work-queue` | - | Screens | Specified |
| **EP-QRY-02** | GetInvoiceDetail | `QUERY` | `GET /ap/invoices/{id}` | - | Screens, 03.02 | Specified |
| **EP-QRY-03** | ListExceptions | `QUERY` | `GET /ap/exceptions` | - | 07.01 – 07.07 | Specified |
| **EP-QRY-04** | ListMyApprovals | `QUERY` | `GET /ap/approvals/mine` | - | 08.01 – 08.08 | Specified |
| **EP-QRY-05** | GetAuditTrail | `QUERY` | `GET /ap/invoices/{id}/audit-trail` | - | 11.08 | Specified |
| **EP-QRY-06** | GetApAging | `QUERY` | `GET /ap/reports/aging` | - | 11.07 | Specified |
| **EP-QRY-07** | SearchInvoices | `QUERY` | `GET /ap/invoices` | - | 02.08, 05.03, 10.07 | Specified |
| **EP-QRY-08** | ListIntakeQueue | `QUERY` | `GET /ap/documents` | - | 02.01 – 02.10 | Specified |
| **EP-REF-01** | OnVendorBankDetailsChanged | `EVENT HANDLER` | VendorBankDetailsChanged (mast... | Possibly T10b | 01.02, 04.02 | Specified |
| **EP-REF-02** | OnVendorBlockStatusChanged | `EVENT HANDLER` | VendorBlocked / VendorUnblocke... | Possibly T10b | 04.10 | Specified |
| **EP-REF-03** | OnVendorDeactivated | `EVENT HANDLER` | VendorDeactivated (master data... | None | 01.06 | Specified |
| **EP-REF-04** | OnPurchaseDataChanged | `EVENT HANDLER` | PoChanged, ContractChanged, Co... | Possibly T05 | 06.11, 07.01, 05.04 | Specified |
| **EP-REF-05** | OnGoodsReturnedOrShipmentCancelled | `EVENT HANDLER` | GoodsReturned, ShipmentCancell... | Possibly T10b | 05.06 | Specified |
| **EP-REF-06** | OnVendorDisputeOrClaimChanged | `EVENT HANDLER` | VendorDisputeOpened/Closed, Qu... | Possibly T10b | 05.05, 07.07, 09.05 | Specified |
| **EP-REF-07** | RaiseMasterDataRequest | `COMMAND` | `POST /ap/master-data-requests` | None | 01.03, 01.06, 06.11 | Specified |
| **EP-SCR-01** | ScreenInvoice | `EVENT HANDLER` | ap.invoice.captured.v1 (after ... | Invoice T03: Captured → Screened (n... | 03.08, 04.01 – 04.09, 01. | Specified |
| **EP-SCR-02** | RecordOutOfBandVerification | `COMMAND` | `POST /ap/invoices/{id}/holds/{holdId}:verify` | None directly (EP-PRC-01 then advan... | 01.02, 04.01, 04.02, 04.0 | Specified |

## Detailed Entry Point Specifications

## Group 1: Inbound Document Intake & Routing (EP-INT-01 to EP-INT-08)

### EP-INT-01: RegisterInboundDocument

- **Kind**: `COMMAND`
- **Route**: `POST /ap/documents`
- **Trace (Scenarios)**: 02.01, 02.02, 02.03, 02.04
- **Trigger**: Mail connector, portal or EDI feed, or AP Clerk upload (paper scan, phone photo)

#### Inputs & Parameters
```yaml
  channel (email · portal_edi · paper · photo)
  received_at (paper: date opened)
  sender, subject
  files[] (content, media type)
  structured_payload (EDI / portal, optional)
  legal_entity_hint (optional)
  physical_original_retained (paper)
  contracts/schemas/register_inbound_document.json
```

#### Preconditions & Guards
- File types allowed (PDF, image, XML/UBL)
- At least one file or a structured payload

#### Processing Logic
- CODE Store each file as an Attachment and compute its fingerprint (hash)
- CODE Create Inbound Document in status received

#### State Changes & Named Transitions
- **Transition**: `Inbound Document: (new) → received`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.document.received.v1`
- **Idempotency Rule**: `Channel message ID (email Message-ID, EDI interchange ID); for uploads, file fingerprint +
sender + day`

#### Error Conditions
- `UNSUPPORTED_FILE → refused, nothing stored`
- `EMPTY_SUBMISSION → refused`

#### Acceptance Tests (Given / When / Then)
- **Test**: Email with one PDF creates one received document and one attachment
- **Test**: Same email delivered twice creates one document

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-INT-02: ClassifyDocument

- **Kind**: `EVENT HANDLER *(PROPOSED)*`
- **Trace (Scenarios)**: 02.05, 02.06, 02.07, 12.07
- **Trigger**: ap.document.received.v1

#### Inputs & Parameters
```yaml
  document_id
```

#### Preconditions & Guards
- Document in status received

#### Processing Logic
- CODE Apply rules first: EDI document type, keywords such as 'proforma', 'credit
- note', 'statement', 'delivery note'
- AI DECIDES If rules are inconclusive, AI classifier proposes a class; accepted at or above
- the confidence threshold
- PERSON Below threshold: classification review task for the AP Clerk (EP-INT-03)

#### State Changes & Named Transitions
- **Transition**: `Inbound Document: received → classified`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.document.classified.v1`
- **Idempotency Rule**: `Event ID (inbox)`

#### Error Conditions
- `CLASSIFIER_UNAVAILABLE → retry with backoff, then review task`

#### Acceptance Tests (Given / When / Then)
- **Test**: A document labelled 'Proforma' is classified proforma
- **Test**: A packing slip with no amount is classified delivery_note
- **Test**: Classifier confidence below threshold opens a review task and leaves status received

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-INT-03: ConfirmClassification

- **Kind**: `COMMAND`
- **Route**: `POST /ap/documents/{id}:classify`
- **Trace (Scenarios)**: 02.05, 02.06, 02.07
- **Trigger**: AP Clerk, from the intake queue

#### Inputs & Parameters
```yaml
  document_id
  classification
  expected_version
  contracts/schemas/confirm_classification.json
```

#### Preconditions & Guards
- Document in status received or classified (not yet converted)
- Actor has role AP Clerk

#### Processing Logic
- PERSON Clerk sets or corrects the class
- CODE Record the override and the original proposal in history

#### State Changes & Named Transitions
- **Transition**: `Inbound Document: received → classified (or reclassified)`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.document.classified.v1`
- **Idempotency Rule**: `idempotency_key from the client`

#### Error Conditions
- `ALREADY_CONVERTED → refused; correct on the invoice instead`

#### Acceptance Tests (Given / When / Then)
- **Test**: Clerk reclassifies a 'delivery note' as invoice and the invoice is created

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-INT-04: CheckDocumentDuplicate

- **Kind**: `EVENT HANDLER`
- **Trace (Scenarios)**: 02.08
- **Trigger**: ap.document.classified.v1

#### Inputs & Parameters
```yaml
  document_id
```

#### Preconditions & Guards
- Document in status classified

#### Processing Logic
- CODE Compare file fingerprint with live documents
- CODE For structured payloads, compare vendor + normalised invoice number +
- amount with live invoices
- CODE On a match, mark duplicate and link to the original

#### State Changes & Named Transitions
- **Transition**: `Inbound Document: received/classified → duplicate (only on a match)`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.document.duplicate_detected.v1`
- **Idempotency Rule**: `Event ID (inbox)`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Same PDF by email and as a scan of the paper copy: second becomes duplicate
- **Test**: Different invoice with the same number but another vendor is not a duplicate

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-INT-05: IdentifyLegalEntity

- **Kind**: `EVENT HANDLER *(PROPOSED)*`
- **Trace (Scenarios)**: 01.07, 02.10
- **Trigger**: ap.document.classified.v1 (after the duplicate check)

#### Inputs & Parameters
```yaml
  document_id
```

#### Preconditions & Guards
- Document classified, not duplicate

#### Processing Logic
- CODE Match entity-specific PO numbers, contract references, delivery and billing
- addresses, VAT IDs
- AI SUGGESTS If rules find no single entity, AI proposes one from the document text
- PERSON AP Clerk confirms in the intake queue when rules did not decide

#### State Changes & Named Transitions
- **Transition**: `None (sets legal_entity_id)`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.document.entity_identified.v1`
- **Idempotency Rule**: `Event ID (inbox)`

#### Error Conditions
- `NO_ENTITY_MATCH → intake review task; if the addressee is not a group company, the`
- `clerk returns it (EP-INT-06)`

#### Acceptance Tests (Given / When / Then)
- **Test**: Freight invoice quoting entity B's PO number is routed to entity B
- **Test**: Invoice addressed to a similarly named outside company opens a review task

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-INT-06: ReturnDocumentToSender

- **Kind**: `COMMAND`
- **Route**: `POST /ap/documents/{id}:return`
- **Trace (Scenarios)**: 02.10
- **Trigger**: AP Clerk

#### Inputs & Parameters
```yaml
  document_id
  return_reason
  message_to_sender
  expected_version
  contracts/schemas/return_document.json
```

#### Preconditions & Guards
- Document not yet converted
- Reason given

#### Processing Logic
- PERSON Clerk decides and writes the reason
- CODE Queue the return message through the notification service

#### State Changes & Named Transitions
- **Transition**: `Inbound Document: classified → returned`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.document.returned.v1`
- **Idempotency Rule**: `idempotency_key`

#### Error Conditions
- `ALREADY_CONVERTED → reject the invoice instead (EP-EXC-09)`

#### Acceptance Tests (Given / When / Then)
- **Test**: Returned document cannot be converted later

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-INT-07: FileNonInvoiceDocument

- **Kind**: `COMMAND`
- **Route**: `POST /ap/documents/{id}:file`
- **Trace (Scenarios)**: 02.06, 02.07
- **Trigger**: AP Clerk, or automatically after classification as delivery_note or statement

#### Inputs & Parameters
```yaml
  document_id
  filed_against_ref (shipment or vendor)
  note
  contracts/schemas/file_document.json
```

#### Preconditions & Guards
- Classification is delivery_note or statement

#### Processing Logic
- CODE Suggest the shipment (delivery note) or vendor (statement) from
- references on the document
- PERSON Clerk confirms when the suggestion is not unique
- CODE For a delivery note, record that the real invoice must later match against this
- delivery

#### State Changes & Named Transitions
- **Transition**: `Inbound Document: classified → filed`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.document.filed.v1`
- **Idempotency Rule**: `idempotency_key / event ID`

#### Error Conditions
- `SHIPMENT_NOT_FOUND → leave classified, open review task`

#### Acceptance Tests (Given / When / Then)
- **Test**: Delivery note is filed against its shipment and never becomes an invoice

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-INT-08: CreateInvoiceFromDocument

- **Kind**: `EVENT HANDLER`
- **Trace (Scenarios)**: 02.01 – 02.05, 02.09, 04.10, 12.07
- **Trigger**: ap.document.entity_identified.v1 for classes invoice, credit_note, proforma, non_payable

#### Inputs & Parameters
```yaml
  document_id
```

#### Preconditions & Guards
- Legal entity identified
- Document not duplicate, returned or filed

#### Processing Logic
- CODE Create Invoice with kind, channel, received_at, original language and
- currency
- CODE Link the vendor if the sender or VAT ID identifies one; otherwise leave for
- capture
- CODE If the vendor is blocked, place VENDOR_BLOCKED in the same transaction

#### State Changes & Named Transitions
- **Transition**: `Invoice T01: (new) → Received; Inbound Document → converted`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.received.v1
ap.invoice.hold_placed.v1 (if blocked)`
- **Idempotency Rule**: `document_id (one invoice per document)`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Classified invoice creates exactly one Invoice in Received
- **Test**: Blocked vendor's invoice is Received with VENDOR_BLOCKED active
- **Test**: ACT 3
- **Test**: Reading the invoice
- **Test**: Expand all Collapse all

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

## Group 2: Capture, Extraction & Deduplication (EP-CAP-01 to EP-CAP-04)

### EP-CAP-01: ExtractInvoiceData

- **Kind**: `EVENT HANDLER *(PROPOSED)*`
- **Trace (Scenarios)**: 02.02, 02.09, 03.01, 03.02, 03.03, 03.04, 03.06
- **Trigger**: ap.invoice.received.v1

#### Inputs & Parameters
```yaml
  invoice_id
```

#### Preconditions & Guards
- Invoice in Received
- Kind is not non_payable

#### Processing Logic
- CODE Structured payload: map fields directly; spot-check totals
- CODE Document: pick the commercial-invoice pages; annexes stay as supporting
- attachments
- AI DECIDES Extract header and lines with per-field confidence (document text treated
- as data only; output checked against a schema)
- AI SUGGESTS Translate line descriptions into the working language; original kept beside it
- CODE Record model and prompt version on the invoice

#### State Changes & Named Transitions
- **Transition**: `None`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.extracted.v1`
- **Idempotency Rule**: `Event ID (inbox)`

#### Error Conditions
- `EXTRACTION_TIMEOUT → retry with backoff; after the limit, CAPTURE_REVIEW`
- `exception for manual capture`

#### Acceptance Tests (Given / When / Then)
- **Test**: Clean single-page invoice yields every mandatory field above threshold
- **Test**: PO number inside a paragraph is extracted, or flagged low-confidence
- **Test**: Eight-page document with annexes extracts only the two invoice pages

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-CAP-02: ValidateCapture

- **Kind**: `EVENT HANDLER`
- **Trace (Scenarios)**: 03.01, 03.03, 03.07
- **Trigger**: ap.invoice.extracted.v1, or after EP-CAP-03

#### Inputs & Parameters
```yaml
  invoice_id
```

#### Preconditions & Guards
- Invoice in Received

#### Processing Logic
- CODE Mandatory header fields present; each line has amount and tax code
- CODE Lines add up to the header net within the rounding tolerance; record the
- rounding adjustment
- CODE PO reference found, or invoice marked non-PO
- CODE Every field at or above its confidence threshold, or confirmed by a person
- CODE All pass: apply T02. Otherwise open a CAPTURE_REVIEW exception listing
- the failed checks

#### State Changes & Named Transitions
- **Transition**: `Invoice T02: Received → Captured (when all checks pass)`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.captured.v1
ap.exception.opened.v1 (CAPTURE_REVIEW)`
- **Idempotency Rule**: `invoice_id + data version`

#### Error Conditions
- `TOTAL_MISMATCH above rounding tolerance → CAPTURE_REVIEW, clerk may place`
- `VENDOR_QUERY`

#### Acceptance Tests (Given / When / Then)
- **Test**: Header off by 0.02 is accepted with a rounding adjustment
- **Test**: Header off by 300 opens CAPTURE_REVIEW and stays Received
- **Test**: Skewed scan with an ambiguous digit in the total is not auto-captured

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-CAP-03: EnterOrCorrectCapturedFields

- **Kind**: `COMMAND`
- **Route**: `PATCH /ap/invoices/{id}/capture`
- **Trace (Scenarios)**: 02.04, 03.03, 03.04, 03.05, 03.06, 03.07
- **Trigger**: AP Clerk, from the capture review screen

#### Inputs & Parameters
```yaml
  invoice_id
  header fields (changed only)
  lines[] (added, changed, removed)
  expected_version
  contracts/schemas/correct_capture.json
```

#### Preconditions & Guards
- Invoice in Received or Captured (later states use the material-change rule)
- Actor has role AP Clerk

#### Processing Logic
- PERSON Clerk enters fields by hand (phone photo, handwritten) or corrects
- extracted ones
- CODE Record each field change with old and new value in history
- CODE Mark corrected fields as person-confirmed, then rerun EP-CAP-02

#### State Changes & Named Transitions
- **Transition**: `None directly (EP-CAP-02 may then apply T02)`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.data_changed.v1`
- **Idempotency Rule**: `idempotency_key`

#### Error Conditions
- `VERSION_CONFLICT → reload and retry`

#### Acceptance Tests (Given / When / Then)
- **Test**: Photo invoice captured by hand reaches Captured
- **Test**: A margin note 'OK to pay' does not create an approval

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-CAP-04: CheckInvoiceNumberDuplicate

- **Kind**: `EVENT HANDLER *(PROPOSED)*`
- **Trace (Scenarios)**: 02.08, 05.03
- **Trigger**: ap.invoice.captured.v1

#### Inputs & Parameters
```yaml
  invoice_id
```

#### Preconditions & Guards
- Invoice in Captured
- Vendor and invoice number known

#### Processing Logic
- CODE Normalise the vendor invoice number (case, spaces, leading zeros,
- separators)
- CODE Exact match with a live invoice (same vendor, legal entity, normalised
- number, amount): apply T11 with reason DUPLICATE and link both
- CODE Near match (same number, different amount, or one character apart): open
- DUPLICATE_REVIEW exception and place VENDOR_QUERY

#### State Changes & Named Transitions
- **Transition**: `Invoice T11: Captured → Rejected (exact duplicate only)`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.rejected.v1 or ap.exception.opened.v1`
- **Idempotency Rule**: `Event ID (inbox)`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Invoice already Posted with the same number and amount is rejected as duplicate
- **Test**: Same number with a different amount opens a review, not a rejection
- **Test**: NOTE Adds exception type DUPLICATE_REVIEW to the state model.
- **Test**: ACT 4
- **Test**: Trust and compliance
- **Test**: Expand all Collapse all

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

## Group 3: Screening, Compliance & Trust (EP-SCR-01 to EP-SCR-02)

### EP-SCR-01: ScreenInvoice

- **Kind**: `EVENT HANDLER`
- **Trace (Scenarios)**: 03.08, 04.01 – 04.09, 01.06, 01.08
- **Trigger**: ap.invoice.captured.v1 (after EP-CAP-04), or EP-PRC-01 after a screening hold is released

#### Inputs & Parameters
```yaml
  invoice_id
```

#### Preconditions & Guards
- Invoice in Captured
- No active hold

#### Processing Logic
- CODE Vendor active and not blocked; inactive vendor opens
- MASTER_DATA_REQUEST
- CODE IBAN on invoice equals vendor master; otherwise TRUST_REVIEW
- CODE VAT ID checked in VIES (timeout and retry; if unavailable after the limit,
- COMPLIANCE hold with reason 'VIES unavailable')
- CODE VAT treatment fits vendor country and goods: domestic, EU reverse charge
- (0% with 'BTW verlegd'), import; otherwise COMPLIANCE
- CODE Dutch mandatory invoice fields present; otherwise COMPLIANCE
- CODE Sender address on a known vendor domain; otherwise TRUST_REVIEW
- CODE First invoice from this vendor, or bank details changed within the configured
- window: TRUST_REVIEW
- CODE Invoice number breaks the vendor's usual pattern: TRUST_REVIEW
- CODE Currency or language differs from the vendor's agreed ones: note on the
- invoice (no hold)
- CODE Urgency wording is recorded; it never changes any check
- CODE No holds placed: apply T03

#### State Changes & Named Transitions
- **Transition**: `Invoice T03: Captured → Screened (no hold placed)`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.screened.v1
ap.invoice.hold_placed.v1 (per failed check)`
- **Idempotency Rule**: `Event ID (inbox) + invoice data version`

#### Error Conditions
- `VIES_UNAVAILABLE → as above, never treated as valid`

#### Acceptance Tests (Given / When / Then)
- **Test**: IBAN differing from master places TRUST_REVIEW and stays Captured
- **Test**: German vendor charging 19% on a B2B invoice places COMPLIANCE
- **Test**: First invoice from a new vendor needs TRUST_REVIEW released before Screened
- **Test**: 'Please pay today' changes nothing in the outcome

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-SCR-02: RecordOutOfBandVerification

- **Kind**: `COMMAND`
- **Route**: `POST /ap/invoices/{id}/holds/{holdId}:verify`
- **Trace (Scenarios)**: 01.02, 04.01, 04.02, 04.03, 04.05, 12.03
- **Trigger**: AP Controller, after calling the vendor on a known number

#### Inputs & Parameters
```yaml
  invoice_id
  hold_id (TRUST_REVIEW)
  method (phone call to number on file …)
  contact reached
  outcome (confirmed · not confirmed)
  note
  contracts/schemas/record_verification.json
```

#### Preconditions & Guards
- Actor has role AP Controller
- Contact number used is the one on file, not one from the invoice or email (recorded)
- Segregation of duties: actor did not change this vendor's bank details

#### Processing Logic
- PERSON Controller verifies and records the outcome
- CODE Confirmed: release the TRUST_REVIEW hold. Not confirmed: keep the hold,
- open a fraud exception and notify the Finance Director

#### State Changes & Named Transitions
- **Transition**: `None directly (EP-PRC-01 then advances)`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.hold_released.v1 or ap.exception.opened.v1`
- **Idempotency Rule**: `idempotency_key`

#### Error Conditions
- `FORBIDDEN_SOD → refused`

#### Acceptance Tests (Given / When / Then)
- **Test**: Confirmed verification releases TRUST_REVIEW and the invoice moves to Screened
- **Test**: The controller who changed the bank details cannot record the verification
- **Test**: ACT 5
- **Test**: Should we pay this?
- **Test**: Expand all Collapse all

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

## Group 4: Eligibility & Prior Commitments (EP-ELG-01 to EP-ELG-03)

### EP-ELG-01: CheckEligibility

- **Kind**: `EVENT HANDLER *(PROPOSED)*`
- **Trace (Scenarios)**: 05.02 – 05.06, 05.08
- **Trigger**: ap.invoice.screened.v1, or EP-PRC-01

#### Inputs & Parameters
```yaml
  invoice_id
```

#### Preconditions & Guards
- Invoice in Screened
- No active hold
- Kind is standard or credit_note (proforma goes to EP-ELG-02)

#### Processing Logic
- CODE Same vendor + normalised number already Posted (AP) or posted in the ERP:
- apply T11 with reason DUPLICATE_PAID and notify the vendor
- CODE Contract referenced is ended or expired: open exception for the AP
- Controller proposing rejection (T11)
- CODE No PO, no contract and no shipment reference: place
- OBLIGATION_UNCONFIRMED and open BUYER_CONFIRMATION for the buyer
- named
- CODE Linked shipment cancelled or returned: open exception proposing rejection
- or a credit-note request
- CODE Open vendor-level dispute: place DISPUTE
- CODE Vendor credit-risk flag: place CREDIT_RISK for the Finance Director
- CODE Nothing found: apply T04

#### State Changes & Named Transitions
- **Transition**: `Invoice T04: Screened → Eligible (or T11 for a paid duplicate)`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.eligible.v1
ap.invoice.hold_placed.v1
ap.exception.opened.v1`
- **Idempotency Rule**: `Event ID (inbox) + data version`

#### Error Conditions
- `ERP_LOOKUP_UNAVAILABLE → retry; never pass the duplicate check without an`
- `answer`

#### Acceptance Tests (Given / When / Then)
- **Test**: Invoice number already posted is rejected as DUPLICATE_PAID
- **Test**: Invoice with no PO or contract waits for buyer confirmation
- **Test**: Vendor with an open dispute: invoice held alongside the dispute
- **Test**: NOTE Eligible is a proposed state (see the state model).

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-ELG-02: DecideProforma

- **Kind**: `COMMAND`
- **Route**: `POST /ap/invoices/{id}:decide-proforma`
- **Trace (Scenarios)**: 02.05, 05.01
- **Trigger**: AP Controller, from the work queue (proforma in Screened)

#### Inputs & Parameters
```yaml
  invoice_id
  decision (quote_only · advance_due)
  advance_amount (if due)
  arrangement_ref
  expected_version
  contracts/schemas/decide_proforma.json
```

#### Preconditions & Guards
- Invoice kind proforma, status Screened
- Advance due only if the vendor has a recorded advance arrangement
- Advance amount ≤ proforma gross

#### Processing Logic
- CODE Show the vendor's recorded arrangement, if any
- PERSON Controller decides
- CODE Advance due: create Advance Payment (requested) and start its approval
- route (same mechanism as invoices)
- CODE Apply T14 Closed with reason quote_only or advance_raised

#### State Changes & Named Transitions
- **Transition**: `Invoice T14: Screened → Closed; Advance Payment: (new) → requested`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.closed.v1
ap.advance.requested.v1`
- **Idempotency Rule**: `idempotency_key`

#### Error Conditions
- `NO_ARRANGEMENT → advance_due refused; the controller records the arrangement`
- `with the master-data owner first`

#### Acceptance Tests (Given / When / Then)
- **Test**: Proforma from a vendor without an arrangement can only close as quote
- **Test**: Advance creates one Advance Payment and closes the proforma

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-ELG-03: SupersedeInvoice

- **Kind**: `COMMAND`
- **Route**: `POST /ap/invoices/{id}:supersede`
- **Trace (Scenarios)**: 04.06, 04.07, 04.09, 05.07
- **Trigger**: AP Clerk, when a corrected or reissued invoice arrives (system proposes the link)

#### Inputs & Parameters
```yaml
  old_invoice_id
  new_invoice_id
  reason
  expected_versions
  contracts/schemas/supersede_invoice.json
```

#### Preconditions & Guards
- Same vendor and legal entity
- Old invoice non-terminal and not Posted
- New invoice not yet beyond Captured

#### Processing Logic
- CODE Propose candidates: same vendor and reference, or a credit-and-rebill
- pattern
- PERSON Clerk confirms the link
- CODE Apply T13 to the old invoice; link both ways; move active holds to the new
- invoice or cancel them; release PO reservations

#### State Changes & Named Transitions
- **Transition**: `Invoice T13: any non-terminal → Superseded (old invoice)`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.superseded.v1`
- **Idempotency Rule**: `idempotency_key`

#### Error Conditions
- `OLD_ALREADY_POSTED → refused; handle as a credit note`

#### Acceptance Tests (Given / When / Then)
- **Test**: Reissue with correct reverse-charge wording supersedes the original; only the new one
- **Test**: can reach Ready to post
- **Test**: ACT 6
- **Test**: Matching
- **Test**: Expand all Collapse all

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

## Group 5: Matching Engine & Variance (EP-MAT-01 to EP-MAT-06)

### EP-MAT-01: RunMatch

- **Kind**: `EVENT HANDLER`
- **Trace (Scenarios)**: 06.01 – 06.12, 10.05
- **Trigger**: ap.invoice.eligible.v1; EP-MAT-05 (rematch); EP-MAT-04 after an AWAITING_RECEIPT hold
is released

#### Inputs & Parameters
```yaml
  invoice_id
```

#### Preconditions & Guards
- Invoice in Eligible
- No active hold other than the one being re-evaluated

#### Processing Logic
- CODE Pick the match type per line: PO for services → two-way; PO for goods →
- three-way; contract → non-PO contract; shipment charge → non-PO
- shipment; credit note → link to original
- CODE Pair lines to PO lines on exact keys (PO line number, item code)
- AI SUGGESTS Unpaired lines: AI proposes pairings from descriptions; a person confirms
- (EP-MAT-02) before they count
- CODE Compare price and quantity per line against the tolerance rule version in
- force (default zero)
- CODE Three-way: compare with received quantities; partial receipt matches only
- what was received
- CODE Check the PO's open balance (installments, fully invoiced POs)
- CODE Contract: compare amount and timing with the schedule
- CODE Write a Match Result per line
- CODE All within tolerance: apply T05 and reserve PO balance. Otherwise place
- MATCH_VARIANCE (with VARIANCE_REVIEW to the buyer) or
- AWAITING_RECEIPT

#### State Changes & Named Transitions
- **Transition**: `Invoice T05: Eligible → Matched (all lines within tolerance or accepted)`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.matched.v1
ap.invoice.hold_placed.v1
ap.match.completed.v1`
- **Idempotency Rule**: `invoice_id + data version + reference-data snapshot`

#### Error Conditions
- `PO_NOT_FOUND → MATCH_VARIANCE (no_reference) + VENDOR_QUERY`
- `PO_EXHAUSTED → MATCH_VARIANCE + VENDOR_QUERY`
- `REFERENCE_DATA_STALE (sync older than the limit) → retry after sync`

#### Acceptance Tests (Given / When / Then)
- **Test**: Exact PO match moves to Matched
- **Test**: Price 3% over with zero tolerance places MATCH_VARIANCE
- **Test**: Invoice before receipt places AWAITING_RECEIPT (not a failure)
- **Test**: 500 invoiced, 480 received places MATCH_VARIANCE (quantity)
- **Test**: Consolidated invoice over three POs writes one result per line
- **Test**: Third installment matches against the remaining balance

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-MAT-02: ConfirmLinePairing

- **Kind**: `COMMAND *(PROPOSED)*`
- **Route**: `POST /ap/invoices/{id}/lines:pair`
- **Trace (Scenarios)**: 06.09
- **Trigger**: AP Clerk, when AI has proposed pairings

#### Inputs & Parameters
```yaml
  invoice_id
  pairings[] (line_id, po_line_ref)
  expected_version
  contracts/schemas/confirm_pairing.json
```

#### Preconditions & Guards
- Invoice in Eligible
- Each PO line belongs to this vendor and legal entity

#### Processing Logic
- PERSON Clerk confirms or changes each pairing
- CODE Rerun EP-MAT-01

#### State Changes & Named Transitions
- **Transition**: `None directly`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.data_changed.v1`
- **Idempotency Rule**: `idempotency_key`

#### Error Conditions
- `PO_LINE_FOREIGN → refused`

#### Acceptance Tests (Given / When / Then)
- **Test**: Confirmed pairing is used on the next match run

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-MAT-03: AcceptVariance

- **Kind**: `COMMAND`
- **Route**: `POST /ap/invoices/{id}/matches/{matchId}:accept`
- **Trace (Scenarios)**: 06.02, 06.07, 06.11, 07.01
- **Trigger**: Buyer or AP Controller, from the variance review

#### Inputs & Parameters
```yaml
  invoice_id
  match_id
  reason
  expected_version
  contracts/schemas/accept_variance.json
```

#### Preconditions & Guards
- Variance within the actor's acceptance limit (configuration)
- Reason given
- Actor is not the invoice's submitter

#### Processing Logic
- PERSON Buyer confirms the price change was agreed, or the Controller accepts
- CODE Record acceptance on the Match Result; release MATCH_VARIANCE; apply
- T05 if nothing else is open

#### State Changes & Named Transitions
- **Transition**: `Invoice T05: Eligible → Matched`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.matched.v1
ap.invoice.hold_released.v1`
- **Idempotency Rule**: `idempotency_key`

#### Error Conditions
- `OVER_LIMIT → refused; escalate`

#### Acceptance Tests (Given / When / Then)
- **Test**: Buyer accepting a 2% variance within limit moves the invoice to Matched
- **Test**: Acceptance above the buyer's limit is refused

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-MAT-04: ReleaseAwaitingReceipt

- **Kind**: `EVENT HANDLER`
- **Trace (Scenarios)**: 06.06, 06.08, 07.04
- **Trigger**: GoodsReceiptRecorded (reference data)

#### Inputs & Parameters
```yaml
  po_ref, po_line_ref, received_quantity
```

#### Preconditions & Guards
- Invoices with an active AWAITING_RECEIPT on that PO line

#### Processing Logic
- CODE Release the hold for each such invoice
- CODE Trigger EP-MAT-01

#### State Changes & Named Transitions
- **Transition**: `None directly`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.hold_released.v1`
- **Idempotency Rule**: `Receipt event ID`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Logging the receipt releases the hold and the invoice matches on the next run

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-MAT-05: RematchInvoice

- **Kind**: `COMMAND`
- **Route**: `POST /ap/invoices/{id}:rematch`
- **Trace (Scenarios)**: 06.11, 07.01
- **Trigger**: AP Clerk; or automatically from EP-REF-04 when a PO or contract changes

#### Inputs & Parameters
```yaml
  invoice_id
  reason
  contracts/schemas/rematch.json
```

#### Preconditions & Guards
- Invoice in Eligible with MATCH_VARIANCE, or Matched with a stale result

#### Processing Logic
- CODE Mark the current Match Result stale; release MATCH_VARIANCE; run EP-
- MAT-01

#### State Changes & Named Transitions
- **Transition**: `Possibly T05`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.match.completed.v1`
- **Idempotency Rule**: `idempotency_key / event ID`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: After the buyer corrects the PO price in the ERP, the rematch passes

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-MAT-06: LinkCreditNote

- **Kind**: `COMMAND`
- **Route**: `POST /ap/invoices/{id}:link-credit-note`
- **Trace (Scenarios)**: 07.02, 10.05
- **Trigger**: AP Clerk; the system proposes the original invoice

#### Inputs & Parameters
```yaml
  credit_note_id
  original_invoice_id or 'unallocated'
  expected_version
  contracts/schemas/link_credit_note.json
```

#### Preconditions & Guards
- Credit note in Eligible
- Same vendor and legal entity
- Credit amount ≤ the original's open amount

#### Processing Logic
- CODE Propose the original from references on the credit note
- PERSON Clerk confirms
- CODE Set corrects_invoice_id; apply T05 to the credit note; release
- CREDIT_NOTE_PENDING on the original

#### State Changes & Named Transitions
- **Transition**: `Credit note T05: Eligible → Matched`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.matched.v1
ap.invoice.hold_released.v1`
- **Idempotency Rule**: `idempotency_key`

#### Error Conditions
- `EXCEEDS_OPEN_AMOUNT → refused`

#### Acceptance Tests (Given / When / Then)
- **Test**: Linking the requested credit note releases the original's hold
- **Test**: Credit note with no original can be marked unallocated (later scope)
- **Test**: ACT 7 + CROSS-CUTTING
- **Test**: Holds, exceptions, rejection
- **Test**: Expand all Collapse all

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

## Group 6: Exceptions, Holds & Escalation (EP-EXC-01 to EP-EXC-09)

### EP-EXC-01: PlaceHold

- **Kind**: `COMMAND`
- **Route**: `POST /ap/invoices/{id}/holds`
- **Trace (Scenarios)**: 05.05, 05.08, 07.02, 07.05, 07.06, 07.07, 09.05
- **Trigger**: AP Clerk, AP Controller or Finance Director (by hold type), or internal call from other entry
points

#### Inputs & Parameters
```yaml
  invoice_id
  hold_type
  scope (invoice · line_id · allocation_id)
  reason_text
  open_exception (bool)
  contracts/schemas/place_hold.json
```

#### Preconditions & Guards
- Invoice non-terminal and not Posted
- Actor may place this hold type (configuration)

#### Processing Logic
- PERSON Person places the hold (or a system rule does)
- CODE If the invoice is Ready to post, withdraw it to Approved in the same
- transaction (T10b)
- CODE Optionally open the matching Exception

#### State Changes & Named Transitions
- **Transition**: `None, or T10b: Ready to post → Approved`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.hold_placed.v1
ap.invoice.withdrawn_from_posting.v1`
- **Idempotency Rule**: `idempotency_key`

#### Error Conditions
- `INVOICE_TERMINAL → refused`
- `FORBIDDEN → refused`

#### Acceptance Tests (Given / When / Then)
- **Test**: Dispute placed on an Approved invoice keeps it Approved and frozen
- **Test**: Hold placed on Ready to post withdraws it to Approved in one transaction

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-EXC-02: ReleaseHold

- **Kind**: `COMMAND`
- **Route**: `POST /ap/invoices/{id}/holds/{holdId}:release`
- **Trace (Scenarios)**: 04.10, 05.02, 05.05, 05.08, 07.05, 07.07, 08.08
- **Trigger**: Role allowed for the hold type (configuration)

#### Inputs & Parameters
```yaml
  invoice_id
  hold_id
  release_note
  expected_version
  contracts/schemas/release_hold.json
```

#### Preconditions & Guards
- Hold active
- Actor may release this hold type
- Linked exception resolved, or the release note says why not needed

#### Processing Logic
- PERSON Person releases with a note
- CODE Write history; EP-PRC-01 then tries the next transition

#### State Changes & Named Transitions
- **Transition**: `None directly`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.hold_released.v1`
- **Idempotency Rule**: `idempotency_key`

#### Error Conditions
- `FORBIDDEN → refused`
- `EXCEPTION_OPEN → refused unless the note is given`

#### Acceptance Tests (Given / When / Then)
- **Test**: Budget owner releasing BUDGET_REVIEW lets approval complete
- **Test**: AP Clerk cannot release a TRUST_REVIEW

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-EXC-03: OpenException

- **Kind**: `COMMAND`
- **Route**: `POST /ap/exceptions`
- **Trace (Scenarios)**: 07.01, 07.04
- **Trigger**: Any AP role; most exceptions are opened by other entry points

#### Inputs & Parameters
```yaml
  invoice_id or document_id
  type
  owner_role
  description
  due_at (default from configuration)
  contracts/schemas/open_exception.json
```

#### Preconditions & Guards
- Target exists and is non-terminal

#### Processing Logic
- PERSON Person opens it
- CODE Assign by owner role; start the service-level timer

#### State Changes & Named Transitions
- **Transition**: `Exception: (new) → open`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.exception.opened.v1`
- **Idempotency Rule**: `idempotency_key`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Manual exception appears in the owner's queue with a due date

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-EXC-04: AssignException

- **Kind**: `COMMAND`
- **Route**: `POST /ap/exceptions/{id}:assign`
- **Trace (Scenarios)**: 07.01 – 07.07
- **Trigger**: Owner role member (take) or AP Controller (assign)

#### Inputs & Parameters
```yaml
  exception_id
  assignee_ref
```

#### Preconditions & Guards
- Exception open or in progress
- Assignee has the owner role

#### Processing Logic
- PERSON Take or assign

#### State Changes & Named Transitions
- **Transition**: `Exception: open → in progress`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.exception.assigned.v1`
- **Idempotency Rule**: `idempotency_key`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Taking an exception moves it to in progress

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-EXC-05: ContactParty

- **Kind**: `COMMAND *(PROPOSED)*`
- **Route**: `POST /ap/exceptions/{id}:contact`
- **Trace (Scenarios)**: 03.03, 03.07, 04.05, 04.06, 04.07, 04.09, 05.02, 06.03, 07.01, 07.02, 07.04
- **Trigger**: Exception assignee

#### Inputs & Parameters
```yaml
  exception_id
  party (vendor · buyer · warehouse · approver · master_data_owner)
  message
  channel
  contracts/schemas/contact_party.json
```

#### Preconditions & Guards
- Exception in progress
- For vendors: contact details from the vendor master, never from the invoice

#### Processing Logic
- AI SUGGESTS AI drafts the message from the exception (for example a reissue request
- naming the missing field)
- PERSON Assignee edits and sends
- CODE Set waiting_on; the timer keeps running

#### State Changes & Named Transitions
- **Transition**: `Exception: in progress → waiting`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.exception.waiting.v1`
- **Idempotency Rule**: `idempotency_key`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Reissue request to the vendor sets waiting_on vendor
- **Test**: Vendor address is taken from master data even if the invoice shows another

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-EXC-06: ResolveException

- **Kind**: `COMMAND`
- **Route**: `POST /ap/exceptions/{id}:resolve`
- **Trace (Scenarios)**: 05.02, 07.01 – 07.07, 08.04
- **Trigger**: Exception assignee

#### Inputs & Parameters
```yaml
  exception_id
  resolution_code
  resolution_note
  release_hold (bool)
  follow_up (none · reject_invoice · rematch · resubmit)
  contracts/schemas/resolve_exception.json
```

#### Preconditions & Guards
- Exception open, in progress or waiting
- Resolution code allowed for this exception type

#### Processing Logic
- PERSON Assignee resolves
- CODE Release the linked hold if asked; run the follow-up (EP-EXC-09, EP-MAT-05,
- EP-APR-08)

#### State Changes & Named Transitions
- **Transition**: `Exception: → resolved`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.exception.resolved.v1`
- **Idempotency Rule**: `idempotency_key`

#### Error Conditions
- `CODE_NOT_ALLOWED → refused`

#### Acceptance Tests (Given / When / Then)
- **Test**: Buyer confirms an off-catalogue purchase: hold released, invoice moves to Eligible
- **Test**: Buyer cannot confirm: follow-up rejects the invoice

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-EXC-07: EscalateOverdueExceptions

- **Kind**: `EVENT HANDLER`
- **Trace (Scenarios)**: 07.04, 07.05
- **Trigger**: Schedule (every 15 minutes, configurable)

#### Inputs & Parameters
```yaml
  none
```

#### Preconditions & Guards
- Exceptions past due_at and not resolved

#### Processing Logic
- CODE Write an Escalation Record; raise escalation_level; notify the next role in the
- escalation chain (configuration)

#### State Changes & Named Transitions
- **Transition**: `None (status unchanged)`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.exception.escalated.v1`
- **Idempotency Rule**: `exception_id + level (each level once)`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Missing receipt unresolved after its due time escalates to the AP Controller once

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-EXC-08: EscalateException

- **Kind**: `COMMAND`
- **Route**: `POST /ap/exceptions/{id}:escalate`
- **Trace (Scenarios)**: 07.05
- **Trigger**: Exception assignee or AP Controller

#### Inputs & Parameters
```yaml
  exception_id
  to_role or to_user
  reason
```

#### Preconditions & Guards
- Exception not resolved

#### Processing Logic
- PERSON Person escalates
- CODE Write an Escalation Record and notify

#### State Changes & Named Transitions
- **Transition**: `None`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.exception.escalated.v1`
- **Idempotency Rule**: `idempotency_key`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Escalated dispute shows its escalation history in the audit trail

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-EXC-09: RejectInvoice

- **Kind**: `COMMAND`
- **Route**: `POST /ap/invoices/{id}:reject`
- **Trace (Scenarios)**: 02.10, 05.02, 05.04, 05.06, 07.03, 08.04
- **Trigger**: AP Clerk or AP Controller (by reason code)

#### Inputs & Parameters
```yaml
  invoice_id
  reason_code
  message_to_vendor
  expected_version
  contracts/schemas/reject_invoice.json
```

#### Preconditions & Guards
- Invoice non-terminal and not Posted
- Reason code allowed for the actor

#### Processing Logic
- PERSON Person rejects
- CODE Apply T11; cancel active holds; close open exceptions; release PO
- reservations; queue the vendor message

#### State Changes & Named Transitions
- **Transition**: `Invoice T11: any non-terminal → Rejected`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.rejected.v1`
- **Idempotency Rule**: `idempotency_key`

#### Error Conditions
- `ALREADY_POSTED → refused; use a credit note`

#### Acceptance Tests (Given / When / Then)
- **Test**: Rejected invoice cannot move again; a resubmission arrives as a new document
- **Test**: ACT 8
- **Test**: Coding and approval
- **Test**: Expand all Collapse all

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

## Group 7: Coding & Approval Workflow (EP-APR-01 to EP-APR-08)

### EP-APR-01: ProposeCoding

- **Kind**: `EVENT HANDLER *(PROPOSED)*`
- **Trace (Scenarios)**: 08.05, 08.07
- **Trigger**: ap.invoice.matched.v1

#### Inputs & Parameters
```yaml
  invoice_id
```

#### Preconditions & Guards
- Invoice in Matched

#### Processing Logic
- CODE PO lines: take GL and cost object from the PO
- CODE Non-PO: vendor default or coding rule (configuration)
- AI SUGGESTS Still uncoded: AI proposes GL and cost object from history, with confidence
- CODE Record coding_source per line

#### State Changes & Named Transitions
- **Transition**: `None`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.coding_proposed.v1`
- **Idempotency Rule**: `Event ID (inbox)`

#### Error Conditions
- `CODING_SERVICE_UNAVAILABLE → leave uncoded for the clerk`

#### Acceptance Tests (Given / When / Then)
- **Test**: PO-backed lines are coded from the PO without AI
- **Test**: AI proposal is never confirmed without a person (v1)
- **Test**: NOTE Whether high-confidence AI coding may be auto-accepted is on the decision map as a new
- **Test**: point.

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-APR-02: ConfirmCoding

- **Kind**: `COMMAND`
- **Route**: `PUT /ap/invoices/{id}/coding`
- **Trace (Scenarios)**: 08.05, 08.07, 12.05
- **Trigger**: AP Clerk

#### Inputs & Parameters
```yaml
  invoice_id
  lines[] (gl_account_ref, cost_object)
  allocations[] (optional)
  is_capex
  expected_version
  contracts/schemas/confirm_coding.json
```

#### Preconditions & Guards
- Invoice in Matched
- GL accounts and cost objects valid on the invoice date
- Allocations add up to 100% per invoice or line

#### Processing Logic
- PERSON Clerk confirms or edits coding and allocations
- CODE Validate against reference data; then EP-APR-03 runs

#### State Changes & Named Transitions
- **Transition**: `None directly`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.coding_confirmed.v1`
- **Idempotency Rule**: `idempotency_key`

#### Error Conditions
- `INVALID_COST_OBJECT → refused`
- `ALLOCATION_NOT_100 → refused`

#### Acceptance Tests (Given / When / Then)
- **Test**: IT support split 40/60 over two cost centres is accepted; 40/50 is refused

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-APR-03: SubmitForApproval

- **Kind**: `EVENT HANDLER`
- **Trace (Scenarios)**: 08.01, 08.02, 08.05, 08.07, 08.08
- **Trigger**: ap.invoice.coding_confirmed.v1, or EP-APR-08 (resubmit)

#### Inputs & Parameters
```yaml
  invoice_id
```

#### Preconditions & Guards
- Invoice in Matched
- No active hold
- Coding complete

#### Processing Logic
- CODE Resolve the route from the current approval matrix version: owner per
- allocation, capital project owner for capex, a second approver above the
- threshold
- CODE Create the Approval Request (with matrix version) and its tasks
- CODE Check budget per allocation; if exhausted, place BUDGET_REVIEW on that
- allocation and tell the budget owner
- CODE Apply T06

#### State Changes & Named Transitions
- **Transition**: `Invoice T06: Matched → Awaiting approval`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.approval_requested.v1
ap.invoice.hold_placed.v1 (BUDGET_REVIEW)`
- **Idempotency Rule**: `invoice_id + data version`

#### Error Conditions
- `NO_APPROVER_FOUND → exception to the AP Controller; stays Matched`

#### Acceptance Tests (Given / When / Then)
- **Test**: Invoice above the threshold gets two tasks: owner then Finance Director
- **Test**: Capex invoice routes to the project owner
- **Test**: Exhausted budget places BUDGET_REVIEW but still opens the tasks

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-APR-04: DecideApprovalTask

- **Kind**: `COMMAND`
- **Route**: `POST /ap/approvals/tasks/{id}:decide`
- **Trace (Scenarios)**: 08.01 – 08.04, 08.06
- **Trigger**: Approver (assignee or delegate), from the approval view

#### Inputs & Parameters
```yaml
  task_id
  decision (approve · reject)
  reason (required for reject)
  expected_version
  contracts/schemas/decide_task.json
```

#### Preconditions & Guards
- Task pending and assigned to the actor
- Segregation of duties: actor did not submit this invoice or change this vendor's bank
- details
- Earlier steps of a sequential route are approved
- BUDGET_REVIEW on this allocation is released

#### Processing Logic
- PERSON Approver decides
- CODE Approve, last task: apply T07; calculate due date from the payment-terms
- snapshot and baseline-date rule; if urgent, open POST_APPROVAL_REVIEW
- CODE Reject: cancel the remaining tasks; apply T08; place APPROVAL_RETURNED;
- open APPROVER_REJECTED for the AP Controller

#### State Changes & Named Transitions
- **Transition**: `Invoice T07: Awaiting approval → Approved, or T08: → Matched`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.approval.task_decided.v1
ap.invoice.approved.v1 or ap.invoice.approval_returned.v1`
- **Idempotency Rule**: `task_id + decision (a second identical decision is a no-op)`

#### Error Conditions
- `FORBIDDEN_SOD → refused`
- `NOT_ASSIGNEE → refused`

#### Acceptance Tests (Given / When / Then)
- **Test**: Single owner approval moves the invoice to Approved with a due date
- **Test**: Rejection sends it back to Matched with a hold and an exception
- **Test**: Approver who submitted the invoice is refused

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-APR-05: ReassignApprovalTask

- **Kind**: `COMMAND / EVENT HANDLER`
- **Route**: `POST /ap/approvals/tasks/{id}:reassign`
- **Trace (Scenarios)**: 08.03
- **Trigger**: AP Controller; or automatically when the assignee is marked absent and a delegate is on
file

#### Inputs & Parameters
```yaml
  task_id
  new_assignee_ref
  reason
```

#### Preconditions & Guards
- Task pending
- New assignee holds the required role and authority limit
- Segregation of duties holds for the new assignee

#### Processing Logic
- CODE Automatic: take the delegate from the delegation table
- PERSON Manual: the Controller picks someone
- CODE Close the task as reassigned; create a new pending task; record
- delegated_from

#### State Changes & Named Transitions
- **Transition**: `Approval Task: pending → reassigned (+ new pending task)`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.approval.task_reassigned.v1`
- **Idempotency Rule**: `idempotency_key / absence event ID`

#### Error Conditions
- `INSUFFICIENT_AUTHORITY → refused`

#### Acceptance Tests (Given / When / Then)
- **Test**: Approver on leave: task moves to the delegate automatically

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-APR-06: MarkUrgent

- **Kind**: `COMMAND`
- **Route**: `POST /ap/invoices/{id}:mark-urgent`
- **Trace (Scenarios)**: 08.06, 04.04
- **Trigger**: Finance Director

#### Inputs & Parameters
```yaml
  invoice_id
  reason
```

#### Preconditions & Guards
- Invoice not yet Approved
- Actor has role Finance Director

#### Processing Logic
- PERSON Finance Director marks it urgent
- CODE Set urgent on the Approval Request; on approval, an after-the-fact review
- exception is created

#### State Changes & Named Transitions
- **Transition**: `None`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.marked_urgent.v1`
- **Idempotency Rule**: `idempotency_key`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Urgent flag never skips screening, matching or approval
- **Test**: Approved urgent invoice gets a POST_APPROVAL_REVIEW exception

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-APR-07: VoidApprovalOnMaterialChange

- **Kind**: `EVENT HANDLER *(PROPOSED)*`
- **Trace (Scenarios)**: Material-change rule
- **Trigger**: ap.invoice.data_changed.v1 where status is Matched or later

#### Inputs & Parameters
```yaml
  invoice_id
  changed fields
```

#### Preconditions & Guards
- Change is material (amounts, quantities, lines, PO references, coding, allocations,
- vendor, legal entity)

#### Processing Logic
- CODE Amounts, lines or PO references changed: mark Match Results stale; void
- approvals; apply T12b → Eligible
- CODE Coding or allocations changed after approval started: void the request;
- apply T12 → Matched

#### State Changes & Named Transitions
- **Transition**: `Invoice T12 or T12b`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.approval_voided.v1`
- **Idempotency Rule**: `Event ID (inbox)`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Changing a line amount after approval sends the invoice back to Eligible and voids the
- **Test**: approval
- **Test**: Changing a cost centre after approval needs re-approval only

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-APR-08: ResubmitForApproval

- **Kind**: `COMMAND`
- **Route**: `POST /ap/invoices/{id}:resubmit`
- **Trace (Scenarios)**: 08.04
- **Trigger**: AP Controller, after resolving an approver's rejection

#### Inputs & Parameters
```yaml
  invoice_id
  note
```

#### Preconditions & Guards
- Invoice in Matched with APPROVAL_RETURNED active
- Exception APPROVER_REJECTED resolved

#### Processing Logic
- PERSON Controller resubmits
- CODE Release APPROVAL_RETURNED; run EP-APR-03

#### State Changes & Named Transitions
- **Transition**: `Leads to T06`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.hold_released.v1`
- **Idempotency Rule**: `idempotency_key`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Resubmitted invoice gets a new Approval Request with the current matrix version
- **Test**: ACT 8 → ERP
- **Test**: Hand-off to posting
- **Test**: Expand all Collapse all

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

## Group 8: Release for Posting & ERP Integration (EP-PST-01 to EP-PST-03)

### EP-PST-01: ReleaseForPosting

- **Kind**: `EVENT HANDLER`
- **Trace (Scenarios)**: 08.01, 09.05, 11.01
- **Trigger**: ap.invoice.approved.v1, or EP-PRC-01 after the last hold on an Approved invoice is released

#### Inputs & Parameters
```yaml
  invoice_id
```

#### Preconditions & Guards
- Invoice Approved
- No active hold on the invoice or its lines

#### Processing Logic
- CODE Set the accounting date proposal: the goods-receipt date when goods
- arrived in an earlier period than the invoice
- CODE Build the posting payload: header, lines, VAT treatment, coding, allocations,
- due date, accounting date proposal
- CODE Apply T09 and write the posting command to the outbox for the ERP adapter

#### State Changes & Named Transitions
- **Transition**: `Invoice T09: Approved → Ready to post`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.ready_to_post.v1
erp.posting.requested.v1 (command to the adapter)`
- **Idempotency Rule**: `invoice_id + approval_request_id`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Approved invoice with an active dispute stays Approved

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-PST-02: HandlePostingAcknowledged

- **Kind**: `EVENT HANDLER *(PROPOSED)*`
- **Trace (Scenarios)**: Open point: posting in v1
- **Trigger**: erp.posting.accepted.v1 (from the adapter)

#### Inputs & Parameters
```yaml
  invoice_id
  erp_document_ref
  posted_at
```

#### Preconditions & Guards
- Invoice in Ready to post

#### Processing Logic
- CODE Store erp_document_ref; apply T10

#### State Changes & Named Transitions
- **Transition**: `Invoice T10: Ready to post → Posted`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.posted.v1`
- **Idempotency Rule**: `Adapter message ID`

#### Error Conditions
- `UNKNOWN_INVOICE → dead-letter queue and alert`

#### Acceptance Tests (Given / When / Then)
- **Test**: Acknowledgement moves the invoice to Posted; a repeat is ignored
- **Test**: NOTE Depends on the open decision on posting in v1.

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-PST-03: HandlePostingRefused

- **Kind**: `EVENT HANDLER *(PROPOSED)*`
- **Trace (Scenarios)**: Open point: posting in v1
- **Trigger**: erp.posting.refused.v1 (from the adapter)

#### Inputs & Parameters
```yaml
  invoice_id
  refusal_code
  message
```

#### Preconditions & Guards
- Invoice in Ready to post

#### Processing Logic
- CODE Apply T10b; open POSTING_REFUSED exception for the AP Controller with
- the ERP's reason (for example closed period, invalid account)

#### State Changes & Named Transitions
- **Transition**: `Invoice T10b: Ready to post → Approved`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.posting_refused.v1
ap.exception.opened.v1`
- **Idempotency Rule**: `Adapter message ID`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Closed-period refusal returns the invoice to Approved with an exception
- **Test**: ACT 1
- **Test**: Reacting to reference data
- **Test**: Expand all Collapse all

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

## Group 9: Master Reference Data Events & Requests (EP-REF-01 to EP-REF-07)

### EP-REF-01: OnVendorBankDetailsChanged

- **Kind**: `EVENT HANDLER`
- **Trace (Scenarios)**: 01.02, 04.02
- **Trigger**: VendorBankDetailsChanged (master data)

#### Inputs & Parameters
```yaml
  vendor_id
  changed_at
  changed_by
```

#### Preconditions & Guards
- -

#### Processing Logic
- CODE Place TRUST_REVIEW on every non-terminal, not-Posted invoice of this
- vendor (Ready to post is withdrawn first)
- CODE Store changed_at and changed_by for screening and segregation-of-
- duties checks

#### State Changes & Named Transitions
- **Transition**: `Possibly T10b`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.hold_placed.v1 (per invoice)`
- **Idempotency Rule**: `Master-data event ID`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Bank change holds the vendor's queued invoices until verified

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-REF-02: OnVendorBlockStatusChanged

- **Kind**: `EVENT HANDLER`
- **Trace (Scenarios)**: 04.10
- **Trigger**: VendorBlocked / VendorUnblocked (master data)

#### Inputs & Parameters
```yaml
  vendor_id
  blocked (bool)
  reason
```

#### Preconditions & Guards
- -

#### Processing Logic
- CODE Blocked: place VENDOR_BLOCKED on open invoices. Unblocked: release it

#### State Changes & Named Transitions
- **Transition**: `Possibly T10b`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.hold_placed.v1 / hold_released.v1`
- **Idempotency Rule**: `Master-data event ID`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Unblocking releases only VENDOR_BLOCKED, not other holds

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-REF-03: OnVendorDeactivated

- **Kind**: `EVENT HANDLER`
- **Trace (Scenarios)**: 01.06
- **Trigger**: VendorDeactivated (master data)

#### Inputs & Parameters
```yaml
  vendor_id
```

#### Preconditions & Guards
- -

#### Processing Logic
- CODE If open invoices or POs exist, open MASTER_DATA_REQUEST so the owner
- can close or reassign them

#### State Changes & Named Transitions
- **Transition**: `None`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.exception.opened.v1`
- **Idempotency Rule**: `Master-data event ID`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Deactivating a vendor with an open invoice raises a request to its owner

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-REF-04: OnPurchaseDataChanged

- **Kind**: `EVENT HANDLER`
- **Trace (Scenarios)**: 06.11, 07.01, 05.04
- **Trigger**: PoChanged, ContractChanged, ContractEnded (reference data)

#### Inputs & Parameters
```yaml
  po_ref or contract_ref
```

#### Preconditions & Guards
- -

#### Processing Logic
- CODE Invoices with MATCH_VARIANCE on that PO or contract: run EP-MAT-05
- CODE Contract ended: open exception for invoices not yet Matched

#### State Changes & Named Transitions
- **Transition**: `Possibly T05`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.match.completed.v1`
- **Idempotency Rule**: `Reference event ID`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Buyer corrects the PO price; the held invoice rematches and passes

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-REF-05: OnGoodsReturnedOrShipmentCancelled

- **Kind**: `EVENT HANDLER`
- **Trace (Scenarios)**: 05.06
- **Trigger**: GoodsReturned, ShipmentCancelled (reference data)

#### Inputs & Parameters
```yaml
  po_ref / shipment_ref
  quantities
```

#### Preconditions & Guards
- -

#### Processing Logic
- CODE For affected invoices not yet Posted, place CREDIT_NOTE_PENDING or
- MATCH_VARIANCE and open an exception

#### State Changes & Named Transitions
- **Transition**: `Possibly T10b`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.hold_placed.v1`
- **Idempotency Rule**: `Reference event ID`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Returned goods hold the invoice until a credit note or rejection

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-REF-06: OnVendorDisputeOrClaimChanged

- **Kind**: `EVENT HANDLER *(PROPOSED)*`
- **Trace (Scenarios)**: 05.05, 07.07, 09.05
- **Trigger**: VendorDisputeOpened/Closed, QualityClaimRaised/Resolved (owning domain, still to
decide)

#### Inputs & Parameters
```yaml
  vendor_id or shipment_ref
  dispute_or_claim_id
  state
```

#### Preconditions & Guards
- -

#### Processing Logic
- CODE Opened: place DISPUTE or QUALITY_CLAIM on linked open invoices. Closed:
- release them

#### State Changes & Named Transitions
- **Transition**: `Possibly T10b`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.invoice.hold_placed.v1 / hold_released.v1`
- **Idempotency Rule**: `Source event ID`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Dispute opened after approval keeps the invoice Approved and frozen

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-REF-07: RaiseMasterDataRequest

- **Kind**: `COMMAND`
- **Route**: `POST /ap/master-data-requests`
- **Trace (Scenarios)**: 01.03, 01.06, 06.11
- **Trigger**: AP Clerk or AP Controller; also opened by other entry points

#### Inputs & Parameters
```yaml
  subject (vendor, contract, PO, receipt)
  subject_ref
  requested_change
  evidence_attachment_ids
  contracts/schemas/master_data_request.json
```

#### Preconditions & Guards
- -

#### Processing Logic
- PERSON Person describes the needed change
- CODE Open MASTER_DATA_REQUEST exception, waiting on the master-data
- owner

#### State Changes & Named Transitions
- **Transition**: `None`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.exception.opened.v1
masterdata.change_requested.v1 (to the owning domain)`
- **Idempotency Rule**: `idempotency_key`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Two vendor records with the same VAT ID raise a merge request; AP changes nothing
- **Test**: itself
- **Test**: CROSS-CUTTING
- **Test**: Process manager
- **Test**: Expand all Collapse all

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

## Group 10: Process Orchestration (EP-PRC-01)

### EP-PRC-01: AdvanceInvoice

- **Kind**: `EVENT HANDLER *(PROPOSED)*`
- **Trace (Scenarios)**: All v1 acts
- **Trigger**: ap.invoice.hold_released.v1, ap.exception.resolved.v1, ap.invoice.data_changed.v1

#### Inputs & Parameters
```yaml
  invoice_id
```

#### Preconditions & Guards
- Invoice non-terminal
- No active hold

#### Processing Logic
- CODE Look up the entry point that moves the invoice on from its current status
- (Captured → EP-SCR-01, Screened → EP-ELG-01, Eligible → EP-MAT-01,
- Matched → EP-APR-03 if coding confirmed, Approved → EP-PST-01) and call
- it

#### State Changes & Named Transitions
- **Transition**: `Whichever transition the called entry point applies`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `(those of the called entry point)`
- **Idempotency Rule**: `Event ID (inbox)`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Releasing the last hold on an Approved invoice moves it to Ready to post
- **Test**: Releasing one of two holds changes nothing
- **Test**: NOTE Keeps 'what happens next' in one place, so released holds never leave invoices stuck.
- **Test**: ACTS 8, 11
- **Test**: Queries for screens and audit
- **Test**: Expand all Collapse all

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

## Group 11: Queries & Work Queues (EP-QRY-01 to EP-QRY-08)

### EP-QRY-01: ListWorkQueue

- **Kind**: `QUERY`
- **Route**: `GET /ap/work-queue`
- **Trace (Scenarios)**: Screens
- **Trigger**: UI: work queue screen

#### Inputs & Parameters
```yaml
  role / user (from login)
  filters: legal entity, status, hold type, due
```

#### Preconditions & Guards
- Results limited to the user's legal entities and roles

#### Processing Logic
- CODE Return invoices and exceptions needing this user's action, oldest due first

#### State Changes & Named Transitions
- **Transition**: `-`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `-`
- **Idempotency Rule**: `-`

#### Error Conditions
- `-`

#### Acceptance Tests (Given / When / Then)
- **Test**: Budget owner sees only approvals and budget reviews for their cost centres

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-QRY-02: GetInvoiceDetail

- **Kind**: `QUERY`
- **Route**: `GET /ap/invoices/{id}`
- **Trace (Scenarios)**: Screens, 03.02
- **Trigger**: UI: invoice detail screen

#### Inputs & Parameters
```yaml
  invoice_id
```

#### Preconditions & Guards
- User may see this legal entity

#### Processing Logic
- CODE Return header, lines, source document pages, extracted-field confidence,
- holds, exceptions, match results, approvals

#### State Changes & Named Transitions
- **Transition**: `-`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `-`
- **Idempotency Rule**: `-`

#### Error Conditions
- `NOT_FOUND`

#### Acceptance Tests (Given / When / Then)
- **Test**: Detail shows the source document beside the extracted fields

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-QRY-03: ListExceptions

- **Kind**: `QUERY`
- **Route**: `GET /ap/exceptions`
- **Trace (Scenarios)**: 07.01 – 07.07
- **Trigger**: UI: exception queue

#### Inputs & Parameters
```yaml
  filters: type, owner role, waiting_on, overdue
```

#### Preconditions & Guards
- -

#### Processing Logic
- CODE Return exceptions with linked invoice and hold

#### State Changes & Named Transitions
- **Transition**: `-`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `-`
- **Idempotency Rule**: `-`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Overdue filter returns escalated items first

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-QRY-04: ListMyApprovals

- **Kind**: `QUERY`
- **Route**: `GET /ap/approvals/mine`
- **Trace (Scenarios)**: 08.01 – 08.08
- **Trigger**: UI: approval view

#### Inputs & Parameters
```yaml
  user (from login)
```

#### Preconditions & Guards
- -

#### Processing Logic
- CODE Return pending tasks with invoice summary, match outcome, budget
- snapshot, attachments

#### State Changes & Named Transitions
- **Transition**: `-`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `-`
- **Idempotency Rule**: `-`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Delegate sees reassigned tasks

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-QRY-05: GetAuditTrail

- **Kind**: `QUERY`
- **Route**: `GET /ap/invoices/{id}/audit-trail`
- **Trace (Scenarios)**: 11.08
- **Trigger**: UI: audit trail screen; auditor export

#### Inputs & Parameters
```yaml
  invoice_id
  format (screen · export)
```

#### Preconditions & Guards
- Role AP Controller or auditor

#### Processing Logic
- CODE Return state history, holds, approvals, match results and field changes in
- time order, with actors and reasons

#### State Changes & Named Transitions
- **Transition**: `-`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `-`
- **Idempotency Rule**: `-`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Trail months later shows who approved, against which PO and receipt, and what
- **Test**: changed

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-QRY-06: GetApAging

- **Kind**: `QUERY`
- **Route**: `GET /ap/reports/aging`
- **Trace (Scenarios)**: 11.07
- **Trigger**: UI: aging report; monthly review

#### Inputs & Parameters
```yaml
  legal entity
  as_of date
  group by (status, hold type, vendor, due bucket)
```

#### Preconditions & Guards
- -

#### Processing Logic
- CODE Aggregate open invoices by status, active holds and due-date buckets

#### State Changes & Named Transitions
- **Transition**: `-`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `-`
- **Idempotency Rule**: `-`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Invoices on DISPUTE are visible in aging, not hidden

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-QRY-07: SearchInvoices

- **Kind**: `QUERY`
- **Route**: `GET /ap/invoices`
- **Trace (Scenarios)**: 02.08, 05.03, 10.07
- **Trigger**: UI search; duplicate checks; statement comparison (later)

#### Inputs & Parameters
```yaml
  vendor, number, amount range, dates, status
```

#### Preconditions & Guards
- -

#### Processing Logic
- CODE Search across current-state rows

#### State Changes & Named Transitions
- **Transition**: `-`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `-`
- **Idempotency Rule**: `-`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Search by normalised number finds variants with spaces or leading zeros

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

### EP-QRY-08: ListIntakeQueue

- **Kind**: `QUERY`
- **Route**: `GET /ap/documents`
- **Trace (Scenarios)**: 02.01 – 02.10
- **Trigger**: UI: intake screen

#### Inputs & Parameters
```yaml
  filters: status, channel, classification
```

#### Preconditions & Guards
- -

#### Processing Logic
- CODE Return documents not yet converted, filed, duplicate or returned

#### State Changes & Named Transitions
- **Transition**: `-`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `-`
- **Idempotency Rule**: `-`

#### Error Conditions
- `Standard errors only`

#### Acceptance Tests (Given / When / Then)
- **Test**: Unclassified documents appear oldest first
- **Test**: CROSS-CUTTING
- **Test**: Configuration
- **Test**: Expand all Collapse all

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---

## Group 12: Configuration Governance & Dual Control (EP-CFG-01)

### EP-CFG-01: PublishConfigurationVersion

- **Kind**: `COMMAND *(PROPOSED)*`
- **Route**: `POST /ap/config/versions`
- **Trace (Scenarios)**: Configuration (per client)
- **Trigger**: AP Controller (configuration admin screen); a second person approves

#### Inputs & Parameters
```yaml
  config_area (tolerances · confidence thresholds · hold permissions · exception timers ·
  rounding tolerance · trust windows · acceptance limits · coding rules)
  values
  effective_from
  approver_ref
  contracts/schemas/config_version.json
```

#### Preconditions & Guards
- Values pass the configuration schema
- Approver is a different person from the author

#### Processing Logic
- PERSON Author drafts; a second person approves
- CODE Store a new immutable version; entry points read the version in force and
- record it

#### State Changes & Named Transitions
- **Transition**: `Configuration version: draft → published`

#### Events Emitted / Consumed
- **Outputs (Outbox)**: `ap.config.published.v1`
- **Idempotency Rule**: `idempotency_key`

#### Error Conditions
- `SAME_AUTHOR_APPROVER → refused`

#### Acceptance Tests (Given / When / Then)
- **Test**: Raising a tolerance applies only to match runs after effective_from

- **Implementation Status**: `SPECIFIED — Ready for Implementation`

---
