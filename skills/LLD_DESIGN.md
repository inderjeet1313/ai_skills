# Generate Low-Level Design (LLD) Document

You are an expert software architect generating a Low-Level Design document for the Digital Dirham CBDC platform. Follow these instructions precisely.

## Input Required

The user will provide:
1. **LLD Name** — e.g., "Mobile App & Onboarding and User Management"
2. **FSD Reference** — the FSD document to base the LLD on. Each LLD has ONE dedicated FSD:
   - Mobile App & Onboarding and User Management → `docs/design/reference/FSD/FSD_Mobile_App_V0.2a.md`
   - Business Portal → `docs/design/reference/FSD/FSD_Business_Portal_v0.2.md`
   - Top-up & Withdrawal → `docs/design/reference/FSD/FSD_Top_Up_and_Withdrawal_v0.2.md`
   - P2P → `docs/design/reference/FSD/FSD_P2P_RTP_v0.2.md`
   - P2M → `docs/design/reference/FSD/FSD_P2M_v0.2.md`
   - User Onboarding (standalone) → `docs/design/reference/FSD/User Onboarding fsd.md` (markdown) or `docs/design/reference/FSD/20260109 FSD Onboarding V0.1.docx` (original docx)
   Do NOT mix FSDs across LLDs.
3. **API Spec Reference** — if applicable (e.g., `api-spec/lfi-api-spec.json`)
4. **Scope** — which FSD sections to cover

## Step 1: Read All Source Documents

Before writing anything:
1. Read the FSD document completely — every section, every user story, every acceptance criterion
2. Read the API spec if provided
3. Read the relevant backend code to verify what's actually implemented:
   - Entity classes: `backend/common/src/main/java/com/takamul/backend/model/*.java`
   - Enum classes: `backend/common/src/main/java/com/takamul/backend/enums/*.java`
   - Controllers: `backend/mobile-api/src/main/java/com/takamul/backend/controller/` (for mobile), `backend/web-api/` (for portal), `backend/lfi-api/` (for LFI callbacks)
   - Services: `backend/wallet-service/src/main/java/com/takamul/backend/service/`
   - DTOs: `backend/common/src/main/java/com/takamul/backend/dto/`
4. Read `backend/CLAUDE.md` for architecture context
5. Read the existing LLD at `docs/design/mobile-onboarding-lld-complete.md` as a reference for the level of detail, structure, and patterns expected
6. **MANDATORY — Read these cross-cutting reference documents for Security, Performance, and Integration sections:**
   - **Security Design Document:** `docs/design/reference/SDD/Digital_Dirham_SDD_v0.2 3.docx` — THE authoritative source for Section 4 (Security) details: auth architecture, encryption standards, key management, compliance requirements, threat model, trust boundaries, PQC standards
   - **Infrastructure Design Document:** `docs/design/reference/Infra/v1.6 - rCBDC Infrastructure Design Document Draft-Mar2026.docx` — THE authoritative source for Section 3 (Performance/Scalability) and Section 5 (Integration) details: deployment topology, network architecture, infrastructure components, availability design, 3-DC architecture
   These are .docx files. If your environment cannot read docx, ask the user to convert them to .md or .pdf first.

   **⚠️ CRITICAL: All security, infrastructure, performance, and integration information in LLD Sections 3, 4, and 5 MUST come from these two authoritative documents — NOT from previously generated LLDs.** Previously generated LLDs may contain inaccurate, outdated, or hallucinated information in these sections. Always go back to the source .docx files. Do NOT copy security/infra content from one LLD to another. Each LLD must independently derive its Sections 3-5 content from the SDD and Infrastructure Design docs.

## Step 2: FSD Coverage Analysis

Create a mapping of EVERY FSD section/subsection to determine:
- Does it need backend design? (API, data model, business logic)
- Does it need frontend/mobile design? (screens, navigation, state management, local storage)
- Does it need both?
- Is it purely UI/UX with no technical design needed? (still mention it with a one-liner)
- Is it NOT YET IMPLEMENTED in the backend? (flag as Open Question)

