# Plan Mode Prompt — Guitar Fretboard (Portrait + Chord Identifier)

You are in **PLAN MODE**. Do **not** write, edit, or generate any application
code yet. Your only job in this phase is to investigate, think, and produce a
reviewable implementation plan for a single-file guitar fretboard app.

The target artifact is `fretboard.html`: a self-contained interactive page that
draws a portrait guitar fretboard and names the chord formed by selected notes.

## Plan-mode rules

- No code generation, no file writes, no scaffolding. Plan only.
- Ask clarifying questions before proposing a plan if requirements are unclear.
- Separate **facts** (what the requirements say) from **assumptions** (what you
  infer) and **open questions** (what you need decided).
- Prefer incremental, verifiable milestones over one big deliverable.
- Every plan step must have a concrete "done when" check.
- End by asking for approval before moving to build mode.

## What to investigate

1. **Scope & constraints**
   - Single HTML file, vanilla JS, no frameworks, no build step, no CDNs.
   - Must run from `file://` in a modern browser.
   - Responsive and print-friendly.
2. **Inputs / domain model**
   - 6 strings, 5 frets, portrait orientation (vertical strings, horizontal frets).
   - Tuning low → high: `E A D G B E` with open MIDI `[40,45,50,55,59,64]`.
   - Playable positions: open (fret 0) through fret 5.
3. **Existing reference behavior** (from `fretboard.html`)
   - Left-click toggles a note (one per string, un-mutes the string).
   - Right-click mutes a string (clears its selected note).
   - Clear resets all state; fret markers (frets 3 & 5) toggle on/off.
   - Readout identifies: nothing / single note / interval / full chord with
     inversions, scale degrees, slash bass, and alternative readings.
4. **Algorithm**
   - Pitch-class set matching against a prioritized chord-template table.
   - Ranking: root position first, then common-chord priority.
5. **UX & visual language**
   - Warm paper/card aesthetic defined by CSS custom properties.
   - Accessible SVG (`role="img"`, `aria-label`) and `aria-pressed` toggle.

## Plan structure to produce

Present your plan in these sections:

1. **Goal** — one-paragraph statement of what will be built.
2. **Requirements summary** — functional and non-functional, as a checklist.
3. **Assumptions** — explicitly listed, each flagged for confirmation.
4. **Open questions** — decisions needed from the user (e.g. tuning variants,
   fret count, flat vs sharp spelling, mobile tap vs long-press behavior).
5. **Milestones** — ordered, each with:
   - What is built
   - "Done when" verification check
   - Estimated relative effort
6. **Proposed milestone breakdown** (starting point):
   - M1: Static portrait fretboard (board, frets, nut, string labels, numbers)
   - M2: Note dots and click-to-select interaction + state
   - M3: Right-click mute, Clear, and fret-marker toggle
   - M4: Chord analysis engine (pitch-class matching + ranking)
   - M5: Readout UI (all cases incl. inversions, degrees, alternatives)
   - M6: Responsiveness, accessibility, print styling, polish
7. **Risks & mitigations** — e.g. coordinate conversion for right-click on a
   scaled SVG, tie-breaking ambiguous chord names, accidental spelling.
8. **Test / verification plan** — concrete manual test cases:
   - Open low E only → single note E
   - E + G# + B → E major
   - E + G# + B + D → E7
   - E + B → power chord / perfect 5th
   - Muted strings render an `x` and are excluded from analysis
   - Clear returns to "No notes selected"
   - Toggles update `aria-pressed`
9. **Definition of done** — the acceptance criteria for the finished file.

## Output format

- Use clear Markdown headings and checklists.
- Keep it concise; no code blocks except tiny pseudo-steps if truly needed.
- Conclude with: "Approve this plan, or tell me what to change, and I'll move
  to build mode."
