# Property Tax Service (PBB-P2) Operational & Business Rules

This document defines the strict business logic, validation constraints, security protocols, and operational rules that govern the ARCHITAX system. These rules ensure data integrity, regulatory compliance, and seamless execution of the workflow defined in 01_WORKFLOW.md.

## 1. Intake & Data Validation Rules

To prevent garbage-in-garbage-out (GIGO) scenarios, the system enforces strict validation at the `RECEIVED` stage before a request can progress.

### 1.1 Mandatory Citizen Data

The system will block submission if the following fields are empty or invalid:

- **Full Name:** Must match the physical KTP.
- **NIK (National ID):** Must be exactly 16 numeric digits.
- **Active Phone Number:** Must be a valid mobile format (required for WhatsApp notifications).
- **Property Address / SPPT Number:** Must be provided to identify the tax object.
- **Service Type:** Must be explicitly selected from the predefined system list.

### 1.2 Mandatory Physical Documents

Officer 1 must verify the physical presence of:

- Completed and signed PBB-P2 Application Form.
- Copy of Applicant’s KTP.
- Latest PBB Payment Receipt (SPPT) or relevant property proof.
- **Rule:** If any are missing, Officer 1 must immediately raise a `MISSING_DOCUMENT` issue, placing the request `ON_HOLD`.

---

## 2. Digital Asset & File Management Rules

To ensure the "digital twin" is legally and operationally viable, strict rules govern the digitization process at the `DIGITALLY_ARCHIVED` stage.

### 2.1 File Specifications

- **Allowed Formats:** .pdf, .jpg, .jpeg, .png.
- **Maximum File Size:** 5 MB per individual file; 50 MB maximum per Bundle.
- **Naming Convention:** System auto-generates filenames as: `[RequestID]_[DocumentType]_[Timestamp].[ext]` (e.g., `REQ-2026-001_KTP_202601151030.pdf`).

### 2.2 Quality Standards

- **Resolution:** Minimum 200 DPI.
- **Legibility:** All text, stamps, and signatures must be clearly readable.
- **Orientation:** Documents must be upright (not rotated 90 or 180 degrees).
- **Rule:** If a scan fails these standards, Officer 3 must raise an `INVALID_SCAN` issue. The system will not allow progression to `MANIFESTED` until a compliant file replaces the invalid one.

---

## 3. Batch Processing Constraints (Bundles & Manifests)

To maintain physical and digital integrity during high-volume processing, the system enforces strict boundaries on batch creation.

### 3.1 Bundle Constraints (Officer 2)

- **Capacity:** Minimum 1, Maximum 50 requests per Bundle.
- **Homogeneity:** All requests in a single Bundle should ideally be of the same Service Type (system warning if mixed, but allowed).
- **Pre-Lock Validation:** The system will block the "Lock Bundle" action if any request within the selection is currently in an `ON_HOLD` state.

### 3.2 Manifest Constraints (Officer 4)

- **Capacity:** Minimum 1, Maximum 20 Bundles per Manifest.
- **Pre-Lock Validation:** The system will block the "Lock Manifest" action if any included Bundle is not in the `DIGITALLY_ARCHIVED` state.

### 3.3 Post-Lock Amendment Rules

Once a Bundle or Manifest is locked, it is immutable by standard Officers.

- **Exclusion:** If a bundle must be removed after Manifest locking (e.g., due to a `LOST_FILE`), Officer 5 must flag it as `EXCLUDED_FROM_DISPATCH`.
- **Physical Annotation:** Officer 5 must manually cross out the excluded bundle on the printed Manifest Cover Letter, initial it, and scan/upload this annotated copy as part of the Proof of Delivery (PoD).

---

## 4. Role-Based Access Control (RBAC) & Security Rules

ARCHITAX enforces the Principle of Least Privilege to guarantee Role Separation and data privacy.

### 4.1 Data Visibility Rules

- **Stage-Bound Access:** Officers can only view and edit requests currently in their designated stage or issues explicitly assigned to them.
- **Data Masking:** Sensitive citizen data (e.g., full NIK) is masked (e.g., `3201xxxxxxxxxx99`) for Officers who do not have a direct operational need to view it (e.g., Logistics Officer).
- **Supervisor Override:** Supervisors have read-only access to all stages but cannot directly edit citizen data; they can only authorize exceptions or reassignments.

### 4.2 Session & System Security

- **Auto-Logout:** User sessions automatically terminate after 15 minutes of inactivity to prevent unauthorized access at shared workstations.
- **No Hard Deletes:** No user, including the Supervisor, can permanently delete a database record. Records can only be transitioned to the `CANCELLED` / `VOIDED` terminal state, which preserves the audit trail.

