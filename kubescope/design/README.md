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
| 07 | [`07-overlays/`](07-overlays/) | the four layers drawn: palette, help, confirm, time | 06 |
| 08 | [`08-icon/`](08-icon/) | app icon and tray icons, five sizes | 06 |

## Where to start

**Building a pane?** `05-frame/04-Decisions.dc.html` first — six rules that are settled and must not
be re-argued. Then `05-frame/05-Components.dc.html` for the component tree, then the pane's own
screen in `06-panes/`.

**Building a control?** `04-controls/04-Primitives.dc.html`. Twenty primitives, every state drawn,
each with a `MUST NOT` line naming the one way it gets misused.

**Deciding what a pane is for?** `03-action-map/01-Action-Map.dc.html` — 316 actions, every one
naming what it opens. It is the only place that says how often a pane type is actually reached.

## The known gap

**The narrow states.** Every screen is drawn at 1600 × 1000. The chrome degrades by a general rule
(the header and control bar drop items at named pixel widths), but **the bodies do not**, and
thirteen of the 31 screens have no narrow treatment at all — mostly the fifteen that arrived in
round 06. The widths that actually occur when panes are split are 958 · 638 · 572 · 382, and none of
them is drawn. See `../brief/05-design-atnezes.md` §5.

## The numbering

Folder numbers are **reading order**, not delivery order — the material arrived across five rounds
and was regrouped by subject in September 2026. Each page still states its own round in its banner.

Two things moved when it was regrouped: the widget catalogue (`04-controls`) had pages in two
different rounds, and the primitives sheet came from the handles round. Nothing was dropped.

## The conventions

- **Briefs are Hungarian** (`../brief/NN-tema.md`) — they are written to a designer who reads Hungarian.
- **Design output is English** — folder names, page titles, component names, CSS, comments.
- One subject per folder, one page per `.dc.html`, numbered within the folder.
