# FRETCRAWLER v1 — Complete Build Specification
**Date:** June 3, 2026
**Author:** Clark Isachsen
**Stack:** Vanilla HTML + CSS + JavaScript (single file, no build step)
**Current file:** ~/Desktop/fretcrawler/index.html
**Deploy:** Netlify (auto-deploy from GitHub, repo: iceaxesun/fretcrawler)
**Claude Code startup:** `cd ~/Desktop/fretcrawler && ulimit -n 2147483646 && claude`

---

## CURRENT STATE

The existing index.html at fretcrawler.com is a working vanilla HTML/JS app with:
- Interactive fretboard showing scale/mode notes
- Circle of Fifths (basic version)
- Diatonic chord display
- Tone.js audio

**Everything in this spec is either a rebuild of existing features or new additions.
Do not assume the current code is correct — treat this spec as the authoritative target.**

---

## 1. CONCEPT

Fretcrawler is a guitar theory tool for tablet and desktop.

**Tagline:** Explore modes — Build progressions — Find your lead

**Sub-tagline:** Scales · Modes · Circle of Fifths · Progressions · Ear Training

**Two modes:**
- **Explore** — pick instrument, key, mode → see scale on fretboard and staff → navigate CAGED positions and arpeggios
- **Build** — construct chord progressions using the Circle of Fifths → hear them play → see fingerings

**Design philosophy:**
- Tablet-first — all tap targets minimum 44px height
- Functional over decorative — clean dropdowns, not hardware skeuomorphics
- Warm maple aesthetic without visual clutter
- One file — keep it vanilla HTML/JS/CSS, no framework, no build step

---

## 2. VISUAL DESIGN

### Background
- Panel background: maple-bg.jpg (file is in ~/Desktop/fretcrawler/)
- Border-radius: 10px
- Thin border: #8a5828

### Color palette
```
Gold/amber active:     #c8a84a
Gold bright:           #ffd870
Dark control bg:       rgba(15,8,2,0.68)
Dark control bg hover: rgba(35,20,5,0.88)
Border inactive:       rgba(140,80,20,0.40)
Border active:         #c8a84a
Text inactive:         #7a5030 to #8a6040
Text active:           #ffd870
Section labels:        #8a5828
Root note dots:        #c03820 (red)
Scale note dots:       #3a8a5a (green)
CAGED shape dots:      #c8a84a (aged brass/gold)
Arpeggio dots:         #2a8a8a (teal/abalone)
```

### Logo
- File: Fretcrawler_Logo.png (in ~/Desktop/fretcrawler/)
- Transparent PNG — use mix-blend-mode: multiply
- Blends into maple background, looks carved
- Large — fills left portion of header generously
- Do NOT place on dark background

### Typography
- **IBM Plex Mono** — all UI text (Google Fonts)
- **Playfair Display italic** — chord names on cards and staging area
- **Georgia serif** — staff notation (treble clef, time signature numbers)

### Chord card quality colors
```
Major:      rgba(30,50,80,0.85)   border rgba(60,100,160,0.5)
Minor:      rgba(50,30,70,0.85)   border rgba(100,55,140,0.5)
Dom7:       rgba(75,45,10,0.85)   border rgba(160,95,20,0.5)
Diminished: rgba(70,15,15,0.85)   border rgba(140,40,40,0.5)
Suspended:  rgba(15,60,50,0.85)   border rgba(30,110,90,0.5)
Augmented:  rgba(60,40,10,0.85)   border rgba(140,100,20,0.5)
```

---

## 3. LAYOUT STRUCTURE

```
┌────────────────────────────────────────────────────────────────┐
│  HEADER: Logo | Tagline + sub-tagline | Tuner / EarTrain / Theory │
├────────────────────────────────────────────────────────────────┤
│  CONTROLS BAR: Instrument▾ | Tuning▾ | Key▾ | Mode▾ |         │
│                [Explore][Build] | Fret◀▶ | Notes/Intervals |   │
│                Play Scale ↑↓                                   │
├────────────────────────────────────────────────────────────────┤
│  FRETBOARD (full width, dark background)                       │
├────────────────────────────────────────────────────────────────┤
│  STAFF (full width, cream background)                          │
├──────────────────── Build Zone ────────────────────────────────┤
│  EXPLORE MODE: CAGED Navigator (see Section 8)                 │
│  BUILD MODE:  CoF | Diatonic chart / Chord strip | Staging     │
│               Transport bar                                    │
└────────────────────────────────────────────────────────────────┘
```

