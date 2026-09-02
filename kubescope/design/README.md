# The design

Eight folders, ordered the way somebody building should read them. Every `.dc.html` is a standalone
page — open it in a browser, no server needed. The `support.js` beside it is the renderer.

| | folder | what settles here | round |
|---|---|---|---|
| 01 | [`01-directions/`](01-directions/) | the look: **A · Instrument** (chosen) and B · Ledger | 02 |
| 02 | [`02-prototype/`](02-prototype/) | the same direction, clickable | 02 |
| 03 | [`03-action-map/`](03-action-map/) | what every kind can do · the rules · eight questions · the overlay grammar | 04 |
| 04 | [`04-controls/`](04-controls/) | **the widget catalogue** — controls, dropdowns, parts, and the 20 primitives in Svelte terms | 04 · 06 |
| 05 | [`05-frame/`](05-frame/) | handles, drag, selection, **the six decisions**, the component tree, the region map | 04 |
| 06 | [`06-panes/`](06-panes/) | 31 screens: 14 question panes, 10 object panes, 7 frame screens | 05 · 06 |
| 07 | [`07-narrow/`](07-narrow/) | **what the bodies do as they narrow** — the grammar, and every pane that needed a decision, at 958 · 638 · 382 | 07 |
| 08 | [`08-overlays/`](08-overlays/) | the four layers drawn: palette, help, confirm, time | 06 |
| 09 | [`09-icon/`](09-icon/) | app icon and tray icons, five sizes | 06 |

## Where to start

**Building a pane?** `05-frame/04-Decisions.dc.html` first — six rules that are settled and must not
be re-argued. Then `05-frame/05-Components.dc.html` for the component tree, then the pane's own
screen in `06-panes/`.

**Building a control?** `04-controls/04-Primitives.dc.html`. Twenty primitives, every state drawn,
each with a `MUST NOT` line naming the one way it gets misused.

**Deciding what a pane is for?** `03-action-map/01-Action-Map.dc.html` — 316 actions, every one
naming what it opens. It is the only place that says how often a pane type is actually reached.

## The base width is 638, not 1600

`07-narrow` closed the last gap, and it changed the default: **every body is designed at 638 first
and widened**, because designing at 1600 and cutting produces a narrow state that is a list of
absences. Three bands, two thresholds — tight 382, mid 480–803, wide 804 and up.

What is still only at 1600: `06-panes/26` first run and `06-panes/30` bulk action. Neither has a
structure that dies, so both follow the grammar without a drawing — but nobody has checked them.

## The numbering

Folder numbers are **reading order**, not delivery order — the material arrived across five rounds
and was regrouped by subject in September 2026. Each page still states its own round in its banner.

Two things moved when it was regrouped: the widget catalogue (`04-controls`) had pages in two
different rounds, and the primitives sheet came from the handles round. Nothing was dropped.

## The conventions

- **Briefs are Hungarian** (`../brief/NN-tema.md`) — they are written to a designer who reads Hungarian.
- **Design output is English** — folder names, page titles, component names, CSS, comments.
- One subject per folder, one page per `.dc.html`, numbered within the folder.
