# Workflow — Plan → Approve → Build

A two-phase workflow for producing `fretboard.html` (portrait guitar fretboard +
chord identifier). Each phase has its own prompt file and a clear exit gate.

## Files

| File | Role |
|------|------|
| `PLAN_PROMPT.md` | Prompt that puts an agent in **plan mode** (no code). |
| `PLAN_OUTPUT.md` | The filled-in plan produced in plan mode. |
| `BUILD_PROMPT.md` | Prompt/instructions for the **build phase**. |
| `WORKFLOW.md` | This document — how the phases connect. |
| `fretboard.html` | The finished deliverable. |

## Phase 1 — Plan mode

**Input:** `PLAN_PROMPT.md` (+ the existing `fretboard.html` as reference).
**Rules:** investigate and plan only; no code, no file writes.
**Output:** a plan matching `PLAN_OUTPUT.md` — goal, requirements, assumptions,
open questions, milestones with "done when" checks, risks, test plan, and a
definition of done.

**Exit gate (human approval):**
- [ ] Goal and requirements are correct.
- [ ] Assumptions A1–A5 confirmed or corrected.
- [ ] Open questions answered or deferred.
- [ ] Milestones and test cases accepted.

> Do not proceed to Phase 2 until the plan is approved.

## Phase 2 — Build mode

**Input:** `BUILD_PROMPT.md` + the approved `PLAN_OUTPUT.md`.
**Rules:** implement exactly the approved scope, milestone by milestone.
**Output:** a single self-contained `fretboard.html`.

**Build order (from the approved plan):**

1. **M1** Static portrait fretboard — board, frets, nut, numbers, string labels.
2. **M2** Note dots + left-click selection state.
3. **M3** Right-click mute, Clear, fret-marker toggle.
4. **M4** Chord analysis engine (pitch-class matching + ranking).
5. **M5** Readout UI (none / single / interval / chord incl. alternatives).
6. **M6** Responsiveness, accessibility, print styling, polish.

**Per-milestone loop:**
1. Implement the milestone.
2. Run its "done when" check from `PLAN_OUTPUT.md`.
3. Run the relevant verification tests.
4. Commit/save before starting the next milestone.

## Phase 3 — Verify & finish

Run the full test plan from `PLAN_OUTPUT.md`:

- [ ] Single note, major, dominant 7th, power chord.
- [ ] Slash chord / inversion when bass ≠ root.
- [ ] Mute clears note and renders `x`; analysis updates.
- [ ] `dim7` shows alternatives.
- [ ] Clear resets to "No notes selected".
- [ ] Marker toggle flips dots and `aria-pressed`.
- [ ] Responsive + print-friendly + accessible.

**Definition of done:** all boxes in `PLAN_OUTPUT.md` §8 checked.

## Quick rules of thumb

- **Plan mode** optimizes for *being right* — no code, surface assumptions early.
- **Build mode** optimizes for *shipping the approved scope* — no scope creep.
- If the plan is wrong mid-build, **stop and re-plan** rather than improvising.
