---
target: app/pages/chat/room/[id].vue
total_score: 22
p0_count: 1
p1_count: 3
timestamp: 2026-06-28T18-38-09Z
slug: app-pages-chat-room-id-vue
---
## Design Health Score

| # | Heuristic | Score | Key Issue |
|---|-----------|-------|-----------|
| 1 | Visibility of System Status | 2 | Joining state shown; no auto-scroll on new messages, no send loading state |
| 2 | Match System / Real World | 3 | Familiar chat patterns; Enter-to-send in a multi-line field is ambiguous |
| 3 | User Control and Freedom | 2 | Double-scroll trap, no link back to the repair request, no send undo |
| 4 | Consistency and Standards | 3 | On-system visually; 12px meta size is off the defined type scale |
| 5 | Error Prevention | 2 | Enter key fires send in a rows=3 textarea; no visible character limit |
| 6 | Recognition Rather Than Recall | 2 | No repair request context shown in the UI |
| 7 | Flexibility and Efficiency | 2 | No keyboard shortcut spec, one rigid send path |
| 8 | Aesthetic and Minimalist Design | 3 | Clean layout; double-scroll is invisible damage |
| 9 | Error Recovery | 2 | Errors as ephemeral toasts; failed send wipes typed text |
| 10 | Help and Documentation | 1 | No char limit visible, no Enter behavior hinted, no context breadcrumb |
| **Total** | | **22/40** | **Acceptable — significant improvements needed** |

## Anti-Patterns Verdict

**LLM assessment:** Not AI-generated slop. No eyebrows, gradient text, hero metrics, or identical card grids. The chat bubble pattern is conventional and correct. Problems are UX engineering gaps, not aesthetic tells.

**Deterministic scan:** Exit code 0. Detector found zero rule violations.

## Overall Impression

The chat UI is structurally minimal and on-brand, but missing foundational chat behaviors: auto-scroll, a predictable keyboard model, and request context. The max-height: 60vh + outer page scroll creates a double-scroll trap. Fix the scroll model first.

## What's Working

1. Message type differentiation is clear (own=violet/right, others=dark/left, system=centered/bordered).
2. Joining lifecycle is handled with clear state and error redirect.
3. Layout is minimal and on-brand without chrome competing for attention.

## Priority Issues

### [P0] Double-scroll trap + missing auto-scroll
max-height: 60vh with overflow-y: auto inside a scrolling page creates two competing scroll surfaces. New messages never auto-scroll into view. Fix: fill-height flex layout (calc(100dvh - header - input - padding)), auto-scroll composable on every messages.value push.

### [P1] Enter key sends in a multi-line textarea
@keyup.enter fires sendMessage() in a rows=3 textarea. Users expecting Enter for newlines accidentally send. Fix: use @keyup.ctrl.enter (or Shift+Enter for newlines), and label it visibly below the input.

### [P1] No repair request context visible
No indication of which repair request the chat belongs to once inside. Fix: slim context bar above messages showing request subject, customer name, and a back-link.

### [P1] Title flashes "Chat · undefined" during load
repairReq?.subject is undefined while useFetch resolves. Fix: fallback to '...' or a loading guard.

### [P2] Message meta contrast fails WCAG AA on own-message backgrounds
$lightgray400 (#aaaaac) at 12px on $primary700 (#512da8) = ~3.8:1, below 4.5:1 required. Fix: override meta color on --own to $lightgray200 or rgba(#DEDEE7, 0.75).

## Persona Red Flags

**Alex (Power User):** No keyboard send shortcut, no unread indicators, no jump-back-to-request key, isSending shows no loading state.

**Casey (Mobile Customer):** Double-scroll on mobile is a one-swipe disaster, full timestamps on every message are noisy, no message grouping for consecutive same-sender messages.

**Sam (Accessibility):** 3.8:1 meta contrast on violet own messages, textarea missing explicit aria-label, system messages distinguished only by color/position with no semantic role.

## Minor Observations

- toLocaleString() shows full date+time for every message; relative formatting (heute 14:32) is standard.
- No empty input feedback when user clicks Send with empty textarea.
- No character counter at 2000-char limit (design system specifies this behavior).
- $darkgray700 system message uses full border (1px solid $lightgray400) — background tint would be less noisy.
- No message grouping for consecutive same-user messages.

## Questions to Consider

- Enter = send or newline? Pick one convention and make it explicit.
- What happens to typed text on mid-emit socket disconnect?
- Would staff benefit from a sidebar of multiple active chat rooms rather than one tab per request?
