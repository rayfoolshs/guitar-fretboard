# Build Prompt — Guitar Fretboard (Portrait + Chord Identifier)

Build a single self-contained `fretboard.html` file (no build tools, no external
dependencies, no frameworks) that renders an interactive portrait guitar
fretboard and identifies the chord formed by the selected notes.

## Deliverable

One HTML file containing:

- Semantic HTML markup
- All CSS in a single inline `<style>` block
- All JavaScript in a single inline `<script>` block
- No external fonts, CDNs, images, or libraries

It must work by simply opening the file in a browser (works from `file://`).

## Core concept

A 6-string, 5-fret fretboard drawn in **portrait** orientation: the strings run
vertically and the frets run horizontally.

- Tuning (low → high): `E A D G B E`
- Open MIDI notes: `[40, 45, 50, 55, 59, 64]` (E2 A2 D3 G3 B3 E4)
- Playable positions: open string (fret 0) through fret 5
- Left (low E) = string index 0; right (high E) = string index 5

## Visual design

Warm, minimal, print-friendly, "paper/card" aesthetic.

CSS custom properties on `:root`:

```
--wood: #ffffff;
--wood-light: #f5f5f5;
--fret: #cfcfcf;
--string: #b0b0b0;
--ink: #1c1a17;
--note: #ffffff;
--accent: #e0a458;
--accent-dark: #8a5a30;
```

- Page background `#f7f3ec`; font stack `"Helvetica Neue", Helvetica, Arial, sans-serif`
- Body laid out as a centered vertical flex column with generous padding
- Card containers: white background, `1px solid #e6ded1`, `border-radius: 14px`,
  soft shadow `0 6px 24px rgba(60,45,25,.08)`
- Controls are pill-shaped buttons (`border-radius: 999px`)
- SVG scales responsively: `width:100%; max-width:420px`

## Layout

- `h1` title: "Guitar Fretboard"
- Subtitle line: `6 strings · 5 frets · standard tuning (E A D G B E)` followed by
  a second line explaining left-click adds a note and right-click mutes a string
- A controls row containing:
  - A static "Note names" pill (label, not interactive)
  - A "Fret markers" toggle button (`aria-pressed="true"` by default)
  - A "Clear" ghost button
- A card containing the SVG fretboard
- A readout card showing chord analysis (see below)
- A footer legend: `x = muted string · – = not used · print-friendly`

## SVG fretboard rendering

Use an SVG with `viewBox="0 0 400 620"` and `role="img"` plus an
`aria-label` describing the fretboard.

Layout constants:

```js
const FRETS = 5;
const LEFT = 110, TOP = 100, COL = 40, FRET_H = 92, PAD = 28, NUT = 7;
const W = COL * (strings - 1);   // width between outer strings
const H = FRET_H * FRETS;        // total fret height
```

Helper functions:

```js
xOf = i => LEFT + i * COL;   // i = 0..5 (low E .. high E)
yOf = f => TOP + f * FRET_H; // f = 0 (nut) .. 5
```

Derived bounds: `boardL = LEFT - PAD`, `boardR = LEFT + W + PAD`,
`boardT = TOP - NUT`, `boardB = TOP + H + PAD`, `midX = (boardL + boardR) / 2`.

Use `document.createElementNS` with an `el(name, attrs, parent)` helper to build
SVG elements. Redraw by clearing `svg.textContent = ""` before each render.

Draw, in order:

1. **Board**: rounded white rect from boardT..boardB with light border
2. **Frets**: horizontal lines at frets 1..5 (last fret line is 3px, others 2px)
3. **Nut**: light grey rect (`#e2e2e2`) at the top
4. **Fret numbers**: 1..5 centered to the right of the board, plus a "Fret" label above
5. **Strings**: vertical lines; thicker for lower strings
   (`width = 1 + (5 - i) * 0.55`). Muted strings render dashed (`3 5`) in `#e2e2e2`.