**CRITICAL RULES from team discussions (Water/Azim K./Hale A.):**

1. **Every FSD section MUST have a corresponding entry in the LLD.** Even if it's simple (like a landing page or basic CRUD), include at least a brief description. The customer (CBUAE) checks that every FSD feature is represented. If it's simple, the LLD entry can be simple — but it must exist. For example, if a feature is essentially CRUD (create, read, update, delete), a one-liner like "User Profile management — standard CRUD operations via `UserController`" is acceptable.

2. **Each LLD MUST cover both frontend AND backend.** This was clarified in the team call: "the LLD should cover the mobile application / portal front-end side of the design PLUS the backend." Even if the frontend and backend are in separate codebases (e.g., Flutter mobile app vs. Spring Boot backend, or React portal vs. Spring Boot web-api), the LLD for that module must document both perspectives. Use sequence diagrams where both UI and API participants are clearly labeled.

3. **FSD-to-code role name mapping is mandatory.** The FSD uses different role names than the code (e.g., `cb-admin` in FSD → `GLOBAL_ADMIN` in code, `lfi-editor` → `LFI_OPERATOR`). Every LLD with RBAC must include an explicit mapping table showing FSD role names ↔ code role names ↔ scope description.

4. **The FSD Coverage Matrix must use the expanded 6-column format:**
   `| FSD Section | Title | LLD Section | Backend Implementation | Frontend Coverage | Gaps |`
   Each row must list the actual controller/service class (backend), the actual page/component (frontend), and reference any Open Questions for gaps. "Not implemented" is acceptable in the Gaps column but must be flagged.

5. **Open Questions must use the structured format:**
   `| # | Category | Issue | Severity | Status |`
   Categories: "FSD Gap", "Not Implemented", "Business Rule", "Spec Mismatch". Severities: "Critical", "High", "Medium", "Low".

## Step 3: Generate the LLD

### Document Structure (v0.2 Approved Format)

This structure is derived from the 5 customer-approved v0.2 LLDs: Top-Up & Withdrawal, Mobile App Onboarding, Transfers, Business Portal, and LFI Position Rebalancing. Follow this EXACT structure.

**⚠️ The document MUST have exactly 7 top-level sections.** All approved LLDs follow this structure:
1. Overview, 2. Design, 3. Performance/Availability/Scalability, 4. Security, 5. Integration, **6. Metrics**, 7. Reference

