---
target: app/components/repair
total_score: 24
p0_count: 0
p1_count: 3
timestamp: 2026-06-29T07-28-20Z
slug: app-components-repair
---
## Design Health Score

| # | Heuristic | Score | Key Issue |
|---|-----------|-------|-----------|
| 1 | Visibility of System Status | 3 | isBusy gating, status badges, and transition animations all solid; editor gives no loading indicator while saving |
| 2 | Match System / Real World | 2 | RepairWorkItemEditor and RepairStepPhase are entirely in English while the rest of the app is German |
| 3 | User Control and Freedom | 3 | Confirmation dialogs on destructive actions; cancel on all popups; no undo for status transitions |
| 4 | Consistency and Standards | 2 | border-radius ranges from 3px to 16px; opacity layering in SavingsTile vs token colors elsewhere; BEM underscore in Editor vs dashes everywhere else |
| 5 | Error Prevention | 3 | Confirmation dialogs for delete/reset are well-done; Device popup submit button stays enabled with no selection — silent fail |
| 6 | Recognition Rather Than Recall | 3 | Status badges use color + text; parts use chips; timeline colors are unlabeled (no legend) |
| 7 | Flexibility and Efficiency | 2 | "Assign to self" and default-step init are good accelerators; no keyboard shortcuts, no bulk actions |
| 8 | Aesthetic and Minimalist Design | 3 | Card/phase layout is clean and focused; eyebrow labels in SavingsTile add decorative noise |
| 9 | Error Recovery | 2 | Toast errors exist for all async failures but are generic; no field-level validation feedback |
| 10 | Help and Documentation | 1 | No tooltips, no inline field hints, no contextual guidance anywhere |
| **Total** | | **24/40** | **Acceptable — significant improvements needed** |

## Anti-Patterns Verdict

**LLM assessment**: This component set does not read as AI-generated at the system level. The token system is disciplined, the motion work is considered, and the Gantt timeline is a genuinely differentiated visualization. What gives it away locally is the eyebrow pattern in RepairSavingsTile — uppercase + letter-spacing on value card labels — and the proximity to the hero-metric template in that same component.

**Deterministic scan**: 9 advisory findings: RepairSavingsTile.vue:66 border-radius 12px outside scale; RepairTimeline.vue:213 border-radius 3px outside scale; RepairTimeline.vue:249-255 8 hard-coded hex status colors outside DESIGN.md palette (intentional semantic data colors, but undocumented).

## Overall Impression

The hardening and motion work are genuinely good. Two of nine components are entirely in English, making the module feel half-shipped. Fix those two and the score jumps materially. After that, the contrast failure in the timeline is the next biggest risk.

## What's Working

1. Error handling discipline in RepairStepGraph.vue — seven async operations, all wrapped in try/catch with toasts, isBusy gates, and confirmation dialogs for destructive actions.
2. Motion system — staggered phase entrance, stat counter flip, card stagger, all covered by prefers-reduced-motion.
3. RepairTimeline Gantt — custom visualization doing real work, live-updates via 60s interval, handles open-ended active states correctly.

## Priority Issues

**[P1] RepairWorkItemEditor.vue is entirely in English**
- close-text="Cancel", title="Work Item Type", title="Assigned Staff", field labels Title/Order/Labor minutes/Description, status options Pending/In Progress/Blocked/Done, submitText returns Update/Create
- Why: Staff open this editor for every work item creation and edit — highest-traffic form in the module
- Fix: Translate all strings. Status → Ausstehend/In Bearbeitung/Blockiert/Erledigt. Labels → Titel/Position/Arbeitsminuten/Beschreibung. Submit → Erstellen/Speichern.
- Command: /impeccable harden RepairWorkItemEditor.vue

**[P1] RepairStepPhase.vue copy is entirely in English**
- Step count uses English pluralization, "Add step" button (×2), empty state text
- Why: Visible for every phase in every request
- Fix: Count → N Schritt/e; buttons → Schritt hinzufügen / Ersten Schritt hinzufügen; empty state in German
- Command: /impeccable harden RepairStepPhase.vue

**[P1] Contrast failure in RepairTimeline.vue**
- White text at 11px on amber #f59e0b: ~3.0:1 (fails WCAG AA, needs 4.5:1)
- White text at 11px on gray #64748b: ~4.2:1 (borderline fail)
- ON_THE_WAY_TO_SHOP and ON_THE_WAY_TO_CUSTOMER both show "Unterwegs" — indistinguishable
- Fix: Dark text on light/amber statuses; luminance-based switching. Distinguish the two "Unterwegs" labels.
- Command: /impeccable audit RepairTimeline.vue

**[P2] Eyebrow ban in RepairSavingsTile.vue**
- .savings-tile-card-label uses text-transform: uppercase; letter-spacing: 0.05em — absolute design system ban
- Also: opacity: 0.8/0.7 layering instead of token colors; border-radius: 12px off-scale
- Fix: Remove uppercase and letter-spacing. Replace opacity with $lightgray400. Move to 8px border-radius.
- Command: /impeccable quieter RepairSavingsTile.vue

**[P2] Device popup submit not disabled with no selection**
- handleSubmit silently early-returns when no device selected; button shows as enabled
- Fix: :disabled="!selectedDevice" on submit trigger
- Command: /impeccable harden RepairDeviceSelectPopup.vue

## Persona Red Flags

**Alex (Power User)**: No keyboard shortcut to open the editor from a card. Editing the same field on 6 work items requires 6 separate popups — no inline editing or batch status change. "Standardschritte anlegen" and "Assign to self" are good accelerators.

**Sam (Accessibility)**: device-select-item is a div with @click — missing role="button" and tabindex="0", keyboard users cannot navigate the device list. White text on amber #f59e0b fails WCAG contrast. pulse-edge animation in RepairTimeline has no prefers-reduced-motion guard.

**Lena (Workshop Technician)**: Part creation requires 5+ fields on a tablet. Work item action row can show up to 6 small buttons — too dense for reliable tablet tapping.

## Minor Observations

- Dead CSS: step-graph-stat* rules in RepairStepGraph.vue (no matching template elements)
- Dead CSS: step-phase-order-label in RepairStepPhase.vue (no matching template elements)
- Timeline status colors should be registered in DESIGN.md as semantic data tokens
- BEM inconsistency in RepairWorkItemEditor.vue: &-status_label/_buttons should use dashes
- border-radius: 6px in RepairDeviceSelectPopup.vue — should move to 8px (sm token)
- font-size: 12px in RepairSavingsTile breakdown items — off scale (11px or 13px)

## Questions to Consider

- "What would a staff member who creates 20 work items a day want from this editor that they can't do today?"
- "Should the timeline legend explain what each color means, or do techs learn it through repetition?"
- "The device popup has search + click-to-select + a button that does the same thing — which affordance actually works and which one is adding noise?"
