# Plan Output — Guitar Fretboard (Portrait + Chord Identifier)

> This is the filled-in plan produced by `PLAN_PROMPT.md`. It is a planning
> artifact only — no code is written until the plan is approved.

## 1. Goal

Build a single self-contained `fretboard.html` file that renders an interactive
portrait guitar fretboard (6 strings × 5 frets, standard tuning) and identifies
the chord formed by the notes the user selects, using a pitch-class matching
engine — with no frameworks, dependencies, or build step.

## 2. Requirements summary

**Functional**
- [ ] Portrait fretboard: vertical strings, horizontal frets, 6 strings, 5 frets.
- [ ] Standard tuning low → high: `E A D G B E` (open MIDI `[40,45,50,55,59,64]`).
- [ ] Playable positions: open (fret 0) through fret 5.
- [ ] Left-click a note toggles selection; one note per string; selecting un-mutes.
- [ ] Right-click a string toggles mute; muting clears that string's note.
- [ ] Muted strings render an `x`; unmuted-unused positions render `–`.
- [ ] Clear button resets all selections and mutes.
- [ ] Fret-marker toggle (dots at frets 3 and 5) with `aria-pressed` state.
- [ ] Readout identifies: nothing / single note / interval / full chord.
- [ ] Full chord shows root+sym, name, inversion, slash bass if applicable,
      low→high notes with scale degrees, and up to 3 alternative readings.
- [ ] Chord-diagram voicing notation (string 6 → string 1: fret / `x` / `–`).

**Non-functional**
- [ ] Single HTML file; inline CSS and JS; no external assets.
- [ ] Runs from `file://` in a modern browser.
- [ ] Responsive (SVG `max-width:420px`, scales to viewport).
- [ ] Print-friendly.
- [ ] Accessible: SVG `role="img"` + `aria-label`; button `aria-pressed`.

## 3. Assumptions

- [ ] **A1** — The reference behavior in the existing `fretboard.html` is the
      source of truth for all interactions and visuals. *(confirm)*
- [ ] **A2** — Only standard tuning; no alternate tunings needed for v1. *(confirm)*
- [ ] **A3** — Accidentals are spelled as **flats** (`Db`, `Eb`, `Gb`, `Ab`, `Bb`)
      everywhere note names are shown. *(confirm)*
- [ ] **A4** — "Right-click to mute" is acceptable; no touch-specific gesture
      is required for v1, though tap behavior should degrade gracefully. *(confirm)*
- [ ] **A5** — The exact chord-template list and its priority order from the
      reference file are retained unchanged. *(confirm)*

## 4. Open questions

1. Keep right-click as the only mute gesture, or add a touch-friendly
   alternative (long-press / double-tap / mute toggle mode)?
2. Should note spelling follow the detected chord root (e.g. show `F#` in a
   D-major context) or always use flats?
3. Is 5 frets sufficient, or should frets be configurable (e.g. 5/12/15)?
4. Should the chord readout offer a "copy voicing" or share affordance?
5. Any branding/theme requirements beyond the warm paper/card aesthetic?

## 5. Milestones

| # | Milestone | Done when | Effort |
|---|-----------|-----------|--------|
| M1 | Static portrait fretboard | Board, frets, nut, fret numbers, string labels, string lines render in SVG; opens cleanly in browser | S |
| M2 | Note dots + selection | Every position shows a clickable dot+label; left-click selects one per string; state persists across re-render | M |
| M3 | Mute / Clear / markers | Right-click mutes (renders `x`, clears note); Clear resets; marker toggle flips dots and `aria-pressed` | M |
| M4 | Chord analysis engine | Pitch-class sets match templates; ranking prefers root position then priority; returns best + alternatives | L |
| M5 | Readout UI | All cases render correctly (none/single/interval/chord) incl. inversion, degrees, slash bass, alternatives | M |
| M6 | Polish | Responsive scaling, print styles, a11y attributes, code organized into labeled sections | S |

## 6. Risks & mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Right-click coordinates wrong on a scaled SVG | Mutes the wrong string | Convert via `getBoundingClientRect()` → viewBox scale; clamp index to 0–5; ignore out-of-bounds clicks |
| Ambiguous chord names (e.g. symmetrical dim7) | Misleading readout | Rank by root-position match, then template priority; list alternatives |
| Accidentals inconsistent | Confusing note names | Centralize flat/sharp mapping in one helper (`noteLabel`) |
| Template table drift | Engine regressions | Keep `CHORDS` as an ordered data table; add manual tests for common shapes |
| Hidden bug on un-mute path | Stale note/mute state | Single `render()` from one source of truth; state changes only via `toggleNote`/`toggleMute`/clear |

## 7. Test / verification plan

- [ ] Open low E only → readout: single note **E**.
- [ ] `E + G# + B` → **E major** (root position).
- [ ] `E + G# + B + D` → **E7**.
- [ ] `E + B` (low, no 3rd) → **power chord** `[0,7]` / perfect 5th.
- [ ] Move the bass off the root (e.g. `G# + E + B`) → slash chord / inversion shown.
- [ ] Mute a string with a selected note → note clears, `x` renders, analysis updates.
- [ ] `dim7` built on four strings → shows a dim7 plus alternative readings.
- [ ] Clear → readout returns to "No notes selected".
- [ ] Fret-marker toggle → dots appear/disappear and `aria-pressed` flips.
- [ ] Resize window / print preview → layout stays legible and contained.

## 8. Definition of done

- [ ] A single `fretboard.html` opens from `file://` with no console errors.
- [ ] All functional and non-functional requirements above are met.
- [ ] All verification tests pass.
- [ ] Code is vanilla JS, organized into clearly commented sections, with no
      external dependencies.
- [ ] Responsive + print-friendly + accessible attributes present.

---

**Approve this plan, or tell me what to change, and I'll move to build mode.**