```markdown
Low-Level Design
{MODULE NAME IN CAPS}

Prepared by: {Author — e.g., Takamul Engineering}
Date: {today's date — e.g., 24 May 2026}
Version: 0.1
Status: Draft
Module: {module name}

## Document Control

### Revision History

| Date | Version | Change Description | Author |
|------|---------|-------------------|--------|
| {date} | 0.1 | Initial draft — {brief scope summary} | {Author} |

### Approvals

| Date | Version | Approved By | Designation |
|------|---------|-------------|-------------|
| | | | |

---

# 1. Overview

{Background paragraph covering: who needs this, why it's needed, what problem it solves, business context. NOT just a technical summary — think from the customer/stakeholder perspective.}

## 1.1 Requirements

Group requirements by FSD section. Use one of these table formats depending on the module:

**For modules with FSD + BRD references** (Top-Up, Transfers, P2M):

| FSD Section | BRD Ref | Title | Key Requirements |
|-------------|---------|-------|------------------|
| Section 2.1.2.1 | F.TW.TU.001, F.TW.TU.002 | Retail User Top-Up | Top-up form with amount and source of funds; review page; validation |

**For modules with FSD only** (Business Portal):

| FSD Section | Title | Key Requirements |
|-------------|-------|------------------|
| Section 1 | User Access to Business Portal | IAM-based user registration by authorized admin |

**For modules with numbered requirements** (LFI Rebalancing, Wallet Limits):

| # | Requirement | Source |
|---|-------------|--------|
| R1 | Only terminal-SUCCESS transactions included in netting | Acceptance Criteria |

## 1.2 Assumptions

| # | Assumption |
|---|-----------|
| 1 | All monetary amounts are stored in minor currency units (fils). 100 fils = 1 AED. |
| 2 | {assumption} |

## 1.3 Not in Scope

| # | Item | Reason / Covered In |
|---|------|---------------------|
| 1 | Infrastructure, network security, trust boundaries | Covered in Infrastructure Design Document (IDD) |
| 2 | {item} | {reason / which LLD covers it} |

## 1.4 Terms, Acronyms, and Definitions

### 1.4.1 Acronyms

| Acronym | Full Term |
|---------|-----------|
| AED | United Arab Emirates Dirham (fiat currency) |

### 1.4.2 Domain Terms

| Term | Definition |
|------|-----------|
| Available Balance | Wallet balance minus any held amounts |

---

# 2. Design

## 2.1 Process Design

Organize by FSD section groups. For EACH flow, use this consistent pattern with H3 for the flow name and H4 for sub-sections:

### 2.1.x {Flow Name}

{One or two sentences of context if needed.}

#### Technical Flow

{Mermaid sequence diagram showing the full flow. Use `sequenceDiagram` for multi-system interactions.}

#### State Machine

{Mermaid `stateDiagram-v2` — include ONLY if the flow has a session-based lifecycle with multiple states. Omit this sub-section entirely if not applicable.}

#### Processing Notes

| Step | Description |
|------|-------------|
| Initiation | {What triggers the flow, initial validations, what is returned immediately} |
| Screening | {Sanctions screening steps if applicable} |
| Completion | {Transfer execution, state updates, notifications} |
| Error handling | {What happens on failure} |

Use Mermaid diagrams where they add clarity:
- `sequenceDiagram` for multi-system interactions
- `stateDiagram-v2` for state machines
- `flowchart TD` for decision flows

**Include an Error Recovery sub-section** (e.g., 2.1.x Error Recovery) at the end of Process Design covering self-healing, idempotency, and crash recovery patterns.

## 2.2 API Design

**IMPORTANT — API Design section is a reference pointer only.** Per team decision, the LLD does NOT duplicate any API details. The section contains ONLY a brief paragraph pointing the reader to the LFI API Specification.

Use this exact format:

> For complete API details including endpoints, request/response schemas, authentication, error codes, and payload examples relevant to this module, refer to the **LFI API Specification** (`lfi-api-spec.json`).

**Do NOT include:** endpoint summary tables, "Relevant API Endpoints" subsections, detailed parameter tables, request/response field lists, JSON blocks, error code tables, or token claim structures. No subsections under 2.2 — it is a single paragraph.

## 2.3 Data Model

**⚠️ CRITICAL — NO SQL CODE BLOCKS IN THE LLD.** Do NOT include `CREATE TABLE`, `ALTER TABLE`, `CREATE INDEX`, `INSERT INTO`, or any other DDL/DML SQL syntax. SQL lives in Flyway migrations in the codebase, not in the LLD.

### 2.3.1 Entity Relationship Diagram

Mermaid erDiagram with ALL entities relevant to this scope. Include:
- ALL fields (not just key fields)
- ALL relationships
- Field types and constraints
- The `version` field from BaseEntity (createdAt, updatedAt, version) on EVERY entity

### 2.3.2 Table Definitions

For each entity, create a sub-section with a 5-column table:

#### 2.3.2.x {table_name}

| Column | Type | Nullable | Default | Description |
|--------|------|----------|---------|-------------|
| id | UUID | No | gen_random_uuid() | Primary key |
| {column} | {type} | {Yes/No} | {default} | {description} |

For entities with JSONB columns, add a separate JSONB structure table immediately after:

| Field | Type | Description |
|-------|------|-------------|
| {field} | {type} | {description} |

**Alternative 3-column format** (for simpler modules like Business Portal):

| Column | Type | Notes |
|--------|------|-------|
| id | uuid | Primary key |

### 2.3.3 Key Data Constraints

| Entity | Constraint | Details |
|--------|-----------|---------|
| wallets | Unique constraint | wallet_id (14-character pattern) |

Include compound unique constraints and partial indexes described in prose.

### 2.3.4 Enums

For each enum, use an H4 heading and a 2-column table:

#### {EnumName}

| Value | Description |
|-------|-------------|
| {VALUE_1} | {description} |

**CRITICAL — List ALL enum values. Verify against actual .java enum files.**

### 2.3.5 Kafka Topics & Events

**Include only if the module uses Kafka.** Use this format:

| Event | Topic | Producer | Consumer | Payload |
|-------|-------|----------|----------|---------|
| P2PRequestedEvent | wallet.tx.p2p.cmd | WalletService | P2PProcessor | P2PPayload |

For each payload type, add a field table:

#### {PayloadName} — key fields

| Field | Type | Description |
|-------|------|-------------|
| payerWalletId | String | Payer wallet ID |

### 2.3.6 Cache Usage

Document each cache entry. Use the card-style format per cache key:

| Key | {key pattern, e.g., lfi-api-key-config::{lfiId}} |
|-----|------|
| TTL | {TTL with property key reference} |
| Type | {data type} |
| Source of truth | {where authoritative data lives} |
| Written by | {service/class} |
| Read by | {service/class} |

**Alternative tabular format** (for simpler modules):

| Keyspace | Purpose | Type / TTL | Source of truth |
|----------|---------|-----------|-----------------|

---

# 3. Performance, Availability, and Scalability

## 3.1 Throughput

{Event-driven architecture, partitioning strategy, throughput design.}

## 3.2 Latency

{Latency budgets, hot-path analysis.}

## 3.3 Concurrency & Consistency

{Concurrency control patterns, idempotency mechanisms.}

## 3.4 Availability

{Failure modes and mitigations, self-healing patterns.}

## 3.5 Configuration Parameters

Group by category with H3 sub-sections. Use 4-column table:

### 3.5.x {Category Name}

| Parameter | Property Key | Default Value | Description |
|-----------|-------------|---------------|-------------|
| {parameter name} | {actual property key from code} | {default} | {description} |

**Common category groupings** (from approved docs):
- Session & Timeout Configuration
- LFI Integration Configuration
- Event Processing Configuration
- Wallet Limit Caps (if applicable)
- Scheduler Configuration (for batch modules)

---

# 4. Security

## 4.1 Application-Level Security Controls

| Control | Description |
|---------|-------------|
| PII Encryption | {Description or "Not applicable. {reason}"} |
| Idempotency | {Description} |
| Input validation | {Description} |
| Rate limiting | {Description or defer to SDD} |

Use "Not applicable — {reason}" for controls the module does not own.

## 4.2 Authentication and Authorization

| API Surface | Authentication and Authorization |
|-------------|----------------------------------|
| Mobile API | Covered in the Mobile Application document. {Module flows} are initiated only from the mobile application. |
| LFI API (inbound) | {S2S auth description} |

---

# 5. Integration

Split into outbound and inbound sub-sections:

## 5.1 {Outbound Direction Label} (Outbound — CBDC → {External Party})

| API | {Party} Endpoint | Used By |
|-----|-------------------|---------|
| {API name} | {endpoint path} | {which flows use it} |

## 5.2 CBDC-Hosted API (Inbound — {External Party} → CBDC)

| API | CBDC Endpoint | Caller | Auth |
|-----|---------------|--------|------|
| {API name} | {endpoint path} | {caller} | {auth mechanism} |

**Additional optional sub-sections** (from approved docs):
- LFI Gateway Authentication & Caching
- Push Notifications (Firebase Cloud Messaging)
- Wallet-Limit Cap Ownership

---

# 6. Metrics

**⚠️ This section is MANDATORY.** All approved LLDs include Section 6 Metrics.

## 6.1 Instrumentation Overview

{Short paragraph: the platform's tracing/metrics medium, scope statement deferring generic platform instrumentation to the Observability document.}

## 6.2 {Module} Context Fields

| Field | Description | Example |
|-------|-------------|---------|
| Transaction ID | Platform transaction identifier | 550e8400-... |

## 6.3 {Module} Audit Events

**Standard format** (3-column):

| Event | Description | Fields |
|-------|-------------|--------|
| {EVENT_NAME} | {What happened} | {field1, field2, field3} |

**Alternative format** (4-column, for portal-style modules):

| Event | Category | Description | Payload Highlights |
|-------|----------|-------------|-------------------|

## 6.4 Event Processing Metrics

**Include only if the module uses Kafka/event processing.**

| Metric Area | What is Monitored |
|------------|-------------------|
| Consumer lag | Offset lag per topic partition |

## 6.5 Module-Specific Metrics

**Include only if there are module-specific dashboard metrics.**

| Metric | Source |
|--------|--------|
| {metric name} | {dashboard name} |

## 6.6 Recommended Alerts

| Alert | Condition | Severity |
|-------|-----------|----------|
| High consumer lag | Lag exceeds threshold for more than 5 minutes | Warning |

---

# 7. Reference

The Reference section structure varies by module complexity:

**For simple modules** (Top-Up, Business Portal, LFI Rebalancing, Wallet Limits):

## 7.1 Related Documents

**Preferred format** (3-column — use for new LLDs):

| Document | Version | Relevance |
|----------|---------|-----------|
| FSD: {Module} | v0.2 | Primary functional specification for this LLD |
| SDD | v0.2 | Security Design Document |
| Infrastructure Design Document | v1.6 | Infrastructure and deployment topology |

Note: Some approved v0.2 docs use simpler formats (2-col `Document | Reference` or 1-col `Document`). For new LLDs, always use the 3-column format above.

**For complex modules** (Transfers, Mobile App) — add these sub-sections as needed:

## 7.1 FSD Coverage Matrix

| FSD Section | Title | LLD Section | Backend Implementation | Frontend Coverage | Gaps |
|-------------|-------|-------------|----------------------|-------------------|------|

## 7.2 Open Questions and Risks

| # | Category | Issue | Severity | Status |
|---|----------|-------|----------|--------|
| 1 | {category} | {issue} | {severity} | {status} |

## 7.x Module-Specific Reference Annexes (optional)

Add any module-specific reference tables as additional sub-sections (e.g., Transfers has P2P Eligibility Matrix and Purpose of Transfer Codes). These are NOT standardized — include whatever reference data the reader would need.

## 7.{last} Related Documents

{Same 3-column format as above}
```

