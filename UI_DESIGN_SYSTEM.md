# ARCHITAX — UI Design System

**Internal Enterprise Design System Reference**
Version 2.0 · Maintained by the ARCHITAX Design & Frontend Engineering team

---

## How to Use This Document

This document is the single source of truth for the visual language of ARCHITAX, the Property Tax (PBB-P2) Archive & Workflow Management System. It defines color, type, spacing, components, motion, and interaction rules so that any designer, frontend engineer, or AI coding assistant can produce new screens that are visually indistinguishable in _style_ from one another — regardless of who built them or when.

All numeric values (hex codes, pixel sizes, durations) in this document are calibrated estimates derived from visual analysis of an approved aesthetic reference, refined for a government-grade enterprise data application. They should be treated as production-ready defaults, implemented as design tokens (CSS custom properties / a theme object), and adjusted centrally — never overridden ad hoc in individual screens.

This document describes **only the visual system**: color, type, layout, spacing, elevation, motion, and component anatomy. It intentionally does not describe ARCHITAX's business workflows, data model, or feature set — those are specified separately in the product requirements documentation.

---

## 1. Design Philosophy

ARCHITAX's visual personality is built around one governing idea: **calm authority**. The interface must feel like software an institution can trust with permanent public records — never flashy, never playful at the expense of legibility, never ambiguous about what is clickable or what state something is in.

Five descriptors anchor every visual decision:

- **Modern government software.** Contemporary interaction patterns (cards, pills, soft elevation) without ever tipping into consumer-app whimsy. The system should look like it was designed in the present decade, not a legacy 2010s admin panel, while still reading as serious and official.
- **Clean.** Surfaces are uncluttered. Every visible element earns its place. Decoration is never added "to fill space" — empty space itself is a deliberate, structural part of the design.
- **Lightweight.** Visual weight is kept low: thin borders over heavy ones, soft shadows over hard ones, regular-weight body text over bold blocks. The interface should feel airy even when displaying dense administrative data.
- **Spacious.** Generous outer whitespace at the page level, paired with disciplined, tighter spacing inside dense components like tables. This contrast — spacious macro layout, efficient micro layout — is what allows the system to handle archival record density without feeling cramped.
- **Trustworthy.** Color is used sparingly and only with meaning (status, category), never as decoration. Consistency of pattern (one card style, one table style, one button style) builds the kind of predictability that signals reliability over time.
- **Enterprise.** Designed for daily, repeated, professional use by trained operators — not first-time casual visitors. Every pattern optimizes for the 500th use, not the first impression.
- **Highly readable.** Strong contrast ratios, generous line-height, and a restrained type scale prioritize legibility over stylistic flourish at every size, from a 32px page title down to a 12px table caption.
- **Low cognitive load.** Visual hierarchy does the thinking for the user: the most important element on any screen is always the most visually prominent one, and everything else recedes in a predictable, learnable order.

---

## 2. Design Principles

These ten principles are the "why" behind every rule documented in the sections that follow. When a new component or pattern is needed that this document doesn't cover, derive it from these principles rather than inventing a new visual idiom.

**Hierarchy.** Every screen has exactly one primary action and one primary piece of information. Size, weight, and color are used — in that order of preference — to establish what matters most. Hierarchy is established structurally (heading levels, card position) before it is reinforced visually (color, weight).

**Consistency.** A button looks and behaves the same way everywhere it appears. A status color always means the same status, system-wide. Once a pattern is established for one use case, it is reused for every analogous use case rather than re-invented.

**Predictability.** Interactive elements look interactive (cursor, hover state, affordance) and static elements never do. Navigation position, action placement (primary actions trend right-aligned and bottom-right in forms, top-right in page headers), and component anatomy stay fixed across the entire application so users build durable muscle memory.

**Accessibility.** Legibility and operability are non-negotiable baseline requirements, not an enhancement layer applied afterward. Contrast ratios, focus states, and keyboard operability are designed in from the start (see Section 21).

**Information Density.** Density is tunable, not fixed. Default views favor comfortable spacing for scanning; power-user contexts (large tables, bulk record review) offer a compact density mode. The system never sacrifices legibility to fit more on screen — it offers an explicit density choice instead.

**Visual Rhythm.** Repeating spacing units (the 8px scale, see Section 6) and repeating component heights (40px controls, 56px rows) create a steady visual cadence down the page, so the eye can predict where the next element will sit before it's read.

**Progressive Disclosure.** Screens show the minimum needed to orient and act. Secondary detail lives behind a click (expand row, open drawer, open menu) rather than being rendered all at once. Density is added only on demand.

**Whitespace Usage.** Whitespace is treated as an active structural tool, not leftover space. It is used to group related elements (proximity), separate unrelated ones, and give every dense data surface room to breathe before the next section begins.

**Minimal Decoration.** No drop shadows, gradients, or color are applied purely for visual interest. Every shadow communicates elevation; every gradient (limited to the navigation shell, see Section 9) communicates "this is the application frame, not the content"; every color communicates a status or category.

**Visual Priority.** When two elements compete for attention, the one with the higher information value to the user's current task wins — not the one that's most visually elaborate. A red status badge on an overdue record will always outrank a decorative icon.

---

## 3. Color System

Color in ARCHITAX is functional first, branded second. The palette is deliberately restrained: a small set of neutrals carries almost all of the interface, and a small set of saturated accent colors is reserved exclusively for meaning — status, category, and primary action.

### 3.1 Core Palette