---

## 4. HEADER

- **Logo** — left, large (min 90px height), mix-blend-mode: multiply
- **Tagline** — center, Playfair Display italic ~18px, color #4a2e10
- **Sub-tagline** — center below tagline, IBM Plex Mono 9px, uppercase, letter-spacing 2px
- **Right column** — three buttons stacked vertically:
  - TUNER
  - EAR TRAIN
  - THEORY
  - Style: dark glass bg, gold border, gold text, min 44px height, min-width 120px

---

## 5. CONTROLS BAR

Single horizontal strip. All controls min 44px height. Flex row with wrap.

### Dropdown style (all four selectors)
```css
background: rgba(15,8,2,0.68);
border: 1px solid rgba(140,80,20,0.45);
border-radius: 4px;
padding: 10px 14px;
font-family: IBM Plex Mono;
font-size: 12px;
color: #c8a84a;
min-height: 44px;
appearance: none;
/* Gold chevron arrow via background-image SVG */
```

### Instrument dropdown
Options: Guitar · Bass · Ukulele · Mandolin · Banjo (5-str)
onChange → update Tuning dropdown options

### Tuning dropdown (updates based on instrument)
```
Guitar:   Standard EADGBe · Drop D · Open G · Open D · DADGAD ·
          Half step down · Full step down
Bass:     Standard EADG · Drop D · 5-string BEADG
Ukulele:  Standard GCEA  (only option, no dropdown needed)
Mandolin: Standard GDAE  (only option)
Banjo:    Open G · Double C · Sawmill
```

### Root Key dropdown
Ordered by Circle of Fifths:
C · G · D · A · E · B · F#/Gb · Db/C# · Ab · Eb · Bb · F

onChange → update CoF highlight, fretboard, staff, diatonic chart, CAGED positions

### Mode dropdown with optgroups
```
── Diatonic (bright → dark) ──
Lydian
Ionian (Major)    ← default selected
Mixolydian
Dorian
Aeolian (Minor)
Phrygian
Locrian
── Other Scales ──
Harmonic Minor
Melodic Minor
Blues
Pentatonic (Minor)
Major Pentatonic
Whole Tone
Diminished H-W
```
onChange → update fretboard, staff

### Explore / Build toggle
- Two paired buttons, right-aligned in the control bar
- Explore = default active
- Active style: dark bg rgba(35,20,5,0.88), gold border, gold text
- Inactive style: dark glass, muted text
- Controls which content appears in the zone below the staff

### Fret position nav
- Label above: "Fret pos"
- Two buttons: ◀ ▶ (44×44px each)
- Shifts fretboard window: open position → 5th position → 7th → 9th → 12th
- Current position shown in fretboard title bar

### Labels toggle
- Label above: "Labels"
- Two paired buttons: Notes | Intervals
- Notes (default): note names on fret dots
- Intervals: interval numbers (R · 2 · 3 · 4 · 5 · 6 · 7)

### Play scale buttons
- Label above: "Play scale"
- Two buttons: ↑ Asc | ↓ Desc
- Plays scale notes sequentially via Tone.js using selected Voice

---

## 6. FRETBOARD

### Container
- Full panel width
- Background: rgba(14,7,1,0.92)
- Drawn as SVG

### String order — CRITICAL
```
Top of display    = high e (thin string)  = lowercase e
                    B
                    G
                    D
                    A
Bottom of display = low E (thick string)  = uppercase E
```
String thickness increases visually from top to bottom.

### Open string pitch classes (top to bottom)
```javascript
const OPEN = [4, 11, 7, 2, 9, 4];
// e=4, B=11, G=7, D=2, A=9, E=4
```

### Frets displayed
- 12 frets shown (fret 0 through 12)
- Fret 0 = open strings (nut position)
- Fret position shifts when nav arrows used

### Note dots
- Root note: #c03820 red, radius 12, glow halo rgba(192,56,32,0.15) radius 16
- Scale notes: #3a8a5a green, radius 12
- Label inside dot: white, IBM Plex Mono, 12px single char / 9px accidentals

### Accidentals — CRITICAL
```
Sharp keys (C G D A E B) → use sharps: C# D# F# G# A#
Flat keys  (Db Ab Eb Bb F) → use flats: Db Eb Gb Ab Bb
C major has NO accidentals — C D E F G A B only
```