## Quality Checklist (from Team Review — aligned with v0.2 approved format)

Before finalizing, the author MUST verify every item:

### Structure & Title Page
- [ ] Title page has exactly these lines: "Low-Level Design" / "{MODULE NAME}" / "Prepared by:" / "Date:" / "Version: 0.1" / "Status: Draft" / "Module: {name}"
- [ ] NO metadata table — title page uses separate lines in Normal paragraph style
- [ ] Document Control section has Revision History (4-col) and Approvals (4-col) tables
- [ ] Document has exactly 7 top-level sections: 1.Overview, 2.Design, 3.Performance, 4.Security, 5.Integration, 6.Metrics, 7.Reference
- [ ] Version is 0.1 (not 1.0)

### Section 1 — Overview
- [ ] Overview has background/who needs it/why context (NOT just technical summary)
- [ ] Requirements table uses the correct column format for the module type (4-col, 3-col, or numbered)
- [ ] Assumptions use numbered 2-column table (# | Assumption), NOT bullets
- [ ] Not in Scope uses 3-column table (# | Item | Reason / Covered In)
- [ ] Terms split into 1.4.1 Acronyms (Acronym | Full Term) and 1.4.2 Domain Terms (Term | Definition)
- [ ] EVERY FSD section has at least a one-liner in the LLD
- [ ] FSD-to-code role name mapping table included (if RBAC is involved)

### Section 2 — Design
- [ ] Process Design uses Technical Flow + State Machine (optional) + Processing Notes pattern
- [ ] Processing Notes use 2-column table (Step | Description)
- [ ] Error Recovery sub-section exists at end of Process Design
- [ ] API Design section contains ONLY a reference paragraph — NO endpoint tables, NO JSON blocks
- [ ] Data Model heading is "2.3 Data Model" (not "Data Model Design")
- [ ] Table Definitions use per-entity sub-sections with 5-col or 3-col tables (NOT "Schema Changes" in prose)
- [ ] Enums use per-enum H4 headings with 2-col table (Value | Description)
- [ ] ALL enum values listed (verified against actual .java enum files)
- [ ] `version` (from BaseEntity) field shown on EVERY entity in ER diagram
- [ ] ALL JSONB sub-fields listed (read the inner classes in code)
- [ ] Kafka Topics & Events sub-section included if module uses Kafka
- [ ] Cache Usage sub-section included
- [ ] **NO SQL code blocks** — no `CREATE TABLE`, `ALTER TABLE`, `CREATE INDEX`

### Section 3 — Performance
- [ ] Configuration Parameters (3.5) use 4-col table (Parameter | Property Key | Default Value | Description)
- [ ] Config parameters grouped by category sub-sections

### Section 4 — Security
- [ ] 4.1 Application-Level Security Controls uses 2-col table (Control | Description)
- [ ] 4.2 Authentication and Authorization uses 2-col table (API Surface | Authentication and Authorization)
- [ ] Uses "Not applicable — {reason}" for controls the module does not own

### Section 5 — Integration
- [ ] Split into outbound and inbound sub-sections (not a single summary table)
- [ ] Outbound table has: API | Endpoint | Used By
- [ ] Inbound table has: API | CBDC Endpoint | Caller | Auth

### Section 6 — Metrics (MANDATORY)
- [ ] Section 6 Metrics EXISTS (not skipped)
- [ ] 6.1 Instrumentation Overview paragraph exists
- [ ] 6.2 Context Fields table: Field | Description | Example
- [ ] 6.3 Audit Events table: Event | Description | Fields (or 4-col variant)
- [ ] 6.6 Recommended Alerts table: Alert | Condition | Severity

### Section 7 — Reference
- [ ] Related Documents use 3-col table (Document | Version | Relevance) — preferred format
- [ ] FSD Coverage Matrix (if included) uses 6-col format
- [ ] Open Questions (if included) use 5-col format (# | Category | Issue | Severity | Status)

### Sequence Diagrams — CRITICAL (from P2P/P2M review)
- [ ] **Screening PII retrieval:** All cross-LFI sequence diagrams MUST show the two-step process: (1) parallel FetchPii calls to both LFIs, (2) parallel screening with counterparty encrypted PII. Same-LFI diagrams show single screening call with encryptedPii=null.
- [ ] **Explicit DB calls:** Every sequence diagram MUST include explicit database operations (SELECT, INSERT, UPDATE, BEGIN/COMMIT) — not abstract "save" or "persist" calls
- [ ] **Limit checks:** Diagrams for flows that use LimitCheckService (P2P, P2M) MUST show `validateOutgoingLimits()` and `validateIncomingLimits()` calls BEFORE screening. Flows without limit checks (WalletTransfer, MicroMerchantRefund) should NOT show them.
- [ ] **Correct transfer mechanism:** Each processor uses the right transfer approach:
  - P2P, P2M, WalletTransfer → `TransferCompletionService.executeTransfer()` → `atomicTransferEnforcingMaxBalance()` (enforces payee max balance at SQL level)
  - MicroMerchantRefundProcessor → `TransactionProcessorSupport.executeDualWalletTransfer()` (direct dual-wallet ops, no max balance enforcement)
  - RefundService → `TransferCompletionService.executeTransfer()` (synchronous, not via Kafka)
- [ ] **Screening request builders:** Each flow uses the correct `SanctionsScreeningRequest` builder:
  - P2P → `forP2P()`
  - P2M → `forP2M()`
  - Refund/MicroMerchantRefund → `forM2P()` (includes `originalTransactionId`)
  - X2X → `forX2X()`
- [ ] **LFI confirmation methods:** Each flow uses the correct confirmation method:
  - P2P/P2M/X2X success → `confirmTransactionCompletedAsync` → `ConfirmTransactionRequest.transactionCompleted()`
  - P2P/P2M/X2X failure → `confirmTransactionFailedAsync` → `ConfirmTransactionRequest.transactionFailed()`
  - Refund success → `confirmRefundCompletedAsync` → `ConfirmTransactionRequest.refundCompleted(originalTxId)`
  - Refund failure → `confirmRefundFailedAsync` → `ConfirmTransactionRequest.refundFailed(originalTxId)`
  - Cancel success → `confirmTransactionCancelCompletedAsync` → `ConfirmTransactionRequest.cancelCompleted(originalTxId)`
  - Cancel failure → `confirmCancelFailedAsync` → `ConfirmTransactionRequest.cancelFailed(originalTxId)`
- [ ] **Optimistic lock retry count:** Document max retry attempts (e.g., P2P = 2 max attempts)
- [ ] **Screening timeout:** Document the 5-second hard timeout on parallel sanctions screening

### Class Diagrams
- [ ] LfiGateway interface includes ALL confirmation method signatures (including MIT-3652 additions)
- [ ] TransactionProcessorSupport includes `executeDualWalletTransfer()` method
- [ ] LimitCheckService class included with relationship to processors that use it
- [ ] Dependency arrows are correct

### Security & Infrastructure (Sections 3-5) — MUST be sourced from .docx files, NOT from old LLDs
- [ ] **All Sections 3-5 content verified as sourced from SDD and Infra Design .docx files** (NOT copied from other LLDs)

### Final Verification (MANDATORY)
- [ ] Run code verification agent to check: roles/permissions match code, controllers exist, entity fields match, security config correct, frontend pages exist
- [ ] Cross-check every claim in the LLD against actual code — do NOT document features that don't exist in code
- [ ] Verify enum values are complete and names match exactly

## Lessons Learned (Cross-Session)

### PII Retrieval Is Always Missing on First Pass
The two-step cross-LFI screening flow (FetchPii → Screen with counterparty PII) was missed in the initial P2P/P2M LLD. This is because the code is in `LfiGatewayImpl.performCrossLfiScreening()` which is abstracted behind the `LfiGateway.performSanctionsScreening()` interface. **Always read LfiGatewayImpl.java** to understand the actual screening flow, not just the interface.

### Public Key Handling
`FetchPiiRequest` has a single `publicKey` field — the counterparty's public key for encryption. The platform's own key management is via `KmsRegistryService` with `KmsTypeCode.PII_PUBLIC_KEY` (the `buildRequesterPublicKey()` method in LfiGatewayImpl is a placeholder TODO that will be replaced by `KmsRegistryService.getPiiPublicKeyConfig()`).

### Processor Transfer Mechanism Varies
Not all processors use `TransferCompletionService`. MicroMerchantRefundProcessor uses `TransactionProcessorSupport.executeDualWalletTransfer()` directly. Always verify in each processor's code which transfer mechanism is used.

### RefundService Is Synchronous
Unlike all other transfer flows (which are Kafka-driven), RefundService handles both REFUND and CANCEL synchronously. It also extracts MCC/TLN from the bound QR code record for screening requests — this is not provided by the caller.

### Sections 3-5 MUST Be Sourced From Infra & Security Design Docs — NOT From Old LLDs
The SDD (Security Design Document) and Infrastructure Design Document are the ONLY authoritative sources for Sections 3 (Performance), 4 (Security), and 5 (Integration). These are .docx files that MUST be read for every LLD. **Never copy security/infra/performance content from a previously generated LLD into a new one** — always go back to the source documents. Old LLDs may contain inaccurate or hallucinated details in these sections.

### Large LLDs Should Be Generated in Parts
LLDs over 2000 lines should be generated section by section using Edit (append) operations, not as a single Write. This prevents context window issues and allows for iterative refinement.

### Always Verify Against Latest Code Before Finalizing
After generating an LLD, run a code verification pass. Read each processor, service, and DTO referenced in the LLD and compare method signatures, dependencies, and behaviors. Code evolves (e.g., MIT-3652 added new confirmation methods) and the LLD must stay current. **This is mandatory, not optional.** Launch a verification agent that checks: roles/permissions match, controllers exist, entity fields match, security config order numbers match, frontend pages exist.

### v0.2 Consolidation: Metadata Table Removed
In v0.2 of all approved LLDs, the metadata table was removed. The title page now uses plain separate lines: "Low-Level Design" / "{MODULE NAME}" / "Prepared by:" / "Date:" / "Version:" / "Status:" / "Module:". Do NOT generate a metadata table.

### v0.2 Consolidation: Wallet Limit Caps Integrated
The standalone Wallet Limit Caps LLD from v0.1 has been folded into the relevant module LLDs in v0.2. Top-Up now includes "Wallet Limit Cap Requirements" in Section 1.1 and "Wallet Limit Caps" in Section 3.5. Transfers includes "P2P/P2M Outbound Limit Enforcement" in Section 2.1. Mobile App includes "Wallet Limit Cap Initialization" and "Effective Ceiling Resolution" in Process Design. Business Portal includes "Admin-Tier Wallet Limit Cap Adjustment" in Process Design.

### v0.2: Related Documents Format Upgraded
Approved v0.2 docs use 3-column Related Documents tables (Document | Version | Relevance) instead of the simpler 1-column format from v0.1. Use this 3-column format for new LLDs.

### FSD Role Names Differ from Code Role Names
FSD uses: `cb-admin`, `cb-user`, `lfi-admin`, `lfi-editor`, `lfi-viewer`. Code uses: `GLOBAL_ADMIN`, `GLOBAL_OPERATOR`, `LFI_ADMIN`, `LFI_OPERATOR`, `LFI_VIEWER`. Always include an explicit mapping table. Note: FSD `cb-user` maps to `GLOBAL_OPERATOR` (not a viewer role).

### Frontend Section for Portal LLDs
Business Portal and similar web portal LLDs must include a dedicated Frontend Design section covering: technology stack, page structure (list all pages with routes), authentication flow (from browser perspective), permission guards (route-level, component-level, API-level), and localization support. Verify pages exist in the actual frontend codebase.

### mBridge/JISR Wallet ID Format
FSD Section 10 defines the mBridge wallet ID format: `M-{B|E}-XXXXCCLL###`. This is stored in `Lfi.LfiData.mbridgeWalletIds` (List<String>). Document the full pattern breakdown in Key Data Constraints.

### API Design: Reference Pointer Only — No Endpoint Tables or Details
Team decision (Azim K. + team): The API Design section should contain ONLY a brief paragraph referencing the LFI API Specification (`lfi-api-spec.json`). No endpoint summary tables, no parameter tables, no JSON blocks, no error code tables. All API detail lives in one place — the API spec.

### No SQL Code Blocks in Data Model Section
Team feedback (Azim K.): SQL syntax like `CREATE TABLE`, `ALTER TABLE`, `CREATE INDEX` etc. should NOT appear in the LLD. The ER diagram already shows all fields and types; the Key Data Constraints table covers indexes and unique constraints. SQL lives in Flyway migrations in the codebase.

### Section Numbering Consistency
When adding new sections (e.g., Frontend Design as 2.2), remember to renumber ALL subsequent subsections. Also update the FSD Coverage Matrix cross-references.

## Output

Generate the complete LLD as a single markdown file. Place it at `docs/design/{kebab-case-name}-lld-complete.md` (match existing convention — e.g., `docs/design/topup-withdrawal-lld-complete.md`, `docs/design/mobile-onboarding-lld-complete.md`).

If the LLD is very large, break the generation into parts but ensure the final document is cohesive and complete.

## Reference LLDs

Use these v0.2 approved documents as gold standards for structure, detail level, and patterns:
- Top-Up & Withdrawal LLD v0.2 — online/OTC flows with wallet limit integration
- Mobile App Onboarding & User Management LLD v0.2 — comprehensive with JWT lifecycle, PII encryption, Redis data structures
- Transfers LLD v0.2 — P2P/P2M/X2X with detailed sequence diagrams, screening flows, state machines
- Business Portal LLD v0.2 — portal-specific patterns with IAM, RBAC, admin cap management
- LFI Position Rebalancing LLD v0.2 — scheduled batch processing with netting, settlement, self-healing