| Token                    | Hex                        | Usage                                                                                    |
| ------------------------ | -------------------------- | ---------------------------------------------------------------------------------------- |
| `color-primary`          | `#6C5CE0`                  | Primary actions, links, active nav state, focus ring, selection accents                  |
| `color-primary-hover`    | `#5B4BCB`                  | Hover/active state of primary-colored elements                                           |
| `color-primary-tint`     | `#EDE9FB`                  | Light background tint for selected rows, primary-colored badges, subtle highlight panels |
| `color-secondary`        | `#34C7B8`                  | Secondary accent for charts, secondary tags, supporting highlights                       |
| `color-bg-canvas`        | `#F4F4F9`                  | Outermost page background, behind the application shell                                  |
| `color-bg-surface`       | `#FFFFFF`                  | Cards, tables, modals, the main content surface                                          |
| `color-bg-sidebar-start` | `#6FD8E0`                  | Sidebar gradient start (top-left)                                                        |
| `color-bg-sidebar-end`   | `#B6A4F0`                  | Sidebar gradient end (bottom-right)                                                      |
| `color-bg-navbar`        | `#FFFFFF`                  | Top navigation bar background                                                            |
| `color-hover`            | `#F6F5FC`                  | Generic hover background for list rows, menu items, table rows                           |
| `color-active`           | `#EDE9FB`                  | Active/selected background (nav items, selected rows, selected tabs)                     |
| `color-border`           | `#E7E6F0`                  | Default border for inputs, cards, dividers between sections                              |
| `color-divider`          | `#EFEEF6`                  | Lighter hairline used inside cards (e.g., under a card header)                           |
| `color-text-primary`     | `#211E33`                  | Headings, body copy, primary readable text                                               |
| `color-text-secondary`   | `#8B87A0`                  | Meta text, captions, table headers, helper text                                          |
| `color-text-placeholder` | `#B5B1C6`                  | Input placeholder text                                                                   |
| `color-text-disabled`    | `#C7C4D6`                  | Disabled control labels                                                                  |
| `color-success`          | `#2FBE8F`                  | Positive status (verified, completed, approved)                                          |
| `color-success-tint`     | `#E2F8F0`                  | Background for success badges/alerts                                                     |
| `color-warning`          | `#F0A93B`                  | Caution status (pending, due soon, needs review)                                         |
| `color-warning-tint`     | `#FDF1DC`                  | Background for warning badges/alerts                                                     |
| `color-danger`           | `#F0507F`                  | Negative status (overdue, rejected, error)                                               |
| `color-danger-tint`      | `#FDE6ED`                  | Background for danger badges/alerts                                                      |
| `color-info`             | `#6C5CE0`                  | Informational status — reuses primary for consistency                                    |
| `color-info-tint`        | `#EDE9FB`                  | Background for info badges/alerts                                                        |
| `color-focus-ring`       | `rgba(108, 92, 224, 0.35)` | 3px outer glow on focused interactive elements                                           |

### 3.2 Usage Rules

Primary purple is reserved for things the user can _act on_: links, the primary button, the active sidebar item, the focus ring, and the send/submit affordance. It should never appear as a passive decorative fill.

Semantic colors (success, warning, danger) are always paired with a label or icon, never used as the sole indicator of meaning, to satisfy both accessibility requirements and the "trustworthy, unambiguous" philosophy. Each semantic color has a tint variant used as a _background_ (e.g., a warning badge uses `color-warning-tint` as its fill and `color-warning` as its text/icon color, never the saturated color as a large background fill).

The sidebar gradient (`color-bg-sidebar-start` → `color-bg-sidebar-end`, 135° diagonal) is the **one** permitted gradient in the entire system. It exists purely to visually frame the application shell against the page canvas and to give the product a recognizable identity at a glance. It is never used elsewhere — not on buttons, not on cards, not on charts.

Text color follows a strict two-tier system: `color-text-primary` for anything the user is meant to read as content, `color-text-secondary` for anything meant to be scanned as metadata (timestamps, counts, helper captions). No third gray is introduced.

---

## 4. Typography

### 4.1 Typeface

ARCHITAX uses a single geometric/humanist sans-serif family across the entire product — no serif, no monospace except for tabular numerals and document/reference IDs.

```
font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
```

Inter (or an equivalent humanist grotesk with similar metrics, e.g., a licensed alternative) is specified for its excellent legibility at small sizes, true tabular figures for data-heavy tables, and a friendly-but-neutral character that supports the "modern government software" philosophy without feeling cold.

### 4.2 Type Scale

| Role                      | Size | Weight         | Line height | Letter spacing     | Usage                                                   |
| ------------------------- | ---- | -------------- | ----------- | ------------------ | ------------------------------------------------------- |
| Display / Page Title      | 30px | 700 (Bold)     | 1.2         | -0.01em            | One per page, top of content area                       |
| H2 / Card & Section Title | 18px | 600 (Semibold) | 1.3         | -0.005em           | Card headers, panel titles                              |
| H3 / Subsection Title     | 16px | 600 (Semibold) | 1.35        | 0                  | Sub-groupings within a card                             |
| Body                      | 14px | 400 (Regular)  | 1.5         | 0                  | Default reading text, list row primary text             |
| Body Emphasis             | 14px | 600 (Semibold) | 1.5         | 0                  | List row primary text when it functions as a title/link |
| Small / Meta              | 13px | 400 (Regular)  | 1.45        | 0                  | Timestamps, secondary row text, helper text             |
| Caption / Micro           | 12px | 500 (Medium)   | 1.4         | 0.01em             | Table headers (often uppercase), section eyebrows       |
| Button                    | 14px | 600 (Semibold) | 1           | 0                  | All button labels                                       |
| Badge / Tag               | 12px | 600 (Semibold) | 1           | 0.01em             | Status pills, category tags                             |
| Input                     | 14px | 400 (Regular)  | 1.4         | 0                  | Field values; 400 weight even when filled               |
| Sidebar Nav Item          | 14px | 500 (Medium)   | 1.3         | 0                  | Default sidebar label weight                            |
| Sidebar Nav Item — Active | 14px | 600 (Semibold) | 1.3         | 0                  | Active state increases weight, not just color           |
| Sidebar Section Header    | 12px | 600 (Semibold) | 1.3         | 0.04em (uppercase) | "Favorites," "Teams"-style group labels                 |
| Top Navigation Title      | 30px | 700 (Bold)     | 1.2         | -0.01em            | Same as Display, lives in the header region             |

