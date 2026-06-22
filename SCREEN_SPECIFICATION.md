# ARCHITAX — Screen Specification

**Document Classification:** UI Screen Specification
**System:** ARCHITAX — Property Tax (PBB-P2) Archive & Workflow Management System
**Design System Reference:** ARCHITAX UI Design System v2.0
**Document Status:** Final — Production Reference
**Audience:** Frontend Engineers, QA Engineers, UX Designers, Product Owners

> All visual values in this document (colors, spacing, radius, shadow, typography) are specified by token name only. Token definitions live in the Design System (`UI_DESIGN_SYSTEM.md`). Tokens are never overridden ad-hoc at the screen level.

---

## How to Read This Document

Each screen specification follows a consistent structure:

1. **Purpose** — why this screen exists and who uses it
2. **Layout** — shell region usage, column structure, primary surface arrangement
3. **Component Hierarchy** — top-to-bottom, left-to-right breakdown of every rendered element
4. **Interactions** — user actions and system responses
5. **Data Requirements** — fields, sources, and formats
6. **Empty State** — what the screen shows when there is no data
7. **Loading State** — skeleton/placeholder strategy during async fetch
8. **Error State** — how failure is surfaced and resolved
9. **Permissions** — which officer roles can access or act on this screen
10. **Responsive Behavior** — layout changes across breakpoints
11. **Accessibility** — keyboard, screen reader, and contrast notes
12. **Keyboard Shortcuts** — power-user hotkeys scoped to this screen

---

## Screen Index

| # | Screen | Primary Role(s) |
|---|--------|----------------|
| 01 | Dashboard | All |
| 02 | Request Detail | Officer 1, 2, 6 |
| 03 | Create Request | Officer 1 |
| 04 | Bundle Detail | Officer 2, 3, 4, 5 |
| 05 | Manifest Detail | Officer 4, 5 |
| 06 | Correction Detail | Officer 6 |
| 07 | Archive | Officer 3 |
| 08 | Digital Archive Viewer | Officer 3, All (read) |
| 09 | Search | All |
| 10 | Officer Management | Admin |
| 11 | User Management | Admin |
| 12 | Audit Log | Admin, Supervisor |
| 13 | Reports | Supervisor, Admin |
| 14 | Settings | Admin |
| 15 | Notifications | All |
| 16 | Profile | All |
| 17 | Login | — |

---

## 01 — Dashboard

### Purpose

The entry point of the application. Provides a high-density operational summary of the current state of the workflow: pending intake, in-progress bundles, dispatched manifests, and recent activity. Designed for the 500th use — an experienced officer should orient and navigate to their primary task within 3 seconds of arrival.

### Layout