```javascript
const SHARP_KEYS = new Set([0, 7, 2, 9, 4, 11]); // C G D A E B
const COF_TO_PC  = [0, 7, 2, 9, 4, 11, 6, 1, 8, 3, 10, 5];
```

### Fret markers
- Single dot: frets 3, 5, 7, 9 — color #201408
- Double dot: fret 12 (octave marker)

### Nut
- Gold rectangle at fret 0, color #c8a84a

### Fret numbers
- Above fretboard, IBM Plex Mono 11px, color #4a3820
- 0 through 12

### Title bar
"Open Position · C Ionian (Major) · Standard EADGBe"
Updates when key, mode, tuning, or fret position changes.

---

## 7. STAFF

### Container
- Full panel width
- Background: rgba(252,246,228,0.95) — cream
- Height: 64px
- Drawn as SVG

### Staff lines
- 5 horizontal lines
- Bottom line = E4 (line 0), top line = F5 (line 4)
- Color #4a3820, stroke-width 0.9
- Opening barline at left edge

### Treble clef
- Unicode: \uD834\uDD1E
- Font: Georgia serif, 58px
- Bottom of clef sits at bottom staff line, tail extends below
- Color #3a2810
- Must look like published sheet music — properly proportioned

### Time signature
- Stacked numerals after clef
- Top number centered between staff lines 2–4
- Bottom number centered between lines 0–2
- Font: Georgia serif, bold, 19px
- Updates when time sig selection changes in transport

### Notes (Explore mode)
- Scale notes as filled noteheads with stems
- Root note = #c03820, other scale notes = #3a8a5a
- Noteheads: ellipse rx=6 ry=4.5
- Stems: 24px — up if below middle line, down if above
- Ledger lines for notes above or below staff
- Note name labels below staff (7.5px)

### Notes (Build mode)
- Shows stacked chord tones in guitar voicing for the active chord card
- Chord tones stacked vertically on staff as the chord plays

### Mode label
- Right side of staff: "C IONIAN (MAJOR)" — updates with key/mode

---

## 8. EXPLORE MODE — CAGED + ARPEGGIO NAVIGATOR

This fills the build zone when Explore is active.
This is the new core feature. Build it carefully.

### Concept
The build space becomes a CAGED navigator. Select a chord,
step through its five positions up the neck, with arpeggio
tones overlaid within each shape.

**The pedagogical insight:** The arpeggio lives inside the
CAGED shape. The solo is not separate from the chord — it IS
the chord, voiced melodically. Fretcrawler shows this
visually in seconds.

**Example:** Little Wing (Hendrix) — A minor shape at fret 7,
G major. Melody, bass, and chords all within one hand position.
That's CAGED + arpeggio in action.

### Interaction flow
1. User selects a chord on the Circle of Fifths
2. Fretboard (or dedicated neck view in build space) shows
   the first CAGED position — lowest on the neck
3. Prev / Next arrows step through all five positions
   ascending the neck
4. Active position displays:
   - CAGED shape name (E, D, C, A, G)
   - Fret number
   - Chord name
5. Arpeggio overlay toggle — lights up arpeggio tones
   within the active CAGED shape
6. Tone.js voices the chord when position changes

### Visual design
```
CAGED shape fretted notes:  aged brass/gold  #c8a84a
Arpeggio tones overlay:     teal/abalone     #2a8a8a
Inactive positions:         dim markers showing full map
Shape badge:                Playfair Display, large
                            e.g. "E shape · Fret 0"
                                 "D shape · Fret 2"
                                 "C shape · Fret 3"
                                 "A shape · Fret 5"
                                 "G shape · Fret 7"
```

### CAGED position data
For each chord, five positions must be computed.
Each position specifies: shape type, root fret, fingering dots.

The five CAGED shapes for any chord:
- E shape: barre at root fret using open E chord form
- D shape: using open D chord form
- C shape: using open C chord form
- A shape: barre at root fret using open A chord form
- G shape: using open G chord form

Implement for major chords first, then minor.

### Controls in CAGED navigator
```
[← Prev shape]  E shape · Fret 0  [Next shape →]
[ ] Show arpeggio
```

### Phase 2 placeholder — Song Catalog
Chord-by-chord walkthrough of classic songs showing
which CAGED position and arpeggio applies.
First song: Little Wing (Hendrix)
Architecture: position/chord/timing data per song,
manual step-through (next/prev), no real-time sync needed.
Do NOT build this in v1 — reserve the architecture for it.

---

## 9. CIRCLE OF FIFTHS

