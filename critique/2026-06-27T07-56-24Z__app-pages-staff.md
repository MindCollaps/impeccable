---
target: app/pages/staff
total_score: 15
p0_count: 1
p1_count: 3
timestamp: 2026-06-27T07-56-24Z
slug: app-pages-staff
---
# Critique: app/pages/staff

## Design Health Score

| # | Heuristic | Score | Key Issue |
|---|-----------|-------|-----------|
| 1 | Visibility of System Status | 2 | No loading state on setRepairStatus/archiveRequest; no explanation when allWorkItemsDone gate hides action buttons |
| 2 | Match System / Real World | 1 | Raw enum strings as button labels (IN_DIAGNOSIS, WAITING_FOR_PARTS); German/English mixing within a single page; "Reperaturauftrag" typo |
| 3 | User Control and Freedom | 2 | No confirmation on REJECT/CANCEL/ARCHIVE — permanent destructive actions fire on single click |
| 4 | Consistency and Standards | 2 | Native select uses 1px static border + box-shadow vs. UiInputText ghost border; resolveDisplayStatus duplicated across pages |
| 5 | Error Prevention | 1 | No confirmation dialogs for destructive closures; allWorkItemsDone blocks COMPLETE but nothing blocks REJECT/CANCEL |
| 6 | Recognition Rather Than Recall | 2 | Enum labels require memorization; queuePosition buried in h2 string; no tooltips/inline guidance |
| 7 | Flexibility and Efficiency | 1 | No keyboard shortcuts; card requires "Details" button click (no whole-card navigation); no sort/filter beyond status; no batch actions |
| 8 | Aesthetic and Minimalist Design | 2 | Four panels all equal visual weight; device panel packs create + edit + info + repair status + 6+ buttons with no visual separation |
| 9 | Error Recovery | 1 | setRequestState rolls back on error but throws createError which may produce unhandled error page; no user-visible toast on failure; no success confirmation on device save |
| 10 | Help and Documentation | 1 | No tooltips; no inline help; no explanation of why state-change buttons are absent when work items aren't done |
| **Total** | | **15/40** | **Poor — major UX overhaul required** |

## Anti-Patterns Verdict

**LLM assessment**: Not AI-generated slop in the aesthetic sense — the component vocabulary (semantic status badges, flat dark cards, ghost borders) is genuine product design with real personality. The failure is functional, not stylistic: raw enum strings as action buttons, language chaos within a single view, and four structurally equal panels with no hierarchy create a surface that looks coherent from 10 feet but breaks down the moment a staff member needs to act under pressure.

