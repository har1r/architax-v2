# ARCHITAX — Product Requirements Document

**Document Classification:** Enterprise Product Requirements Document (PRD)
**Product:** ARCHITAX — Property Tax (PBB-P2) Archive & Workflow Management System
**Version:** 1.0 — Production Reference
**Status:** Final
**Audience:** Product Managers · Government Executives · Business Analysts · Enterprise Architects · Software Architects · UI/UX Designers · Frontend Developers · Backend Developers · QA Engineers · DevOps Engineers · Project Managers

---

**Document Authority**

This PRD is the business foundation of ARCHITAX. It governs the *what* and *why* of the product. Implementation decisions — architecture, schema, API design, infrastructure — are outside its scope. Three companion documents form the complete specification baseline:

| Document | Classification | Governs |
|----------|---------------|---------|
| `WORKFLOW.md` | Functional Specification | Business process, state machines, business rules, error handling, audit requirements |
| `SCREEN_SPECIFICATION.md` | UI Screen Specification | Every screen's purpose, layout, component hierarchy, interactions, states, permissions |
| `UI_DESIGN_SYSTEM.md` | Design System Reference | Visual language, component library, color, typography, spacing, motion, accessibility |

> **Critical:** This document references and summarizes those specifications. It does not restate them in full and does not supersede them. In any conflict between this PRD and those companion documents, the companion document governs its own domain.

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Business Context](#2-business-context)
3. [Problem Statement](#3-problem-statement)
4. [Stakeholders](#4-stakeholders)
5. [User Personas](#5-user-personas)
6. [Product Scope](#6-product-scope)
7. [Functional Requirements](#7-functional-requirements)
8. [Non-Functional Requirements](#8-non-functional-requirements)
9. [Business Rules Overview](#9-business-rules-overview)
10. [Workflow Overview](#10-workflow-overview)
11. [Screen Overview](#11-screen-overview)
12. [UI Philosophy](#12-ui-philosophy)
13. [Product Constraints](#13-product-constraints)
14. [Risks](#14-risks)
15. [Assumptions](#15-assumptions)
16. [Acceptance Criteria](#16-acceptance-criteria)
17. [Success Metrics & KPIs](#17-success-metrics--kpis)
18. [Future Roadmap](#18-future-roadmap)
19. [Glossary](#19-glossary)

---

## 1. Executive Summary

### 1.1 Project Overview

ARCHITAX is an enterprise-grade Property Tax (PBB-P2) workflow management system designed to digitize, formalize, and audit the complete lifecycle of land and building tax service requests — from taxpayer intake through Central Office adjudication and final resolution.

ARCHITAX is not a citizen-facing portal. It is an internal operational platform used exclusively by trained government officers at the district level to manage a hybrid physical-digital workflow mandated by existing regulatory frameworks.

### 1.2 Background

Indonesia's Property Tax on Rural and Urban Land and Buildings — *Pajak Bumi dan Bangunan Perdesaan dan Perkotaan* (PBB-P2) — was transferred from the national government to district and city authorities under Law No. 28 of 2009 on Regional Taxes and Levies. This transfer mandated district-level collection, administration, and processing of all PBB-P2 service requests.

The service encompasses six distinct application types: Partial Mutation, Full Mutation Update, Full Mutation Regular, Correction, New Tax Object Registration, and Reactivation. Each involves a formal physical document submission from the taxpayer, an internal government review and validation cycle, aggregation into dispatch batches, physical courier dispatch to the Central Office, and — upon return — final status recording.

Across districts, this process has historically been managed through paper registers, spreadsheets, and informal coordination between officers. The absence of a purpose-built digital system has created measurable operational risks including lost records, inconsistent numbering, unclear chain of custody, inability to audit post-dispatch outcomes, and no centralized visibility for supervisors or management.

ARCHITAX is built to resolve exactly these problems while fully preserving the legal and procedural integrity of the existing business process. It digitalizes the administrative layer without altering the regulatory process itself.

### 1.3 Vision

> **ARCHITAX will be the single system of record for every PBB-P2 service request processed at the district level — from the moment a taxpayer submits a physical application to the moment the Central Office renders a final decision.**

Every record in ARCHITAX will be uniquely numbered, permanently auditable, role-accountable, and electronically archived — making the district property tax office paperless in administration while remaining fully compliant with physical-document legal requirements.

### 1.4 Mission

To give every district property tax officer a clear, fast, and reliable digital workspace that eliminates administrative ambiguity, enforces the correct business process, maintains legal compliance, and produces a complete audit trail — without changing the government workflow they already know.

### 1.5 Product Goals

| # | Goal | Horizon |
|---|------|---------|
| G-01 | Eliminate untracked paper registers and spreadsheet-based request management | Launch |
| G-02 | Provide real-time operational visibility into the status of every Request, Bundle, and Manifest | Launch |
| G-03 | Enforce deterministic, rule-bound exception handling (Surgical Isolation and Full Bundle Return) digitally | Launch |
| G-04 | Produce a complete, immutable audit trail for every state change across every record | Launch |
| G-05 | Enable digital archival of every physical document with full traceability back to its originating Request | Launch |
| G-06 | Provide supervisors and management with operational reporting and KPI visibility | Launch |
| G-07 | Support multi-district deployment with district-scoped data isolation | Medium Term |
| G-08 | Serve as the operational backbone for an eventual fully digital submission channel | Long Term |

### 1.6 Business Objectives

1. Reduce average Request processing cycle time by at least 40% within 12 months of full adoption.
2. Achieve zero unrecoverable document loss events by maintaining a complete digital archive of every physical submission.
3. Reach 100% audit trail coverage — every status change on every record is traceable to a specific officer and timestamp.
4. Eliminate ad-hoc exception handling by enforcing the two-mechanism policy (Surgical Isolation, Full Bundle Return) at the system level.
5. Enable supervisors to generate operational reports without manual data aggregation.

### 1.7 Success Metrics

Detailed KPIs are defined in Section 17. Summary:

| Metric | Target |
|--------|--------|
| Request processing cycle time | ≥40% reduction vs. baseline |
| Document loss rate | 0 unrecoverable losses after go-live |
| Audit trail completeness | 100% of state changes logged |
| System availability | ≥99.5% uptime during business hours |
| User adoption rate | ≥90% of officers using the system as primary tool within 60 days |
| Correction rate | ≤5% of dispatched Requests returned via Surgical Isolation or Full Bundle Return |

### 1.8 Key Value Proposition

ARCHITAX converts a fragmented, paper-dependent, error-prone administrative process into a governed, auditable, digitally-archived workflow — without disrupting the legal framework, officer responsibilities, or physical document requirements that the current government process mandates.

It is the difference between a district office that *knows* what happened to every tax request and one that can only *hope* its paper records are complete.

---

## 2. Business Context

### 2.1 Current Manual Process

At a typical district PBB-P2 office without a purpose-built system, the process operates as follows:

1. Taxpayers submit physical application forms at a service counter. Intake officers verify documents and record information into paper registers or local spreadsheets.
2. Validated applications are grouped manually — often by placing physical documents into labeled folders — to form dispatch batches.
3. A dispatch officer creates a handwritten or word-processor-generated cover letter listing the batch contents.
4. Documents are physically delivered to the Central Office. The cover letter is signed and returned as an acknowledgment. A copy may or may not be filed.
5. After weeks or months, the Central Office returns either individual documents (defective requests) or occasionally whole batches, with notes that are transcribed manually into local records.
6. Officers attempt to reconcile returned documents with local records, often relying on memory or physical stapling of notes to original documents.

This process produces no centralized status view, no electronic audit trail, and no mechanism for a supervisor to verify whether the records accurately reflect what has actually occurred.

### 2.2 Current Operational Challenges

The most acute problems in the current state, documented across multiple district offices, include:

- **No system of record for Requests.** Records exist only as physical documents and potentially in disconnected local spreadsheets. There is no single queryable source of truth.
- **No Request numbering discipline.** Different officers have applied different numbering conventions, resulting in duplicates, gaps, and ambiguity when reconciling records across time periods.
- **No Bundle or Manifest tracking.** The concept of a "Bundle" or "Manifest" exists informally in the dispatch process, but there is no digital record of which Requests were grouped, which cover letter was generated, or when dispatch actually occurred.
- **No exception handling audit trail.** When requests are returned, the path of correction and re-dispatch is handled ad-hoc, often without recording the return event, the correction performed, or the new dispatch cycle.
- **No management visibility.** Supervisors have no dashboard, no report, and no real-time status view. Status queries require physically checking paper registers or asking individual officers.
- **No digital archive.** Physical documents, once dispatched, leave the office. If the Central Office or taxpayer later requires a copy, the district office may have no accessible record.

### 2.3 Why Digital Transformation Is Needed

The transfer of PBB-P2 administration to district authorities has placed full legal accountability for processing accuracy, completeness, and timeliness on district offices. This accountability cannot be met reliably with paper-based systems because:

- **Legal liability.** District offices are legally accountable for the accuracy of every dispatched Request. Without an audit trail, they cannot defend their processing records in any dispute.
- **Workload growth.** As districts grow and property data changes, the volume of annual service requests increases. Manual systems do not scale.
- **Regulatory expectations.** National-level government digitalization programs (including the *SPBE — Sistem Pemerintahan Berbasis Elektronik* framework) increasingly require electronically traceable government processes.
- **Central Office efficiency.** A digitally organized district office can produce cover letters, worksheets, and dispatch records that reduce errors at the Central Office reception point, shortening adjudication cycles.
- **Disaster resilience.** A digital archive provides the only reliable protection against physical document loss through fire, flood, or other disasters (addressed directly in `WORKFLOW.md §26, EC-32`).

### 2.4 Government Context

PBB-P2 administration is governed by:

- Law No. 28 of 2009 on Regional Taxes and Levies (transfer of PBB-P2 to regional authorities)
- Ministerial and regional regulations governing property tax service procedures and form requirements
- District-level operational policies governing officer roles, Bundle formation rules, and Manifest dispatch procedures

ARCHITAX does not introduce new legal obligations. It enforces the obligations that already exist — in a digital, auditable form.

### 2.5 Expected Organizational Impact

| Stakeholder Group | Expected Impact |
|---|---|
| Intake Officers (Officer 1) | Structured intake form replaces paper registers; NOP auto-formatting reduces transcription errors; clear validation status replaces informal judgment |
| Bundle/Manifest Officers (O2, O4) | Automated number assignment; system-generated cover letters; pre-dispatch validation gates prevent formation errors |
| Digitization Officer (O3) | Centralized document upload with direct linkage to Request records; digitization status visible to all officers |
| Dispatch Officer (O5) | Digital record of dispatch events; acknowledgment receipt archival; no more manual tracking of "what was sent when" |
| Completion Officer (O6) | Real-time dashboard of pending dispositions; structured return-recording replacing handwritten annotations |
| Supervisors / Management | First-time access to real-time operational KPIs, throughput trends, backlog depth, and correction rates |

### 2.6 Expected Citizen/Taxpayer Impact

ARCHITAX is an internal operations system. Taxpayers do not interact with it directly. However, they benefit materially through:

- **Faster processing.** Reduced cycle times at the district office mean faster resolution of their property tax applications.
- **Reduced re-submission burden.** Accurate intake recording and structured validation reduce errors that cause returns and repeat correction cycles.
- **Status visibility (future).** A future citizen-facing portal layer can be built on ARCHITAX's data to give taxpayers real-time status updates on their applications.

---

## 3. Problem Statement

The following problems are identified as the direct product motivation for ARCHITAX. Each problem is present in the current manual process and is explicitly solved by one or more ARCHITAX features.

| # | Problem | Impact | ARCHITAX Solution |
|---|---------|--------|-------------------|
| P-01 | **Paper dependency.** All records exist only as physical documents. Loss of a document is equivalent to loss of the record. | Permanent, unrecoverable service request data loss | Digital archive with full document traceability per `WORKFLOW.md §17` |
| P-02 | **No unified Request tracking.** Officers cannot query the current status of any Request without checking physical files. | Inability to answer basic questions about processing status | Centralized Request lifecycle tracking with status visible to all authorized officers |
| P-03 | **Inconsistent numbering.** Requests, Bundles, and Manifests are numbered ad-hoc, producing duplicates, gaps, and ambiguity. | Legal identification failures; reconciliation breakdowns | Deterministic, sequential numbering system enforced by the platform per `WORKFLOW.md §18` |
| P-04 | **No Bundle integrity enforcement.** Nothing prevents a Bundle from being formed incorrectly (wrong district, wrong count, no cover letter). | Systemic defects detected only at Central Office — triggering Full Bundle Returns | Pre-dispatch validation rules (VR-01 through VR-10) enforced digitally |
| P-05 | **No Manifest management.** The aggregation of Bundles into a Manifest for dispatch is entirely manual with no digital record. | Lost Manifest cover letters; inability to trace which Bundles were dispatched together | Manifest lifecycle managed end-to-end in the system per `WORKFLOW.md §8` |
| P-06 | **Ad-hoc exception handling.** Returned documents are processed informally, often losing the link between the return event, the correction, and the re-dispatch. | Correction cycles are untraceable; some returns may be silently dropped | Structured Correction module with Surgical Isolation and Full Bundle Return workflows |
| P-07 | **No audit trail.** There is no record of who performed which action at what time. | Legal defensibility is impossible; accountability cannot be assigned | Immutable append-only audit log covering every state change per `WORKFLOW.md §19` |
| P-08 | **No digitization discipline.** Physical documents are scanned inconsistently, or not at all, with no linkage between the scan and the originating Request. | No recoverable copy of physical documents after dispatch | Structured digitization workflow with Officer 3 accountability and Request-level traceability |
| P-09 | **No supervisory visibility.** Supervisors have no real-time or historical view of throughput, backlog, or correction rates. | Management decisions made on incomplete or stale data | Real-time dashboards and exportable reports covering all operational KPIs |
| P-10 | **Communication gaps between officers.** Status handoffs between Officer 1 → 2 → 3 → 4 → 5 → 6 are informal and occasionally missed. | Requests stall at transition points with no notification to the next officer | In-app and external notifications (WhatsApp, email) triggered on status changes |
| P-11 | **Duplicate request processing.** Without a central registry, duplicate submissions from the same taxpayer for the same property object can be processed in parallel without detection. | Conflicting records at Central Office; legal ambiguity | NOP uniqueness validation and duplicate-detection checks per `WORKFLOW.md §EC-02` |
| P-12 | **No role-based access control.** All officers may access all records, regardless of their responsibilities. | Officers can perform actions outside their role, creating accountability gaps | RBAC enforced at every screen and action per officer role per `SCREEN_SPECIFICATION.md` |
| P-13 | **No reporting.** Producing a throughput report, backlog analysis, or correction rate summary requires hours of manual spreadsheet work. | Management is perpetually under-informed; performance improvement is impossible without baseline data | Reports module with pre-built and custom report configurations |
| P-14 | **Disaster vulnerability.** Physical document archives are held at a single location. A fire or flood destroys all records for affected Requests. | Mass, permanent data loss; potential legal liability for the district | Cloud-backed digital archive survivable per `WORKFLOW.md §EC-32` and `RP-06` |

---

## 4. Stakeholders

### 4.1 Taxpayer

| Attribute | Description |
|-----------|-------------|
| **Role in system** | External — submits physical applications at the service counter; does not use ARCHITAX directly |
| **Business goals** | Timely, accurate processing of their property tax application (mutation, correction, new object, reactivation) |
| **Pain points** | Uncertainty about application status; being asked to re-submit due to intake errors; long processing times |
| **ARCHITAX impact** | Faster processing; lower error rates at intake; future status visibility via portal extension |
| **Success criteria** | Application reaches Completed status with no re-submission requirement |
| **System interaction** | Indirect — benefits from improved officer efficiency; no direct system access in current scope |

### 4.2 Officer 1 — Application Intake & Review

| Attribute | Description |
|-----------|-------------|
| **Primary accountability** | Accuracy of all recorded application data; the system-of-record for every Request |
| **Business goals** | Record applications accurately and completely; validate all required documents before submission |
| **Pain points** | Manual transcription errors for NOP and NIK; no instant feedback on whether an application is a duplicate; unclear status after handoff to Officer 2 |
| **ARCHITAX features used** | Create Request, Request Detail, Correction Detail (re-validation), Notifications, Dashboard, Search |
| **Success criteria** | Zero validation failures at Officer 2 or Central Office attributable to Officer 1 intake errors |
| **KPIs** | Intake accuracy rate; average time from submission to Validated status; re-validation rate |

### 4.3 Officer 2 — Bundle Coordination

| Attribute | Description |
|-----------|-------------|
| **Primary accountability** | Formation of valid Bundles; generation of Bundle Cover Letters; initiation of Partial Mutation Worksheets |
| **Business goals** | Form Bundles that pass all validation rules on first attempt; minimize Full Bundle Returns caused by formation errors |
| **Pain points** | Manual counting of Requests per Bundle; no automated check that all Requests in a Bundle are from the same district; generating cover letters manually in word processors |
| **ARCHITAX features used** | Bundle Detail, Dashboard (Active Bundles KPI), Notifications, Search |
| **Success criteria** | Zero Full Bundle Returns attributable to Bundle formation errors (cross-district, count mismatch) |
| **KPIs** | Bundle formation accuracy rate; average time from first Request validated to Bundle dispatched |

### 4.4 Officer 3 — Digitization & Archival

| Attribute | Description |
|-----------|-------------|
| **Primary accountability** | Complete, accurate, legible digitization of every physical Application Document |
| **Business goals** | Digitize every Request's physical documents before the parent Bundle is manifested; maintain archive quality standards |
| **Pain points** | No structured system for tracking which documents have been digitized; no way to flag corrupted scans for re-scan without manual annotation |
| **ARCHITAX features used** | Archive, Digital Archive Viewer, Notifications, Dashboard |
| **Success criteria** | 100% digitization completion before any Bundle reaches Manifested status; zero documents with Digitization Status = Pending at dispatch time |
| **KPIs** | Digitization completion rate; average scan-to-archive time; corrupted/illegible scan rate |

### 4.5 Officer 4 — Manifest Coordination

| Attribute | Description |
|-----------|-------------|
| **Primary accountability** | Formation of Manifests from dispatch-ready Bundles; generation of Manifest Cover Letters |
| **Business goals** | Assemble Manifests that are complete, correctly numbered, and include only Bundles with confirmed digitization |
| **Pain points** | Verifying that every Bundle in a Manifest has a completed Cover Letter and digitized documents — currently done by visual inspection of paper files |
| **ARCHITAX features used** | Manifest Detail, Bundle Detail (read), Dashboard, Notifications |
| **Success criteria** | Every Manifest generated passes all pre-dispatch validation rules on first attempt |
| **KPIs** | Manifest formation accuracy rate; average time from Bundles ready to Manifest dispatched |

### 4.6 Officer 5 — Dispatch & Logistics

| Attribute | Description |
|-----------|-------------|
| **Primary accountability** | Physical dispatch of Manifest packages; digital archival of signed Manifest Cover Letters |
| **Business goals** | Record every dispatch event accurately; ensure the digital archive captures the signed acknowledgment |
| **Pain points** | No centralized log of what was dispatched when; signed cover letters are often misfiled or lost after return |
| **ARCHITAX features used** | Manifest Detail (Record Dispatch), Digital Archive Viewer, Notifications |
| **Success criteria** | Every dispatch event is digitally recorded within 24 hours of physical dispatch; every signed Manifest Cover Letter is archived within 24 hours of receipt |
| **KPIs** | Dispatch recording lag; signed cover letter archival rate |

### 4.7 Officer 6 — Completion Monitoring

| Attribute | Description |
|-----------|-------------|
| **Primary accountability** | Post-dispatch tracking of all Request dispositions; recording Central Office decisions accurately |
| **Business goals** | Close every dispatched Request with an accurate final status; initiate Correction workflows for returns without delay |
| **Pain points** | No centralized view of which Requests are still Pending; manual reconciliation of Central Office return notices against internal records; easy to mis-record a disposition against the wrong Request Number |
| **ARCHITAX features used** | Dashboard (all KPI cards), Request Detail, Correction Detail, Search, Reports, Notifications |
| **Success criteria** | Zero mis-recorded dispositions; all Correction workflows initiated within 2 business days of return receipt |
| **KPIs** | Pending Request closure rate; correction initiation time; disposition accuracy rate |

### 4.8 Supervisor

| Attribute | Description |
|-----------|-------------|
| **Role** | Oversees all operational officers; responsible for service quality and throughput |
| **Business goals** | Real-time visibility into processing volumes, backlogs, and error rates; ability to identify systemic issues before they escalate |
| **Pain points** | Currently dependent on manual reports from officers; no baseline data for performance evaluation |
| **ARCHITAX features used** | Reports, Audit Log, Dashboard (read-all mode), Search |
| **Success criteria** | Can generate any operational report without requesting data from individual officers |
| **KPIs** | Report generation self-sufficiency; issue-detection lead time |

### 4.9 Administrator

| Attribute | Description |
|-----------|-------------|
| **Role** | Manages the operational configuration of ARCHITAX: officer accounts, roles, districts, workflow settings |
| **Business goals** | Maintain accurate user accounts and role assignments; configure system settings to reflect current operational policies |
| **Pain points** | No current system to manage — this role is new and created by ARCHITAX |
| **ARCHITAX features used** | Officer Management, User Management, Settings, Audit Log, Reports |
| **Success criteria** | All officer accounts accurately reflect current staffing and role assignments; no unauthorized access events |
| **KPIs** | Account accuracy rate; time-to-provision for new officers |

### 4.10 System Administrator

| Attribute | Description |
|-----------|-------------|
| **Role** | Infrastructure and platform operations; not a business user |
| **Business goals** | System availability, performance, backup integrity, security compliance |
| **Pain points** | No current system to administer |
| **ARCHITAX features used** | Settings (technical), Audit Log, Monitoring integrations (outside ARCHITAX UI scope) |
| **Success criteria** | ≥99.5% uptime; zero data loss events; successful backup verification |
| **KPIs** | Uptime percentage; mean time to recovery; backup success rate |

### 4.11 Management (District Head / Finance Director)

| Attribute | Description |
|-----------|-------------|
| **Role** | Executive oversight; strategic decision-making for the district property tax function |
| **Business goals** | Evidence of operational improvement; compliance with national digitalization mandates; reduced processing backlogs |
| **Pain points** | No data on current performance; unable to demonstrate compliance or improvement to auditors |
| **ARCHITAX features used** | Reports (exported summaries), Dashboard (read-only, aggregate view) |
| **Success criteria** | Can present auditors and senior government officials with verifiable processing records and performance data |
| **KPIs** | Year-over-year throughput growth; correction rate trend; processing time improvement |

---

## 5. User Personas

### Persona 1 — "Ibu Sari" · Officer 1 · Application Intake Specialist

**Background:** 34 years old. Has worked at the district tax office for 7 years. Previously tracked requests in a shared Excel spreadsheet on a local server. Comfortable with computers but not a power user. Uses the system 6–8 hours a day at a service counter PC.

**Responsibilities:** Receives physical applications from taxpayers, verifies identity documents, records all application data, validates requests, and hands them off to Officer 2.

**Technical skill:** Moderate. Comfortable with web-based forms and standard government administrative software. Unfamiliar with keyboard shortcuts initially but learns quickly.

**Motivations:** Wants her recorded data to be right the first time. Dislikes getting requests returned to her weeks later for correction errors she made at intake.

**Pain points:** Formatting NOP numbers manually; no way to check if a NOP has already been submitted; slow to find records from previous periods.

**Needs:** Fast form entry with auto-formatting; clear validation feedback before submission; searchable record history for the same NOP.

**Goals:** Zero intake errors flagged by Officer 2 or the Central Office. Finish daily intake batch by 15:00.

**Daily workflow:** Receives 5–20 physical applications per day. Enters each into ARCHITAX's Create Request screen. Validates them. Receives task notifications when returned requests come back for re-validation.

---

### Persona 2 — "Pak Budi" · Officer 2 · Bundle Coordinator

**Background:** 41 years old. 12 years at the office. The informal "process expert" — other officers ask him when unsure about rules. Currently maintains a personal notebook tracking which requests are in which batch.

**Responsibilities:** Groups validated requests into Bundles, creates cover letters, initiates worksheets for Partial Mutation cases, and ensures Bundles are ready for manifesting.

**Technical skill:** Moderate-high. Very organized; will master any system that gives him reliable counts and a clear status view.

**Motivations:** Wants the system to replace his personal notebook. Wants to know, at a glance, which Bundles are waiting on Officer 3's digitization.

**Pain points:** Cover letters take 30–45 minutes to produce manually per Bundle. Occasionally forms a Bundle that turns out to have a count mismatch because he miscounted the physical documents.

**Needs:** System-generated cover letters; automated count validation; clear indicator of digitization status per Bundle.

**Goals:** Never have a Full Bundle Return caused by a formation error again.

**Daily workflow:** Reviews the list of validated Requests, groups them into Bundles, runs the cover letter generator, monitors Bundle status on the Dashboard, and responds to digitization-complete notifications to advance Bundles to manifesting.

---

### Persona 3 — "Mbak Dewi" · Officer 3 · Digitization & Archival Specialist

**Background:** 28 years old. The youngest officer on the team. Comfortable with technology; previously managed document uploads on a cloud storage folder shared by all officers with no naming convention.

**Responsibilities:** Scans all physical application documents, uploads them to the archive, and confirms digitization completion per Request and Bundle.

**Technical skill:** High. Fast typist; already uses keyboard shortcuts in other tools.

**Motivations:** Wants the archive to be searchable and organized, not a folder of files named "scan_001.pdf."

**Pain points:** No link between a scanned file and the Request it belongs to. No way to know which documents are pending digitization.

**Needs:** Upload interface that asks for the linked Request number at the time of upload; clear queue of Requests pending digitization; instant feedback if a linked Record does not exist.

**Goals:** Zero Bundles delayed at the Manifest stage because her digitization is incomplete.

**Daily workflow:** Opens the Archive screen each morning, filters by Digitization Status = Pending, works through the queue, uploads scans, marks each complete. Uses the Archive Viewer to verify scan quality before confirming.

---

### Persona 4 — "Pak Hendro" · Officer 6 · Completion Monitor

**Background:** 52 years old. Senior. Has seen every kind of return and error over 20 years. Communicates directly with the Central Office. Currently keeps a physical ledger of all dispatched Requests and their outcomes.

**Responsibilities:** Monitors all dispatched Requests for Central Office decisions, records Completed or Returned outcomes, and initiates Correction workflows for returns.

**Technical skill:** Low-moderate. Has strong institutional knowledge but dislikes learning new software. Needs the system to be simple and obvious, not feature-rich.

**Motivations:** Wants to retire his physical ledger. Wants to know, in one view, every Request that is still Pending.

**Pain points:** Reconciling Central Office return notices with his physical ledger is time-consuming and error-prone. Sometimes records the disposition against the wrong Request Number.

**Needs:** A clear Pending queue showing only his outstanding records; a form that makes it impossible to record a disposition against the wrong Request; plain-language status labels.

**Goals:** Zero mis-recorded dispositions. Close every returned Request into a Correction workflow within one business day of receipt.

**Daily workflow:** Opens the Dashboard. Reviews the Tasks Due card filtered to his pending dispositions. Records each Central Office decision as it arrives. Opens Correction Detail screens to initiate return workflows.

---

### Persona 5 — "Bu Ratna" · Supervisor

**Background:** 47 years old. Head of the property tax service division. Answers to the District Finance Director. Currently produces monthly reports by asking each officer to report their numbers verbally and aggregating them manually.

**Responsibilities:** Performance oversight of all six officer functions; compliance with national digitalization mandates; reporting to management.

**Technical skill:** Moderate. Uses office productivity software daily. Has never used an enterprise workflow system but has asked for one for three years.

**Motivations:** Wants to stop being the one who doesn't know what's happening in her own office. Wants to give management data, not anecdotes.

**Pain points:** Zero real-time visibility. Monthly reports take a full day to compile. No way to know if Officer 6 is recording returns correctly without physically checking his ledger.

**Needs:** Dashboard with all-officer aggregate view; one-click report generation; audit log access for compliance reviews.

**Goals:** Generate a complete monthly operational report in under 30 minutes. Never again be surprised by a backlog she didn't know existed.

**Daily workflow:** Opens the Dashboard once a day to review KPI cards. Runs a weekly throughput report. Responds to exception alerts. Reviews the Audit Log when investigating discrepancies.

---

## 6. Product Scope

### 6.1 In Scope (Version 1.0 — Launch)

- Complete Request lifecycle management: intake, validation, bundling, manifesting, dispatch, completion, correction
- Six PBB-P2 service types: Partial Mutation (PM), Full Mutation Update (FMU), Full Mutation Regular (FMR), Correction (COR), New Tax Object (NTO), Reactivation (REA)
- Bundle management: formation, cover letter generation, Partial Mutation Worksheet, dispatch-readiness tracking
- Manifest management: formation, cover letter generation, dispatch recording, acknowledgment archival
- Exception handling: Surgical Isolation and Full Bundle Return workflows
- Digital archive: document upload, digitization status tracking, archive viewer, quality verification
- Deterministic numbering system: Request, Bundle, and Manifest number generation
- Immutable audit trail covering all state changes across all entities
- Role-based access control for six officer roles plus Supervisor, Admin, and System Admin
- In-app notifications for status changes, assignments, and overdue tasks
- WhatsApp and email notifications for critical workflow events
- Search and filtering across all entity types
- Operational reports and dashboard analytics
- PDF export of cover letters, worksheets, and reports
- Excel export of tabular reports and audit log extracts
- User and officer account management
- System settings: numbering configuration, district setup, service type settings, notification preferences
- Responsive design supporting desktop, laptop, and tablet viewports

### 6.2 Out of Scope (Version 1.0)

- **Citizen-facing portal.** Taxpayers do not interact with ARCHITAX directly. No public submission form, no taxpayer status lookup.
- **Central Office integration.** ARCHITAX does not connect to or exchange data with the Central Office's internal systems. All Central Office decisions are recorded manually by Officer 6.
- **Fully digital document submission.** Physical document submission by taxpayers is not replaced. Digitization remains a parallel officer function.
- **Mobile-native application.** ARCHITAX targets web browser access on desktop and tablet; no native iOS or Android application is in scope.
- **National or provincial multi-district federation.** Version 1.0 targets a single district office deployment. Multi-district federation is a future scope item.
- **Automated scan quality assessment.** Document legibility is verified manually by Officer 3 in the Archive Viewer.
- **Taxpayer communication.** Notifications go to officers only. Taxpayer SMS/WhatsApp status updates are a future scope item.
- **Financial calculations.** ARCHITAX manages the service request workflow, not the tax assessment calculation itself.
- **Integration with national NOP registry** for real-time duplicate detection. NOP uniqueness is enforced within ARCHITAX's own database.

### 6.3 Future Scope (Versioned Roadmap)

See Section 18 for the full roadmap. Highlights:
- Multi-district federation with provincial-level aggregate reporting
- Citizen-facing portal for application status lookup
- Direct Central Office integration via API or structured data exchange
- Fully digital taxpayer application channel (replacing physical intake)
- Mobile application for Officers 5 and 6 in the field
- Automated scan quality assessment
- AI-assisted duplicate detection using NOP and NIK matching

### 6.4 Enterprise Expansion Opportunities

ARCHITAX's workflow engine — hierarchical containment (Request → Bundle → Manifest), deterministic state machines, role-scoped accountability, and immutable audit trail — is architecturally transferable to other government tax and licensing administration domains:

- Motor vehicle tax (PKB) service request management
- Building permit application tracking
- Business license application workflows
- Any multi-officer, document-centric government service requiring a formal chain of custody

---

## 7. Functional Requirements

> This section defines *what* the system must do. It does not define *how*. Screen-level detail is specified in `SCREEN_SPECIFICATION.md`. Business rules are specified in `WORKFLOW.md`. This section bridges both into module-level business requirements.

---

### Module 01 — Authentication & Access Control

**Purpose:** Ensure every system action is performed by an authenticated, authorized actor with a verifiable identity.

**Business Value:** Satisfies the legal accountability requirement that every state change is traceable to a specific officer. Prevents unauthorized access to sensitive taxpayer data.

**Business Description:** Officers must authenticate before accessing any system function. Every session is time-bound. Role assignments govern which screens, actions, and records each officer can access. No officer can perform actions outside their designated role scope.

**Primary Users:** All officers, Admin, Supervisor, System Admin.

**Major Features:**
- Credential-based login (username + password) with session management
- Role-Based Access Control (RBAC) enforced at every screen and every action
- Multi-factor authentication option (configurable by Admin)
- Session timeout after inactivity
- Account lockout after repeated failed login attempts
- Password reset via email
- Login history available to Admin for security review
- All authentication events logged to the immutable audit trail

**Workflow Relationship:** Authentication is the gateway to the entire system. Every officer role maps to specific workflow stage authorities per `WORKFLOW.md §6` and access permissions per `SCREEN_SPECIFICATION.md Appendix C`.

**Inputs:** Username, password, (optionally) second factor.

**Outputs:** Authenticated session; role-scoped access grant; audit log entry.

**Success Indicators:** Zero unauthorized access events; 100% of state changes attributable to an authenticated actor.

---

### Module 02 — Dashboard & Operational Overview

**Purpose:** Give every officer a role-appropriate real-time summary of the workflow state immediately upon login.

**Business Value:** Eliminates the need for officers to manually query the status of outstanding work. Reduces the time-to-task from minutes (checking paper registers) to seconds (reading the Dashboard).

**Business Description:** The Dashboard is the primary landing screen for all users. It presents KPI summary cards, a task queue of pending actions, recent activity, and quick navigation to frequently accessed features. The data shown is scoped to the officer's role — Officer 1 sees pending intake tasks; Officer 6 sees pending disposition tasks; Supervisors see aggregate metrics across all officers.

**Primary Users:** All officers (role-scoped), Supervisor, Admin.

**Major Features:**
- Four KPI stat cards: Pending Intake, Active Bundles, Dispatched Manifests, Completed This Month
- Tasks Due list: records assigned to or owned by the current user requiring action, ordered by due date
- Recent Activity feed: the most recent audit events the user has performed or is involved in
- Favorites: user-pinned quick links to frequently accessed features or records
- KPI delta indicators: comparison against the previous 30 days
- Role-scoped data: Officers see only their own pending work; Supervisors see all

**Workflow Relationship:** The Dashboard provides entry points into all four workflow phases (Intake, Aggregation, Dispatch, Resolution) through the Tasks Due queue and navigation favorites.

**Inputs:** Current officer identity and role; current record states from all modules.

**Outputs:** Role-appropriate operational summary; navigable task queue.

**Success Indicators:** ≥90% of officers begin their daily work from the Dashboard; average time from login to first meaningful action < 30 seconds.

---

### Module 03 — Request Management

**Purpose:** Manage the complete lifecycle of a single PBB-P2 service request — from intake through final disposition.

**Business Value:** The Request is the atomic unit of the entire workflow. Every other module (Bundle, Manifest, Correction, Archive) exists to serve Request lifecycle management. Accurate, complete Request records are the legal foundation of the district office's processing accountability.

**Business Description:** A Request represents a single taxpayer application for one of six PBB-P2 service types. Its lifecycle spans four phases and up to twelve distinct states. Officer 1 creates and validates Requests. Subsequent lifecycle states are driven by other officers' actions in other modules. At all times, any authorized officer can view the current state of any Request and navigate its full history.

**Primary Users:** Officer 1 (create, validate, re-validate), Officer 2 (read, assign to Bundle), Officer 6 (record disposition), Supervisor (read-all), Admin (read/manage).

**Major Features:**
- Structured intake form capturing Primary Data, SPPT Data (Existing), and New Data per `WORKFLOW.md §30.5`
- Multi-record New Data support for Partial Mutation service type (one-to-many split records)
- NOP auto-formatting to canonical format on input
- Draft save and Submit for Validation workflow
- Status progression tracking across all twelve lifecycle states per `WORKFLOW.md §9`
- Audit log tab showing full history of every state change and field modification
- Related records panel showing parent Bundle, Manifest, and any Correction record
- Officer-scoped action buttons: validate (Officer 1), assign to Bundle (Officer 2), flag for correction (Officers 1, 6), record disposition (Officer 6)
- Breadcrumb navigation from Bundle/Manifest down to Request
- NIK masking for non-Officer-1 users

**Workflow Relationship:** Defined in full in `WORKFLOW.md §9` (Request Lifecycle) and `§10.1` (Request State Transition Table). Every state transition is role-restricted as specified therein.

**Inputs:** Physical application form data; officer validation judgment; Central Office disposition decisions.

**Outputs:** Validated Request (eligible for bundling); final disposition status (Completed or Returned); audit trail entries.

**Success Indicators:** 100% of submitted applications reach Validated status before entering Bundle formation; zero Requests with missing data at dispatch time.

---

### Module 04 — Bundle Management

**Purpose:** Manage the formation, validation, cover letter generation, and lifecycle of Bundles — the mid-level aggregation unit grouping Requests for dispatch.

**Business Value:** Bundles are the primary unit of error containment. A well-formed Bundle reduces the risk of Full Bundle Returns. The Bundle Cover Letter is a legal document. Bundle management is where most pre-dispatch quality gates are enforced.

**Business Description:** Officer 2 creates Bundles by selecting validated Requests, assigns a system-generated Bundle Number, conditionally produces the Partial Mutation Worksheet, and generates the Bundle Cover Letter. The system enforces pre-dispatch validation rules that block Bundle progression if any validation constraint is violated. Once dispatched, a Bundle's composition is permanently frozen.

**Primary Users:** Officer 2 (create, manage), Officer 3 (update digitization status), Officer 4 (read, assign to Manifest), Officer 5 (read, confirm dispatch).

**Major Features:**
- Bundle creation with system-assigned Bundle Number per `WORKFLOW.md §18`
- Request assignment with district and service type validation guards
- Automatic Request count reconciliation
- Partial Mutation Worksheet generation (conditional — PM service type only)
- Bundle Cover Letter generation (system-produced, Officer 2 authorized)
- Pre-dispatch validation gate: all member Requests digitized, count matches cover letter, district consistency verified
- Bundle status progression: Formed → Cover Letter Generated → Digitized → Manifested → Dispatched/Legally Valid
- Post-dispatch state management: Partially Returned, Fully Returned/Closed-Superseded, Closed (Completed)
- Bulk Request add/remove with confirmation and audit trail
- Bundle read-only state after dispatch

**Workflow Relationship:** Defined in `WORKFLOW.md §7` (Bundle Lifecycle) and `§10.2` (Bundle State Transition Table). Validation rules VR-01 through VR-06 per `WORKFLOW.md §20`.

**Inputs:** Validated Requests; Officer 2 grouping decision; Officer 3 digitization confirmations.

**Outputs:** Formed Bundle; Bundle Cover Letter (PDF); Partial Mutation Worksheet (conditional, PDF); Bundle status transitions.

**Success Indicators:** Zero Bundles dispatched with validation rule violations; Bundle Cover Letter generation time < 2 minutes per Bundle.

---

### Module 05 — Manifest Management

**Purpose:** Manage the formation, cover letter generation, dispatch recording, and lifecycle of Manifests — the top-level dispatch unit grouping Bundles for physical delivery to the Central Office.

**Business Value:** The Manifest is the legal dispatch container. The moment a Manifest is physically dispatched, both the Manifest and all its constituent Bundles become legally valid — permanently. Accurate Manifest management ensures this legal validity is correctly recorded and the signed acknowledgment is archived.

**Business Description:** Officer 4 creates Manifests by selecting dispatch-ready Bundles, assigns a Manifest Number, and generates the Manifest Cover Letter. Officer 5 records the physical dispatch event and, upon return of the signed cover letter, archives it. The Manifest is never recalled regardless of post-dispatch defects in its contents.

**Primary Users:** Officer 4 (create, manage), Officer 5 (dispatch, acknowledge), Officer 6 (read, monitor dispositions), Supervisor (read).

**Major Features:**
- Manifest creation with system-assigned Manifest Number per `WORKFLOW.md §18`
- Bundle assignment with dispatch-readiness validation guards
- Manifest Cover Letter generation (includes Signature Section)
- Dispatch recording: dispatch date, tracking reference, signed cover letter upload
- Acknowledgment recording: signed Manifest Cover Letter digital archival
- Manifest status progression: Formed → Cover Letter Generated → Dispatched/Legally Valid → Acknowledged → Closed
- Bundle composition view within each Manifest
- Request count roll-up (total Requests across all Bundles)
- Manifest is read-only and non-modifiable after dispatch

**Workflow Relationship:** Defined in `WORKFLOW.md §8` (Manifest Lifecycle) and `§10.3` (Manifest State Transition Table).

**Inputs:** Dispatch-ready Bundles; Officer 5 dispatch confirmation; signed Manifest Cover Letter (physical document → digital scan).

**Outputs:** Formed Manifest; Manifest Cover Letter (PDF); legal validity establishment; archived signed acknowledgment.

**Success Indicators:** Zero Manifests dispatched with undigitized Bundles; signed Manifest Cover Letter archived within 24 hours of physical receipt.

---

### Module 06 — Correction & Exception Handling

**Purpose:** Manage the structured resolution of post-dispatch defects — both Surgical Isolation (individual Request return) and Full Bundle Return — through a tracked, auditable correction workflow.

**Business Value:** This module is the enforcement mechanism for `WORKFLOW.md`'s Principle of Minimum Necessary Disruption. It ensures that returns are always handled via exactly one of the two permitted exception mechanisms, that every return event is traceable, and that corrected Requests re-enter the workflow as a clean new lifecycle cycle.

**Business Description:** Officer 6 receives Central Office return notices and records them in ARCHITAX as Correction records. Each Correction record is typed as either Surgical Isolation or Full Bundle Return. For Surgical Isolation, the specific returned Request(s) transition to Returned-Pending Correction while all other Requests in the Bundle remain unaffected. For Full Bundle Return, all Requests in the Bundle transition to Returned-Pending Correction and the Bundle is permanently closed and superseded. In both cases, Officer 1 re-validates the returned Request(s) and Officer 2 forms a new Bundle.

**Primary Users:** Officer 6 (create Correction, confirm return receipt), Officer 1 (re-validate returned Requests), Officer 2 (form replacement Bundle), Supervisor (read, oversight).

**Major Features:**
- Correction record creation with type selection: Surgical Isolation or Full Bundle Return
- Affected record linkage: original Bundle, Manifest, and specific returned Request(s)
- Central Office reference number capture
- Return receipt confirmation with date and notes
- Automatic status transitions: affected Requests → Returned-Pending Correction; Bundle (Full Return) → Closed-Superseded
- Re-validation workflow: Officer 1 reviews and re-validates returned Requests
- Re-bundling workflow: Officer 2 creates new Bundle from re-validated Requests
- Four-stage correction progress tracker: Reported → Returned → Re-bundled → Re-dispatched
- Correction closure once re-dispatch is confirmed

**Workflow Relationship:** Defined in `WORKFLOW.md §12–15` (Error Handling Strategy, Surgical Isolation, Full Bundle Return, Decision Tree). Business rules BR-03, BR-08, BR-09, BR-10 are enforced by this module.

**Inputs:** Central Office return notice; Officer 6 return recording; Officer 1 re-validation; Officer 2 new Bundle formation.

**Outputs:** Correction record; Request status transitions; Bundle status transition (Full Bundle Return); new Bundle linkage.

**Success Indicators:** 100% of returns processed via this module (zero ad-hoc exception handling); average Correction-to-Re-dispatch cycle ≤ defined SLA.

---

### Module 07 — Digital Archive

**Purpose:** Maintain a complete, traceable digital copy of every physical document associated with every Request, linked to the originating Request number, Bundle number, and Manifest number.

**Business Value:** The digital archive is the disaster recovery backbone of the entire system. It enables the district office to reconstruct any Request's physical document history even if the original physical document is no longer accessible (dispatched, lost, or destroyed). It satisfies `WORKFLOW.md §17` (Digital Document Flow) and BR-11.

**Business Description:** Officer 3 scans physical application documents and uploads them to the archive with a linked Request identifier. Each archive entry records the document type, upload timestamp, uploading officer, and digitization status. The Archive provides a full-text and metadata-searchable interface for locating any archived document. Document quality is verified by Officer 3 in the Archive Viewer before digitization is confirmed complete.

**Primary Users:** Officer 3 (upload, verify, mark complete), all officers (read-only access to their role-accessible records).

**Major Features:**
- Document upload with mandatory linkage to a Request/Bundle/Manifest record
- Batch upload for processing multiple documents in a single operation
- Digitization status workflow: Pending → In Progress → Complete
- Document quality flag: documents can be flagged for re-scan without deleting the original
- Archive Viewer: full-screen document viewer with zoom, rotation, and page navigation for multi-page PDFs
- Annotation and notes per archived document
- Archive search: filter by Record Type, Digitization Status, Date Range, Officer, Service Type, District
- Immutable archive: documents are never deleted; superseded or corrupted versions are annotated but retained per AR-01
- Traceability: every document is linked to its originating Request, Bundle, and Manifest identifiers

**Workflow Relationship:** Parallel to Phase B (Aggregation) per `WORKFLOW.md §3`. Digitization completion is a pre-dispatch gate for Bundle progression (VR-05).

**Inputs:** Physical scanned document files; Request identifier; Officer 3 quality judgment.

**Outputs:** Archived document linked to originating records; Digitization Status = Complete (per Request); digital recovery artifacts.

**Success Indicators:** 100% of dispatched Requests have at least one associated digitized document; zero archive entries with Record Reference missing or incorrect.

---

### Module 08 — Search & Filtering

**Purpose:** Enable any authorized officer to locate any record across all entity types using multiple search and filter dimensions.

**Business Value:** Eliminates the time cost of manually checking paper files or asking colleagues when trying to locate a specific Request, Bundle, or Manifest. Reduces time-to-information from minutes to seconds.

**Business Description:** The Search screen provides a unified, federated search across Requests, Bundles, Manifests, Corrections, and Archive Documents. Officers can search by record ID, NOP, taxpayer name, NIK, or any text field. Filter chips narrow results by entity type, status, service type, date range, officer, and district. Results are grouped by entity type, with match highlighting for the queried terms. Saved searches allow officers to quickly re-apply complex filters.

**Primary Users:** All officers, Supervisor, Admin.

**Major Features:**
- Federated full-text search across all entity types
- Filter chips: Entity Type, Status, Service Type, Date Range, District, Officer
- Match term highlighting in results
- Results grouped by entity type, with counts per tab
- Saved searches: name and recall complex filter configurations
- Recent search history
- Role-scoped results: officers see only records within their access permissions

**Workflow Relationship:** Read-only access to all workflow entities. Supports all phases as a cross-cutting navigation tool.

**Inputs:** Search query; active filter set; officer identity (for scoping).

**Outputs:** Ranked, filtered result set; navigation entry points to all matching record detail screens.

**Success Indicators:** Average time to locate a known record by ID < 5 seconds; average time to locate a record by NOP < 10 seconds.

---

### Module 09 — Notifications

**Purpose:** Proactively alert officers to workflow events that require their attention — without requiring them to poll the Dashboard or Search screen continuously.

**Business Value:** Eliminates workflow stall points caused by handoff delays between officers. When Officer 1 validates a Request, Officer 2 should know immediately. When a Manifest is dispatched, Officer 6 should begin monitoring. Notifications make the workflow self-propelling.

**Business Description:** ARCHITAX delivers in-app, email, and WhatsApp notifications for role-relevant workflow events. Notifications are typed (assignment, status change, overdue, mention, system), actionable (clicking navigates to the relevant record), and tracked (read/unread state). Officers can configure per-type notification preferences. System administrators can configure global notification settings and template content.

**Primary Users:** All officers (receive), Admin (configure templates and delivery settings).

**Notification Types:**

| Type | Trigger | Recipients |
|------|---------|------------|
| Assignment | Record assigned to an officer | Assigned officer |
| Status Change | Record owned by or assigned to officer changes status | Owning/assigned officer |
| Overdue Alert | A Pending Request exceeds defined monitoring SLA | Officer 6, Supervisor |
| Correction Initiated | Correction record opened on a record | Officer 1 (for re-validation), Officer 2 (for re-bundling) |
| Dispatch Completed | Manifest dispatched | Officer 6 (to begin monitoring) |
| System Event | Export ready, system maintenance | All / Admin |
| Mention | Officer mentioned in a note | Mentioned officer |

**Delivery Channels:**
- In-app: notification panel (bell icon) and full-page notification screen
- Email: configurable per notification type and per officer preference
- WhatsApp (via Fonnte API): configurable per notification type; used for high-priority events where officers may not be at their workstation

**Inputs:** Workflow state changes from all modules; officer preference settings.

**Outputs:** In-app notification entries; email messages; WhatsApp messages.

**Success Indicators:** Zero workflow stalls caused by officer not knowing that their action is required; notification delivery latency < 60 seconds from trigger event.

---

### Module 10 — Audit Trail

**Purpose:** Maintain an immutable, complete, chronologically ordered record of every significant state change and user action in the system.

**Business Value:** The audit trail is the legal backbone of ARCHITAX's accountability model. It enables the district office to reconstruct the exact history of any record at any point in time, identify the responsible actor for any action, and produce legally defensible evidence in any dispute or audit.

**Business Description:** Every state change, field modification, status transition, user login, export action, and administrative change is appended to the immutable audit trail as a time-stamped record with actor identity, action type, entity reference, and before/after state snapshot. Records are never deleted or modified — corrections are made by appending new correcting records (per `WORKFLOW.md §RP-05`). The Audit Log screen provides a filterable, searchable, exportable view of these records for Supervisors and Admins.

**Primary Users:** Supervisor (view, compliance review), Admin (view, investigate), System Admin (export for external audit).

**Major Features:**
- Automatic logging of all state changes across all entities (no manual entry)
- Actor identity, timestamp (with timezone), IP address, and action type per entry
- Before/after state diff for field-level changes
- Filter by entity type, action type, date range, actor, record ID
- Expandable row detail showing full diff
- Export to CSV and PDF (filtered range)
- Immutable: no deletion, no editing of historical entries; corrections appended only
- Legal hold flag: specific records can be marked as under legal hold per `WORKFLOW.md §EC-19`

**Workflow Relationship:** Cross-cutting across all four workflow phases. Satisfies `WORKFLOW.md §19` (Audit Trail Requirements) in full, including AR-01 through AR-07.

**Inputs:** All module state change events (automatically captured).

**Outputs:** Immutable audit log entries; exportable compliance reports.

**Success Indicators:** 100% of state changes have a corresponding audit log entry; zero gaps in audit trail continuity for any record.

---

### Module 11 — Reports & Analytics

**Purpose:** Enable Supervisors and management to generate operational reports covering throughput, backlog, officer workload, correction rates, and dispatch timelines without manual data aggregation.

**Business Value:** Converts a previously opaque process into a measurable, improvable operation. Gives management the evidence base to evaluate performance, justify resource allocation, and demonstrate regulatory compliance.

**Business Description:** The Reports module provides pre-built report templates and a configuration layer for customized reporting. Reports are parameterized by date range, district, officer, and service type. Output includes summary KPI cards, charts, and detail tables. Reports are exportable to PDF and Excel. Saved report configurations allow recurring reports to be re-run with consistent parameters.

**Primary Users:** Supervisor, Admin, Management (via exported reports).

**Pre-built Reports:**

| Report | Key Metrics |
|--------|-------------|
| Throughput Summary | Requests submitted, validated, dispatched, completed per period |
| Backlog Analysis | Requests by status and age; average time per lifecycle stage |
| Dispatch Timeline | Average Bundle formation, digitization, and manifesting durations |
| Correction Rate | Surgical Isolation and Full Bundle Return counts, rates, and causes |
| Officer Workload | Actions per officer per period; comparative performance |
| Archive Completeness | Digitization completion rate per Bundle; overdue digitization queue |

**Inputs:** Date range, district, officer, service type parameters; all module data.

**Outputs:** Rendered report with KPI cards, charts, and tables; PDF export; Excel export.

**Success Indicators:** Monthly report generation time < 30 minutes (vs. current ~8 hours manual); Supervisor self-sufficiency rate = 100% (no manual data collection from officers needed).

---

### Module 12 — User & Officer Management

**Purpose:** Enable Administrators to manage officer accounts, role assignments, district assignments, and access permissions.

**Business Value:** Role-based access control is only as good as the accuracy of the role assignments. This module ensures that the right officers have access to the right capabilities at all times, and that account provisioning and deprovisioning are tracked.

**Business Description:** Admins can invite new officers, assign roles and districts, deactivate departing officers, reset passwords, and enforce multi-factor authentication. User Management extends this to all account types, including read-only accounts and auditors. All account management actions are logged to the audit trail.

**Primary Users:** Admin only.

**Major Features:**
- Officer account creation via email invitation
- Role assignment: Officer 1–6, Supervisor, Admin, Read-only, Auditor
- District assignment (governs which district's records the officer can access)
- Account deactivation (records remain; login access revoked)
- Password reset and forced 2FA setup
- Login history per user
- Bulk role and status changes
- Account management audit trail

**Inputs:** New officer details; role and district assignment decisions; deactivation triggers.

**Outputs:** Provisioned officer accounts; deactivated accounts; audit log entries.

**Success Indicators:** Time-to-provision for new officers < 10 minutes; zero orphaned active accounts for departed officers.

---

### Module 13 — Settings

**Purpose:** Allow Administrators to configure system-wide operational parameters that govern the behavior of the workflow engine, numbering system, and notification delivery.

**Business Value:** Operational policies vary between districts and may change over time. Settings ensures that ARCHITAX's enforcement logic reflects actual district policy without requiring code changes.

**Business Description:** Settings provides grouped configuration panels for: Organization (name, district details, fiscal periods), Workflow (numbering prefixes, Bundle rules, Manifest rules), Notifications (email and WhatsApp templates, delivery thresholds), and Data/Compliance (audit log retention, archive retention). All setting changes are protected by confirmation gates and logged to the audit trail. Destructive operations (numbering reset) require typed confirmation.

**Primary Users:** Admin only.

**Key Configurable Parameters:**
- Numbering prefixes and sequence reset rules
- Maximum Requests per Bundle
- Cross-district Bundle policy
- Mixed service type Bundle policy
- Overdue alert threshold (days Pending before Officer 6 is notified)
- Notification delivery preferences (per event type)
- Audit log retention period

**Inputs:** Admin configuration decisions.

**Outputs:** Updated system behavior; audit log entries for all setting changes.

**Success Indicators:** All setting changes take effect immediately (no restart required); zero unauthorized configuration changes.

---

## 8. Non-Functional Requirements

### 8.1 Performance

| Requirement | Target |
|-------------|--------|
| Page load time (initial, authenticated) | < 2 seconds on a standard office LAN connection |
| Search results (first page) | < 1 second for most queries; < 3 seconds for complex federated search |
| Cover letter generation (PDF) | < 5 seconds per document |
| Report generation (standard) | < 15 seconds for pre-built reports on standard date ranges |
| Document upload (per file) | < 10 seconds for files up to 10 MB on a standard LAN connection |
| Audit log query | < 3 seconds for standard filtered queries |

### 8.2 Availability

| Requirement | Target |
|-------------|--------|
| System availability (business hours) | ≥99.5% uptime (Monday–Friday, 07:00–17:00 local) |
| Planned maintenance window | Off-hours only (after 17:00, with 48-hour advance notice to all users) |
| Unplanned downtime recovery (RTO) | ≤ 4 hours |
| Data recovery point objective (RPO) | ≤ 24 hours (daily backups); ≤ 1 hour (incremental backup preferred) |

### 8.3 Reliability

- All data mutations are atomic: a state change either completes fully or rolls back entirely. No partial state changes are persisted.
- File uploads are resumable or retried automatically; a failed upload does not create a partial archive entry.
- The audit trail is append-only and cannot be corrupted by application-level errors.

### 8.4 Scalability

- The system must handle a district office with up to 50 active users and up to 10,000 Requests per calendar year without performance degradation.
- The numbering system (`WORKFLOW.md §18`) supports up to 999,999 Requests, 99,999 Bundles, and 9,999 Manifests per district per year — well above any projected district volume.
- The architecture must support future multi-district federation without requiring data migration or re-numbering of existing records.

### 8.5 Maintainability

- All design tokens are centralized per `UI_DESIGN_SYSTEM.md §24` — no hard-coded values in individual components.
- All business rules are implemented in a single, documented layer — not distributed across UI, API, and database.
- Settings changes take effect without redeployment.
- Code coverage for all workflow state machine logic must be ≥ 90%.

### 8.6 Security

| Requirement | Specification |
|-------------|---------------|
| Authentication | Session-based with JWT; mandatory re-authentication after inactivity timeout |
| Authorization | RBAC enforced server-side on every action; client-side UI gating is supplementary, not sole enforcement |
| Data transmission | TLS 1.2 minimum for all data in transit |
| Sensitive field masking | NIK displayed in masked form to non-Officer-1 roles |
| File storage | Uploaded documents stored in access-controlled cloud storage; direct file URLs are signed and short-lived |
| Audit trail | All authentication events, data mutations, and export actions logged |
| Account lockout | Account locked after 5 consecutive failed login attempts; Admin notification triggered |
| Session management | Sessions invalidated on logout; concurrent session management (Admin-configurable) |

### 8.7 Accessibility

The product meets WCAG 2.1 Level AA as its baseline compliance standard, as specified in `UI_DESIGN_SYSTEM.md §21`. Key requirements:

- All text meets minimum contrast ratios (4.5:1 body text, 3:1 large text)
- The full interface is keyboard-operable without mouse dependency
- All interactive elements display a visible focus ring
- All status indicators convey meaning through both color and text (never color alone)
- Screen reader compatibility for all forms, tables, modals, and status regions

### 8.8 Responsiveness & Browser Support

| Breakpoint | Behavior |
|------------|----------|
| Desktop (≥1440px) | Full layout; expanded sidebar; multi-column grids |
| Laptop (1024–1439px) | Full layout; reduced padding |
| Tablet (768–1023px) | Icon-only sidebar; single-column content; horizontal scroll for tables |
| Mobile (<768px) | Off-canvas drawer sidebar; single column; table rows → stacked cards |

**Supported browsers:** Latest two stable releases of Chrome, Firefox, Safari, and Edge. Internet Explorer is not supported.

### 8.9 Hybrid Paper-Digital Support

ARCHITAX is designed for a hybrid operational environment where physical documents are the legal primary artifact and digital records are a faithful parallel representation. This means:

- The system never requires an action to be physically impossible (e.g., it will not refuse to record a dispatch if the signed cover letter is not yet uploaded — the upload can follow).
- Physical document counts and digital records must remain reconcilable at all times.
- The digital archive is a preservation and recovery tool, not a replacement for physical compliance.

### 8.10 Data Retention

| Data Type | Retention Period |
|-----------|-----------------|
| Request records | Permanent (no deletion) |
| Bundle and Manifest records | Permanent (no deletion) |
| Archived documents | Configurable minimum (suggested: 10 years minimum for government records) |
| Audit log entries | Configurable minimum (suggested: 10 years minimum for government records) |
| Notification history | 12 months |
| Login history | 24 months |

Soft deletion only: no record is hard-deleted from the system. Deleted/superseded records are flagged and excluded from default views but remain queryable by Admins.

### 8.11 Compliance & Auditability

- Every state change is logged per `WORKFLOW.md §19`.
- Audit log exports (CSV/PDF) can be produced for any date range for external audit review.
- Legal hold flags prevent deletion or modification of records under litigation.
- The system maintains chain-of-custody traceability from taxpayer submission to final Central Office decision.

### 8.12 Backup Strategy

- Daily automated full backups of all application data and file storage.
- Backup storage is geographically separated from primary infrastructure per `WORKFLOW.md §A-04`.
- Backup integrity is verified monthly by System Admin.
- Recovery procedure is documented and tested quarterly.

---

## 9. Business Rules Overview

The complete set of binding business rules is defined in `WORKFLOW.md §11` (Business Rules, BR-01 through BR-14). This section provides a categorical summary for orientation. **In any conflict between this summary and `WORKFLOW.md`, `WORKFLOW.md` governs.**

### Category 1 — Hierarchical Containment

| Rule | Summary |
|------|---------|
| BR-01 | A Request belongs to exactly one Bundle at any point in time |
| BR-02 | A Bundle belongs to exactly one Manifest for its entire lifecycle |
| BR-03 | A returned Request is never re-inserted into its original Bundle — only into a new Bundle |

### Category 2 — Legal Validity

| Rule | Summary |
|------|---------|
| BR-06 | Legal validity is established at the moment of physical dispatch and is permanent and irrevocable |
| BR-07 | The Manifest is never recalled, regardless of post-dispatch defects in its contents |

### Category 3 — Exception Handling

| Rule | Summary |
|------|---------|
| BR-08 | Exactly two exception mechanisms exist: Surgical Isolation and Full Bundle Return |
| BR-09 | Surgical Isolation operates at Request granularity; Full Bundle Return at Bundle granularity; neither at Manifest granularity |
| BR-10 | A Bundle subject to Full Bundle Return is permanently closed and its number never reused |

### Category 4 — Document Integrity

| Rule | Summary |
|------|---------|
| BR-04 | Partial Mutation Worksheet is created if and only if the Bundle contains at least one PM Request |
| BR-05 | Bundle Cover Letters have exactly three required elements; Manifest Cover Letters have exactly four |
| BR-11 | Every digitized document must be traceable to its originating Request, Bundle, and Manifest number |

### Category 5 — Numbering & Accountability

| Rule | Summary |
|------|---------|
| BR-12 | Officer roles are non-transferable per workflow instance; the performing officer is recorded regardless of individual staffing |
| BR-13 | A Request's terminal status per lifecycle cycle is binary: Completed or Returned (triggering a new cycle) |
| BR-14 | All numbering is unique, sequential within scope, and permanently assigned — never reused |

---

## 10. Workflow Overview

The complete ARCHITAX business process is defined in `WORKFLOW.md`. This section provides a high-level summary only. **For any implementation decision, `WORKFLOW.md` is the governing reference.**

### Four Macro-Phases

```
Phase A — Intake & Validation
Taxpayer submits physical application → Officer 1 reviews and records
Output: Validated Request

Phase B — Aggregation
Officer 2 forms Bundle → Officer 3 digitizes → Officer 4 forms Manifest
Output: Dispatch-ready Manifest with digital archive

Phase C — Dispatch & Transit
Officer 5 dispatches physically to Central Office → acknowledges
Output: Legally valid Bundle and Manifest

Phase D — Resolution & Completion
Central Office adjudicates → Officer 6 records dispositions
Output: Completed or Returned Requests → Correction cycle if returned
```

### Three Core Entities

| Entity | Role | Key Properties |
|--------|------|----------------|
| Request | Atomic unit — one taxpayer application | Permanent number; 12 lifecycle states; owned by Officer 1 |
| Bundle | Mid-level container — groups Requests | Permanent number; 7 lifecycle states; legal validity at dispatch |
| Manifest | Top-level container — groups Bundles for dispatch | Permanent number; 6 lifecycle states; never recalled |

### Two Exception Mechanisms

Both exception mechanisms, and the decision logic between them, are defined in `WORKFLOW.md §12–15`. Neither mechanism revokes the Manifest's legal validity. Both result in a new Bundle formation cycle for returned Requests.

| Mechanism | Frequency | Granularity | Bundle Effect |
|-----------|-----------|-------------|---------------|
| Surgical Isolation | 90–95% of cases | Individual Request(s) | Bundle remains valid |
| Full Bundle Return | 5–10% of cases | Entire Bundle | Bundle closed and superseded |

---

## 11. Screen Overview

The complete specification for all 17 screens — including layout, component hierarchy, interactions, data requirements, empty/loading/error states, permissions, responsive behavior, accessibility, and keyboard shortcuts — is defined in `SCREEN_SPECIFICATION.md`.

The following summary provides orientation for each screen's business purpose, primary users, and workflow relationship.

| Screen | Purpose | Primary Users | Workflow Relationship |
|--------|---------|--------------|----------------------|
| **01 — Dashboard** | Operational summary; role-scoped task queue; KPI cards | All | Entry point to all four phases; Tasks Due drives daily work routing |
| **02 — Request Detail** | View and act on a single Request's complete lifecycle | Officer 1, 2, 6 | Primary interface for Phase A actions (validate) and Phase D actions (record disposition) |
| **03 — Create Request** | Intake form for recording a new taxpayer application | Officer 1 | Phase A — creates the Submitted/Under Review → Validated Request |
| **04 — Bundle Detail** | View and manage a single Bundle — composition, cover letter, worksheet, status | Officer 2, 3, 4, 5 | Phase B — Bundle formation, cover letter generation, digitization tracking |
| **05 — Manifest Detail** | View and manage a single Manifest — Bundles, cover letter, dispatch, acknowledgment | Officer 4, 5 | Phase B/C — Manifest formation, dispatch recording, acknowledgment archival |
| **06 — Correction Detail** | Track and resolve a single Correction event (Surgical Isolation or Full Bundle Return) | Officer 6, 1, 2 | Phase D — exception handling per `WORKFLOW.md §12–15` |
| **07 — Archive** | Full-width table workspace for managing all digital archive entries | Officer 3 | Phase B — parallel to aggregation; digitization queue and completion tracking |
| **08 — Digital Archive Viewer** | Full-screen viewer for a single archived document | Officer 3, all (read) | Phase B — quality verification before marking digitization complete |
| **09 — Search** | Federated search across all entity types | All | Cross-cutting — record location and navigation tool for all phases |
| **10 — Officer Management** | Create, edit, deactivate officer accounts and role assignments | Admin | Administrative — system configuration, not workflow |
| **11 — User Management** | Manage all user accounts, access levels, and security settings | Admin | Administrative — security and access control |
| **12 — Audit Log** | Filterable, exportable view of the immutable system audit trail | Supervisor, Admin | Cross-cutting — compliance, dispute resolution, accountability review |
| **13 — Reports** | Pre-built and custom operational reports with chart and table output | Supervisor, Admin | Cross-cutting — performance measurement and management reporting |
| **14 — Settings** | System-wide configuration of workflow rules, numbering, notifications | Admin | Administrative — policy enforcement configuration |
| **15 — Notifications** | In-app notification feed; notification preferences | All | Cross-cutting — workflow event alerting for all phases |
| **16 — Profile** | Personal account information, security, notification preferences | All | Administrative — per-user identity and preferences |
| **17 — Login** | Authentication entry point | All (pre-auth) | Pre-workflow — gateway to the entire system |

---

## 12. UI Philosophy

The complete visual language, component library, and interaction specifications are defined in `UI_DESIGN_SYSTEM.md`. This section summarizes the governing philosophy for team alignment. **For all visual decisions, `UI_DESIGN_SYSTEM.md` is the governing reference.**

### 12.1 Visual Identity: Calm Authority

ARCHITAX's visual personality is built around a single governing idea: the interface must feel like software an institution can trust with permanent public records. Every visual decision — color, spacing, typography, motion — serves this goal.

The aesthetic reference is a contemporary enterprise application (clean sidebar with identity gradient, white content surfaces, soft card elevation, semantic status colors) adapted for a government-grade administrative context. It is modern without being playful, clean without being cold.

### 12.2 Design Language Summary

| Property | Specification |
|----------|--------------|
| Primary color | `#6C5CE0` — violet-purple; used exclusively for actions, links, active states, and focus rings |
| Sidebar background | Diagonal gradient from `#6FD8E0` to `#B6A4F0` — the one permitted gradient in the system |
| Surface color | `#FFFFFF` — cards, modals, tables |
| Canvas color | `#F4F4F9` — outermost page background |
| Typography | Inter (or equivalent humanist grotesk); tabular figures for all numeric data |
| Type scale | 30px display → 12px caption; 9 defined roles |
| Spacing scale | 8px base unit (4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80px) |
| Border radius | 8px (controls) → 20px (modals) → 999px (pills/tags) |
| Status colors | Success `#2FBE8F`, Warning `#F0A93B`, Danger `#F0507F`, Info `#6C5CE0` |
| Motion | Nothing animates longer than 300ms; motion confirms actions, never entertains |

### 12.3 Interaction Philosophy

- **One primary action per screen.** Every page has exactly one most-important action. Secondary actions are visually subordinate.
- **Progressive disclosure.** Screens show the minimum to orient and act. Detail lives behind a click.
- **Predictable placement.** Primary actions are top-right in page headers and bottom-right in forms — always.
- **Density choice, not density imposition.** Tables offer Comfortable (56px rows) and Compact (40px rows) density modes. The user decides when to increase density.

### 12.4 Navigation Structure

Four levels of navigation:

1. **Sidebar (Level 1):** Primary destinations — persistent, always visible or icon-collapsed
2. **Page Title (Level 2):** Current location — named in 30px/700 display text
3. **Tabs (Level 3):** Sub-views of the same destination (e.g., Request tabs: Primary Data, SPPT Data, New Data, Documents, Audit Log)
4. **Breadcrumb (Level 4):** Record depth on detail pages (e.g., "Bundles › BND-2024-00042")

### 12.5 Enterprise Usability

ARCHITAX is designed for the 500th use, not the first impression. This means:

- Officers who use the system 6–8 hours a day for years should find it fast, predictable, and low-fatigue
- Keyboard shortcuts are first-class features, not afterthoughts
- Table density modes and bulk actions serve power users without compromising readability for standard users
- Consistent patterns across all 17 screens mean skills transfer instantly between modules

### 12.6 Accessibility

WCAG 2.1 Level AA compliance is a non-negotiable baseline. Key commitments:

- All text: minimum 4.5:1 contrast ratio
- Full keyboard operability: every action accessible without a mouse
- Status meaning: never communicated by color alone
- Focus states: always visible, never suppressed
- Screen reader support: semantic HTML, ARIA roles, and live regions throughout

---

## 13. Product Constraints

### 13.1 Government Regulations

- PBB-P2 processing procedures are governed by national tax regulations and regional implementing regulations. ARCHITAX does not modify these procedures; it enforces them digitally.
- Physical document submission remains legally required at this time. ARCHITAX cannot replace the physical submission requirement; it can only complement it with digital tracking.
- Data retention requirements for government tax records are defined by national archival regulations and are configurable in ARCHITAX Settings but cannot be set below the legal minimum.

### 13.2 Legal Compliance

- The legal validity of a dispatched Bundle or Manifest is established by the physical dispatch act, not by any digital event in ARCHITAX. ARCHITAX records this event but does not itself constitute the legal act.
- Audit trail records must be non-repudiable. The system cannot permit deletion or backdating of audit entries.
- NIK (national identity numbers) is sensitive personal data governed by applicable data protection regulations. Access to full NIK values is restricted to Officer 1 and Admin.

### 13.3 Physical Document Dependency

- ARCHITAX operates in parallel with, not in replacement of, the physical document chain of custody. Physical documents move independently of digital records.
- The system cannot prevent physical document loss once documents leave the district office; it can only ensure that digital copies exist to enable recovery.
- Officer 3's digitization function is the sole bridge between the physical and digital systems. Any digitization quality failures directly affect disaster recovery capability.

### 13.4 Hybrid Workflow

- Officers perform digital actions in ARCHITAX on the same day as or the day after the corresponding physical actions. Real-time digital-physical synchronization is an operational goal, not a technical enforcement.
- The system allows recording of dispatch events after the fact (Officer 5 records dispatch date, which may be today or a past date) to accommodate the reality that physical actions precede digital recording.

### 13.5 Infrastructure Assumptions

- The deployment environment provides internet connectivity at district office workstations sufficient to support a web-based application. If the internet connection is intermittent, the application may be temporarily unavailable — offline mode is not in scope.
- Cloud storage for the digital archive must be contracted and provisioned before go-live. Assumed capacity: minimum 1 TB at launch, scalable.
- District offices provide standard desktop or laptop computers with a modern browser. Mobile-native support is out of scope for Version 1.0.

### 13.6 Operational Limitations

- ARCHITAX does not integrate with the Central Office's internal systems. Central Office decisions are entered manually by Officer 6. The accuracy of this data is dependent on Officer 6's diligence.
- The system assumes each officer is assigned to a single district. Multi-district officer assignments are not supported in Version 1.0.
- Taxpayer communication (status notifications to taxpayers) is out of scope. Officers who wish to communicate processing status to taxpayers do so outside the system.

---

## 14. Risks

### 14.1 Business Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| B-01: District regulations change after deployment, requiring workflow modifications | Medium | High | Settings module provides configurable rules; formal change management process for workflow-level changes |
| B-02: Scope creep expands to Central Office integration before Version 1.0 stability is proven | High | Medium | Strict V1.0 scope boundary in this document; Central Office integration explicitly in future roadmap |
| B-03: Management expectations exceed V1.0 delivery (e.g., expecting citizen portal) | Medium | Medium | Executive briefing on scope before kick-off; Section 6 scope document as reference |
| B-04: District operational budget insufficient for cloud storage growth | Low | Medium | Storage cost estimation in infrastructure planning; tiered archival strategy in roadmap |

### 14.2 Operational Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| O-01: Officers continue using paper registers alongside ARCHITAX, creating divergent records | High | High | Change management program; Dashboard designed to make parallel record-keeping visibly redundant; Supervisor report visibility to detect non-adoption |
| O-02: Officer 6 mis-records dispositions due to unfamiliarity with new interface | Medium | High | UI design specifically targets this risk: Correction Detail screen makes incorrect assignment structurally difficult; Audit Log enables rapid detection and correction |
| O-03: Digitization backlog prevents Bundles from reaching Manifested state | Medium | Medium | Dashboard digitization KPI; Officer 3 task queue; Supervisor notification on overdue digitization |
| O-04: Officer staffing changes leave roles unassigned, blocking workflow stage progression | Medium | High | Officer Management module enables rapid provisioning; Admin receives notification of unassigned officer roles |

### 14.3 Technical Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| T-01: Slow internet at district office impairs application performance | Medium | Medium | Performance targets set conservatively; offline read capability as future enhancement |
| T-02: Large file uploads fail or are truncated on slow connections | Medium | Low | Resumable upload implementation; max file size limit enforced at upload |
| T-03: PDF generation service unavailable blocks cover letter creation | Low | High | PDF generation decoupled from state machine; failure returns graceful error without blocking workflow |
| T-04: Cloud storage vendor outage makes archived documents unavailable | Low | High | Multi-region storage with geographic redundancy per `WORKFLOW.md §A-04` and `RP-06` |

### 14.4 Data Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| D-01: Data entry errors by Officer 1 propagate through the workflow before being detected | High | Medium | Pre-dispatch validation gates catch structural errors; post-dispatch errors handled by Surgical Isolation workflow |
| D-02: Audit trail gap caused by a failed write during a state transition | Low | Critical | Atomic transactions ensure state change and audit entry succeed together or both fail |
| D-03: Numbering collision under concurrent Bundle or Manifest formation | Low | High | Centralized numbering service with locking per `WORKFLOW.md §A-08` and `EC-21, EC-22` |

### 14.5 Security Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| S-01: Unauthorized access to taxpayer NIK data | Low | Critical | NIK masking by role; audit log of all NIK access attempts; server-side RBAC enforcement |
| S-02: Brute-force account compromise | Low | High | Account lockout after 5 failed attempts; Admin alert on lockout event |
| S-03: Admin account compromised, leading to mass role re-assignment | Very Low | Critical | Audit trail for all admin actions; MFA strongly recommended for Admin accounts; least-privilege principle |

### 14.6 Adoption Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| A-01: Officers resist adopting the system due to habit or skepticism | High | High | UI designed for ease of the 500th use; onboarding training; Supervisor dashboard visibility creates adoption accountability |
| A-02: Training insufficient for Officer 1 and Officer 6 who have the most complex interactions | Medium | High | Role-specific training modules; system-generated help text and tooltips for complex fields |
| A-03: Supervisor does not use Reports module, losing the management visibility value proposition | Medium | Medium | Report generation made extremely fast (< 30 minutes); automated scheduled report delivery option in roadmap |

---

## 15. Assumptions

The following assumptions are made at the time of this document's writing. If any assumption proves false, the affected sections of this PRD and its companion documents should be reviewed.

| # | Assumption | Risk if False |
|---|------------|--------------|
| AS-01 | Each district officer has access to a desktop or laptop PC with a modern browser and reliable LAN/internet connectivity during business hours | Application unavailability during work hours; may require offline mode development |
| AS-02 | The existing six service types (PM, FMU, FMR, COR, NTO, REA) cover all PBB-P2 applications the district processes | New service types require workflow and schema additions |
| AS-03 | Officer roles (1–6) represent distinct functional responsibilities; in small offices, one individual may hold multiple roles | No system change required; audit trail records the role performed, not the individual (BR-12) |
| AS-04 | The Central Office's adjudication decisions are received as return notices and recorded manually by Officer 6 — no direct digital channel from Central Office exists | If Central Office does provide a digital channel, Module 06 must be extended to consume it |
| AS-05 | Physical document submission by taxpayers remains a legal requirement for the foreseeable future | If fully digital submission is mandated, Phase A must be redesigned |
| AS-06 | Cloud storage and internet infrastructure are procured and operational before go-live | Deployment blocked; digital archive unavailable |
| AS-07 | All officers will be trained before go-live; ARCHITAX does not need to be self-explanatory for completely untrained users | Higher error rates at launch; may require enhanced in-app guidance |
| AS-08 | A single district deployment is the target for Version 1.0; multi-district federation is addressed in Version 2.0 | Multi-district requirements in V1.0 scope would require significant architectural additions |
| AS-09 | The numbering sequence for Requests, Bundles, and Manifests resets annually per district (configurable in Settings) | Numbering format must be extended for very high-volume districts without annual reset |
| AS-10 | Fonnte WhatsApp API is available and contractually usable for government operational notification purposes | WhatsApp notification channel must be replaced with an alternative |

---

## 16. Acceptance Criteria

The following criteria define the minimum conditions for accepting ARCHITAX as production-ready. All criteria must be met before go-live is authorized.

### 16.1 Business Acceptance

| Criterion | Condition |
|-----------|-----------|
| BA-01 | All six PBB-P2 service types can be intake-recorded without data loss or truncation |
| BA-02 | A complete Request lifecycle — from intake to Completed status — can be executed end-to-end in the system without any offline paper steps for the digital recording portion |
| BA-03 | Both exception mechanisms (Surgical Isolation, Full Bundle Return) are executable end-to-end through the Correction module |
| BA-04 | All 14 business rules (BR-01 through BR-14) are enforced by the system; none can be bypassed by any officer role |
| BA-05 | A Supervisor can generate every pre-built report without requesting data from any officer |
| BA-06 | An Admin can provision a new officer account and have it operational within 10 minutes |

### 16.2 Operational Acceptance

| Criterion | Condition |
|-----------|-----------|
| OA-01 | All pre-dispatch validation rules (VR-01 through VR-10) are enforced at the correct lifecycle stage |
| OA-02 | Every state change across every entity type produces a corresponding audit log entry |
| OA-03 | The audit log cannot be edited or deleted by any user role, including Admin |
| OA-04 | PDF cover letter generation (Bundle and Manifest) produces a correctly formatted, complete document |
| OA-05 | Notification delivery (in-app, email, WhatsApp) is confirmed working for every defined trigger event |
| OA-06 | A digital archive document can be uploaded, linked to a Request, and retrieved by any authorized officer |
| OA-07 | The system correctly enforces RBAC: no officer can perform an action outside their designated role |

### 16.3 User Acceptance

| Criterion | Condition |
|-----------|-----------|
| UA-01 | At least 80% of officers who complete a structured training session can complete their primary daily tasks without assistance after one day of practice |
| UA-02 | No officer reports a critical workflow action that is impossible or confusing to perform in the system |
| UA-03 | The Dashboard loads and is useful within 3 seconds on the target office hardware |
| UA-04 | The search screen returns results for a known Record ID in under 5 seconds |
| UA-05 | Officer satisfaction score (post-launch survey) averages ≥ 3.5 / 5.0 |

### 16.4 Quality Acceptance

| Criterion | Condition |
|-----------|-----------|
| QA-01 | All P1 (critical) and P2 (high) bugs are resolved before go-live |
| QA-02 | Code coverage for workflow state machine logic ≥ 90% |
| QA-03 | Accessibility: no WCAG 2.1 Level AA violations on any of the 17 screens |
| QA-04 | Performance: all page load targets in Section 8.1 are met under simulated concurrent load (10 simultaneous users) |
| QA-05 | Security: no critical or high severity findings in a pre-go-live security review |
| QA-06 | Backup and recovery procedures are documented, tested, and verified at least once before go-live |

---

## 17. Success Metrics & KPIs

### 17.1 Processing Efficiency

| KPI | Baseline (est.) | V1.0 Target | Measurement Method |
|-----|----------------|-------------|-------------------|
| Average Request-to-Validated time | ~2 days (manual) | ≤ 1 day | Timestamp: Submitted → Validated |
| Average Validated-to-Dispatched time | ~5 days (manual) | ≤ 3 days | Timestamp: Validated → Dispatched |
| Average Dispatched-to-Completed time | Unmeasured | Baseline established at launch | Timestamp: Dispatched → Completed |
| Bundle formation time (Officer 2) | ~90 min/Bundle | ≤ 30 min/Bundle | Timestamp: Bundle Created → Cover Letter Generated |
| Manifest formation time (Officer 4) | ~60 min/Manifest | ≤ 20 min/Manifest | Timestamp: Manifest Created → Dispatched |

### 17.2 Quality & Accuracy

| KPI | Target | Measurement Method |
|-----|--------|-------------------|
| Intake accuracy rate (no Officer 2 rework) | ≥ 97% | Requests reaching Bundle status without Officer 1 re-validation ÷ total Requests |
| Bundle formation accuracy rate (no Full Bundle Returns caused by formation errors) | ≥ 99% | Full Bundle Returns caused by formation error ÷ total Bundles dispatched |
| Correction rate (Surgical Isolation) | ≤ 5% of dispatched Requests | Surgical Isolation events ÷ total dispatched Requests |
| Document loss rate (unrecoverable) | 0 events after go-live | Count of unrecoverable document loss incidents |
| Disposition accuracy rate (no mis-recorded outcomes) | ≥ 99.5% | Dispositions requiring correction ÷ total recorded dispositions |

### 17.3 System Performance

| KPI | Target | Measurement Method |
|-----|--------|-------------------|
| System availability (business hours) | ≥ 99.5% | Total uptime ÷ total business hours, measured monthly |
| Mean time to recovery (MTTR) | ≤ 4 hours | Average time from incident detection to service restoration |
| Audit trail completeness | 100% | State changes with audit entries ÷ total state changes; verified by quarterly audit |
| Backup success rate | 100% | Successful backups ÷ scheduled backup runs, measured monthly |

### 17.4 User Adoption

| KPI | Target | Measurement Method |
|-----|--------|-------------------|
| Officer adoption rate | ≥ 90% using ARCHITAX as primary tool within 60 days | Officers performing ≥1 action per working day ÷ total active officers |
| Parallel record-keeping rate (paper register use) | 0% within 90 days | Supervisor survey + spot audit |
| User satisfaction score | ≥ 3.5 / 5.0 | Quarterly officer survey |
| Support ticket rate | ≤ 2 tickets / officer / month in months 2–6 | Help desk ticket log |

### 17.5 Management Value

| KPI | Target | Measurement Method |
|-----|--------|-------------------|
| Report generation time (Supervisor) | ≤ 30 minutes for any pre-built report | Supervisor self-report; system-measured report generation time |
| Management reporting self-sufficiency | 100% (no manual data collection from officers) | Survey: "Did you need to ask an officer for data to complete this report?" |
| Audit response time (external audit request) | ≤ 2 hours to produce requested audit trail export | Measured from audit request to export delivery |

---

## 18. Future Roadmap

### 18.1 Short Term (V1.1 — 3 to 6 months post-launch)

- **Officer mobile web optimization:** Improve tablet and mobile experience for Officers 5 and 6 who may need to record dispatch and disposition events in the field.
- **Scheduled report delivery:** Automatically deliver pre-configured reports to Supervisor and management email on a weekly or monthly cadence.
- **Taxpayer WhatsApp status notifications:** Extend the notification system to allow Officer 6 to send a WhatsApp message to a taxpayer confirming their Request's final status.
- **Bulk import for backlog migration:** Import historical Request records from existing spreadsheets to give the system a populated baseline at launch.
- **Enhanced duplicate detection:** NOP + NIK combination check with a warning (not a hard block) when an officer submits a Request that matches an existing open Request's NOP.

### 18.2 Medium Term (V2.0 — 6 to 18 months post-launch)

- **Multi-district federation:** A single ARCHITAX instance serves multiple district offices, each with isolated data and district-scoped user management, feeding a provincial-level aggregate reporting layer.
- **Advanced reporting:** Trend analysis over multi-year periods; comparison across districts (provincial view); configurable KPI dashboard for management.
- **Automated scan quality assessment:** Machine-assisted legibility scoring for uploaded documents; automatic flag for re-scan if a quality threshold is not met.
- **QR code and barcode integration:** Print QR codes on Bundle and Manifest cover letters that link directly to the digital record; use barcodes on physical documents to streamline Officer 3's linking process.
- **SLA monitoring and escalation:** Configurable SLA thresholds per workflow stage; automated escalation notifications to Supervisor when SLAs are breached.
- **Enhanced citizen portal foundation:** A read-only status lookup page (no ARCHITAX login required) that allows taxpayers to check their Request status using their Request Number and NIK.

### 18.3 Long Term (V3.0 — 18 to 36 months post-launch)

- **Central Office integration:** Structured digital data exchange with the Central Office's system, eliminating manual disposition recording by Officer 6 and replacing it with an automated status feed.
- **Fully digital taxpayer submission channel:** An online application form allowing taxpayers to submit PBB-P2 service requests digitally (subject to regulatory authorization), reducing the physical document dependency for eligible service types.
- **AI-assisted intake validation:** Machine learning-based NOP and address verification against national cadastral data sources; auto-classification of service type from scanned application documents.
- **National compliance reporting:** Integration with national SPBE reporting requirements; automated production of standardized digital government service compliance reports.

### 18.4 Enterprise Vision

ARCHITAX's long-term vision is to become the standard government property tax workflow management platform across Indonesian district and city governments — a SaaS-model platform where each subscribing district operates its own isolated deployment under a centralized platform umbrella, enabling national-level analytics while preserving district-level data sovereignty.

The workflow engine's three architectural principles (Hierarchical Containment, Legal Validity Upon Dispatch, Minimum Necessary Disruption) and the deterministic state machine defined in `WORKFLOW.md` are designed to remain stable as the foundation for this enterprise vision without structural redesign.

---

## 19. Glossary

| Term | Definition |
|------|------------|
| **Request** | The atomic unit of the ARCHITAX workflow. A single taxpayer application for one of six PBB-P2 service types (PM, FMU, FMR, COR, NTO, REA). A Request has a permanent, unique Request Number and progresses through up to twelve lifecycle states. See `WORKFLOW.md §9`. |
| **Bundle** | The mid-level aggregation container. Officer 2 groups one or more validated Requests into a Bundle, which is assigned a permanent, unique Bundle Number. A Bundle becomes legally valid at the moment of physical dispatch. See `WORKFLOW.md §7`. |
| **Manifest** | The top-level dispatch container. Officer 4 groups one or more dispatch-ready Bundles into a Manifest, assigned a permanent, unique Manifest Number. A Manifest is physically dispatched to the Central Office and is never recalled. See `WORKFLOW.md §8`. |
| **Surgical Isolation** | The default post-dispatch exception mechanism. One or more specific defective Requests are returned from the Central Office while their parent Bundle and Manifest remain legally valid and unaffected. Affects 90–95% of exception cases. See `WORKFLOW.md §13`. |
| **Full Bundle Return** | The escalation post-dispatch exception mechanism. An entire Bundle is returned and permanently invalidated (Closed/Superseded) when the Bundle's formation itself is structurally defective. The parent Manifest remains valid. Affects 5–10% of exception cases. See `WORKFLOW.md §14`. |
| **PBB-P2** | *Pajak Bumi dan Bangunan Perdesaan dan Perkotaan* — Indonesia's rural and urban land and building property tax, administered at the district/city level since 2012. |
| **NOP** | *Nomor Objek Pajak* — the unique tax object identifier assigned to every property. The primary identifier for a property in the PBB-P2 system. |
| **NIK** | *Nomor Induk Kependudukan* — Indonesia's national citizen identity number. Used to identify the taxpayer of record for a property. |
| **Central Office** | The external government authority that receives dispatched Manifests, adjudicates submitted Requests, and returns defective items. The Central Office's internal adjudication logic is outside ARCHITAX's governance. |
| **Bundle Cover Letter** | The system-generated legal document accompanying a Bundle during dispatch. Contains exactly three elements: Bundle Number, Bundle Summary, and Request List. |
| **Manifest Cover Letter** | The system-generated legal document accompanying a Manifest during dispatch. Contains exactly four elements: Manifest Number, Manifest Summary, Bundle List, and Signature Section. |
| **Partial Mutation Worksheet** | A supplementary document generated by Officer 2 only when a Bundle contains at least one Partial Mutation (PM) Request. Details the area and value split calculations for the affected tax objects. |
| **Audit Trail** | The immutable, append-only log of every significant state change, user action, and system event in ARCHITAX. Required for legal accountability and operational compliance. See `WORKFLOW.md §19`. |
| **Digital Archive** | The collection of scanned physical documents stored and linked to their originating Request, Bundle, and Manifest records. Maintained by Officer 3. Serves as the disaster recovery source for physical document loss. |
| **Hierarchical Containment** | One of the three core workflow principles. A Request is contained within a Bundle; a Bundle is contained within a Manifest. Containment is one-directional and cannot be reversed after dispatch. |
| **Legal Validity Upon Dispatch** | One of the three core workflow principles. Both a Bundle and its parent Manifest become permanently legally valid the moment Officer 5 physically dispatches them. This validity cannot be retroactively revoked. |
| **Minimum Necessary Disruption** | One of the three core workflow principles. When a defect is discovered, ARCHITAX resolves it at the smallest organizational unit capable of containing it — never escalating to a higher container than necessary. |
| **RBAC** | Role-Based Access Control. The permission model governing which screens, data, and actions each officer role can access. Enforced server-side on every request. |
| **Officer 1–6** | The six functional officer roles defined in `WORKFLOW.md §6`, each accountable for a specific workflow stage. A single staff member may hold multiple roles; the role performed is what is recorded in the audit trail. |
| **Hybrid Process** | ARCHITAX's operational context: a process in which physical documents are the primary legal artifact and digital records run in parallel as an administrative and archival layer. Neither fully replaces the other at this stage. |
| **Service Type** | One of six categories of PBB-P2 application: Partial Mutation (PM), Full Mutation Update (FMU), Full Mutation Regular (FMR), Correction (COR), New Tax Object (NTO), Reactivation (REA). Each follows the same workflow; differences are limited to intake validation and the conditional PM Worksheet. |
| **Disposition** | The final Central Office decision on a dispatched Request: either Completed (favorable) or Returned (defective). Recorded by Officer 6 in ARCHITAX upon receipt of the Central Office's decision. |
| **State Machine** | The deterministic model defining all permitted lifecycle states and transitions for Requests, Bundles, and Manifests. See `WORKFLOW.md §10` for complete state transition tables. |
| **SPBE** | *Sistem Pemerintahan Berbasis Elektronik* — Indonesia's Electronic-Based Government System framework, which mandates digital transformation across government service delivery. |

---

*End of PRODUCT_REQUIREMENTS.md — ARCHITAX Property Tax (PBB-P2) Workflow Management System*

*Companion documents:*
- *`WORKFLOW.md` — Functional Specification (governs all business process and rule decisions)*
- *`SCREEN_SPECIFICATION.md` — UI Screen Specification (governs all screen-level design decisions)*
- *`UI_DESIGN_SYSTEM.md` — Design System Reference (governs all visual and component decisions)*