### Placement
- **In Build mode:** lower left of build zone, free floating, no border ring
- **In Explore mode:** compact, upper area, used as chord/key selector

### SVG dimensions
~208×208px, viewBox="-104 -104 208 208"

### Structure
- 12 outer wedges (major keys)
- 12 inner ring segments (relative minors)
- Dark center core with current key name

### Wedge colors
```javascript
const COF_COLORS = [
  "#2e7a56", // C
  "#3a6898", // G
  "#5a4898", // D
  "#8a3878", // A
  "#983838", // E
  "#985018", // B
  "#587818", // F#
  "#1a8868", // Db
  "#1a5888", // Ab
  "#582888", // Eb
  "#881858", // Bb
  "#686008"  // F
];
```

### Active wedge
- Fill: #c8a84a (gold)
- Drop shadow glow filter
- Stroke: #ffd870

### Labels
- Outer (major keys): white, IBM Plex Mono, 10px (12px active), weight 600
- Inner (relative minors): rgba(220,220,220,0.8), 7px
- Center: current key 20px bold gold + "KEY" label 6.5px below

### Bevel highlight
- Thin lighter arc at outer edge of each wedge
- rgba(255,255,255,0.14)

### Behavior
- Tap wedge → selects key → updates key dropdown, fretboard, staff, diatonic chart
- In Build mode: tap wedge → sends chord to staging area
- Two-way sync: key dropdown changes also rotate CoF highlight

---

## 10. BUILD MODE — DIATONIC CHORD CHART

Shown in build zone when Build mode is active,
above the chord progression strip.

- Label: "Diatonic chords — tap to stage"
- 7 chord buttons, one per scale degree
- Each button: chord name (Playfair italic 17px) + roman numeral below
- Color coded by chord quality (see Section 2 palette)
- Min height 44px per button
- Tap → sends chord to staging area, auto-plays via Tone.js

### Diatonic chord qualities (major scale)
```
I   = major
ii  = minor
iii = minor
IV  = major
V   = dominant 7
vi  = minor
vii = diminished
```

---

## 11. CHORD STAGING AREA

Right side of build zone. Always visible when Build mode active.

### Purpose
Holds the current chord selection before it is committed
to the progression. The user auditions here — changes quality,
picks duration — then confirms.

### Compact view (default)
- Chord name: Playfair Display italic, 26px, white
- Quality label beside name: 13px, rgba(255,255,255,0.65)
- Duration row: whole ♩ half ♩ quarter ♩ eighth ♩ + dot modifier
- Checkmark confirm button: "✓ Add to progression"
  - Green background rgba(0,140,55,0.8)
  - Min height 44px

### On tap of chord name → quality popover
Options: maj · min · 7 · maj7 · min7 · sus2 · sus4 · dim · aug
Selecting quality:
- Updates staging card display
- Plays chord immediately via Tone.js
- Updates fretboard (chord voicing) and staff (stacked notes)

### On tap of duration symbol
- Symbol highlights (gold)
- Dot modifier tap adds 50% duration
- Card width preview updates

### On tap of checkmark
- Chord drops into progression strip
- Staging area clears
- Ready for next CoF selection

### Auto-behavior
- Chord plays automatically when it arrives in staging

### Staging card style
- Background: rgba(150,105,15,0.82) warm amber
- Border: rgba(200,168,74,0.55) gold
- Border-radius: 5px
- Box-shadow: 0 0 12px rgba(200,168,74,0.15)

---

## 12. CHORD PROGRESSION STRIP

Horizontal scrolling strip of chord cards.

### Card widths (duration-based)
```
Whole note:    110px
Half note:      64px
Quarter note:   40px
Eighth note:    28px
Dotted adds 50% to each
```

### Card contents
- × remove button (top-right, small, hover to reveal)
- Chord name: Playfair Display italic, 17px, white
- Quality: 7px, rgba(255,255,255,0.45)
- Duration row: whole/half/quarter/eighth + dot (tappable)
- Mini fret diagram if card wide enough (min ~54px)
- Narrow cards (eighth): floating diagram tooltip on tap

### Card behavior
- Tap chord name → plays via Tone.js
- Fretboard shows chord voicing
- Staff shows stacked chord tones
- Card active state: gold border glow

### Add card button
- Dashed border, + symbol, same height as cards
- Tap → next CoF selection goes to staging

---

## 13. TRANSPORT BAR

Below the chord progression strip. All controls min 44px.