**Deterministic scan**: The staff page files themselves are clean (0 findings). Component dependencies have 33 findings:
- **26 × `design-system-radius`**: Hardcoded border-radius values (`999px`, `3px`, etc.) across 12 components bypass the 4px/8px token pair defined in DESIGN.md. Worst in `HomeImagePlaceholder.vue` (5×), `RepairStepPhase.vue` (3×), `RepairWorkItemCard.vue` (2×).
- **10 × `design-system-color`**: Hardcoded colors in `RepairTimeline.vue` (#64748b, #3b82f6, #f59e0b, #ef4444, #8b5cf6, #06b6d4, #14b8a6) are Tailwind-style values that bypass the design system's semantic color map entirely. The project's semantic warning is `#bb9d58`; the timeline uses `#f59e0b`. These are materially different colors from a different system.
- **1 × `side-tab` (warning)**: `RepairWorkItemCard.vue` line 122 — `border-left: 4px solid var(--accent-color)` — this is an absolute ban in DESIGN.md ("side-stripe borders, never intentional").

The detector caught things Assessment A missed: the RepairTimeline color set is a completely parallel color system coexisting with the official one, and the border-left side-stripe is the single most cited AI-generated anti-pattern.

## Overall Impression

The bones are solid — the dark substrate, semantic status vocabulary, and optimistic updates are real design decisions. The problem is the detail page: it's a 4-column equal-weight grid that surfaces everything simultaneously (device info, repair status phases, work items, customer notes) with no visual hierarchy telling staff where to look first. Combine that with raw enum button labels, no destructive action confirmation, and a language that can't decide if it's German or English, and the most complex surface in the tool is also the most dangerous one to use under time pressure.

## What's Working

- **Semantic status badges** — the color-to-state mapping (violet=active, amber=waiting, green=done, rose=closed) is consistent everywhere and immediately legible without training. Staff know where a request stands at a glance.
- **The allWorkItemsDone gate** — blocking COMPLETE until all work items are done is smart workflow enforcement. The system knows the sequence; it protects staff from premature closure.
- **Optimistic updates in setRequestState** — immediate local status mutation with rollback on error is the right pattern. The UI feels fast; failures don't corrupt state.

## Priority Issues

**[P0] No confirmation on REJECT / CANCEL / ARCHIVE**
- **Why it matters**: These are permanent, customer-facing business decisions. REJECT ends the repair relationship. CANCEL may trigger communication obligations. ARCHIVE closes the record. All fire on a single click with no confirmation, no consequence preview, no undo. One misclick costs a customer.
- **Fix**: Gate all three behind `CommonPopup` with subject line + plain-language consequence string: "Das lehnt die Reparaturanfrage von [customer] dauerhaft ab. Dies kann nicht rückgängig gemacht werden." Require explicit confirm click, not just dismiss.
- **Suggested command**: `/impeccable harden app/pages/staff/request/[id].vue`

**[P1] Raw RepairStatus enum strings as button labels**
- **Why it matters**: "IN_DIAGNOSIS", "WAITING_FOR_PARTS", "IN_OUTGOING" are code identifiers, not workshop language. Staff operating under time pressure must mentally translate enum convention to repair phase meaning before every click. The RepairTimeline already has a German STATUS_LABELS map — it's unused here.
- **Fix**: Apply the existing German STATUS_LABELS map (from `RepairTimeline.vue`) to `repairStatusActions` button rendering. "Diagnose" beats "IN_DIAGNOSIS" in every possible way.
- **Suggested command**: `/impeccable clarify app/pages/staff/request/[id].vue`

**[P1] German/English language mixing within a single page**
- **Why it matters**: "Reperaturauftrag von" (typo + German), "Customer Notes" (English), "Frag den Customer" (hybrid), "Display Name" (English field label), "Hat versucht" (German freeform) — the detail page switches languages at the word level. This erodes trust and signals an incomplete product.
- **Fix**: Decide on one language for all UI copy (likely German, given the target market). Fix "Reperaturauftrag" → "Reparaturauftrag". "Customer Notes" → "Kundennotizen". Establish a single-language pass as a prerequisite to any other copy work.
- **Suggested command**: `/impeccable clarify app/pages/staff/request/[id].vue`

**[P1] Up to 6+ equal-weight primary buttons for repair status transitions**
- **Why it matters**: `repairStatusActions` renders every possible status transition as a violet primary button with no indication of which is the expected next step vs. a regression or edge case. There is a deterministic sequence (RECEIVED → IN_DIAGNOSIS → WAITING_FOR_PARTS → IN_REPAIR → IN_QA → IN_OUTGOING). That sequence is invisible in the UI.
- **Fix**: Show only the single next-step transition as primary. Demote out-of-sequence options to ghost/secondary buttons inside a "Weitere Status" disclosure. The sequence is known; express it visually.
- **Suggested command**: `/impeccable layout app/pages/staff/request/[id].vue`

**[P2] Request list cards not clickable — only the "Details" button navigates**
- **Why it matters**: Every `.req` card is a visual affordance that implies clickability. Staff must locate and click a button at the bottom of each card rather than tapping the card body. Two interactions where one is expected.
- **Fix**: Wrap `.req` in a `NuxtLink` to `/staff/request/${r.id}`. The "Details" button can remain as a visual anchor. The whole card should be the hit area.
- **Suggested command**: `/impeccable polish app/pages/staff/request/index.vue`

**[P2] RepairTimeline uses a parallel, incompatible color system**
- **Why it matters**: The Gantt bars use Tailwind-style hardcoded colors (`#f59e0b` for warning, `#3b82f6` for primary, `#ef4444` for error) instead of the design system's semantic tokens (`#bb9d58`, `#7c59bc`, `#bb5866`). Customers see the timeline — the status colors they've learned from status badges mean something different in the Gantt chart.
- **Fix**: Replace all 7 hardcoded hex values in `RepairTimeline.vue` with the project's `colorsList` tokens: `colorsList.warning600`, `colorsList.primary600`, `colorsList.error600`, etc.
- **Suggested command**: `/impeccable colorize app/components/repair/RepairTimeline.vue`

**[P2] Side-stripe border in RepairWorkItemCard (absolute DESIGN.md ban)**
- **Why it matters**: `RepairWorkItemCard.vue:122` — `border-left: 4px solid var(--accent-color)` — this is the single most cited AI-generated anti-pattern. It's explicitly prohibited in DESIGN.md.
- **Fix**: Replace with a full-border container tinted with the accent color at 10% opacity, or a leading status badge/icon. Remove the side stripe entirely.
- **Suggested command**: `/impeccable polish app/components/repair/RepairWorkItemCard.vue`

## Persona Red Flags

**Alex (Power User / Staff veteran)**: Cards require navigating to the "Details" button rather than clicking anywhere on the card — double the clicks for every record. No keyboard shortcuts for primary workflow actions (accept, set status, archive). No sort controls on the request list (newest, most urgent, by status, by assigned user). Repair status transitions require scanning 6 equal-weight violet buttons with no sequencing cue. No batch operations on the list.

**Jordan (First-Timer / New staff member)**: "IN_DIAGNOSIS" and "WAITING_FOR_PARTS" as action button labels require knowing the codebase enum convention before operating a core workflow. The complete/accept/reject buttons are hidden until allWorkItemsDone with no explanation of what needs to happen first. "Reperaturauftrag" typo undermines trust on the first page a new employee sees. History view shows no dates — no way to understand how long past repairs took.

**Sam (Accessibility)**: `.req` list cards are not keyboard-focusable — no interactive role, no tab stop, no cursor indicator. Six repair status buttons rendered via `v-for` from enum values have no ARIA differentiation — all announce the same raw enum text. Status meaning is conveyed primarily through badge background color; screen readers hear the enum value but not the semantic role. No `focus-visible` style on cards.

**Riley (Stress Tester)**: `archiveRequest` and `saveRepairDevice` have no loading state — double-submit is possible without visual feedback. Device selection popup uses `overflow-scroll` with no search on what could be a 200+ item catalog. `queuePosition` appended as a string in `h2` wraps awkwardly on long subject lines. Saving repair device has no success toast — staff can't tell if the save worked.

**Project-specific persona — Klaus (Workshop Owner / Admin)**: Reviews repair history for business decisions. `history.vue` shows no completion timestamps, no duration data, no filtering beyond raw status — the archive is a list with no analytical value. The savings tile renders zeros on unpriced requests rather than hiding until data exists.

## Minor Observations

- `resolveDisplayStatus` is duplicated in `index.vue` and `history.vue` — extract to a shared composable
- Native `<select>` in `index.vue` uses `border: 1px solid $lightgray150` and `box-shadow` on focus — contradicts the ghost border pattern (`2px transparent → primary500`) used everywhere else. Replace with `CommonSelector` or a consistently styled select
- No pagination on the request list — 500+ open requests will render into a single unmanaged scroll
- The savings tile renders a table of zeros on new/unpriced requests — should be hidden when `laborCost === 0 && partsCost === 0 && newPurchaseValue === 0`
- 26 components with hardcoded `border-radius` values (not using the 4px/8px token pair) — low priority but systematic drift worth a sweep

## Questions to Consider

- "What if the request list surfaced urgency signals — time stalled in current status, unanswered customer messages — so staff could triage without opening each card?"
- "Should REJECT and CANCEL ever sit at the same visual level as COMPLETE? What if all destructive closures lived behind a single 'Close request' overflow with a required confirmation step?"
- "The detail page requires scanning four separate panels to understand the complete repair state — what would a single status strip at the top look like that told the full story (request status + repair phase + work item progress + last activity) before the panels?"

---
Score: 15/40 | P0: 1 | P1: 3 | Target: app/pages/staff
