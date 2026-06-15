# Property Tax Service (PBB-P2) Role-Based Access Control (RBAC)

This document defines the strict permission boundaries for all actors within the ARCHITAX system. It enforces the Principle of Least Privilege and Role Separation, ensuring that officers can only interact with data and execute actions relevant to their specific stage in the workflow.

## 1. Core Access Principles

- **Stage-Bound Access**: Operational Officers (1–5) can only view and edit requests, bundles, or manifests that are currently in their designated workflow stage, or issues explicitly assigned to them.
- **No Hard Deletes**: The "Delete" action is universally disabled. Records can only be transitioned to a `CANCELLED / VOIDED` terminal state by a Supervisor, preserving the immutable audit trail.
- **Data Masking**: Sensitive Personally Identifiable Information (PII), such as the full 16-digit NIK, is masked (e.g., `3201xxxxxxxxxx99`) for roles that do not require it for verification (e.g., Logistics & Delivery Officer).
- **Supervisor Override**: Critical exceptions (reassignment, reopening locked batches, cancellation) are strictly reserved for the Supervisor / Escalation Manager and are heavily logged.

## 2. Entity-Level Permission Matrix

This matrix defines the high-level permissions each role has over the core system entities.

**Legend:**

- ✅ = Full Permission
- 👁️ = Read-Only / View Only
- ⚠️ = Conditional Permission (Requires specific state or Supervisor approval)
- ❌ = No Access / Action Blocked

| Action / Entity              | Officer 1 (Intake) | Officer 2 (Bundling) | Officer 3 (Digitization) | Officer 4 (Manifest) | Officer 5 (Logistics) | Officer 6 (Monitoring) | Supervisor |
| :--------------------------- | :----------------: | :------------------: | :----------------------: | :------------------: | :-------------------: | :--------------------: | :--------: |
| **REQUEST**                  |                    |                      |                          |                      |                       |                        |            |
| View Request (Current Stage) |         ✅         |          ✅          |            ✅            |          ✅          |          ✅           |           ✅           |  👁️ (All)  |
| Edit Request Metadata        |      ✅ (REC)      |          ❌          |            ❌            |          ❌          |          ❌           |           ❌           | ⚠️ (Over)  |
| Raise Issue (Any Stage)      |         ✅         |          ✅          |            ✅            |          ✅          |          ✅           |           ✅           |     ✅     |
| Cancel / Void Request        |         ❌         |          ❌          |            ❌            |          ❌          |          ❌           |           ❌           |     ✅     |
| **BUNDLE**                   |                    |                      |                          |                      |                       |                        |            |
| View Bundle Details          |         👁️         |          ✅          |            ✅            |          ✅          |          ✅           |           👁️           |  👁️ (All)  |
| Create & Add to Bundle       |         ❌         |          ✅          |            ❌            |          ❌          |          ❌           |           ❌           |     ❌     |
| Lock Bundle                  |         ❌         |          ✅          |            ❌            |          ❌          |          ❌           |           ❌           |     ❌     |
| Reopen Locked Bundle         |         ❌         |          ❌          |            ❌            |          ❌          |          ❌           |           ❌           |     ✅     |
| **MANIFEST**                 |                    |                      |                          |                      |                       |                        |            |
| View Manifest Details        |         ❌         |          ❌          |            ❌            |          ✅          |          ✅           |           👁️           |  👁️ (All)  |
| Create & Add to Manifest     |         ❌         |          ❌          |            ❌            |          ✅          |          ❌           |           ❌           |     ❌     |
| Lock Manifest                |         ❌         |          ❌          |            ❌            |          ✅          |          ❌           |           ❌           |     ❌     |
| Flag as EXCLUDED             |         ❌         |          ❌          |            ❌            |          ❌          |          ✅           |           ❌           |     ✅     |
| **ISSUE MANAGEMENT**         |                    |                      |                          |                      |                       |                        |            |
| Raise New Issue              |         ✅         |          ✅          |            ✅            |          ✅          |          ✅           |           ✅           |     ✅     |
| Update to IN_PROGRESS        |         ✅         |          ✅          |            ✅            |          ✅          |          ✅           |           ✅           |     ✅     |
| Update to RESOLVED           |         ✅         |          ✅          |            ✅            |          ✅          |          ✅           |           ✅           |     ✅     |
| Verify & Close Issue         |         ❌         |          ❌          |            ❌            |          ❌          |          ❌           |           ✅           |     ✅     |
| **SYSTEM & AUDIT**           |                    |                      |                          |                      |                       |                        |            |
| Reassign Task / Issue        |         ❌         |          ❌          |            ❌            |          ❌          |          ❌           |           ❌           |     ✅     |
| View Full Audit Log          |         ❌         |          ❌          |            ❌            |          ❌          |          ❌           |           👁️           |     ✅     |
| Export Audit Log / Reports   |         ❌         |          ❌          |            ❌            |          ❌          |          ❌           |           ✅           |     ✅     |
| Hard Delete Record           |         ❌         |          ❌          |            ❌            |          ❌          |          ❌           |           ❌           |     ❌     |

