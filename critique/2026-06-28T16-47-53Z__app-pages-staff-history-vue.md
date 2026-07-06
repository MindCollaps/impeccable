---
target: pages/staff/history.vue
total_score: 14
p0_count: 1
p1_count: 5
timestamp: 2026-06-28T16-47-53Z
slug: app-pages-staff-history-vue
---
## Design Health Score

| # | Heuristic | Score | Key Issue |
|---|-----------|-------|-----------|
| 1 | Visibility of System Status | 1 | `pending` and `error` from `useFetch` are discarded; loading shows false empty state |
| 2 | Match System / Real World | 2 | One English label in a German UI; slash-delimited device string is machine-formatted |
| 3 | User Control and Freedom | 2 | Read-only view—no destructive actions—but no search, filter, or page navigation |
| 4 | Consistency and Standards | 1 | Diverges from sister page `staff/request/index.vue` on layout, aria, error handling, `resolveDisplayStatus` logic |
| 5 | Error Prevention | 2 | No destructive actions, but false empty-state-during-load is a confidence-eroding design error |
| 6 | Recognition Rather Than Recall | 2 | Basic info visible; no date, no customer, no filter means finding a ticket requires scanning |
| 7 | Flexibility and Efficiency | 1 | No search, no filter, no sort, no keyboard shortcuts |
| 8 | Aesthetic and Minimalist Design | 2 | Cards are clean but `UiLabeledText` signals interactivity on a read-only field; no hierarchy above cards |
| 9 | Error Recovery | 0 | API failure shows "Keine historischen Anfragen" — a silent lie with no retry path |
| 10 | Help and Documentation | 1 | No contextual help, empty state has no guidance, status display logic is opaque |
| **Total** | | **14/40** | **Poor — major gaps, primarily sins of omission** |

## Anti-Patterns Verdict

**LLM assessment:** Not AI slop in the visual sense. Looks like an unfinished first draft. Card composition is textbook starter layout; nothing wrong with the parts, nothing considered about the whole. A history page is fundamentally an archive — its purpose is retrieval. This page was built as if its purpose were display. No date. No search. No way to locate a specific ticket without scrolling.

**Deterministic scan:** Exit code 0 — no anti-pattern hits. File too minimal to trigger rules.

## Overall Impression

A 59-line stub that renders historical repair requests with no loading state, no error handling, no date information, and no retrieval tools. The sister page is substantially more capable. History should be the more powerful view — it's where you go to look something up — but it's currently the weaker one.

## What's Working

1. Design system fidelity — correct tonal stack, ghost borders follow system vocabulary.
2. Clean card anatomy — subject → status → device → issue → link is logically ordered.
3. Correct empty state component — `<common-box>` centered is right for empty page.

## Priority Issues

**[P0] No loading or error state** — `pending` and `error` from `useFetch` silently dropped. Every user sees false empty state during load; API failure shows "Keine historischen Anfragen" permanently. Fix: destructure `pending`/`error`, show skeleton/loader, add retry on error. `/impeccable harden`

**[P1] No page title or landmark** — Page starts with `h2` inside a card. No `<common-page>`, no `h1`. Fix: wrap in `<common-page title="Verlauf">`. `/impeccable audit`

**[P1] "Suspected issue" label is English** — Line 16. All other strings are German. Fix: change to "Fehlerbeschreibung". `/impeccable clarify`

**[P1] Dates absent from archive page** — No `createdAt` or completion date. Staff can't locate a specific historical repair. Fix: display `r.createdAt` formatted as German date near card header. `/impeccable layout`

**[P1] `UiStatus` contrast failure on amber and green** — No `color` declaration in `Ui-Status.vue`. Inherits near-white (#DEDEE7) which produces ~1.7:1 on amber (#bb9d58) and green (#66bb58). Fix: add `color: $darkgray1000` to `.status`. `/impeccable audit`

**[P1] Details button missing aria-label** — All "Details" buttons announce identically to screen readers. Fix: add `:aria-label="\`Details: ${ r.subject }\`"`. `/impeccable audit`

**[P2] No search or filtering** — Archive grows unboundedly. Fix: add text search on subject/device fields, reuse filter pattern from `staff/request/index.vue`. `/impeccable shape`

**[P2] Slash-delimited device string** — `name / model / brand` looks machine-generated. Fix: use middle-dot helper matching `request/index.vue`. `/impeccable clarify`

**[P2] `resolveDisplayStatus` diverges from active queue** — Same request may show different status depending on page. Fix: align logic or extract shared util. `/impeccable harden`

## Persona Red Flags

**Marco (Workshop Technician):** Needs to find a specific old repair. No date, no search — 200 cards sorted by nothing visible. Gives up after 30 seconds.

**Sam (Screen Reader):** First heading is a repair subject `h2` — no page landmark. `<label>` wraps display text with no associated input (semantic error). All "Details" buttons announce identically.

**Riley (Stress Tester):** 500 repairs render simultaneously with no pagination. Page paint freezes. Bottom cards unreachable without scrolling all 500.

## Minor Observations

- Card has no hover state; entire card looks static even though a button lives inside.
- `<label>` in `LabeledText.vue` wraps display text with no `for` — semantic error; use `<span>` or `<p>`.
- Details button missing `size="S"` used in sister page — 40px vs expected 32px height.
- `UiStatus` falls back to raw enum string for unmapped statuses — silent regression risk.
