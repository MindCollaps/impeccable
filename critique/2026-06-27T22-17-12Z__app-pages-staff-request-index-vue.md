---
target: app/pages/staff/request/index.vue
total_score: 19
p0_count: 0
p1_count: 3
timestamp: 2026-06-27T22-17-12Z
slug: app-pages-staff-request-index-vue
---
## Design Health Score

| # | Heuristic | Score | Key Issue |
|---|-----------|-------|-----------|
| 1 | Visibility of System Status | 2 | No loading skeleton; no request count; no filter-result summary |
| 2 | Match System / Real World | 2 | German/English language mix throughout the same page |
| 3 | User Control and Freedom | 3 | Reset filters present; list has no destructive actions |
| 4 | Consistency and Standards | 2 | Native `<select>` visually breaks next to `UiInputText` in the same row |
| 5 | Error Prevention | 2 | API failure maps to misleading "no requests" empty state |
| 6 | Recognition Rather Than Recall | 2 | Queue position `(1)` is unlabeled; device line has no field labels |
| 7 | Flexibility and Efficiency of Use | 1 | No keyboard nav, no bulk actions, no sorting |
| 8 | Aesthetic and Minimalist Design | 2 | All card content has equal visual weight; suspected issue is detail-level noise |
| 9 | Error Recovery | 2 | API error produces wrong empty state; no error UI |
| 10 | Help and Documentation | 1 | No tooltips, no contextual help anywhere |
| **Total** | | **19/40** | **Poor — significant improvements needed** |

## Anti-Patterns Verdict

**LLM assessment**: The page doesn't look AI-generated in the decorative sense — no eyebrows, no gradient text, no hero metrics. But it reads as undesigned MVP scaffolding: a column of flat cards with no hierarchy differentiation, a filter row with two mismatched input controls side-by-side, and mixed-language copy on a page that should be consistently German. The design system tokens are used correctly where they appear, but the components aren't composed into a coherent page design.

**Deterministic scan**: 0 findings. No absolute-ban anti-patterns detected in `app/pages/staff/request/index.vue`. The sparseness of the page means there's nothing to trigger pattern detection — the problem here is absence, not excess.

**Visual overlays**: Browser injection was not attempted (no browser automation in this environment).

## Overall Impression

This is a working page that hasn't been designed. The semantic status badges and the "no results vs. no data" empty state distinction show good product thinking. But staff landing here a dozen times per shift deserve a page that helps them triage at a glance — and currently every card forces full reading because there's no hierarchy. The single biggest opportunity: differentiate the primary triage signal (subject + status) from the secondary context (device details, suspected issue) within each card.

## What's Working

1. **Semantic status vocabulary is correctly applied.** The `ui-status` component maps each workflow state to the right color from the design system. Amber for waiting, violet for active/accepted, green for done, red for cancelled. This is the clearest, most correct element on the page.

2. **Filter empty state distinction.** The page correctly separates "Bisher keine Anfragen" (no data exists) from "No requests match the selected filters" (filters excluded everything). This prevents a classic confusion that sends users looking for bugs when filters are just active.

3. **Ghost border pattern on `UiLabeledText`.** The suspected issue field uses the correct ghost border treatment — transparent at rest, colored on hover. This is the design system used right.

## Priority Issues

**[P1] Language inconsistency: German and English mixed on the same page**
- **Why it matters**: Staff using a German-language workshop tool hit English labels ("Device type", "Reset filters", "Details", "No requests match…") immediately after seeing German status labels and German empty states. This is disorienting and signals an unfinished product. The CLAUDE.md states the interface is in German.
- **Fix**: Translate all UI copy to German. "Gerätetyp", "Filter zurücksetzen", "Details", "Keine Anfragen entsprechen den Filtern".
- **Suggested command**: `/impeccable clarify`