## 3. Stage-Specific Action Permissions (The "Happy Path")

| Workflow Stage               | Primary Owner | Allowed Actions                                             | Blocked Actions                                              |
| :--------------------------- | :------------ | :---------------------------------------------------------- | :----------------------------------------------------------- |
| **1. RECEIVED**              | Officer 1     | Register metadata, Print Receipt, Raise Issues.             | Edit after moving to BUNDLED; Delete.                        |
| **2. BUNDLED**               | Officer 2     | Select requests, Generate Bundle ID, Lock, Raise Issues.    | Edit individual request metadata; Unlock (needs Supervisor). |
| **3. DIGITALLY_ARCHIVED**    | Officer 3     | Upload/Replace scans, Verify, Raise Issues, Advance.        | Alter bundle composition; Advance if ON_HOLD.                |
| **4. MANIFESTED**            | Officer 4     | Select bundles, Generate Manifest ID, Lock, Raise Issues.   | Edit bundle contents; Unlock (needs Supervisor).             |
| **5. DISPATCHED**            | Officer 5     | Record dispatch details (courier, time), Raise Issues.      | Alter manifest contents.                                     |
| **6. DELIVERY_CONFIRMATION** | Officer 5     | Upload signed Manifest (PoD), Link to record, Raise Issues. | Advance if PoD is missing/invalid.                           |
| **7. UNDER_MONITORING**      | Officer 6     | Record completion data, Verify/Close issues, Raise Issues.  | Edit historical data.                                        |
| **8. COMPLETED**             | System / O6   | View Only, Export Audit Log.                                | Modifications, edits, new issues.                            |

## 4. Data Visibility & Masking Rules

| Data Field            | O1  | O2  | O3  | O4  | O5  | O6  | Supervisor |
| :-------------------- | :-: | :-: | :-: | :-: | :-: | :-: | :--------: |
| Full Name             | ✅  | ✅  | ✅  | ✅  | ✅  | ✅  |     ✅     |
| Full NIK (16 digits)  | ✅  | ✅  | ✅  | ✅  | ⚠️  | ✅  |     ✅     |
| Phone Number          | ✅  | ⚠️  | ⚠️  | ⚠️  | ⚠️  | ✅  |     ✅     |
| Property Address/SPPT | ✅  | ✅  | ✅  | ✅  | ✅  | ✅  |     ✅     |
| Uploaded Documents    | ✅  | ✅  | ✅  | ✅  | ✅  | ✅  |     ✅     |
| Internal Audit Notes  | ❌  | ❌  | ❌  | ❌  | ❌  | ✅  |     ✅     |

_Note: If Officer 5 (Logistics) encounters a delivery issue requiring citizen contact, they must request the unmasked phone number through an official system prompt, which logs the access request for audit purposes._

## 5. Special Authorization Rules (Supervisor Exclusives)

Every execution of these actions triggers a mandatory `mandatory_note` in the Audit Log:

- **Bundle / Manifest Reopening**: Supervisor reviews the issue, approves the unlock, and the system logs `BUNDLE_REOPENED` or `MANIFEST_UNLOCKED`.
- **Task / Issue Reassignment**: Supervisor reassigns the entity to a different officer. Regular officers cannot reassign their own tasks.
- **Terminal State Authorization (`CANCELLED / VOIDED`)**: Supervisor executes the transition, inputs the mandatory cancellation reason, and the system seals the record and notifies the citizen via WhatsApp.
- **Manual System Override**: Supervisor can force a state transition, bypassing standard validation rules (heavily audited).

## 6. System Administrator Permissions (Technical Role)

- **User Management**: Create, disable, or modify Officer and Supervisor accounts and role assignments.
- **System Configuration**: Define SLA time limits, configure WhatsApp API credentials, and manage Issue Type dropdowns.
- **Database Maintenance**: Execute backups and monitor system health.
- **Restriction**: System Administrators cannot view unmasked citizen PII or alter operational workflow data unless explicitly granted temporary, audited access by the Head of Office for forensic investigation.