### 4.3 Spacing & Rhythm Rules

Paragraph spacing within body copy uses a flat 8px margin between blocks. Heading-to-content spacing is always 12px (H2/H3 to the content directly beneath it) and section-to-section spacing is always 32px, keeping the vertical rhythm aligned to the 8px base spacing scale (Section 6).

Numerals in tables, IDs, dates, and currency use tabular (fixed-width) figures so that columns of numbers align vertically without manual padding — critical for a records/archive system where operators scan long columns of reference numbers and tax assessment values.

---

## 5. Layout System

### 5.1 Page Structure

The application uses a fixed two-region shell: a persistent left sidebar and a fluid main content region containing a top header and scrollable body. The shell itself is desktop-first — designed and validated at 1440×900 first, then adapted down.

| Property                            | Value                                                             |
| ----------------------------------- | ----------------------------------------------------------------- |
| Application shell max width         | 1600px (centers on ultra-wide monitors)                           |
| Sidebar width (expanded)            | 264px                                                             |
| Sidebar width (collapsed)           | 72px                                                              |
| Content max width                   | 1280px (centered within the content region for very wide screens) |
| Content grid                        | 12-column, 24px gutter                                            |
| Default container padding (desktop) | 32px horizontal, 32px vertical                                    |
| Default container padding (tablet)  | 24px                                                              |
| Default container padding (mobile)  | 16px                                                              |

### 5.2 Grid Behavior

Dashboard-style screens use a 2-column macro layout at desktop width (roughly 60/40 or 65/35 split — primary data on the left, supporting widgets on the right), collapsing to a single stacked column below 1024px. Record/table-heavy screens use the full content width as a single column with the table as the dominant element. Form screens cap line length at roughly 640–720px even on wide screens, since long input rows hurt scannability.

### 5.3 Desktop-First Philosophy

Because ARCHITAX is an internal operator tool used primarily on desktop and laptop monitors during the working day, layout decisions are optimized for 1280px+ viewports first. Mobile and tablet are treated as "graceful degradation" breakpoints (Section 20) for occasional access, not as the primary design target — but they are never neglected or broken.

---

## 6. Spacing System

All spacing in the system is drawn from a single 8px base scale, with one 4px half-step for the tightest relationships (icon-to-label gaps, inline badge padding). No arbitrary spacing values (e.g., 10px, 18px, 30px) are used anywhere in the system.

| Token      | Value | Primary usage                                                                                                                    |
| ---------- | ----- | -------------------------------------------------------------------------------------------------------------------------------- |
| `space-1`  | 4px   | Icon-to-label gap; internal padding of small badges/tags; gap between a label and its required-indicator asterisk                |
| `space-2`  | 8px   | Gap between related inline elements (icon button + icon button); paragraph spacing; gap between sidebar nav items                |
| `space-3`  | 12px  | Internal vertical padding of inputs and buttons; gap between a table row's internal columns at compact density                   |
| `space-4`  | 16px  | Default gap between stacked elements in a card; compact card padding; gap between a heading and a single line of supporting text |
| `space-5`  | 20px  | Comfortable card padding (mobile/tablet default)                                                                                 |
| `space-6`  | 24px  | Standard card padding (desktop default); gap between cards in a grid; gap between sidebar group sections is 24–32px              |
| `space-8`  | 32px  | Gap between major page sections; page header bottom margin before content begins                                                 |
| `space-10` | 40px  | Gap between the top navigation bar and the start of page content on spacious dashboard views                                     |
| `space-12` | 48px  | Gap between unrelated page regions (e.g., end of a table, start of a new full-width section)                                     |
| `space-16` | 64px  | Outer page padding on very large desktop viewports (≥1600px); top padding above a page's Display title in hero-style views       |

**Rule of thumb:** as you move from "inside a component" → "between components" → "between sections" → "between page regions," spacing should always increase, never plateau or decrease. This monotonic increase is what produces the system's signature spacious-but-organized feel.

---

## 7. Border Radius