```
┌─────────────────────────────────────────────────────────────┐
│ Sidebar (264px)   │ Top Navigation (76px, sticky)           │
│                   ├─────────────────────────────────────────┤
│                   │ Page Body (scrollable)                  │
│                   │  ┌──── KPI Row (4 stat cards) ────────┐ │
│                   │  ├──── Tasks Due ──── Favorites ───────┤ │
│                   │  ├──── Recent Activity ── Quick Links ─┤ │
│                   │  └─────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

Page body uses 32px top padding, 32px horizontal padding (desktop). Content max-width 1200px, left-aligned within the content region.

### Component Hierarchy

**Top Navigation**
- Page title: "Home" — Display style (30px/700, `color-text-primary`)
- Global search field (pill, 360px → 480px on focus)
- Notification bell with unread badge count
- User avatar (36px, with dropdown menu)

**KPI Row — 4 Stat Cards (equal-width grid, 24px gutter)**

Each stat card is a Standard Card with:
- Colored Icon Tile (32px icon, 48px tile, one of: `color-primary`, `color-secondary`, `color-success`, `color-warning`)
- Metric label: Small/Meta style, `color-text-secondary`
- Count: 30px/700, `color-text-primary`
- Delta indicator: 12px/500 with directional arrow icon; green (`color-success`) for positive, red (`color-danger`) for negative, compared to previous 30 days
- Cards: "Pending Intake", "Active Bundles", "Dispatched Manifests", "Completed This Month"

**Tasks Due Card (left column, ~60% width)**
- Card header: "Tasks Due" (H2) + count badge (danger-colored if overdue items exist) + "All Tasks" text link (right-aligned) + overflow ⋯ button
- Header divider
- Simplified List Rows (56px each, comfortable density):
  - Leading status badge (color-coded by record type)
  - Primary text: Record ID + truncated title (Body Emphasis)
  - Meta: Due date (Small, `color-text-secondary`; red if past due)
  - Tag: Service type pill (category color)
  - Avatar group: assigned officers
- Pagination: "Showing 1–10 of N" + Previous/Next ghost buttons

**Favorites Card (right column, ~40% width)**
- Card header: "Favorites" (H2) + edit icon button (right)
- Header divider
- Interactive Tile grid (2×2), each tile:
  - Colored Icon Tile (64px, rounded-square, 12px radius)
  - Label: 13px/600
  - Subtitle: 12px/400, `color-text-secondary`
  - Hover: Level 1 → Level 2 shadow, 150ms ease-out

**Recent Activity Card (left column, full width below Tasks Due)**
- Card header: "Recent Activity" (H2) + "View All" text link
- Header divider
- Simplified List Rows (48px each):
  - Leading avatar (32px, officer who acted)
  - Primary text: action description with linked record ID
  - Timestamp: Small, `color-text-secondary`, right-aligned

**Message / Quick Links Card (right column)**
- Card header: "Quick Links" (H2) + edit icon
- Header divider
- List of navigable shortcuts — each row: 16px icon + label + trailing chevron

### Interactions

- Clicking any row in Tasks Due opens the relevant record detail screen
- Clicking a Favorites tile navigates to that feature section
- Clicking the notification bell opens the Notification Panel (Section 15)
- KPI cards are non-interactive (static)
- "All Tasks" link navigates to Search screen pre-filtered by "assigned to me, open"
- Row hover: `color-hover` background, 100ms ease; cursor: pointer

### Data Requirements

| Field | Source | Format |
|-------|--------|--------|
| Pending Intake count | Request aggregate | Integer |
| Active Bundles count | Bundle aggregate | Integer |
| Dispatched Manifests count | Manifest aggregate | Integer |
| Completed This Month | Request aggregate (current calendar month) | Integer |
| Tasks Due rows | Request + Bundle records assigned to user, ordered by due_date ASC | List |
| Recent Activity | Audit log, last 50 events | List |
| Favorites | User preference store | List of shortcuts |

### Empty State

If no tasks are due, the Tasks Due card renders the Empty State component:
- 48px outline inbox icon, `color-text-secondary`
- Heading: "No tasks pending" (16px/600)
- Body: "All caught up. New intake requests will appear here." (14px/400, `color-text-secondary`)
- No primary action button (purely informational)

### Loading State

On first load, all four KPI cards render as Skeleton Loaders (shimmer, full card silhouette). List rows in Tasks Due and Recent Activity render as 3 skeleton rows each (text blocks at 60% and 40% width, icon circle). Tiles in Favorites render as skeleton squares.

### Error State

If the dashboard data fetch fails, an Inline Alert (warning variant) appears at the top of the page body:
- Icon: outline alert-triangle
- Text: "Dashboard data could not be loaded. Try refreshing the page."
- Action button: "Retry" (ghost, small)
- KPI cards show `—` instead of counts; lists show zero rows with error messaging inline.

### Permissions

All authenticated users see the Dashboard. KPI cards show counts scoped to records the user's role can access. Tasks Due shows only records assigned to or owned by the current user unless the user is a Supervisor or Admin (who see all).

### Responsive Behavior

- Desktop (≥1440px): KPI row = 4 columns; Tasks Due + Favorites = 60/40 split; Recent Activity + Quick Links = 60/40
- Laptop (1024–1439px): Same structure, 32px → 24px card padding
- Tablet (768–1023px): KPI row = 2×2 grid; all column pairs collapse to full width stacked
- Mobile (<768px): Single column; KPI row = 2×2 grid with compact cards; list rows remain full width; sidebar becomes off-canvas drawer

### Accessibility

- KPI cards: `role="region"` with descriptive `aria-label` per card
- Task list uses `role="list"` / `role="listitem"`
- All text link and icon button controls have explicit `aria-label`
- Delta indicators include screen-reader text: e.g., "increased by 12 from last month"

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `G H` | Navigate to Home (Dashboard) |
| `N` | Open Create Request modal |
| `/` | Focus global search |
| `?` | Open keyboard shortcut help modal |

---

## 02 — Request Detail

### Purpose

The canonical view of a single Request — the atomic unit of the ARCHITAX workflow. Officers use this screen to review intake data, track workflow progression, view audit history, and initiate actions (validate, assign to bundle, flag for correction, record disposition).

### Layout

```
┌──────────────────────────────────────────────────────────────┐
│ Sidebar │ Top Nav: breadcrumb + page title + action buttons  │
│         ├────────────────────────────┬─────────────────────  │
│         │ Left: Main Detail (70%)   │ Right: Side Panel(30%)│
│         │  ─ Header Info Band       │  ─ Status & Timeline  │
│         │  ─ Tab Bar                │  ─ Assigned Officers   │
│         │  ─ Tab Content            │  ─ Related Records     │
│         │  ─ Audit Log Section      │  ─ Actions            │
│         └────────────────────────────┴──────────────────────  │
└──────────────────────────────────────────────────────────────┘
```

Left column max-width: 720px. Right side panel: 320px, fixed, sticky within the viewport (scrolls independently). 32px gutter between columns.

### Component Hierarchy

**Top Navigation**
- Breadcrumb: "Requests › [Request ID]" — 13px, `color-text-secondary` with final segment in `color-text-primary`/600
- Page title: Request ID in Display style (e.g., "REQ-2024-000012")
- Action button cluster (right-aligned):
  - "Validate" — Primary button (visible to Officer 1 when status = Draft)
  - "Assign to Bundle" — Secondary button (visible to Officer 2 when status = Validated)
  - "Flag for Correction" — Ghost button with `color-danger` text
  - Overflow ⋯ menu: Print, Export PDF, Copy ID, Archive

**Info Band (below Nav, above tabs)**
Full-width band inside the content card, 3-column grid:
- Column 1: Service Type Tag (large badge, category color) + NOP (tabular numerals, 16px/600)
- Column 2: Taxpayer name (16px/600) + NIK (13px, `color-text-secondary`)
- Column 3: Status Badge (current status, semantic color) + Created date + Created by officer avatar

**Tab Bar (underline style)**
Tabs: "Primary Data" | "SPPT Data" | "New Data" | "Documents" | "Audit Log"

**Tab: Primary Data**
Two-column form grid (read-only by default, editable for Officer 1 when status = Draft):
- Service Number (text, tabular)
- Service Type (tag/badge display)
- Tax Object Number / NOP (text, tabular)
- District (text)
- Submission Date (date)

**Tab: SPPT Data (Existing)**
Two-column label/value grid:
- Owner Name
- Owner Address
- Block / RT / RW
- Object Address / Block / RT / RW
- Land Area (numeric, m²)
- Building Area (numeric, m²)

**Tab: New Data**
Two-column label/value grid (for PM service type: repeatable section with "Record N of N" header and Previous/Next navigation):
- Owner Name
- Owner Address
- Block / RT / RW
- Object Address / Block / RT / RW
- Land Area (numeric, m²)
- Building Area (numeric, m²)
- Proof of Ownership (document reference link)

**Tab: Documents**
File list table (4 columns): File name | Type | Uploaded by | Date | Actions (view, download)

**Tab: Audit Log**
Chronological timeline (newest first):
- Actor avatar + name
- Action label (status change, field edit, assignment)
- Timestamp
- Optional diff block (before → after for field changes)

**Right Side Panel**

Status & Timeline Card:
- Current status badge (large, semantic color)
- Progress stepper (5 steps: Draft → Validated → In Bundle → Dispatched → Completed/Returned)
- Each step: icon (checkmark/circle/clock), label, officer name, timestamp

Assigned Officers Card:
- Officer 1 (Intake): avatar + name
- Officer 2 (Bundle): avatar + name (or "Unassigned" in `color-text-secondary`)
- Officer 6 (Disposition): avatar + name

Related Records Card:
- Parent Bundle: ID link + status badge
- Parent Manifest: ID link + status badge
- Correction record (if any): ID link

Actions Card:
- Contextual buttons stacked vertically based on status + user role
- E.g., for Officer 6: "Mark Completed" (primary), "Mark Pending" (secondary), "Record Return" (danger)

### Interactions

- Tab switching: instant (no page load), content cross-fades 150ms
- "Validate" button: opens confirmation modal (single field: notes, optional); on confirm, status changes to Validated, toast appears "Request validated"
- "Assign to Bundle" button: opens a drawer with a bundle selector (searchable dropdown + create new bundle option)
- "Flag for Correction" button: opens Correction Detail creation flow (modal → new Correction record)
- Overflow menu: standard dropdown, 150ms open animation
- In PM service type, New Data tab shows Previous/Next chevrons to cycle between split-object records
- Audit Log entries are expandable (accordion) to reveal full field-change diff

### Data Requirements

| Field | Source | Notes |
|-------|--------|-------|
| Request ID | requests.id | Formatted: REQ-{YEAR}-{6-digit seq} |
| Service Type | requests.service_type | One of: PM, FMU, FMR, COR, NTO, REA |
| NOP | requests.nop | Tabular numerals |
| NIK | requests.nik | Masked: show last 4 digits only for non-Officer-1 users |
| Primary / SPPT / New Data | requests.primary_data, sppt_data, new_data | JSON groups |
| Status | requests.status | Enum: Draft, Validated, InBundle, Dispatched, Completed, Pending, Returned |
| Audit entries | audit_log where entity_id = request.id | Ordered DESC by created_at |
| Related bundle/manifest | bundles.id, manifests.id | Via FK lookup |

### Empty State

Individual tab empty states:
- Documents tab with no uploads: outline upload-cloud icon + "No documents attached. Upload supporting files." + "Upload Document" secondary button
- New Data tab for non-PM types: "Not applicable for this service type." (no icon, plain text)

### Loading State

Right panel and info band load first (skeleton shimmers). Tab content loads after; tabs show skeleton rows (3 per tab) until data arrives. The breadcrumb and page title always render immediately from the URL slug.

### Error State

If the Request cannot be fetched (deleted, unauthorized, network error):
- Replace content with a centered error state card:
  - 48px outline alert-circle icon
  - Heading: "Request could not be loaded" (16px/600)
  - Body: specific error message from API
  - "Go Back" ghost button + "Retry" secondary button

### Permissions

| Action | Officer 1 | Officer 2 | Officer 3 | Officer 4 | Officer 5 | Officer 6 | Admin |
|--------|-----------|-----------|-----------|-----------|-----------|-----------|-------|
| View | ✓ | ✓ | ✓ (read) | ✓ (read) | ✓ (read) | ✓ | ✓ |
| Edit / Validate | ✓ (Draft only) | — | — | — | — | — | ✓ |
| Assign to Bundle | — | ✓ | — | — | — | — | ✓ |
| Record Disposition | — | — | — | — | — | ✓ | ✓ |
| Flag Correction | ✓ | ✓ | — | — | — | ✓ | ✓ |

NIK field is masked for all roles except Officer 1 and Admin.

### Responsive Behavior

- Desktop: Two-column layout (70/30)
- Laptop: Two-column layout, right panel narrows to 280px
- Tablet: Right panel collapses into a "Details" tab added to the tab bar; main content is full width
- Mobile: All tabs in tab bar; right panel content moved to bottom of page after audit log

### Accessibility

- Status badge includes `aria-label` describing both the color and the text label
- Progress stepper steps use `aria-current="step"` on the active step
- Tab bar uses `role="tablist"`, each tab `role="tab"` with `aria-selected`
- Confirmation modal traps focus and returns focus to trigger button on close

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `E` | Edit primary data (Officer 1 only, Draft status) |
| `V` | Validate request (Officer 1 only) |
| `B` | Assign to bundle (Officer 2 only) |
| `1–5` | Switch between tabs |
| `Esc` | Close any open modal/drawer |
| `Backspace` | Navigate back to previous list |

---

## 03 — Create Request

### Purpose

The intake form used by Officer 1 to record a new taxpayer application (Request) into the system. Produces a Draft Request. This is the most form-intensive screen in the system and must handle the special multi-record New Data case for Partial Mutation (PM) service type.

### Layout

```
┌─────────────────────────────────────────────────────────────┐
│ Sidebar │ Top Nav: breadcrumb + page title                  │
│         ├─────────────────────────────────────────────────  │
│         │ Single centered form column (~720px max-width)    │
│         │  ─ Section: Primary Data                         │
│         │  ─ Section: SPPT Data (Existing)                 │
│         │  ─ Section: New Data  [+ Add Record for PM]      │
│         │  ─ Section: Document Upload                      │
│         │  ─ Form Footer: Cancel | Save Draft | Submit     │
│         └─────────────────────────────────────────────────  │
└─────────────────────────────────────────────────────────────┘
```

Form is rendered in a single white card (`color-bg-surface`, 16px radius, Level 1 shadow). Sections are separated by 32px vertical space and a `color-divider` hairline. No sidebar tabs — the form is a single long scroll.

### Component Hierarchy

**Top Navigation**
- Breadcrumb: "Requests › New Request"
- Page title: "Create Request"
- No action buttons in header (form footer has all actions)

**Section 1 — Primary Data**
Section eyebrow: "PRIMARY DATA" (Caption/Micro, uppercase)
Fields (two-column grid, 16px column gap):
- Service Number (Text Input, required, full-width) — placeholder: e.g., "2024/001"
- Service Type (Select / Dropdown, required) — options: Partial Mutation, Full Mutation Update, Full Mutation Regular, Correction, New Tax Object, Reactivation
- Tax Object Number / NOP (Text Input, required, tabular font) — placeholder: "XX.XX.XXX.XXX.XXX-XXXX.X"
- District (Select, required, pre-populated from user's assigned district)
- Submission Date (Date Picker, required, defaults to today)

**Section 2 — SPPT Data (Existing)**
Section eyebrow: "SPPT DATA — EXISTING"
Fields (two-column grid):
- Owner Name (Text Input, required)
- Owner Address (Textarea, 3 rows, required)
- Block (Text Input)
- RT (Text Input, narrow)
- RW (Text Input, narrow)
- Object Address (Textarea, 3 rows, required)
- Object Block / RT / RW (same as above)
- Land Area (Text Input + "m²" suffix unit label, numeric, required)
- Building Area (Text Input + "m²" suffix, numeric, required)

**Section 3 — New Data**
Section eyebrow: "NEW DATA"
Subheading: "Record 1" (H3, 16px/600) with "Add Another Record" ghost button (only visible when service type = PM)
Same fields as SPPT Data plus:
- Proof of Ownership (Text Input or file picker)

For PM: each additional New Data record appears as a collapsible accordion card beneath the first, labeled "Record 2", "Record 3", etc. Each card has an × remove button (icon-only, ghost, danger hover). A divider separates records. A sticky floating "Add Record" button (pill, primary color, Level Floating shadow) appears when a PM type is selected.

**Section 4 — Document Upload**
Section eyebrow: "SUPPORTING DOCUMENTS"
Drag-and-drop upload zone:
- 2px dashed `color-border`, 8px radius, 80px height at rest
- Icon: 20px outline upload-cloud, `color-text-secondary`
- Label: "Drag files here or click to browse"
- Accepted types: PDF, JPG, PNG (noted below zone)
Uploaded files appear as a list (file name + size + × remove button) beneath the drop zone.

**Form Footer (sticky bottom, 72px, `color-bg-surface`, 1px `color-border` top border)**
Right-aligned button group:
- "Cancel" — Ghost button (navigates back without saving)
- "Save Draft" — Secondary button (saves without validating)
- "Submit for Validation" — Primary button (triggers validation rules; shows inline errors if failed)

### Interactions

- Service Type selection: dynamically shows/hides "Add Another Record" option; shows PM worksheet notice banner if PM selected
- NOP field: auto-formats input to the canonical XX.XX.XXX.XXX.XXX-XXXX.X pattern on blur
- Required field validation: triggered on "Submit" click (not on blur, to reduce noise during entry)
- Invalid fields: border becomes `color-danger`, error message appears beneath the field (13px, `color-danger`)
- "Save Draft" succeeds silently with a toast: "Request saved as draft"
- "Submit" opens confirmation modal: "Submit this request for validation? Officer 1 data becomes the permanent system-of-record." → Confirm (Primary) | Cancel (Ghost)
- Accordion records for PM: expand/collapse on header click; chevron rotates 90°, 180ms

### Data Requirements

All fields map directly to the `requests` table fields documented in WORKFLOW.md §30.5. Service Type drives the required field set — the system enforces COR, NTO, REA service types do not need a Proof of Ownership field; only PM/FMU/FMR do.

### Empty State

N/A — Create Request is always rendered as a blank form on load.

### Loading State

On initial render, Service Type dropdown and District dropdown may require async option loading. During that fetch: the dropdowns render as skeleton inputs. The rest of the form renders immediately. If data for a pre-populated form (e.g., "create from existing") is loading, the full form shows skeleton inputs.

### Error State

Server-side validation failure (after Submit): Inline Alert (danger variant) at the top of the form body:
- "This request could not be submitted. Please correct the errors below."
- Individual field errors appear in-line beneath each invalid field.

Network failure on submit: Toast (danger): "Submission failed. Check your connection and try again." Submit button reverts from loading spinner to normal state.

### Permissions

Create Request is available only to Officer 1 and Admin. Officer 1 can only create Requests for their assigned district.

### Responsive Behavior

- Desktop/Laptop: Two-column field grid, 720px max-width centered
- Tablet: Two-column collapses to single column; sticky footer remains
- Mobile: Single column, all touch targets ≥44px; date picker uses native mobile date input

### Accessibility

- All form fields have explicit `<label>` elements (never placeholder-as-label)
- Required fields marked with `aria-required="true"` and a visible asterisk (*) with a "required" `title`
- Error messages linked to fields via `aria-describedby`
- PM "Add Record" button announces "Add another New Data record" via `aria-label`
- Confirmation modal traps focus, `role="dialog"`, `aria-modal="true"`

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `Tab` | Advance to next field |
| `Shift+Tab` | Previous field |
| `Ctrl+S` / `Cmd+S` | Save Draft |
| `Ctrl+Enter` / `Cmd+Enter` | Submit for Validation |
| `Esc` | Cancel / close confirmation modal |

---

## 04 — Bundle Detail

### Purpose

Displays a single Bundle — a named group of validated Requests. Used by Officer 2 (Bundle Coordinator) to review the Bundle's composition, generate the Bundle Cover Letter, and mark the Bundle as dispatch-ready. Also used by Officers 3, 4, and 5 for read access and downstream actions.

### Layout

Two-column layout: left main panel (70%) and right side panel (30%), identical in structure to Request Detail. A data table listing all constituent Requests is the dominant element in the main panel.

### Component Hierarchy

**Top Navigation**
- Breadcrumb: "Bundles › [Bundle ID]"
- Page title: Bundle ID (e.g., "BND-2024-00042")
- Action buttons (right-aligned, context-sensitive):
  - "Generate Cover Letter" — Secondary (Officer 2, status = Open)
  - "Mark Dispatch-Ready" — Primary (Officer 2, when Cover Letter generated)
  - "Add Requests" — Ghost (Officer 2, when status = Open)
  - Overflow ⋯: Print, Export PDF, Archive

**Info Band (3-column)**
- Column 1: Bundle ID (tabular, 16px/600) + Service Type composition summary (e.g., "3× PM · 2× FMR")
- Column 2: District + fiscal period
- Column 3: Status badge + Request count + Created date/officer

**Tab Bar**
Tabs: "Requests" | "Cover Letter" | "Worksheet" (only if Bundle contains PM Requests) | "Audit Log"

**Tab: Requests**
Full data table (comfortable density, sticky header):

| Column | Width | Notes |
|--------|-------|-------|
| Checkbox | 40px | Bulk select |
| Request ID | 140px | Tabular, link |
| Taxpayer Name | 180px | Truncated with tooltip |
| NOP | 160px | Tabular |
| Service Type | 120px | Status badge (category color) |
| Submitted | 110px | Date, tabular |
| Status | 130px | Status badge (lifecycle color) |
| Actions | 60px | ⋯ icon button (remove from bundle, view) |

Bulk action bar (appears above table when ≥1 row selected):
- "{N} selected" + "Remove from Bundle" danger button + deselect × button

Table footer: pagination control + "Showing N of M Requests" + density toggle (Comfortable/Compact)

**Tab: Cover Letter**
Preview panel: generated Bundle Cover Letter document displayed as a formatted read-only block (white surface, border, 16px padding). Fields shown: Bundle Number, District, Fiscal Period, Request List (numbered), Officer 2 signature line. "Download PDF" secondary button at top-right of preview.

**Tab: Worksheet** (PM only)
If Bundle contains PM Requests, shows the Partial Mutation Worksheet: a table of NOP splits with before/after land area and building area, and calculated deltas. "Download PDF" button.

**Tab: Audit Log** — same pattern as Request Detail §02

**Right Side Panel**

Status Card:
- Status badge (large)
- Progress stepper: Open → Cover Letter Generated → Dispatch-Ready → Dispatched

Parent Manifest Card:
- Manifest ID link + status badge
- "Assign to Manifest" button (Officer 4, only if no manifest assigned)

Composition Summary Card:
- Service type breakdown (mini bar or count list)
- Total Request count
- District

Actions Card:
- Contextual buttons per role/status (same pattern as Request Detail)

### Interactions

- Row click → opens Request Detail in same tab
- "Add Requests" → opens a side Drawer with a searchable table of Validated Requests not yet in a Bundle; checkboxes; "Add Selected" primary button
- "Remove from Bundle" (bulk or row overflow) → confirmation modal; on confirm, removes Requests and re-counts
- "Generate Cover Letter" → loading spinner on button; generated document appears in Cover Letter tab; toast success
- "Mark Dispatch-Ready" → confirmation modal + Officer 2 signature prompt (text field for full name, acting as an e-signature record)
- Worksheet tab available only for PM-containing Bundles; tab is hidden otherwise

### Data Requirements

| Field | Source |
|-------|--------|
| Bundle ID | bundles.id |
| Status | bundles.status |
| Requests | requests WHERE bundle_id = bundle.id |
| Cover Letter body | Generated from bundle fields + request list |
| Worksheet data | requests WHERE service_type = PM AND bundle_id = bundle.id |
| Parent Manifest | manifests WHERE bundle.manifest_id |

### Empty State

Requests tab with no requests:
- 48px outline folder-open icon
- "No requests in this bundle yet."
- "Add Requests" primary button

### Loading State

Table shows 5 skeleton rows (checkbox + 6 text blocks per row). Side panel cards show skeleton content blocks.

### Error State

If Bundle not found: same centered error state as Request Detail §02.
If Cover Letter generation fails: toast (danger) + "Retry" action in toast.

### Permissions

| Action | Officer 2 | Officer 3 | Officer 4 | Officer 5 | Admin |
|--------|-----------|-----------|-----------|-----------|-------|
| View | ✓ | ✓ | ✓ | ✓ | ✓ |
| Add/Remove Requests | ✓ | — | — | — | ✓ |
| Generate Cover Letter | ✓ | — | — | — | ✓ |
| Mark Dispatch-Ready | ✓ | — | — | — | ✓ |
| Assign to Manifest | — | — | ✓ | — | ✓ |

### Responsive Behavior

Same as Request Detail §02. Table on tablet/mobile: rows convert to stacked label/value mini-cards (per Design System §20).

### Accessibility

- Table: `role="grid"`, column headers use `scope="col"`, `aria-sort` on sorted column
- Bulk action bar: `aria-live="polite"` announces count when selection changes
- All icon-only buttons have `aria-label`

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `A` | Open Add Requests drawer (Officer 2 only) |
| `G` | Generate Cover Letter |
| `Space` | Toggle row selection (when focused on a row) |
| `Ctrl+A` | Select all rows |
| `1–4` | Switch tabs |
| `Esc` | Close drawer / modal |

---

## 05 — Manifest Detail

### Purpose

Displays a single Manifest — the top-level dispatch unit grouping multiple Bundles. Used by Officer 4 (Manifest Coordinator) to compose the Manifest, generate the Manifest Cover Letter, and finalize for dispatch. Officer 5 records dispatch and archives the signed Manifest Cover Letter.

### Layout

Identical two-column shell to Bundle Detail §04. The dominant content is a table of constituent Bundles (not Requests).

### Component Hierarchy

**Top Navigation**
- Breadcrumb: "Manifests › [Manifest ID]"
- Page title: Manifest ID (e.g., "MFT-2024-0008")
- Action buttons:
  - "Add Bundles" — Ghost (Officer 4, Open status)
  - "Generate Manifest Cover Letter" — Secondary (Officer 4)
  - "Record Dispatch" — Primary (Officer 5, status = Cover Letter Generated)
  - Overflow ⋯: Print, Export, Archive

**Info Band (3-column)**
- Column 1: Manifest ID + Dispatch date (if dispatched)
- Column 2: District + Fiscal Period + Bundle count
- Column 3: Status badge + Created date/officer

**Tab Bar**
Tabs: "Bundles" | "Cover Letter" | "Dispatch Record" | "Audit Log"

**Tab: Bundles**
Data table (comfortable density):

| Column | Width | Notes |
|--------|-------|-------|
| Checkbox | 40px | Bulk |
| Bundle ID | 140px | Link |
| Request Count | 100px | Right-aligned, tabular |
| Service Types | 160px | Comma-separated type tags |
| District | 130px | Text |
| Status | 130px | Status badge |
| Actions | 60px | ⋯ (remove, view) |

**Tab: Cover Letter**
Same pattern as Bundle Cover Letter tab. Manifest Cover Letter includes: Manifest Number, District, Fiscal Period, Bundle List (numbered), Signature Section for Officer 4. "Download PDF" button.

**Tab: Dispatch Record**
Empty until Officer 5 records dispatch. After recording:
- Dispatch date (date/time)
- Dispatched by: Officer 5 avatar + name
- Physical tracking reference (optional text field)
- Signed Manifest Cover Letter: attached file viewer link
- Acknowledgment received: toggle (yes/no) + date received back

**Tab: Audit Log** — same pattern

**Right Side Panel**
- Status Card with 4-step progress stepper: Open → Cover Letter Generated → Dispatched → Acknowledged
- Composition Summary: Bundle count, total Request count, district breakdown
- Dispatch Timeline: key dates (created, cover letter generated, dispatched, acknowledged)
- Actions Card: contextual per role/status

### Interactions

- "Add Bundles" → Drawer with searchable table of Dispatch-Ready Bundles not yet in a Manifest
- "Record Dispatch" → Modal form: dispatch date (date picker, required), tracking reference (text), upload signed cover letter (file picker) → on submit, status → Dispatched; toast success
- "Acknowledge" toggle in Dispatch Record tab → confirmation + date field
- Clicking a Bundle row → navigates to Bundle Detail

### Data Requirements

| Field | Source |
|-------|--------|
| Manifest ID | manifests.id |
| Bundles | bundles WHERE manifest_id = manifest.id |
| Request count | SUM of bundle.request_count |
| Cover Letter | Generated from manifest + bundle list |
| Dispatch record | manifests.dispatch_date, dispatch_officer, tracking_ref, signed_cover_letter_file |

### Empty State

Bundles tab empty: 48px outline layers icon + "No bundles in this manifest yet." + "Add Bundles" primary button

### Loading State

Same skeleton pattern as Bundle Detail.

### Error State

Same centered error state as other detail screens.

### Permissions

| Action | Officer 4 | Officer 5 | Admin |
|--------|-----------|-----------|-------|
| View | ✓ | ✓ | ✓ |
| Add/Remove Bundles | ✓ | — | ✓ |
| Generate Cover Letter | ✓ | — | ✓ |
| Record Dispatch | — | ✓ | ✓ |
| Record Acknowledgment | — | ✓ | ✓ |

All other officers may view (read-only).

### Responsive Behavior

Same as Bundle Detail. On mobile, Bundles table converts to stacked mini-cards.

### Accessibility

Same as Bundle Detail. File upload in dispatch modal uses `aria-required` and announces upload progress via `aria-live`.

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `A` | Open Add Bundles drawer (Officer 4) |
| `D` | Record Dispatch (Officer 5) |
| `1–4` | Switch tabs |
| `Esc` | Close drawer / modal |

---

## 06 — Correction Detail

### Purpose

Records and tracks the lifecycle of a Correction — either a Surgical Isolation (individual Request returned) or Full Bundle Return. Created by Officers 1 or 6 when a defect is identified. Officer 6 uses this screen to monitor resolution and re-dispatch.

### Layout

Two-column layout (70/30). Narrower than other detail screens — the right panel is more prominent here because the correction workflow is action-heavy.

### Component Hierarchy

**Top Navigation**
- Breadcrumb: "Corrections › [Correction ID]"
- Page title: Correction ID (e.g., "COR-2024-000005")
- Action buttons:
  - "Confirm Receipt of Return" — Primary (Officer 6, status = Pending Return)
  - "Re-assign to New Bundle" — Secondary (Officer 2, after receipt confirmed)
  - "Close Correction" — Ghost (after re-dispatch confirmed)

**Info Band (3-column)**
- Column 1: Correction Type badge — "Surgical Isolation" (warning) or "Full Bundle Return" (danger)
- Column 2: Defect description summary (truncated, expandable)
- Column 3: Status badge + Created date

**Tab Bar**
Tabs: "Summary" | "Affected Records" | "Resolution" | "Audit Log"

**Tab: Summary**
Two-column label/value grid:
- Correction Type
- Original Bundle ID (link)
- Original Manifest ID (link)
- Defect Description (full text, reads as a block of body copy)
- Reported by (avatar + name)
- Reported date
- Central Office reference number (if provided)
- Expected return date

**Tab: Affected Records**
Table of affected Requests (Surgical Isolation) or all Requests in the Bundle (Full Bundle Return):
- Request ID | Taxpayer Name | Service Type | Original Status | Correction Status
Each row links to the Request Detail.

**Tab: Resolution**
Two states:

Before resolution:
- Empty state: "Awaiting return confirmation from Central Office."
- "Record Return Receipt" button (Officer 6)

After resolution:
- Return date
- Return notes (text block)
- New Bundle ID assigned (link, once re-bundled by Officer 2)
- Re-dispatch date (once re-dispatched)
- Resolution status: Resolved / Pending Re-dispatch

**Tab: Audit Log** — same pattern

**Right Side Panel**
- Status Card: 4-step stepper: Reported → Returned → Re-bundled → Re-dispatched
- Related Records: original Bundle, Manifest, new Bundle (once created)
- Actions Card: contextual

### Interactions

- "Confirm Receipt of Return" → modal: return date (date picker) + notes (textarea) → on confirm, status → Returned; affected Requests' statuses updated to Returned
- "Re-assign to New Bundle" → opens Bundle assignment drawer (same as Request Detail flow), but scoped to the affected Requests
- "Close Correction" → confirmation modal; sets status → Resolved

### Data Requirements

| Field | Source |
|-------|--------|
| Correction ID | corrections.id |
| Type | corrections.type (surgical_isolation, full_bundle_return) |
| Affected Requests | requests WHERE correction_id = correction.id |
| Original Bundle/Manifest | FK lookups |
| Resolution fields | corrections.return_date, return_notes, new_bundle_id |

### Empty State

Resolution tab before action taken (described above).

### Loading State

Skeleton shimmers on all tabs and side panel cards.

### Error State

Same centered error state as other detail screens.

### Permissions

| Action | Officer 1/2 | Officer 6 | Admin |
|--------|-------------|-----------|-------|
| View | ✓ (read) | ✓ | ✓ |
| Confirm Return | — | ✓ | ✓ |
| Re-assign to Bundle | ✓ | — | ✓ |
| Close Correction | — | ✓ | ✓ |

### Responsive Behavior

Same as other detail screens.

### Accessibility

- Correction Type badge uses `aria-label="Correction type: Surgical Isolation"` (not color-only)
- All status transitions announced via `aria-live="polite"` toast messages

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `R` | Confirm Receipt of Return (Officer 6) |
| `1–4` | Switch tabs |
| `Esc` | Close modal |

---

## 07 — Archive

### Purpose

Officer 3's primary workspace. Displays all digital archive entries (scanned and uploaded documents associated with Requests, Bundles, and Manifests). Officers can upload new documents, search and filter, and verify digitization status.

### Layout

```
┌────────────────────────────────────────────────────────────┐
│ Sidebar │ Top Nav: title + search + filters               │
│         ├────────────────────────────────────────────────  │
│         │ Filter Chips Row (horizontal, scrollable)       │
│         │ Full-width Data Table (sticky header)           │
│         │ Pagination footer                               │
└────────────────────────────────────────────────────────────┘
```

No right side panel. The Archive is a full-width list/table workspace.

### Component Hierarchy

**Top Navigation**
- Page title: "Archive"
- Search field (inline in nav bar, 480px) — scoped to archive entries
- Action buttons: "Upload Document" — Primary; "Batch Upload" — Secondary; ⋯ overflow

**Filter Chips Row** (24px below nav, 16px left-aligned, 8px gap between chips)
Filter chips (removable, outlined):
- Record Type: All | Request | Bundle | Manifest
- Digitization Status: All | Pending | In Progress | Complete
- Date Range: picker
- Officer 3 (assignee filter)
- Service Type
- District
- "+ Add Filter" ghost chip at end

**Data Table** (full-width, comfortable density, sticky header)

| Column | Width | Notes |
|--------|-------|-------|
| Checkbox | 40px | Bulk |
| Document ID | 140px | Tabular link |
| Record Reference | 180px | Linked to the parent Request/Bundle/Manifest |
| Record Type | 110px | Badge (Request/Bundle/Manifest) |
| Document Type | 130px | e.g., Application Form, ID Copy, Map |
| Uploaded By | 120px | Avatar + name |
| Upload Date | 110px | Tabular date |
| Digitization Status | 140px | Status badge |
| Actions | 60px | ⋯ (view, download, reassign, delete) |

**Bulk Action Bar** (appears on selection):
- "{N} selected" + "Mark Complete" success button + "Reassign" secondary + "Delete" danger + × deselect

**Table Footer**
Pagination + "Showing N of M entries" + page size selector + density toggle

### Interactions

- Row click → opens Digital Archive Viewer (§08) in same tab
- "Upload Document" → opens modal: record reference picker (search), document type select, file drop zone → on confirm, creates archive entry with status "Pending"
- "Batch Upload" → opens multi-file drop zone modal with a mapping table (file → record reference, document type) — each row is editable
- Filter chips: clicking × removes filter; filters are additive (AND logic); filter state persists in URL query params
- Row overflow → View (opens viewer), Download (direct download), Reassign (changes assigned officer), Delete (confirmation modal)
- "Mark Complete" (bulk) → changes digitization status to Complete on all selected

### Data Requirements

| Field | Source |
|-------|--------|
| Archive entries | archive_documents table, all columns |
| Record references | FK to requests / bundles / manifests |
| Digitization status | archive_documents.digitization_status |
| File URL | archive_documents.file_url (signed URL from storage) |

### Empty State

When no entries match the current filters:
- 48px outline archive icon + "No archive entries match the current filters."
- "Clear Filters" ghost button (resets all chips)

When filters are clear and archive is empty:
- 48px outline upload-cloud icon + "No documents in the archive." + "Upload Document" primary button

### Loading State

Filter chips load immediately. Table shows 8 skeleton rows with standard shimmer.

### Error State

Table error: Inline Alert (danger) above the table: "Archive data could not be loaded." + "Retry" ghost button.
Upload failure: Toast (danger) with the specific error reason.

### Permissions

| Action | Officer 3 | Admin |
|--------|-----------|-------|
| View Archive | ✓ | ✓ |
| Upload Documents | ✓ | ✓ |
| Batch Upload | ✓ | ✓ |
| Mark Complete | ✓ | ✓ |
| Delete | — (soft delete only) | ✓ |

Officers 1, 2, 4, 5, 6 can view (read-only) documents linked to records they can access.

### Responsive Behavior

- Desktop/Laptop: Full table, all columns visible
- Tablet: Columns: Document ID, Record Reference, Status, Actions (4 visible); others accessible via row expand
- Mobile: Table converts to stacked mini-card rows

### Accessibility

- Filter chips: `role="group"` with `aria-label="Filters"`, each chip a `<button>` with `aria-pressed` if active
- Table: `aria-rowcount` set to total matching entries; `aria-colcount` set appropriately
- Upload modal drag-and-drop: provides keyboard-accessible file browse alternative, file list announced via `aria-live`

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `U` | Open Upload Document modal |
| `B` | Open Batch Upload modal |
| `/` | Focus search field |
| `F` | Focus first filter chip |
| `Space` | Toggle row selection (row focused) |
| `Ctrl+A` | Select all |
| `Esc` | Clear selection / close modal |

---

## 08 — Digital Archive Viewer

### Purpose

A full-screen document viewer for inspecting a single archived file. Used by all roles for read access. Officer 3 also uses it to update digitization status and add annotations/notes.

### Layout

```
┌────────────────────────────────────────────────────────────┐
│ Sidebar (icon-only, auto-collapsed) │ Viewer Shell         │
│                                     │  ─ Viewer Top Bar    │
│                                     ├─────────────────┬───│
│                                     │ Document Canvas │Inf│
│                                     │ (80%)          │o  │
│                                     │                │Pan│
│                                     │                │el │
│                                     │                │(20│
│                                     │                │%)  │
└────────────────────────────────────────────────────────────┘
```

Sidebar auto-collapses to icon-only (72px) in this screen to maximize document canvas width. Viewer occupies the full remaining viewport height (document canvas is fixed height, no page scroll — the document itself scrolls within its canvas).

### Component Hierarchy

**Viewer Top Bar** (56px, `color-bg-surface`, 1px `color-border` bottom)
Left cluster:
- Back chevron button → returns to Archive list
- Breadcrumb: "Archive › [Document ID]"
- Document title (14px/600)

Center cluster:
- Previous/Next document arrows (cycle through filtered list without returning to archive)
- Page N of M indicator (for multi-page PDFs)

Right cluster:
- Zoom controls: − zoom level % + (dropdown for preset %, fit-to-width, fit-to-page)
- Rotation button (90° clockwise)
- Download button
- "Mark Complete" badge-style button (Officer 3 only, success style)
- ⋯ overflow: Print, Open in new tab, Report issue

**Document Canvas** (80% of viewer area)
- PDF/image rendered in a scrollable container, white background, 8px inner padding
- For PDFs: page breaks are visually separated by 8px `color-bg-canvas` gaps
- Zoom is handled via CSS transform; no re-fetch on zoom
- Thumbnail rail (collapsible, 80px wide, left edge of canvas): mini page thumbnails for multi-page PDFs; clicking thumbnail scrolls to that page

**Info Panel** (20% right column, sticky, independently scrollable)

Document Info Card:
- Document ID
- Document Type
- Digitization Status (editable select for Officer 3)
- Upload Date / Uploaded By

Linked Record Card:
- Record Type badge + Record ID link
- Record status badge

Notes Card:
- Existing notes (chronological, avatar + text + timestamp)
- "Add Note" text area (3 rows) + "Save Note" primary button

Actions Card:
- "Mark Complete" primary (Officer 3)
- "Reassign" secondary
- "Flag Issue" ghost (danger)

### Interactions

- Zoom: − and + buttons change zoom in 25% increments; dropdown presets: 50%, 75%, 100%, 125%, 150%, Fit Width, Fit Page
- Keyboard scroll: Arrow keys scroll the document canvas; Page Up / Page Down change pages in PDFs
- Thumbnail click: smooth-scrolls canvas to the target page, 200ms ease
- "Mark Complete" updates digitization status inline; badge animates success (color transition, 200ms); info panel status updates immediately (optimistic)
- "Add Note" submit: appends note to the chronological list without page reload; textarea clears
- "Flag Issue" → opens a modal: issue type select + description textarea + submit
- Previous/Next arrows: cross-fades document canvas content, 150ms; breadcrumb and info panel update

### Data Requirements

| Field | Source |
|-------|--------|
| File URL | archive_documents.file_url (signed, short-lived) |
| Document metadata | archive_documents (all columns) |
| Notes | archive_notes WHERE document_id |
| Linked record | FK via archive_documents.record_type + record_id |

### Empty State

If file URL is not available (deleted, not yet uploaded): centered placeholder showing 48px outline file-x icon + "Document file not available." + "Report Issue" ghost button.

### Loading State

Document canvas shows a skeleton shimmer overlay while the file loads; the info panel renders immediately from metadata (no file dependency).

### Error State

If file fails to load: centered error card within the canvas: "Document could not be loaded." + Download (as fallback) + "Retry" ghost button.

### Permissions

All authenticated users can view documents linked to records within their role's access scope. Only Officer 3 and Admin can update digitization status or add notes.

### Responsive Behavior

- Desktop/Laptop: Full layout as described
- Tablet: Info panel slides behind a drawer (toggle button in top bar)
- Mobile: Info panel hidden by default; accessible via bottom sheet triggered by an info icon button

### Accessibility

- Document canvas: `role="document"` with `aria-label="Document viewer"`
- Thumbnail rail items: `role="button"`, `aria-label="Page N"`, `aria-current="true"` on active page
- Zoom level announced via `aria-live="polite"` when changed
- All toolbar buttons have `aria-label`; keyboard-operable zoom and navigation

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `←` `→` | Previous / Next document |
| `↑` `↓` | Scroll document canvas |
| `Page Up` / `Page Down` | Previous / Next page (PDF) |
| `+` / `-` | Zoom in / out |
| `0` | Reset zoom to 100% |
| `F` | Fit to width |
| `N` | Add note (focus textarea) |
| `Esc` | Close viewer → return to Archive |

---

## 09 — Search

### Purpose

The global search and filter hub for finding any record across all entities (Requests, Bundles, Manifests, Corrections, Archive Documents). Supports power users who need to locate a specific record by ID, taxpayer name, NOP, or other attributes.

### Layout

```
┌─────────────────────────────────────────────────────────────┐
│ Sidebar │ Top Nav: "Search" title                          │
│         ├─────────────────────────────────────────────────  │
│         │ Search Bar (large, full-width, centered)         │
│         │ Filter Chips Row                                 │
│         │ Results Tabs: All | Requests | Bundles | ...     │
│         │ Results Table / List                             │
│         │ Pagination                                       │
└─────────────────────────────────────────────────────────────┘
```

### Component Hierarchy

**Search Bar**
Pill input (full-width, 56px height, prominent), `color-bg-surface`, Level 1 shadow:
- Leading search icon (24px, `color-text-secondary`)
- Placeholder: "Search by record ID, NOP, taxpayer name, NIK…"
- Trailing × clear button (appears when input has value)
- Shortcut hint: `⌘K` displayed inside the input (right side, `color-text-placeholder`)

**Filter Chips Row** (appears after first search or with saved filters)
- Entity type (active tab mirrors this)
- Status (multi-select)
- Service Type (multi-select)
- Date range
- Officer / Assigned to
- District
- "+ Add Filter"

**Results Tabs**
"All (N)" | "Requests (N)" | "Bundles (N)" | "Manifests (N)" | "Corrections (N)" | "Documents (N)"
Each count updates live as search executes. Active tab bold, 2px primary underline.

**Results Table** (varies per tab)

"All" tab: mixed results, grouped by entity type with a section header label ("Requests", "Bundles", etc.), showing top 3 per type with "See all N" text link.

Single-entity tabs: full data table matching the design of each entity's primary list view.

Columns for "Requests" tab:

| Column | Notes |
|--------|-------|
| Request ID | Tabular, link; matched query term highlighted in `color-primary-tint` |
| Taxpayer Name | Match highlighted |
| NOP | Tabular, match highlighted |
| Service Type | Badge |
| Status | Badge |
| District | Text |
| Date | Tabular |

**No-results state** shown inline within each tab when count = 0.

**Recent Searches** (shown below search bar when input is focused, before typing):
- Up to 5 recent queries, each a clickable chip
- "Clear history" text link (right-aligned)

**Saved Searches** (shown below recent, if any exist):
- Named saved searches with bookmark icon
- "Save current search" ghost button appears in filter chip row after a search is run

### Interactions

- Typing in search bar: debounced 300ms, then fires API query; results update all tabs simultaneously
- Tab click: filters results to that entity type; URL updates with ?entity=requests
- Filter chips: same additive AND logic as Archive (§07); state persists in URL
- Row click: navigates to the relevant record detail screen
- "Save current search" → modal: enter name → saves to user preferences → appears in Saved Searches
- Match highlighting: the matched substring in ID, name, NOP fields is wrapped in a `<mark>` styled with `color-primary-tint` background, `color-primary` text
- ⌘K / Ctrl+K global shortcut: focuses this search bar from any screen (or opens a command palette modal)

### Data Requirements

Federated search across: `requests`, `bundles`, `manifests`, `corrections`, `archive_documents`. Backend returns a unified result set with entity type and ID for routing.

### Empty State

Before any search (fresh page load):
- 48px outline search icon, `color-text-secondary`
- "Search across all records" (18px/600)
- "Try searching by NOP, taxpayer name, Request ID, or Bundle ID." (14px, `color-text-secondary`)
- Recent Searches section below

No results state (after search with zero matches):
- 48px outline search-x icon
- "No results for '{ query }'"
- "Try different keywords or adjust your filters." + "Clear Filters" ghost button

### Loading State

Results skeleton: 5 rows per visible tab, each row with standard shimmer blocks. Tab counts show "…" while loading.

### Error State

If search fails: Inline Alert (danger) above results: "Search could not be completed. Try again." + "Retry" button.

### Permissions

Search returns only records the current user is authorized to view, scoped by role. Officer 3 sees Archive results only; Officer 1 sees only Requests. Supervisor/Admin see all entities.

### Responsive Behavior

- Desktop: Full layout
- Tablet: Filter chips wrap to 2 rows if needed; results table adjusts to fewer visible columns
- Mobile: Search bar full-width; filter chips collapse to a "Filter" button opening a bottom sheet; results show as stacked mini-cards

### Accessibility

- Search input: `role="searchbox"`, `aria-label="Search all records"`, `aria-controls` pointing to results region
- Results region: `role="region"`, `aria-live="polite"`, `aria-label="Search results"`, announces count changes
- Match highlights: `<mark>` elements are semantically correct; screen readers announce highlighted matches naturally

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `⌘K` / `Ctrl+K` | Focus search from any screen |
| `↓` / `↑` | Navigate results (when search is focused) |
| `Enter` | Open focused result |
| `Tab` | Move to filter chips |
| `Esc` | Clear search / dismiss |
| `S` | Save current search (when results are showing) |

---

## 10 — Officer Management

### Purpose

Admin-only screen for creating, editing, deactivating, and managing officer accounts. Configures role assignments, district assignments, and access permissions.

### Layout

Full-width table layout (same shell as Archive §07, no right side panel).

### Component Hierarchy

**Top Navigation**
- Page title: "Officer Management"
- Action buttons: "Invite Officer" — Primary; "Export List" — Secondary

**Filter Chips Row**
- Role: All | Officer 1–6 | Admin | Supervisor
- Status: All | Active | Inactive
- District: (multi-select)

**Data Table**

| Column | Width | Notes |
|--------|-------|-------|
| Checkbox | 40px | Bulk |
| Officer | 200px | Avatar (32px) + Full Name (Body Emphasis) + email (Small, secondary) |
| Role | 140px | Role badge (category color per role) |
| District | 130px | Text |
| Status | 100px | Active (success badge) / Inactive (neutral) |
| Last Active | 120px | Relative timestamp |
| Actions | 60px | ⋯ (Edit, Deactivate, Resend Invite, Reset Password) |

**Bulk Action Bar**: "Deactivate Selected" danger + "Change Role" secondary

### Interactions

- "Invite Officer" → Modal: Full Name (text), Email (text), Role (select), District (select), "Send Invitation" primary button
- Row ⋯ → "Edit": opens an Edit Officer drawer (same fields as invite + Status toggle)
- "Deactivate" → confirmation modal with warning: "Deactivated officers cannot log in. Their records remain in the system."
- "Resend Invite" → toast confirmation: "Invitation resent to {email}"
- "Reset Password" → confirmation modal → sends password reset email

### Data Requirements

| Field | Source |
|-------|--------|
| Officers | users table |
| Role | users.role |
| District | users.district_id |
| Last Active | users.last_active_at |
| Status | users.status |

### Empty State

No officers found (filtered): "No officers match the current filters." + "Clear Filters"

### Loading State

8 skeleton rows with avatar circles and text blocks.

### Error State

Table error: Inline Alert above table + "Retry".

### Permissions

Admin-only screen. Not accessible to any Officer role.

### Responsive Behavior

- Desktop: Full table
- Tablet/Mobile: Condensed columns (Officer, Role, Status, Actions); others in expandable row

### Accessibility

- Invite modal: focus trap, all fields labeled, `aria-required`
- Role badge: includes text label (not color-only)

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `I` | Open Invite Officer modal |
| `Esc` | Close modal / drawer |
| `Ctrl+A` | Select all rows |

---

## 11 — User Management

### Purpose

Admin-only screen for managing the full user population (distinct from Officer Management which is role-focused). Handles login credentials, access levels, and account lifecycle.

### Layout

Identical structure to Officer Management §10. The distinction: this screen shows all accounts (including non-officer system users, read-only accounts, external auditors), while §10 is scoped to operational officers.

### Component Hierarchy

Same data table structure as Officer Management, with additional columns:
- Account Type: Officer | Auditor | Read-Only | System
- 2FA Status: Enabled / Disabled (badge)
- Login Method: Password | SSO

Additional actions in row ⋯:
- Force 2FA Setup
- Disable Account
- View Login History

**"Login History" Drawer**: opens from row action, shows a table of login events (date/time, IP, device, success/failure).

### Interactions

Same as Officer Management plus:
- "Force 2FA Setup" → sends email prompting user to set up 2FA; status badge updates to "Pending"
- "View Login History" → opens a right-side drawer (250ms slide-in) with a paginated log table

### Data Requirements

Same as Officer Management plus: `users.account_type`, `users.two_factor_enabled`, `users.login_method`, `login_events` table.

### Permissions

Admin-only.

### All other specs (Empty/Loading/Error/Responsive/Accessibility/Keyboard) mirror Officer Management §10.

---

## 12 — Audit Log

### Purpose

System-wide append-only log of every significant state change and user action. Used by Supervisors and Admins for compliance, forensics, and dispute resolution. Provides the complete chain of custody required by WORKFLOW.md §19.

### Layout

Full-width table, same shell as Archive/Officer Management.

### Component Hierarchy

**Top Navigation**
- Page title: "Audit Log"
- Action buttons: "Export Log" — Secondary (exports filtered results as CSV or PDF)

**Filter Chips Row**
- Entity Type: All | Request | Bundle | Manifest | Correction | Archive | User | System
- Action Type: All | Created | Updated | Status Change | Deleted | Login | Export
- Date Range: date range picker (required to constrain large datasets; defaults to "Last 7 days")
- Actor: user search (autocomplete)
- Record ID: text filter

**Data Table** (compact density by default — this is a high-volume read-only log)

| Column | Width | Notes |
|--------|-------|-------|
| Timestamp | 160px | Full date + time, tabular, UTC offset shown |
| Actor | 160px | Avatar (24px) + name |
| Action | 160px | Action type badge (neutral by default; danger for deletions) |
| Entity | 120px | Entity type badge |
| Record ID | 140px | Tabular link to the affected record |
| Description | auto | Plain text summary of what changed |
| IP Address | 120px | Monospace, `color-text-secondary` |

No bulk actions. No edits. Read-only.

Table footer: pagination (50 rows per page default) + "Showing N of M entries" + export button repeated here.

**Expandable Row** (click anywhere on row):
Reveals full diff block (before → after) for field changes, shown as a two-column table inside the expanded row.

### Interactions

- Date range is mandatory (API enforces it); defaults auto-populate to "Last 7 days"
- Row click → expands in-place; chevron icon appears at far right of each row (same accordion pattern)
- "Export Log" → opens modal: choose format (CSV / PDF), confirm the current filters being exported, → triggers async export → toast: "Export is being prepared. You'll be notified when it's ready." (Notification Panel)
- Record ID links → navigate to the relevant detail screen

### Data Requirements

| Field | Source |
|-------|--------|
| Audit entries | audit_log table (all columns) |
| Diff data | audit_log.before_state, after_state (JSON) |
| Actor | users JOIN on audit_log.actor_id |

### Empty State

No entries for current filter: "No audit entries match the current filters." + "Adjust filters" — no clear button (date range is always required, so it's never truly empty)

### Loading State

12 skeleton rows, compact density.

### Error State

Inline Alert above table: "Audit log could not be loaded." + "Retry"

### Permissions

Supervisor and Admin only. Officers cannot access this screen.

### Responsive Behavior

- Desktop: Full table
- Tablet: Condense to Timestamp, Actor, Action, Description; IP in expandable row
- Mobile: Stack as cards (each log entry = one card)

### Accessibility

- Expandable rows use `aria-expanded` and `aria-controls`; keyboard: Enter/Space to expand
- Timestamps include `<time datetime="...">` with ISO 8601 value
- IP addresses in monospace font: use `<code>` element

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `E` | Export log (opens modal) |
| `Space` / `Enter` | Expand/collapse focused row |
| `Esc` | Collapse all / close modal |

---

## 13 — Reports

### Purpose

Generates pre-defined and custom operational reports for Supervisors and Admins. Covers throughput, backlog, officer performance, correction rates, and dispatch timelines.

### Layout

```
┌─────────────────────────────────────────────────────────────┐
│ Sidebar │ Top Nav: "Reports" title + action buttons         │
│         ├──────────────────────┬──────────────────────────  │
│         │ Report Selector      │ Report Canvas              │
│         │ (left, 280px)        │ (right, fluid)             │
│         │  ─ Report Categories │  ─ Report Header           │
│         │  ─ Saved Reports     │  ─ Filters & Params        │
│         │                      │  ─ Charts + Tables         │
│         └──────────────────────┴──────────────────────────  │
└─────────────────────────────────────────────────────────────┘
```

### Component Hierarchy

**Left Report Selector Panel** (280px, `color-bg-surface`, 1px `color-border` right border, full height)

Section: "Pre-built Reports"
- Throughput Summary
- Backlog Analysis
- Dispatch Timeline
- Correction Rate
- Officer Workload
- Archive Completeness

Section: "Saved Reports" (user's saved custom configurations)

Each report listed as a Sidebar Nav Item style row (40px, icon + label, hover state), with the active report getting the white pill selection treatment.

"+ New Custom Report" ghost row at bottom

**Right Report Canvas**

Report Header Card:
- Report name (H2)
- Description (13px, `color-text-secondary`)
- Last generated timestamp

Parameters Bar (full-width, below header card):
- Date range (date picker, required)
- District (select, optional)
- Officer (multi-select, optional)
- Service Type (multi-select, optional)
- "Run Report" primary button (right-aligned)

Report Body (after run):
- KPI summary row (3–4 stat cards, same as Dashboard KPI cards)
- Chart section (bar, line, or grouped bar depending on report type — built with the product's chosen charting library, using `color-primary`, `color-secondary`, `color-success`, `color-warning` as chart palette, in that priority order)
- Detail table (same table component, with export-per-section option)

**Export Controls** (top-right of report canvas, right-aligned):
- "Export PDF" Secondary button
- "Export CSV" Ghost button
- "Save Report Config" Ghost button with bookmark icon

### Interactions

- Selecting a report in the left panel: instantly loads the report template into the canvas (parameters are cleared, body shows empty state prompting user to run)
- "Run Report" → triggers data fetch; loading state on canvas body; KPI cards and charts appear as data arrives
- "Save Report Config" → modal: give this report config a name → saves to "Saved Reports" in left panel
- Chart data points: hover shows tooltip (value + label, Level 3 shadow); clicking a bar/segment filters the table below to matching records
- "Export PDF" → async export → notification toast when ready

### Data Requirements

Report-specific aggregations from `requests`, `bundles`, `manifests`, `corrections`, `audit_log`. All queries are parameterized by date range, district, officer, and service type.

### Empty State

Before running any report (fresh report canvas):
- 48px outline bar-chart icon + "Configure parameters and run the report."
- "Run Report" primary button (same as in params bar)

### Loading State

After "Run Report" click: parameters bar button shows spinner; KPI cards show skeleton shimmers; chart area shows a single large skeleton block; table shows skeleton rows.

### Error State

Report generation failure: Inline Alert (danger) below params bar: "Report could not be generated." + error detail + "Retry" ghost.

### Permissions

Supervisor and Admin only.

### Responsive Behavior

- Desktop: Two-column shell (left selector + right canvas)
- Tablet: Left selector collapses to a horizontal tab strip above the canvas
- Mobile: Left selector hidden behind a hamburger; report canvas full-width

### Accessibility

- Charts use `role="img"` with `aria-label` summarizing the chart data; a "View as table" toggle provides the data in an accessible table format
- Report selector items: `role="listitem"`, active item `aria-current="true"`

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `R` | Run report |
| `E` | Export PDF |
| `↑` `↓` | Navigate report list (in selector panel) |
| `S` | Save report configuration |

---

## 14 — Settings

### Purpose

System configuration for Admins. Covers system-wide settings: numbering system configuration, district setup, service type settings, notifications configuration, integration settings, and data retention policies.

### Layout

Same two-column shell as Reports (left nav panel + right settings canvas). Settings categories in the left panel; individual setting groups in the right canvas.

### Component Hierarchy

**Left Settings Nav** (280px)
Section groups:
- General
  - Organization Details
  - Districts
  - Fiscal Periods
- Workflow
  - Numbering System
  - Service Types
  - Bundle Rules
  - Manifest Rules
- Notifications
  - Email Notifications
  - In-App Notifications
- Integrations
  - Export Formats
  - API Access
- Data & Compliance
  - Audit Log Retention
  - Archive Retention
  - Data Export

**Right Settings Canvas**
Each settings group renders as one or more Standard Cards. Within each card:

Card header: group name (H2) + optional "Edit" icon button (pencil, ghost) in the header row — settings are read-only by default and editable only after clicking Edit (reduces accidental changes)

Settings fields in read-only mode: label/value pairs (same as detail screen tabs)
In edit mode: the same fields become editable inputs; a floating sticky action bar appears at the bottom of the canvas with "Save Changes" primary + "Cancel" ghost.

Sample: Numbering System card:
- Request Sequence Prefix (text, e.g., "REQ")
- Bundle Sequence Prefix (text, e.g., "BND")
- Manifest Sequence Prefix (text, e.g., "MFT")
- Sequence Year Reset (toggle: Yes/No)
- Current Sequence Numbers (read-only, tabular)
- "Reset Sequence" danger button (protected by a confirmation modal with typed confirmation)

Sample: Bundle Rules card:
- Maximum Requests per Bundle (number input, validates against BR-17)
- Cross-District Bundles Allowed (toggle: Yes/No, default: No per A-01)
- Mixed Service Types per Bundle (toggle: Yes/No per A-02)

### Interactions

- "Edit" icon button: transitions the card from read-only to edit mode (smooth 150ms fade of label/value → input transition)
- Any change: marks the setting as "Unsaved" (a small orange dot on the card header + the sticky save bar appears)
- "Save Changes" → confirmation modal for destructive settings (e.g., numbering reset); otherwise saves directly with a toast
- Toggle controls: animate thumb position, 150ms ease
- "Reset Sequence" → modal: "Type CONFIRM to proceed" text field; primary button stays disabled until exact string is matched

### Data Requirements

All configuration values from a `system_settings` key-value store, grouped by category.

### Empty State

N/A — Settings always have values (system defaults if not customized).

### Loading State

Settings canvas shows skeleton label/value pairs while loading.

### Error State

Save failure: Inline Alert (danger) in the sticky save bar: "Settings could not be saved. {reason}" + "Retry".

### Permissions

Admin-only. No other role can access or modify Settings.

### Responsive Behavior

- Desktop: Two-column shell
- Tablet: Left nav collapses to tab strip
- Mobile: Left nav in bottom sheet

### Accessibility

- Toggle controls: `role="switch"`, `aria-checked` state
- Sticky save bar: `aria-live="polite"` announces unsaved state
- Destructive confirmation modal: typed confirmation field has `aria-label="Type CONFIRM to proceed"`

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `↑` `↓` | Navigate settings categories |
| `E` | Edit focused card |
| `Ctrl+S` / `Cmd+S` | Save changes |
| `Esc` | Cancel edit mode |

---

## 15 — Notifications

### Purpose

Displays all system notifications for the current user: assignment alerts, status changes, mentions, system events, and task reminders. Accessible as both an in-page full-screen list and as a slide-out panel triggered from the notification bell in the top navigation.

### Notification Panel (slide-out, from bell icon)

Width: 400px. Slides in from the right edge of the top navigation bar (250ms cubic-bezier). Has its own scroll region, max-height: 80vh. Closes on outside click or Escape.

**Panel Header**:
- "Notifications" (H2)
- Unread count badge (danger, pill)
- "Mark all read" text link (right-aligned)
- Settings icon button (links to notification preferences in Settings §14)
- × close button

**Notification List** (grouped: "Today", "Yesterday", "Earlier")

Each notification row (56px, comfortable):
- Leading icon (20px, semantic color per notification type) OR avatar (32px) for person-initiated events
- Primary text: action description (14px/400, `color-text-primary` if unread, `color-text-secondary` if read)
- Secondary text: Record ID link + timestamp (13px, `color-text-secondary`)
- Trailing unread dot (8px, `color-primary`, appears only if unread)
- Row hover: `color-hover` background; click marks as read and navigates to the related record

**Panel Footer**:
- "View all notifications" text link → full-screen Notifications page

### Full-Screen Notifications Page

Layout: Full-width list (no table — this is a notification feed, not a data grid). Filter chips at top: All | Unread | Assignments | Status Changes | System.

Same notification row structure as panel but with wider text area (no truncation). Bulk actions: "Mark all read" | "Clear all".

### Notification Types

| Type | Icon | Color | Trigger |
|------|------|-------|---------|
| Assignment | person-plus | `color-primary` | A record assigned to the user |
| Status Change | refresh-cw | `color-info` | A record the user owns changes status |
| Mention | at-sign | `color-secondary` | User mentioned in a note |
| Overdue Alert | alert-circle | `color-danger` | A task is past its due date |
| System Event | info | `color-text-secondary` | System maintenance, export ready, etc. |
| Correction | alert-triangle | `color-warning` | A Correction record is opened on the user's record |

### Interactions

- Notification row click: marks as read (dot disappears, text dims to secondary color) + navigates to the related record
- "Mark all read": optimistic update — all dots disappear instantly, then API confirms
- Swipe left (mobile): reveals "Dismiss" action (danger)
- New notifications arriving while panel is open: slide in from the top with a subtle 200ms ease-in animation; count badge increments

### Data Requirements

| Field | Source |
|-------|--------|
| Notifications | notifications WHERE user_id = current_user.id |
| Read state | notifications.read_at (null = unread) |
| Related record | notifications.entity_type, entity_id |
| Notification type | notifications.type |

### Empty State

Panel empty: 48px outline bell icon + "You're all caught up." (centered, `color-text-secondary`)
Full page with filter active: "No notifications match this filter." + "Show all" ghost button

### Loading State

Panel: 5 skeleton notification rows (icon circle + two text blocks each)
Full page: 10 skeleton rows

### Error State

Toast (danger): "Notifications could not be loaded." + "Retry"

### Permissions

All authenticated users have their own notification feed. System notifications are visible to all; role-specific notifications only appear for the relevant role.

### Responsive Behavior

- Desktop: Panel 400px, full-page layout with filter chips
- Tablet/Mobile: Panel full-width (100vw slide-over); full page is single-column

### Accessibility

- Bell icon button: `aria-label="Notifications, {N} unread"`, updates count dynamically
- Notification panel: `role="dialog"`, `aria-label="Notifications"`, focus trapped while open
- New notification arrival: `aria-live="polite"` region announces "New notification: {summary}"
- Unread dot: `aria-label="Unread"` on the dot element; not color-only — the full-weight text also signals unread state

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `N` | Toggle notification panel |
| `M` | Mark all notifications as read (panel focused) |
| `↓` `↑` | Navigate notification rows (panel focused) |
| `Enter` | Open focused notification's record |
| `Esc` | Close notification panel |

---

## 16 — Profile

### Purpose

Allows the currently signed-in user to view and update their personal information, change their password, manage notification preferences, and review their recent activity.

### Layout

Centered content (max-width 720px), single column, standard shell. No side panel.

### Component Hierarchy

**Top Navigation**
- Page title: "Profile"
- Action buttons: "Edit Profile" — Secondary; "Change Password" — Ghost

**Profile Hero Card** (prominent, top of page)
- Avatar (64px, L size per Design System §17) — clickable to open image picker
- Full Name (H2, 18px/600)
- Role badge (category color)
- District (13px, `color-text-secondary`)
- Email (13px, `color-text-secondary`)
- Last active: "Active N minutes ago" or specific date (13px, `color-text-secondary`)

**Personal Information Card**
Read-only label/value grid (edit mode triggered by "Edit Profile" button):
- Full Name
- Email (read-only after initial set, change requires Admin action)
- Phone (optional)
- Department / Unit
- District

**Security Card**
- Password: "••••••••" (masked) + "Change Password" ghost button
- Two-Factor Authentication: toggle + status badge (Enabled/Disabled) + "Configure" text link
- Active Sessions: "N active sessions" + "View all / Sign out other sessions" text link → opens a modal listing sessions (device, IP, last active, "Revoke" button per session)

**Notification Preferences Card**
Quick toggles (same style as Settings toggles):
- Email notifications: on/off
- In-app notifications: on/off
- Per-type toggles (collapsed by default, expandable accordion):
  - Assignment notifications
  - Status change notifications
  - Overdue alerts
  - System events

**Recent Activity Card**
Last 10 actions performed by the current user (same row style as Dashboard Recent Activity, but scoped to self). "View Full Audit History" text link (links to Audit Log filtered by current user).

### Interactions

- "Edit Profile" → fields transition to edit mode; sticky save bar appears
- "Change Password" → modal: Current Password + New Password + Confirm New Password (all inputs type=password, with show/hide toggle per field) + "Update Password" primary
- Avatar click → file picker; selected image uploads immediately; avatar updates optimistically with loading spinner overlay
- 2FA toggle → opens setup wizard modal (if enabling) or confirmation modal (if disabling)
- "View all sessions" → modal with paginated session table; "Revoke All Others" danger button
- Notification preference toggles: save automatically on change (no save button needed); each change fires an optimistic update with a silent toast on success

### Data Requirements

| Field | Source |
|-------|--------|
| User profile | users table (current_user) |
| Sessions | user_sessions WHERE user_id = current_user.id |
| Notification prefs | user_notification_preferences |
| Recent activity | audit_log WHERE actor_id = current_user.id, last 10 |

### Empty State

Recent Activity card with no history: "No recent activity recorded."

### Loading State

Hero card loads immediately from session data. Other cards show skeleton content while fetching.

### Error State

"Profile data could not be loaded." Inline Alert with "Retry". For save failures: toast (danger).

### Permissions

All authenticated users can view and edit their own profile. Changing email requires Admin intervention. Role and District are read-only (Admin-managed).

### Responsive Behavior

Single column at all breakpoints. On mobile, hero card stacks vertically (avatar above name/details). Form fields are full-width.

### Accessibility

- Avatar upload: file input with `aria-label="Upload profile photo"`, visually hidden but keyboard-focusable
- Password inputs: `autocomplete="current-password"` / `"new-password"` for password manager support
- Show/hide password toggle: `aria-label="Show password"` / `"Hide password"`, toggles `aria-pressed`
- 2FA wizard modal: multi-step, focus moves to first field of each step on advance

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `E` | Enter Edit Profile mode |
| `P` | Open Change Password modal |
| `Ctrl+S` / `Cmd+S` | Save profile changes |
| `Esc` | Cancel edit / close modal |

---

## 17 — Login

### Purpose

The unauthenticated entry point. Allows officers to sign in to ARCHITAX with their credentials. Not accessible once authenticated (redirects to Dashboard on load if a valid session exists).

### Layout

```
┌─────────────────────────────────────────────────────────────┐
│  Full-page split:                                           │
│  Left panel (45%, sidebar gradient bg) │ Right (55%, white) │
│   ─ ARCHITAX logo + wordmark           │ ─ Login form card  │
│   ─ Product tagline                    │   centered         │
│   ─ Version info (bottom-left)         │                    │
└─────────────────────────────────────────────────────────────┘
```

No sidebar or top navigation. The application shell is not present — this is a pre-auth screen. The split background serves as the sole brand expression.

### Component Hierarchy

**Left Panel** (45% viewport width)
Background: sidebar gradient (`color-bg-sidebar-start` → `color-bg-sidebar-end`, 135°)
Content (vertically centered, 48px horizontal padding):
- ARCHITAX logo mark (SVG, 48px)
- Wordmark: "ARCHITAX" (28px/700, white)
- Tagline: "Property Tax Archive & Workflow Management" (16px/400, `rgba(255,255,255,0.8)`)
- Bottom-left: Version label (12px/500, `rgba(255,255,255,0.5)`) + Government/district identifier

**Right Panel** (55% viewport width)
Background: `color-bg-canvas`
Content: single centered login card (420px max-width, `color-bg-surface`, 20px radius, Level Modal shadow, 40px padding):

Card header (centered):
- "Sign in to ARCHITAX" (H2, 18px/600)
- "Use your assigned credentials to continue." (13px, `color-text-secondary`)

Form fields:
- Email / Username label + Text Input (40px, full-width, autocomplete="username")
- Password label + Password Input (40px, full-width, autocomplete="current-password") + trailing show/hide toggle icon
- "Forgot password?" text link (13px, right-aligned, below password field)
- "Sign In" Primary button (full-width, large — 48px height for this screen only; the sole primary action)

Error inline (below button, only when auth fails):
- Inline Alert, danger variant: "Invalid email or password. Please try again."

Bottom of card (centered):
- "Trouble signing in? Contact your system administrator." (12px, `color-text-secondary`)

### Interactions

- Submit on button click or Enter key in any field
- Loading state: button label → spinner + "Signing in…", button disabled, fields disabled
- Auth success → 200ms cross-fade → redirect to Dashboard (or the originally requested URL if redirected here from a deep link)
- Auth failure → shake animation on the card (300ms, ±6px, 2 cycles) + Inline Alert appears; fields remain filled; password field cleared; focus moves to password field
- "Forgot password?" → navigates to a simple `/forgot-password` page (separate, out of scope for this specification)
- If session already valid on page load → immediate redirect to Dashboard with no login form shown

### Data Requirements

- POST credentials to `/auth/login`
- On success: receives session token, stored in httpOnly cookie
- No data pre-population (email field is always empty on fresh load; browser autofill operates normally)

### Empty State

N/A — form is always shown.

### Loading State

Button loading state as described above. No skeleton loaders (the form renders instantly — no async data dependency before display).

### Error State

- 401 Unauthorized (wrong credentials): Inline Alert (danger) inside the card
- 429 Too Many Requests (rate limit): Inline Alert (warning): "Too many attempts. Please wait N minutes before trying again." + countdown timer updating every second
- 503 Service Unavailable: Inline Alert (danger): "ARCHITAX is temporarily unavailable. Please try again shortly."
- Account locked (5 failed attempts): Inline Alert (danger): "Your account has been locked. Contact your administrator." + "Sign In" button disabled

### Permissions

No authentication required. If accessed while authenticated, redirect to Dashboard.

### Responsive Behavior

- Desktop: Left/right split as described
- Tablet: Left panel collapses — only the logo/wordmark appears above the login card (no full gradient panel); login card is centered full-width
- Mobile: Left panel hidden entirely; login card is full-width (100vw) with no card border-radius at bottom edge; gradient visible only as a top header band (80px)

### Accessibility

- Form fields: `<label>` elements for all inputs; autocomplete attributes for password manager support
- Error messages: `aria-live="assertive"` for auth failure messages (immediate announcement)
- Show/hide password: `<button type="button">` with `aria-label` toggling between "Show password" and "Hide password"; `aria-pressed` state
- Rate limit countdown: `aria-live="polite"` to avoid repetitive announcements; updates every 10 seconds (not every second) for screen reader sanity
- Card shake animation: suppressed under `prefers-reduced-motion`; error is still shown

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `Tab` | Advance to next field |
| `Enter` | Submit form (from any field) |
| `Shift+Tab` | Previous field |

---

## Appendix A — Component Cross-Reference

| Component (Design System §23) | Screens Used |
|-------------------------------|-------------|
| Application Shell | 01–16 |
| Sidebar Navigation | 01–16 |
| Top Navigation Bar | 01–16 |
| Card (Standard) | 01, 02, 04, 05, 06, 10–16 |
| Card (Interactive Tile) | 01 |
| Data Table | 02, 04, 05, 07, 10, 11, 12 |
| Simplified List Row | 01, 15 |
| Status Badge | All |
| Tag / Filter Chip | 07, 09, 10, 11, 12, 13 |
| Avatar / Avatar Group | 01, 02, 04, 10, 11, 15, 16 |
| Button (all variants) | All |
| Text Input | 03, 06, 14, 16, 17 |
| Select / Dropdown | 03, 06, 14 |
| Textarea | 03, 15 |
| Checkbox | 04, 05, 07, 10, 11 |
| Date Picker | 03, 05, 12, 13, 14 |
| Search Field | 09 |
| Modal / Dialog | 02–08, 10–17 |
| Drawer / Side Panel | 04, 05, 11, 15 |
| Tooltip | All |
| Toast | All |
| Inline Alert / Banner | 01–17 |
| Pagination Control | 01, 02, 04, 05, 07, 09–12 |
| Breadcrumb | 02–08 |
| Tabs (underline) | 02, 04, 05, 06, 09 |
| Accordion / Expandable Row | 03, 12 |
| Progress Bar / Stepper | 02, 04, 05, 06 |
| Empty State | All |
| Skeleton Loader | All |

---

## Appendix B — Status Color Matrix

| Status | Semantic Color | Token | Usage |
|--------|---------------|-------|-------|
| Draft | Info | `color-info-tint` / `color-info` | Requests not yet validated |
| Validated | Success | `color-success-tint` / `color-success` | Requests ready for bundling |
| In Bundle | Info | `color-primary-tint` / `color-primary` | Requests assigned to a Bundle |
| Dispatched | Secondary | — / `color-secondary` | Bundles/Manifests sent to Central Office |
| Completed | Success | `color-success-tint` / `color-success` | Final positive resolution |
| Pending | Warning | `color-warning-tint` / `color-warning` | Awaiting Central Office action |
| Returned | Danger | `color-danger-tint` / `color-danger` | Surgical Isolation or Bundle Return in progress |
| Resolved | Neutral | — / `color-text-secondary` | Correction closed |

---

## Appendix C — Officer Role × Screen Access Matrix

| Screen | O1 | O2 | O3 | O4 | O5 | O6 | Admin | Supervisor |
|--------|----|----|----|----|----|----|-------|------------|
| Dashboard | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Request Detail | ✓ | R | R | R | R | ✓ | ✓ | R |
| Create Request | ✓ | — | — | — | — | — | ✓ | — |
| Bundle Detail | R | ✓ | R | R | R | R | ✓ | R |
| Manifest Detail | R | R | R | ✓ | ✓ | R | ✓ | R |
| Correction Detail | ✓ | ✓ | — | — | — | ✓ | ✓ | R |
| Archive | R | R | ✓ | R | R | R | ✓ | R |
| Digital Archive Viewer | R | R | ✓ | R | R | R | ✓ | R |
| Search | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Officer Management | — | — | — | — | — | — | ✓ | — |
| User Management | — | — | — | — | — | — | ✓ | — |
| Audit Log | — | — | — | — | — | — | ✓ | ✓ |
| Reports | — | — | — | — | — | — | ✓ | ✓ |
| Settings | — | — | — | — | — | — | ✓ | — |
| Notifications | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Profile | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| Login | * | * | * | * | * | * | * | * |

**Legend:** ✓ = Full access | R = Read-only | — = No access | * = Pre-auth only

---

_End of SCREEN_SPECIFICATION.md — ARCHITAX Property Tax (PBB-P2) Archive & Workflow Management System_
_Design System Reference: UI_DESIGN_SYSTEM.md v2.0 | Workflow Reference: WORKFLOW.md v1.0_