**[P1] Native `<select>` breaks the design system in the filter row**
- **Why it matters**: The status dropdown uses a native `<select>` with `border: 1px solid $lightgray150`, `box-shadow` on focus, and `background: $darkgray900` — none of which match the adjacent `UiInputText` (2px ghost border, no shadow, surface-elevated background). Two different visual affordances for filtering sit side-by-side. Every time a user's eye moves across the filter row they register the inconsistency as "something is off."
- **Fix**: Replace the native `<select>` with `CommonSelector` (or the equivalent custom dropdown component already in the system) so all filter controls share the same ghost-border, surface-elevated, primary-focus visual language.
- **Suggested command**: `/impeccable polish`

**[P1] No loading state during data fetch**
- **Why it matters**: `useFetch` is async. While the request list loads, the page renders nothing — it falls through to the empty state box ("Bisher keine Anfragen"), which makes the page look broken or empty before data arrives. Staff who reload this page frequently will experience content pop-in on every visit.
- **Fix**: Use the `pending` ref from `useFetch` and show a skeleton or the `UiLoader` component while `pending.value === true`.
- **Suggested command**: `/impeccable harden`

**[P2] Zero visual hierarchy within request cards**
- **Why it matters**: Each card stacks subject, status badge, device details, suspected issue, and Details button with roughly equal visual weight. Staff triaging 20+ requests must read every card fully because there's no scannable primary signal. The suspected issue field is detail-view content that adds cognitive load at the list level.
- **Fix**: Establish a two-tier card structure: primary row (subject + status badge side-by-side, plus queue position labeled as "Position"), secondary row (device name only — omit model/brand at list level), and move suspected issue behind the Details link or into a collapsible/tooltip. Make subject weight 600 and device details weight 400 in muted color.
- **Suggested command**: `/impeccable layout`

**[P2] Queue position is unlabeled and cryptic**
- **Why it matters**: `{{ r.subject }} {{ r.queuePosition ? `(${r.queuePosition})` : '' }}` renders as e.g. "Broken screen (3)" with no indication of what "3" means. Queue position? Priority level? Count of something? Staff must already know, newcomers won't.
- **Fix**: Separate it from the subject heading and label it — either as a small labeled chip ("Position: 3") or with an icon tooltip.
- **Suggested command**: `/impeccable clarify`

## Persona Red Flags

**Alex (Staff Technician as power user)**: Opens this page 15+ times per shift. No sort controls — can't sort by date received, urgency, or status to see what needs attention first. No keyboard navigation between cards (must mouse to each "Details" button). No request count indicator ("12 open requests"). The suspected issue text forces full-card reading for every request to find the one to act on. Alex will build a workaround (browser filter, Ctrl+F) rather than use the built-in tools.

**Sam (Accessibility-dependent)**: The "Details" button on every card has no context in its accessible name — a screen reader reads 20 identical "Details" buttons with no way to distinguish them. Each button needs `aria-label="Details: {{ r.subject }}"` or the button text must include the subject. The device info line (`r.deviceName / r.deviceModel / r.deviceBrand`) inside `div.req-details` has no screen reader label — it's raw text with slashes.

**Florian (Workshop Manager — project-specific)**: Manages 3–5 technicians; comes to this page to answer "what needs to be picked up now?" The flat unsorted list with no timestamps, no time-in-queue indicators, and no filtering by assignee makes this question unanswerable from the list view. The suspected issue text filling each card adds noise rather than helping him delegate. He needs to know: status, device type, how long it's been waiting. Two of those three are buried or absent.

## Minor Observations

- `resolveDisplayStatus()` applies a business rule (ACCEPTED + first work item done → show historical status) but this logic is invisible to the staff — there's no tooltip or indicator explaining why a card's status might show something other than "Accepted."
- The `req-details` div renders `r.deviceName / r.deviceModel / r.deviceBrand` with no null-handling on display — if any of these are empty strings, the slashes still render (e.g. "iPhone / / Apple").
- `UiLabeledText` has `font-size: 15px` for the label in its CSS, which is larger than the system's 13px label spec from DESIGN.md. Minor divergence.
- The filter grid uses `grid-template-columns: minmax(0, 1fr) 240px auto` — on a wide monitor the search field can grow very wide while the status dropdown is fixed at 240px. A max-width on the row would prevent an overly stretched search input.
- No `aria-live` region for when filters update the list — screen readers won't announce that results changed.