| Element                                           | Radius                                                                         |
| ------------------------------------------------- | ------------------------------------------------------------------------------ |
| Application shell (outer frame)                   | 24px                                                                           |
| Card                                              | 16px                                                                           |
| Compact card / widget tile                        | 12px                                                                           |
| Modal                                             | 20px                                                                           |
| Dropdown / popover panel                          | 12px                                                                           |
| Drawer                                            | 16px (leading edge only)                                                       |
| Button (rectangular)                              | 8px                                                                            |
| Input / Select / Textarea                         | 8px                                                                            |
| Search field / pill button                        | 999px (full pill)                                                              |
| Tag / Badge / Chip                                | 999px (full pill)                                                              |
| Avatar (default)                                  | 999px (circle)                                                                 |
| Avatar (square variant, e.g., colored icon tiles) | 12px                                                                           |
| Table outer container                             | 16px (top corners only need to round if the table sits directly inside a card) |
| Table row / cell                                  | 0px (rows are square; only the table's outer container is rounded)             |

Radius increases with a component's visual "weight": small interactive controls (buttons, inputs) get a moderate 8px, larger resting surfaces (cards, modals) get a more generous 16–20px, and fully pill-shaped elements (search, tags, avatars) signal "this is a discrete, draggable/removable/identity token," not a content container.

---

## 8. Shadows

Shadows are used exclusively to communicate elevation (what is "above" what), never for decoration. They are soft, low-opacity, and cool-toned (matching the violet-leaning neutral palette) rather than pure black.

| Level           | Value                                                                | Usage                                                                                                                  |
| --------------- | -------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Level 0         | `none`                                                               | Flat resting elements: sidebar items, table rows, inline content                                                       |
| Level 1         | `0 1px 2px rgba(33, 30, 51, 0.04), 0 1px 1px rgba(33, 30, 51, 0.03)` | Default resting elevation for cards and the navbar                                                                     |
| Level 2         | `0 4px 12px rgba(33, 30, 51, 0.07)`                                  | Hover state on interactive cards/tiles; open dropdown menus                                                            |
| Level 3         | `0 10px 24px rgba(33, 30, 51, 0.10)`                                 | Popovers, date pickers, notification panels                                                                            |
| Modal           | `0 20px 48px rgba(33, 30, 51, 0.18)`                                 | Modal dialogs over a dimmed backdrop                                                                                   |
| Dropdown        | `0 6px 16px rgba(33, 30, 51, 0.08)`                                  | Context menus, select panels                                                                                           |
| Floating Action | `0 8px 20px rgba(108, 92, 224, 0.30)`                                | Primary floating action button — the one place a colored (not neutral) shadow is used, tinted to match `color-primary` |

The application shell itself sits on the page canvas with a Level 2-equivalent shadow, visually separating "the product" from "the browser/desktop background" — reinforcing that everything inside the rounded shell is one cohesive system.

---

## 9. Sidebar System

The sidebar is the primary navigation anchor and the single most identity-bearing surface in the system.

| Property                                             | Value                                                                      |
| ---------------------------------------------------- | -------------------------------------------------------------------------- |
| Width (expanded)                                     | 264px                                                                      |
| Width (collapsed, icon-only)                         | 72px                                                                       |
| Outer padding                                        | 24px top/sides                                                             |
| Background                                           | Diagonal gradient, `color-bg-sidebar-start` → `color-bg-sidebar-end`, 135° |
| Nav item height                                      | 40px                                                                       |
| Gap between nav items                                | 4px                                                                        |
| Gap between groups (Primary nav / Favorites / Teams) | 28px                                                                       |
| Icon size                                            | 20px, stroke-based, 1.75px stroke weight                                   |
| Icon-to-label gap                                    | 12px                                                                       |

**Active state:** the active nav item renders on a solid white pill (`color-bg-surface`) at full item width, radius 10px, with a Level 1 shadow, primary-colored icon and semibold primary-colored text. This is the only place inside the sidebar where a flat white surface interrupts the gradient — it is intentional and exists purely to make "where am I" unmistakable against the colorful background.

**Hover state:** non-active items get a translucent white overlay (`rgba(255,255,255,0.35)`) on hover, radius 10px, no shadow — a lighter touch than the active state so the two are never confused.

**Section title:** 12px, uppercase, 600 weight, 0.04em letter-spacing, rendered in a darker, higher-contrast tone against the gradient (`rgba(33,30,51,0.65)`) so it remains legible across the gradient's color range. 8px margin-bottom before the first item in its group.

**Favorites section:** a flat list of pinned/starred items, each row using the same 40px height and pill-hover pattern as primary nav, prefixed with a small outline star icon at 16px.

**Workspace/Group section ("Teams"-equivalent):** expandable rows. Each row shows a small colored rounded-square icon (12px radius, 24px size, one flat accent color per group) + label + a trailing chevron at the far right. Expanding reveals an indented sub-list (additional 16px left indent) of child items, each at the same 40px row height. Chevron rotates 90° on expand, 180ms ease.

**Scrollbar:** thin (6px), fully transparent at rest, fading to a low-contrast white-translucent thumb (`rgba(255,255,255,0.4)`) on hover/scroll only — never a default OS scrollbar.

**Bottom/utility actions:** persistent actions like "add a new group" render as a ghost-style row identical in height/padding to a nav item, with a leading "+" icon in a dashed or outline circle, sitting at the end of its section rather than visually separated — reinforcing that it's an extension of the list, not a separate toolbar.

**Avatar:** the signed-in user's avatar (32px circle) lives in the top navigation bar, not the sidebar, keeping the sidebar focused purely on wayfinding.

**Animation:** expand/collapse of the whole sidebar animates width over 200ms ease-in-out; labels fade out (120ms) slightly before the width transition completes when collapsing, and fade in (150ms) slightly after the width transition completes when expanding, avoiding any text "squishing" artifact.

---

## 10. Top Navigation

| Property          | Value                                                                                                                                                                                       |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Height            | 76px                                                                                                                                                                                        |
| Background        | `color-bg-navbar` (white), flush with the content surface                                                                                                                                   |
| Page title        | Display style (30px/700), left-aligned, vertically centered in the bar                                                                                                                      |
| Search bar        | Pill, 999px radius, 40px height, 360px default width (grows to 480px on focus), background `color-hover`, left-aligned leading search icon at 16px, placeholder in `color-text-placeholder` |
| Avatar            | 36px circle, right-aligned, with an optional 8px status/notification dot                                                                                                                    |
| Notification icon | 20px outline bell, positioned left of the avatar with 16px gap, badge count in a small `color-danger` circle, top-right offset -2px/-2px                                                    |
| Spacing           | 24px between major header elements (title block, search, notification, avatar)                                                                                                              |
| Alignment         | Title left, utility cluster (search → notification → avatar) right, single row, space-between                                                                                               |
| Sticky behavior   | Sticky to the top of the scrollable content region; remains fixed while page content scrolls beneath it                                                                                     |
| Shadow            | None at scroll position 0; transitions to Level 1 shadow once content has scrolled beneath it (`scrollY > 0`), giving a subtle separation cue without a permanent hard line                 |

---

## 11. Cards

Cards are the primary content-grouping container for dashboard and summary views.

| Property        | Value                                                                                                                                                        |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Padding         | 24px (standard), 20px (compact/tablet)                                                                                                                       |
| Corner radius   | 16px                                                                                                                                                         |
| Background      | `color-bg-surface`                                                                                                                                           |
| Border          | 1px solid `color-border` (used in addition to, not instead of, the Level 1 shadow — both together produce the soft, defined edge seen throughout the system) |
| Title           | H2 style (18px/600), top-left of the card header row                                                                                                         |
| Subtitle/meta   | Small style (13px/400, `color-text-secondary`), directly under the title with 2px gap, or inline as a count badge next to the title                          |
| Icon            | Optional leading icon at 18px before the title, or a colored icon tile (Section 17-adjacent pattern) for "feature tile" style cards                          |
| Header divider  | 1px `color-divider` hairline beneath the header row, 16px margin below the header text and 16px above the card's primary content                             |
| Hover           | Only applies to _interactive_ cards (clickable tiles): Level 1 → Level 2 shadow transition, 150ms ease; static information cards never change on hover       |
| Selection       | 2px `color-primary` border replacing the default 1px neutral border, plus a faint `color-primary-tint` background wash                                       |
| Elevation       | Level 1 at rest; Level 2 on hover for interactive cards only                                                                                                 |
| Spacing in grid | 24px gutter between cards, both horizontal and vertical                                                                                                      |
| Grid behavior   | 2-up on desktop dashboards, collapsing to 1-up at tablet/mobile (Section 20)                                                                                 |

A header row commonly includes, right-aligned, one or two of: a text link ("View all"-equivalent, primary color, 13px/600), a small count badge, and an icon-only overflow ("⋯") button — always in that left-to-right priority order, with 12px between each.

---

## 12. Tables

ARCHITAX is a records-heavy product, so the table component is one of the most-used and most-tuned patterns in the system. Two density modes exist: **Comfortable** (default) and **Compact** (opt-in, for power users reviewing large record sets).

| Property                | Comfortable | Compact |
| ----------------------- | ----------- | ------- |
| Row height              | 56px        | 40px    |
| Cell horizontal padding | 16px        | 12px    |
| Header row height       | 44px        | 36px    |

**Header:** 12px caption-style text, `color-text-secondary`, 600 weight, often uppercase with 0.04em letter-spacing; 1px `color-border` bottom border separating header from body; header cells are left-aligned for text columns and right-aligned for numeric columns (critical for scanning assessed values and reference numbers).

**Row hover:** background shifts to `color-hover`, 100ms ease, cursor becomes pointer if the row is clickable; no shadow or scale change (rows must stay perfectly aligned, unlike cards).

**Selected row:** background `color-primary-tint`, plus a 3px solid `color-primary` left-edge accent bar, so multi-selected rows remain visible even when scrolled and the checkbox column is out of view.

**Status badge:** rendered in the rightmost-but-one column by convention, using the pill badge component (Section 15) with semantic coloring.

**Action menu:** trailing column, icon-only overflow button (20px, ghost style), invisible at rest (opacity 0) and fading to opacity 1 on row hover/focus — keeping rows visually quiet until the user's attention is on that specific row.

**Pagination:** bottom-right of the table container, format "Showing 1–25 of 248," with ghost-style previous/next chevron icon buttons (36px) and, optionally, a compact page-size selector dropdown to its left.

**Sorting:** clickable header labels show a small 12px caret on hover; the actively sorted column keeps its caret visible and renders its header label in `color-text-primary` instead of secondary, with the caret oriented to match sort direction.

**Sticky header:** for any table taller than the viewport, the header row sticks to the top of the scrollable region and gains a Level 1 shadow once the body has scrolled beneath it — identical pattern to the top navigation bar's scroll behavior, reinforcing system-wide consistency.

---

## 13. Forms

| Component            | Specification                                                                                                                                                                        |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Input (text)         | Height 40px, padding 10px 14px, 1px solid `color-border`, radius 8px, 14px/400 value text, placeholder in `color-text-placeholder`                                                   |
| Input — focus        | Border becomes `color-primary`; an additional 3px outer glow using `color-focus-ring` is applied around the entire control                                                           |
| Dropdown / Select    | Same base styling as text input, with a trailing 16px chevron icon; open panel uses Dropdown shadow, 12px radius, 36px item rows, `color-hover` on item hover                        |
| Textarea             | Same base styling, 80px minimum height, vertical-only resize handle                                                                                                                  |
| Checkbox             | 18×18px box, 4px radius, 1.5px border at rest; checked state fills solid `color-primary` with a white 1.5px-stroke checkmark icon                                                    |
| Radio                | 18×18px circle, 1.5px border at rest; checked state shows an 8px solid `color-primary` inner dot centered in the circle                                                              |
| Date picker          | Popover panel, Level 2 shadow, 12px radius; selected date is a filled `color-primary` circle, today's date is an outline-only circle, disabled dates render in `color-text-disabled` |
| Search field         | Pill radius (999px), 40px height, leading icon, otherwise matches the input spec                                                                                                     |
| Validation — error   | Border becomes `color-danger`; 12px helper text in `color-danger` appears 4px below the field, prefixed with a small warning icon                                                    |
| Validation — success | Border becomes `color-success`; optional trailing check icon inside the field                                                                                                        |
| Disabled             | Background `color-hover`, text `color-text-disabled`, border unchanged but lower contrast, cursor `not-allowed`, no focus ring possible                                              |
| Read-only            | No border, transparent/very light background tint, text in full `color-text-primary` (distinguished from disabled by remaining fully legible), no focus ring                         |
| Required indicator   | A `color-danger` asterisk, `space-1` (4px) after the field label, never used on its own without a label                                                                              |

Field labels sit 8px above their control, 13px/600 weight, `color-text-primary`. Helper text (non-error) sits 4px below the control, 13px/400, `color-text-secondary`. Vertical gap between stacked form fields is `space-5` (20px); gap between distinct form sections is `space-8` (32px).

---

## 14. Buttons

| Variant     | Background         | Text                                    | Border             | Hover                                                | Active                    |
| ----------- | ------------------ | --------------------------------------- | ------------------ | ---------------------------------------------------- | ------------------------- |
| Primary     | `color-primary`    | White                                   | none               | `color-primary-hover`                                | darken further ~6%        |
| Secondary   | `color-bg-surface` | `color-text-primary`                    | 1px `color-border` | `color-hover` background                             | `color-active` background |
| Ghost       | transparent        | `color-primary` or `color-text-primary` | none               | `color-hover` background                             | `color-active` background |
| Danger      | `color-danger`     | White                                   | none               | darken ~8%                                           | darken further ~6%        |
| Icon button | transparent        | inherits context color                  | none               | `color-hover` background, radius matches button size | `color-active` background |

| Size             | Height | Horizontal padding | Text size |
| ---------------- | ------ | ------------------ | --------- |
| Small            | 32px   | 12px               | 12px      |
| Medium (default) | 40px   | 16px               | 14px      |
| Large            | 48px   | 20px               | 16px      |

All buttons share an 8px corner radius (except icon-only buttons used inside pill contexts, which may be circular) and a 150ms ease background-color transition on hover/active.

**Loading state:** the label is replaced by a centered spinner (matching the button's text color) and the button's width is locked to its pre-loading width to prevent layout shift; the control becomes non-interactive but does not visually dim.

**Disabled state:** 40% opacity applied to the entire button, no hover/active transitions fire, cursor `not-allowed`.

Primary buttons are reserved for the single most important action in any given context (one primary button per form/modal/toolbar); every other action uses Secondary or Ghost, preserving the "one primary action per screen" hierarchy principle from Section 2.

---

## 15. Status Components

**Badge:** the core status-communication unit. Pill shape (999px radius), 4px vertical / 10px horizontal padding, 12px/600 text, background = semantic `-tint` color, text/icon = semantic full color. Used for single, non-removable status labels (e.g., a record's current state).

**Tag:** visually identical to a badge but includes an optional trailing 12px "×" remove icon, used wherever a value is user-removable (e.g., an applied filter, a selected category in a multi-select).

**Chip:** a tag variant used specifically for active filters in toolbar contexts; may include a small leading icon in addition to the label and remove icon, and uses a neutral (`color-hover` background, `color-text-primary` text) palette rather than semantic color, since a filter chip's color shouldn't be confused with a status badge's color.

**Progress bar:** 6px height track in `color-border`, 999px radius, filled portion in `color-primary` (or `color-success` for "complete" states) with the same radius; optional percentage label in 12px/600 `color-text-secondary` right-aligned above the bar.

**Toast:** floating notification, bottom-right by default, 360px width, white background, Level 3 shadow, 12px radius, leading 20px semantic-colored status icon, 14px message text, auto-dismiss after 5s indicated by a thin progress underline that depletes left-to-right. Enters via a 200ms ease-out slide-and-fade from the bottom edge; exits via a faster 150ms ease-in fade.

**Alert (inline banner):** full-width within its container, 4px solid left border in the semantic color, `-tint` background fill, 16px padding, leading semantic icon, message text, optional dismiss "×" top-right.

**Notification panel:** triggered from the top navigation bell icon, dropdown-style panel (Dropdown shadow, 12px radius, 360px width), list of notification rows at 64px height each (avatar/icon + two lines of text + timestamp), `color-hover` on row hover, unread rows marked with a small `color-primary` dot in the leading position.

---

## 16. Icons

- **Style:** outline/stroke-based throughout — no filled icon glyphs in the default state (filled variants may be used sparingly to indicate an _active/selected_ toggle state only, e.g., a filled star for "favorited").
- **Stroke weight:** 1.75px at default sizes, scaling proportionally at larger icon sizes (e.g., ~2px at 24px+).
- **Corner treatment:** rounded line caps and joins throughout — no sharp/mitered corners — to match the friendly-but-professional character of the typeface and the rounded card/button geometry.
- **Sizes:** 16px (inline/dense contexts, table action icons), 18–20px (default — nav items, buttons, form field icons), 24px (empty states, feature callouts, larger status icons).
- **Spacing:** 8px gap between an icon and its adjacent text label; icons are optically centered against the cap-height of adjacent text, not the full line-height box.
- **Color:** icons default to `currentColor`, inheriting the text color of their context; semantic icons (status, alerts) take the matching semantic color explicitly.

---

## 17. Avatar System

| Size | Pixels | Usage                                       |
| ---- | ------ | ------------------------------------------- |
| XS   | 24px   | Dense table rows, notification rows         |
| S    | 32px   | Default list-row avatar, sidebar references |
| M    | 40px   | Top navigation, header contexts             |
| L    | 64px   | Profile/detail hero contexts                |

**Grouping:** overlapping stacks use a -8px left margin per subsequent avatar, with a 2px white ring (`border: 2px solid color-bg-surface`) around each to maintain separation against busy backgrounds. When more than 4 collaborators/assignees exist, the 4th position becomes a "+N" circle in a neutral `color-hover` background with `color-text-secondary` text instead of an image.

**Status indicator:** an 8–10px circle anchored to the bottom-right of the avatar, with its own 2px white ring, colored per state (`color-success` = active, `color-text-disabled` gray = inactive, `color-warning` = away/pending).

**Fallback:** when no image is available, avatars render as initials (first + last initial, uppercase, white, 600 weight) over a flat, deterministic background color selected from a fixed 8-color accent rotation keyed to the user's identifier — ensuring the same person always gets the same fallback color across the entire application.

---

## 18. Navigation Pattern

ARCHITAX's information architecture is expressed through four nested levels:

1. **Sidebar (Level 1 — Destinations):** the persistent left rail; always visible (or collapsed to icons) and represents the top-level sections of the product.
2. **Page Header (Level 2 — Current Location):** the Display-style title in the top navigation area names exactly where the user is.
3. **Tabs (Level 3 — Sub-views):** underline-style tabs sit directly beneath the page header when a destination has multiple views of the same data (e.g., a list view vs. a board view of the same record set). Active tab: 2px solid `color-primary` bottom border, 14px/600 text in `color-text-primary`; inactive tabs: 14px/500 text in `color-text-secondary`, no border, with `color-hover` background on hover. 24px gap between tab labels.
4. **Breadcrumb (Level 4 — Record depth):** used only on deeply nested record detail pages, 13px text, `color-text-secondary` for ancestor segments and `color-text-primary`/600 for the current segment, separated by a 14px chevron icon with 8px gaps on either side.

**Context navigation:** record-specific secondary actions and metadata live in a right-side drawer (Section 9-adjacent drawer pattern) rather than crowding the primary page header, keeping the main navigation chrome stable regardless of what record is open.

**Section navigation:** very long single-page detail views (e.g., a record with many data groupings) use a sticky in-page sub-header with anchor jump links rather than paginating into multiple page loads, preserving context for the user.

---

## 19. Motion System

Motion in ARCHITAX is purposeful and fast — it confirms what just happened, it never delays the user, and it is the first thing simplified or removed under a "reduced motion" accessibility preference.

| Interaction                           | Duration             | Easing                         | Notes                                                                                   |
| ------------------------------------- | -------------------- | ------------------------------ | --------------------------------------------------------------------------------------- |
| Hover (color/background)              | 120–150ms            | ease                           | No movement, color/background only                                                      |
| Interactive card hover (lift)         | 150ms                | ease-out                       | Shadow Level 1 → Level 2; optional 1.01–1.02 scale, never more                          |
| Fade (tooltips, menu items appearing) | 150ms                | ease                           | Opacity only                                                                            |
| Dropdown/menu open                    | 150ms                | ease-out                       | Scale 95% → 100% + opacity 0 → 1, transform-origin nearest the trigger                  |
| Drawer / side panel                   | 250ms                | cubic-bezier(0.2, 0.8, 0.2, 1) | Slides in from the right edge                                                           |
| Toast                                 | 200ms in / 150ms out | ease-out / ease-in             | Slide + fade from the screen edge                                                       |
| Modal                                 | 200ms in / 150ms out | ease-out / ease-in             | Backdrop fade + panel scale 95% → 100%                                                  |
| Sidebar collapse/expand               | 200ms                | ease-in-out                    | Width transition; label opacity offset slightly from the width animation                |
| Skeleton loading shimmer              | 1.5s loop            | linear                         | Used for content placeholders during data fetch                                         |
| Button spinner                        | 800ms loop           | linear                         | Rotation only                                                                           |
| Route/page content change             | 150–200ms            | ease                           | Cross-fade of content region only; sidebar and top nav remain static (persistent shell) |

**General rule:** nothing in the interface animates longer than 300ms. Enterprise operators perform the same actions hundreds of times a day; motion exists to provide feedback, never to entertain, and any animation that begins to feel slow on the 50th repetition is too slow.

---

## 20. Responsive Rules

| Breakpoint | Range       | Sidebar                                                                  | Content grid           | Tables                                                                                | Forms                                                                |
| ---------- | ----------- | ------------------------------------------------------------------------ | ---------------------- | ------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| Desktop    | ≥1440px     | Expanded (264px)                                                         | Up to 2–3 columns      | Full multi-column table                                                               | Capped line length (~720px), centered or left-aligned within content |
| Laptop     | 1024–1439px | Expanded (264px)                                                         | 2 columns              | Full table, reduced outer padding                                                     | Same as desktop                                                      |
| Tablet     | 768–1023px  | Auto-collapses to icon-only (72px), expandable via a flyout on hover/tap | 1 column (cards stack) | Full table retained, horizontal scroll permitted within the table container if needed | Single column, full-width fields                                     |
| Mobile     | <768px      | Off-canvas drawer, triggered by a hamburger control in the top bar       | 1 column               | Rows convert to stacked label/value mini-cards (no horizontal scroll)                 | Single column, all touch targets ≥44px                               |

At every breakpoint, the core component anatomy (radius, color, type scale) stays identical — only layout density, column count, and the sidebar's visibility mode change. This is what keeps the product feeling like "the same system" regardless of device.

---

## 21. Accessibility

- **Contrast:** all body text maintains a minimum 4.5:1 contrast ratio against its background; large text (≥18px bold or ≥24px regular) maintains a minimum 3:1. Semantic colors were chosen and tinted specifically to clear these thresholds against both white and tint backgrounds.
- **Keyboard navigation:** the full interface — sidebar, header, every table row action, every form control, every modal/drawer — is operable via keyboard alone, in a logical top-to-bottom, left-to-right tab order matching visual layout.
- **Focus:** every focusable element shows a visible focus ring (`color-focus-ring`, 3px, 2px offset) at all times; focus styling is never suppressed, including on custom-built controls like checkboxes, the date picker, and dropdowns.
- **ARIA:** the sidebar uses `role="navigation"`; the active item carries `aria-current="page"`; expandable sidebar groups carry `aria-expanded`; toasts and async status changes use `aria-live="polite"` regions; icon-only buttons always carry an `aria-label`.
- **Screen reader support:** all icon-only controls and avatar images include meaningful text alternatives; status badges announce their semantic meaning as text (not implied by color alone); table headers use `scope="col"` and, where applicable, `scope="row"`.
- **Touch target:** every interactive element maintains a minimum 44×44px hit area on touch-capable viewports, even where the visible element (e.g., a 32px icon button) is smaller — achieved via invisible padding rather than visually enlarging the control.

---

## 22. Enterprise UX Rules

ARCHITAX is used by trained operators, daily, for years — not by casual first-time visitors. Every rule below exists to optimize for that reality:

- **Dense information without clutter.** Generous macro whitespace (page padding, section gaps) paired with efficient micro spacing (table rows, form fields) lets the system display real archival density without ever feeling chaotic.
- **Fast scanning.** Left-aligned text columns, right-aligned numeric columns, and consistent semantic color-coding let an experienced operator identify a record's status from peripheral vision alone, without reading every label.
- **Minimal distraction.** No marketing-style imagery, illustration, or decorative gradients appear anywhere inside the working application surfaces — the single sidebar gradient is the one deliberate exception, reserved purely for shell identity.
- **Long-term usability.** Typography, iconography, and color choices were selected to avoid short-lived visual trends (no heavy skeuomorphism, no excessive blur/glassmorphism, no neon palettes), so the system remains comfortable to use — and inexpensive to maintain — for years.
- **Operator efficiency.** Compact density modes, keyboard operability, persistent sidebar/filter state, and bulk-action affordances in tables are first-class, not afterthoughts — this is software people work _in_, not software people occasionally visit.
- **Predictable patterns.** A single card pattern, a single table pattern, and a single form pattern are reused across the entire application; a new screen should always feel instantly familiar to anyone who has used any other screen in the system.

---

## 23. Component Inventory

The following is the canonical list of reusable visual components defined by this system. Any new screen should be composed from this inventory before a new component pattern is proposed.

- Application Shell (Sidebar + Top Navigation + Content Region)
- Sidebar Navigation Item
- Sidebar Section Header
- Sidebar Expandable Group (with chevron + child rows)
- Top Navigation Bar / Page Header
- Global Search Field
- User Avatar Menu
- Notification Bell + Notification Panel
- Card (Standard, Compact, Interactive Tile)
- Colored Icon Tile (used for feature/shortcut tiles)
- Data Table (Header Row, Body Row, Sticky Header variant, Compact density variant)
- Simplified List Row (lightweight table row used inside dashboard cards)
- Status Badge
- Tag (removable)
- Filter Chip
- Avatar / Avatar Group
- Button (Primary, Secondary, Ghost, Danger, Icon-only) × (Small, Medium, Large)
- Text Input
- Select / Dropdown
- Textarea
- Checkbox
- Radio
- Date Picker
- Search Field
- Modal / Dialog
- Drawer / Side Panel
- Tooltip
- Toast
- Inline Alert / Banner
- Pagination Control
- Breadcrumb
- Tabs (underline style)
- Accordion / Expandable Row
- Progress Bar
- Empty State
- Skeleton Loader

---

## 24. Implementation Notes

This system should be implemented as a centralized design token layer (CSS custom properties, a Tailwind theme extension, or an equivalent theme object) consumed by every component — never as hard-coded values inside individual screens or components. Recommended token namespacing mirrors the tables above: `color-*`, `space-*`, `radius-*`, `shadow-*`, `font-*`, and `motion-*`.

When extending this system for a new screen or component not explicitly covered above, derive every decision from Section 2's principles in this priority order: hierarchy and predictability first, then consistency with the closest existing pattern in Section 23, then accessibility compliance per Section 21 — and only then visual refinement. A new pattern that cannot be justified against these principles should be brought back to the design system owners before being shipped.

---

_End of document._
