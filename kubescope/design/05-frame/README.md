# Round 04 — handles, gestures, selection

From `brief/04-fogantyuk-kijeloles.md`. Six pages. Everything is drawn at 1:1, because the previous
round's numbers-without-drawings is what produced invented handles.

| file | what it settles | brief |
|---|---|---|
| `01-Handles.dc.html` | split handle in five states, the pane's three states, six tab-strip cases, seven cursors | §5.1 §5.3 §5.4 §5.5 |
| `02-Drag.dc.html` | ten frames from grab to release · five targets · the refusal | §5.2 |
| `03-Selection.dc.html` | six specimens, what is selectable per pane, the gestures, the lifetime table | §6 |
| `04-Decisions.dc.html` | the six contradictions, each with the loser named and what it was right about | §7 |
| `05-Components.dc.html` | 27 frame components + 4 stores: composition, inventory, props, `MUST NOT` | deliverable |
| `06-Primitives.dc.html` | 22 controls, indicators, charts and surfaces, every state drawn | deliverable |

## The decisions in one place

1. **Focus** — the code's header tint, plus a cyan header hairline. No ring, no geometry change.
2. **Header** — three buttons at 17px (`⊞ ⋯ ✕`). Filter is the control bar's field; maximise is `⌘⏎`.
3. **Placement preview** — the live rehearsal, not one rectangle. A rectangle cannot show the split
   that collapses behind you, so it lies while you decide.
4. **Split** — new pane 40%, threshold 804px, and below it `⊞` becomes *open as a tab* rather than
   greying out.
5. **Row's 2px left edge** — status keeps it. Selection is fill plus a 1px outline around a run.
6. **Selected row fill** — 8% on a neutral row, 10% *composited* on a tinted one.

Plus two the brief left open and one the customer asked for mid-round:

- **Multi-selection exists.** `⌘`/`⇧` click, `⇧`-arrow, `⌘A` over the filtered set. Arrows move the
  cursor and select nothing; `Space` toggles.
- **A selection never shrinks in silence.** Rows hidden by a filter are counted; dead rows stay as
  tombstones; the action bar states the number it would really touch.
- **The divider is never a colour channel.** A pane's border is 1px `--ks-line` and its grip 3 × 20px
  `--ks-line-strong`, both neutral. Every hue already means something on the data — status on the
  row's 2px mark, cluster identity in the header and the fleet rows — so a coloured pane edge adds a
  fourth meaning exactly where the eye separates panes. Applied across all 31 pane screens
  (30 coloured edges removed); the tiled screens gained the neutral grip at each divider instead.
- **The split handle is visible at rest** — a 3 × 20px grip in `--ks-line-strong` at the midpoint.
  Rest, hover and drag are one object at three sizes: 20px grey, 30px cyan, then the whole edge.

## Two things to reconcile before building

- The brief cites `kubescope-frame-design.dc.html` and `kubescope-panel-design.dc.html`, neither of
  which is in the repo. Its **verbatim code values** were taken as the authority: cyan `#3fe0d0`,
  error row `rgba(255,77,109,.05)`, selected row 8%, header 28px, control bar 26px, tab strip 22px,
  pane minimum 320 × 180. The neutral greys come from `design/01-directions/`, since nothing quotes them.
- Six lines of prose in those two plans are now wrong. `04-Decisions.dc.html` names each one.

## Everything the brief asked for is here

The region map (§7/1 of the earlier frame brief) landed as page 7 of this round; the last two pane
screens — bulk action and the end-to-end incident — are `06-panes/30` and `31`. Nothing from
`brief/04-fogantyuk-kijeloles.md` or the earlier frame brief is outstanding.

The one thing still needing the customer rather than the designer: six lines of prose in
`kubescope-frame-design` / `kubescope-panel-design` now contradict what is built and what page 4
decides. Each is named there so the documents can be corrected.