6. **String labels**: tuning note above each string, plus a string-number label
   (`6s` .. `1s`) above that
7. **Fret markers**: grey dots at frets 3 and 5 (toggleable)
8. **Notes**: a clickable circle (r = 11) plus a text label at every
   string/fret position. Open notes sit above the nut. Selected notes use the
   accent fill + accent-dark stroke (`.note-dot.sel`).
9. **Mutes**: a grey "×" (`#8a8175`, 2.5px strokes) above each muted string

Note labels use flats for accidentals: map `#` names through
`FLAT = ["C","Db","D","Eb","E","F","Gb","G","Ab","A","Bb","B"]`.

## Interaction model

State:

```js
const selection = new Map(); // stringIndex -> fret
const muted = new Set();     // stringIndex muted (not played)
let showMarkers = true;
```

- **Left-click a note**: toggle it. Only one note per string. Selecting a note
  un-mutes that string.
- **Right-click a string**: toggles mute. Muting a string clears its selection.
  Convert the click to SVG coordinates using `getBoundingClientRect()` and clamp
  to the nearest string index. Ignore clicks outside the board bounds.
- **Clear**: empty `selection` and `muted`.
- **Fret markers**: toggle the marker dots and update `aria-pressed`.

Re-render the whole board on every state change.

## Chord analysis engine

Represent notes as `{ s, f, midi: OPEN_MIDI[s] + f, pc: midi % 12 }`, sorted
low → high pitch. The lowest picked note is the **bass**.

Keep a prioritized `CHORDS` list of templates as `{ sym, name, iv }`, where `iv`
is the set of intervals from the root. Order matters: earlier = higher tie-break
priority (common chords first). Include at least:

- major `[0,4,7]`, minor `[0,3,7]`
- dominant 7th `[0,4,7,10]`, major 7th `[0,4,7,11]`, minor 7th `[0,3,7,10]`
- shell voicings: dominant/maj7/m7 without the 5th
- m7b5, dim7, dim, aug, sus2, sus4, power chord `[0,7]`, mMaj7, 7#5, m6, 6
- add9, madd9, 9, maj9, m9 (with and without the 5th), 6/9, 7sus4

Matching algorithm:

1. Compute the unique pitch classes.
2. For each unique pc as a candidate root, build the sorted interval set
   `(pc - root + 12) % 12` and compare its joined key against each template's
   joined `iv`.
3. Collect all matches as `{ root, chord, ivs, prio }`.
4. Rank: prefer when `bass.pc === root` (root position), then by template
   priority.

## Readout card

On every render, compute the analysis and update the readout:

- **No notes**: show "No notes selected" plus a hint.
- **Single note**: show the note name (flat form), label "Single note", and the
  voicing.
- **Two+ notes with no template match**: if exactly two, name the interval
  (e.g. "perfect 5th") and mark it "incomplete chord"; otherwise
  "Unrecognized voicing". Show a note-name pair heading.
- **Match found**: show `Root + sym` (with a slash bass note, styled in
  `--accent-dark`, when the bass is not the root), the full chord name, and the
  inversion. Show the notes low → high with their scale degrees, and up to 3
  alternative interpretations ("Also reads as: …").

Voicing notation is string 6 → string 1: a fret number, `x` for muted, or `–`
for not used, space-separated.

Use the degree table `["1","b2","2","b3","3","4","b5","5","b6","6","b7","7"]`
and inversion names `["root position","1st inversion","2nd inversion",
"3rd inversion","4th inversion"]`, and an interval-name table.

## Constraints & quality bar

- Vanilla JS only; no frameworks and no build step.
- Accessible: SVG has an `aria-label`; toggle button uses `aria-pressed`.
- Clean, readable, well-commented code organized into clear sections
  (configuration, layout, state, drawing, interaction, analysis, readout, render).
- Responsive and print-friendly.