```
[▶][❚❚][■][⏮] | [⟳ Loop] | [4/4][3/4][6/8] | [Tempo ——●—— 120 BPM] | [Voice▾] | [8 bars] | [Lead Complexity] | [💾]
```

### Playback buttons (circular, ~44px)
▶ Play · ❚❚ Pause · ■ Stop · ⏮ Rewind

### Loop toggle
- ⟳ button, ON by default
- When on: glows gold, progression repeats continuously
- When off: plays once and stops

### Time signature
Three paired buttons: 4/4 · 3/4 · 6/8
Active = gold border and text
onChange → updates staff time sig display

### Tempo
- Label: "Tempo"
- Horizontal slider: 40–240 BPM
- Numeric LED display: shows current BPM in gold
- Label: "BPM"

### Voice dropdown
Piano · Strum · Picked · Bass + Chord · Click only

### Bar counter
- Displays e.g. "8 bars"
- Updates live as cards are added/removed/duration changed
- Calculation: sum of card durations ÷ time signature beats

### Lead complexity (segmented buttons)
```
Pentatonic | In Key | Chord Tones | Follow Chords | Jazz (greyed out)
```
Controls fretboard display during playback:
- Pentatonic: pentatonic scale over full progression
- In Key: full mode scale
- Chord Tones: 1-3-5 of current playing chord
- Follow Chords: chord tones shift with each chord change
- Jazz: reserved, not implemented in v1

### Save/Export button (💾)
Tap → small menu:
- Save (localStorage)
- Download MIDI
- Share Link (URL encoded progression)
- Print Chart

---

## 14. EAR TRAINING MODAL

Accessed via EAR TRAIN button in header.

### Flow
1. Root note displayed large
2. Play button → two notes play (melodic ascending by default)
3. Four answer buttons (interval names, randomized)
4. Select answer → correct/wrong shown
5. Physics tip displayed after each answer
6. Auto-advances after 1.8s
7. Score and streak tracked at top

### Settings
- Diatonic / All 12 intervals
- Melodic / Harmonic
- Ascending / Descending

### Interval data with physics tips
```
Unison:      Reset every 1 cycle — perfect consonance
Minor 2nd:   Ratio 16:15, resets after 240 cycles — maximum dissonance
Major 2nd:   Ratio 9:8, resets after 72 cycles — mild dissonance
Minor 3rd:   Ratio 6:5, resets after 30 cycles — warm and dark
Major 3rd:   Ratio 5:4, resets after 20 cycles — bright and stable
Perfect 4th: Ratio 4:3, resets every 12 cycles — strong consonance
Tritone:     Ratio 45:32, never fully resets — maximum ambiguity
Perfect 5th: Ratio 3:2, resets every 6 cycles — strongest after octave
Minor 6th:   Ratio 8:5, resets after 40 cycles — melancholic
Major 6th:   Ratio 5:3, resets after 15 cycles — open and warm
Minor 7th:   Ratio 7:4, resets after 28 cycles — the blues note
Major 7th:   Ratio 15:8, resets after 120 cycles — yearning tension
Octave:      Ratio 2:1, resets every 2 cycles — pure consonance
```

---

## 15. MUSIC THEORY DATA

### Mode interval sets
```javascript
const MODES = [
  {n:"Lydian",      ivs:[0,2,4,6,7,9,11]},
  {n:"Ionian",      ivs:[0,2,4,5,7,9,11]},  // default
  {n:"Mixolydian",  ivs:[0,2,4,5,7,9,10]},
  {n:"Dorian",      ivs:[0,2,3,5,7,9,10]},
  {n:"Aeolian",     ivs:[0,2,3,5,7,8,10]},
  {n:"Phrygian",    ivs:[0,1,3,5,7,8,10]},
  {n:"Locrian",     ivs:[0,1,3,5,6,8,10]},
  {n:"Harm Minor",  ivs:[0,2,3,5,7,8,11]},
  {n:"Mel Minor",   ivs:[0,2,3,5,7,9,11]},
  {n:"Blues",       ivs:[0,3,5,6,7,10]},
  {n:"Pentatonic",  ivs:[0,3,5,7,10]},
  {n:"Maj Penta",   ivs:[0,2,4,7,9]},
  {n:"Whole Tone",  ivs:[0,2,4,6,8,10]},
  {n:"Dim H-W",     ivs:[0,1,3,4,6,7,9,10]},
];
```

