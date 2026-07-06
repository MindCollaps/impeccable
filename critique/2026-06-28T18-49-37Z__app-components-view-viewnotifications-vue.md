---
target: app/components/view/ViewNotifications.vue
total_score: 19
p0_count: 0
p1_count: 3
timestamp: 2026-06-28T18-49-37Z
slug: app-components-view-viewnotifications-vue
---
## Design Health Score

| # | Heuristic | Score | Key Issue |
|---|-----------|-------|-----------|
| 1 | Visibility of System Status | 2 | No loading state on fetch; no per-item read feedback; read/unread look identical |
| 2 | Match System / Real World | 2 | "Keine Notifications" mixes German + English; timestamp shows full datetime not relative |
| 3 | User Control and Freedom | 2 | No click-outside to close; no Escape key dismiss; delete has no undo |
| 4 | Consistency and Standards | 2 | Three semantic color violations; $lightgray125 border on dark panel; native button inside UiButton div |
| 5 | Error Prevention | 2 | "Delete Read" bulk-deletes with no confirmation; no guard on destructive path |
| 6 | Recognition Rather Than Recall | 3 | Labels present on actions; notification content visible; "NEW" chip helps |
| 7 | Flexibility and Efficiency | 1 | No keyboard navigation; no arrow-key traversal; no Escape to close |
| 8 | Aesthetic and Minimalist Design | 2 | Amber bell on dark header is visually jarring; bright border ring looks like a theming accident |
| 9 | Error Recovery | 1 | Zero try/catch on all 4 async operations — silent failures throughout |
| 10 | Help and Documentation | 2 | Delete icon has no aria-label; "Delete Read" label is ambiguous without tooltip |
| **Total** | | **19/40** | **Poor — significant improvements needed** |

## Anti-Patterns Verdict

**LLM assessment**: No AI slop in the visual sense — this is hand-coded and structurally reasonable. What's present are design-system discipline issues: semantic color violations, a theming inconsistency on the panel border, and missing interaction states.

**Deterministic scan**: One finding, exit code 2. Line 201, `border-radius: 9px` on the unread count badge pill — outside the documented scale (xs: 4px, sm: 8px). Not a false positive.

## Overall Impression

A functionally complete notification bell with accumulated design debt. The single biggest issue: it speaks three visual languages simultaneously — warning amber on the bell (repair semantics), error red on the count badge (cancellation semantics), and a light-mode border on a dark panel (theming drift). Staff who have learned the repair color vocabulary will be momentarily confused by amber appearing in the header when no repair is actually waiting.

## What's Working

1. The dropdown structure is sound — position: absolute, scrollable list within max-height, flex column.
2. The unread count cap "99+" prevents layout-breaking 3-digit numbers.
3. Per-item delete with @click.stop is the right approach — event propagation correctly blocked.

## Priority Issues

**[P1] Semantic color violations on the bell icon and both badges**
- What: Bell icon container uses $warning600 (amber). Unread count badge uses $error500 (red). "NEW" chip also uses $warning600. All violate the Semantic Lock Rule.
- Why it matters: Staff have learned color vocabulary from repair status badges. Amber in the header creates a false "is something waiting in repair queue?" alarm.
- Fix: Bell container → $primary500 background or plain secondary button. Unread count badge → $primary500. "NEW" chip → $primary500/$primary600.

**[P1] Panel border uses light-theme token on dark surface**
- What: border: 1px solid $lightgray125 (#E6E6EB) — near-white border on a $darkgray900 panel. Looks like a theming error.
- Why it matters: Conflicts with dark panel borders throughout the app ($darkgray700/$darkgray800 pattern).
- Fix: Change to border: 1px solid $darkgray700.

**[P1] Zero error handling on all async operations**
- What: loadNotifications(), deleteNotification(), deleteReadNotifications(), markAllAsRead() all have no try/catch.
- Why it matters: Silent failures leave users with stale counts and no recovery path.
- Fix: Wrap each with try/catch + showToast({ mode: ToastMode.Error }).

**[P2] No visual distinction between read and unread notifications**
- What: Unread items look identical to read ones except for a "NEW" chip buried at the bottom of the meta row.
- Fix: Move "NEW" chip inline with subject (top row). Give unread items subtle tint or weight difference.

**[P2] Panel doesn't close on click-outside or Escape**
- What: Panel only closes by clicking the bell again or clicking a notification.
- Fix: Add click-outside handler and Escape keydown listener.

## Persona Red Flags

**Alex (Power User)**: Panel is mouse-only — no keyboard traversal, no Escape to close, no arrow-key navigation.

**Sam (Accessibility)**: Delete button has no aria-label. Nested interactive elements (button inside div[role=button]) break screen reader tab flow. No aria-live for count updates.

**Maria (Workshop Tech)**: Amber bell creates false repair-queue alarm. 8 notifications with no read/unread distinction means she can't triage at a glance. "Delete Read" has no confirmation.

## Minor Observations

- formatTime() has no locale arg — pass 'de-DE' explicitly for German product.
- border-radius: 9px (line 201) outside design system scale — use 50% or 8px.
- z-index: 50 on both sticky header and panel — document a scale (header: 50, dropdown: 60).
- "Keine Notifications" mixes languages — use "Keine Benachrichtigungen" or "No notifications" consistently.
- No loading state while fetching — list snaps in with no transition feedback.