---

## 5. Audit & Immutability Rules

To satisfy the Auditability principle, the system treats the audit log as a legal, append-only ledger.

### 5.1 Immutable Logging

- Every state transition, issue creation, file upload, and note addition generates an immutable Audit Log Event with a UUID, ISO-8601 timestamp, User ID, Role, IP Address, and Device ID.
- Past states cannot be overwritten. If a mistake is made, a new corrective action (e.g., uploading `scan_v2.pdf`) is logged, preserving `scan_v1.pdf` in the history.

### 5.2 Sealed Records

- Once a request reaches `COMPLETED` or `CANCELLED`, the record is sealed.
- **Rule:** No new issues can be raised, no metadata can be altered, and no files can be added or removed. Only "View" and "Export Audit Log" actions are permitted.

### 5.3 Mandatory Note Enforcement

The system will disable the "Submit" or "Resolve" button unless a text note is provided for:

- Raising any `ON_HOLD` issue.
- Marking an issue as `RESOLVED`.
- Reassigning a task.
- Authorizing a Bundle/Manifest reopening.
- Executing a `CANCELLED` / `VOIDED` transition.

---

## 6. Exception Handling & Escalation Matrix

| Issue Type             | Trigger Stage      | Primary Resolver | Max Resolution Time | Escalation Path                               |
| :--------------------- | :----------------- | :--------------- | :------------------ | :-------------------------------------------- |
| `MISSING_DOCUMENT`     | RECEIVED           | Officer 1        | 3 Days              | Officer 1 → Supervisor                        |
| `DATA_MISMATCH`        | RECEIVED           | Officer 1        | 1 Day               | Officer 1 → Supervisor                        |
| `INVALID_SCAN`         | DIGITALLY_ARCHIVED | Officer 3        | 1 Day               | Officer 3 → Supervisor                        |
| `WRONG_BUNDLE`         | BUNDLED            | Officer 2        | 1 Day               | Officer 2 → Supervisor (Requires Reopen Auth) |
| `MISSING_BUNDLE`       | MANIFESTED         | Officer 4        | 1 Day               | Officer 4 → Supervisor                        |
| `MISSING_CONFIRMATION` | DISPATCHED         | Officer 5        | 3 Days              | Officer 5 → Supervisor                        |
| `LOST_FILE`            | Any Stage          | Supervisor       | 5 Days              | Supervisor → Head of Office                   |
| `SLA_EXCEEDED`         | Any Stage          | Supervisor       | 1 Day               | Supervisor → Head of Office                   |

> **Note:** The SLA timer for the main workflow automatically pauses while a request is in the `ON_HOLD` state, resuming only when the issue is marked `CLOSED`.

---

## 7. Physical Document Lifecycle Rules

While ARCHITAX manages the digital workflow, strict rules govern the physical paper trail.

### 7.1 Post-Completion Disposition

Once a request reaches the `COMPLETED` stage, the physical file is no longer needed for daily processing.

- **Local Holding (Days 1–30):** Physical bundles are moved from active processing trays to a secure, access-controlled "Local Holding" storage area within the origin office.
- **Central Physical Archive (Days 31–365):** Once a month, Officer 5 batches all `COMPLETED` physical files and formally transfers them to the Central Government Physical Archive facility, logging the transfer date in ARCHITAX.
- **Secure Destruction (Year 5+):** In accordance with national property tax data retention laws, physical documents reaching the end of their legal retention period are securely shredded by the Central Archive.
- **Critical Rule:** The digital twin in ARCHITAX remains permanently sealed and immutable for historical auditing, even after the physical paper is destroyed.

---

## 8. Notification & Communication Rules

### 8.1 Internal Handoff Notifications

- **Trigger:** Successful stage transition (e.g., `BUNDLED` → `DIGITALLY_ARCHIVED`).
- **Action:** System sends a dashboard alert and/or email to the designated Owner of the next stage.

### 8.2 External Citizen Notifications (WhatsApp)

- **Request Registered:** Sent immediately upon `RECEIVED`.
- **Action Required:** Sent immediately when an `ON_HOLD` issue (e.g., `MISSING_DOCUMENT`) is raised, instructing the citizen on what to bring to the Frontdesk.
- **Dispatched:** Sent when status changes to `DISPATCHED`, providing an estimated completion date.
- **Completed:** Sent when status changes to `COMPLETED`, instructing the citizen to collect the result at the Frontdesk.
- **Cancelled:** Sent if the Supervisor executes a `CANCELLED` / `VOIDED` transition, including the high-level reason.