### Key mappings
```javascript
const COF_TO_PC  = [0,7,2,9,4,11,6,1,8,3,10,5];
const SHARP_KEYS = new Set([0,7,2,9,4,11]); // C G D A E B
const SHARP_N    = ["C","C#","D","D#","E","F","F#","G","G#","A","A#","B"];
const FLAT_N     = ["C","Db","D","Eb","E","F","Gb","G","Ab","A","Bb","B"];
```

### Diatonic chord qualities
```
Scale degree:  I    ii   iii  IV   V    vi   vii
Quality:       maj  min  min  maj  dom7 min  dim
Roman:         I    ii   iii  IV   V    vi   vii°
```

---

## 16. AUDIO (Tone.js)

### Scale playback
Play Ascending/Descending buttons → notes play sequentially
Duration per note: 0.4s default
Voice setting determines synth

### Chord playback (staging + cards)
Tap chord name → plays chord voicing via Tone.js
Use guitar voicing (open chords), not root position
Chord plays automatically when arriving in staging

### Progression playback
Loop plays chord sequence in order at current tempo
Each chord plays for its notated duration
Fretboard switches to chord voicing as each chord plays
Staff switches to stacked chord tones

### CAGED position playback
Each position change voices the chord at that register
Timbre subtly changes as position moves up the neck
(higher positions = slightly brighter)

### Voice types
```
Piano:       Tone.Sampler or PolySynth, piano envelope
Strum:       arpeggiated chord, fast strum pattern
Picked:      single notes, picked envelope
Bass+Chord:  bass note beat 1, chord beats 2-3
Click:       metronome only
```

---

## 17. BUILD ORDER FOR CLAUDE CODE

Build one component at a time.
After each component: screenshot check before continuing.
Do not proceed if something looks wrong.

```
Step 1:  Panel + Header
         Logo with mix-blend-mode:multiply, tagline, utility buttons
         Verify logo looks carved into maple, not on dark bg

Step 2:  Controls bar
         All 4 dropdowns + Explore/Build toggle + nav controls
         All 44px minimum height
         Verify tuning dropdown updates when instrument changes

Step 3:  Fretboard
         Static C major first, verify correct strings and note names
         CRITICAL: e top, E bottom. C major = no accidentals
         Then make reactive to dropdowns

Step 4:  Staff
         Treble clef + time sig + scale notes
         Must look like published sheet music
         Then make reactive

Step 5:  Circle of Fifths
         Colorful wedges, no border ring, floating
         Two-way sync with key dropdown
         In Build mode: tap → sends to staging

Step 6:  Explore Mode — CAGED Navigator
         Five positions for selected chord
         Prev/Next stepping
         Arpeggio overlay toggle
         Tone.js voicing per position

Step 7:  Build Mode — Diatonic chord chart
         7 buttons, tap → staging area
         Color coded by quality

Step 8:  Chord staging area
         Display + quality popover + duration selector + confirm

Step 9:  Chord progression strip
         Cards with width = duration
         Duration row inside each card
         Remove button

Step 10: Transport bar
         Play/Pause/Stop/Rewind/Loop
         Time sig + Tempo slider + Voice + Bar counter
         Lead complexity selector
         Save/Export button (stub is fine for v1)

Step 11: Tone.js audio integration
         Scale playback, chord playback, progression loop

Step 12: Ear Training modal
         Full quiz flow with physics tips

Step 13: Polish
         Text sizes — nothing below 10px readable
         Tap target audit — everything 44px+
         Test on tablet viewport
```

---

## 18. FILES IN ~/Desktop/fretcrawler/

```
index.html              ← current working app (read before touching)
Fretcrawler_Logo.png    ← transparent PNG, use mix-blend-mode:multiply
maple-bg.jpg            ← panel background texture
FRETCRAWLER_SPEC.md     ← this document
```

Reference mockup: fretcrawler_v10.html (in Downloads)
Reference screenshots: Affinity mockup PNGs (in Downloads)
Best layout reference: Screenshot_2026-05-31_at_9_54_29_PM.png

---

## 19. OPENING PROMPT FOR CLAUDE CODE SESSION

Copy and paste this exactly when launching Claude Code:

"Read FRETCRAWLER_SPEC.md fully before writing any code.
Then read index.html and describe the current architecture.
Do not write any code yet.
Tell me what exists, what needs to be rebuilt, and what is new.
Then we will proceed one step at a time per the build order in Section 17."

---

*End of specification. Build with care.*
*One step at a time. Screenshot after each.*
