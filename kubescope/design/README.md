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
| 06 | [`06-panes/`](06-panes/) | 32 screens: 14 question panes, 10 object panes, 8 frame screens | 05 · 06 · 08 |
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

## Round 08 — the empty tab

`06-panes/32-Empty-Tab.dc.html` arrived on its own, after the regrouping, and settles what `⌘T`
opens: **a launcher, not a page.** One floating 680 × 458 bar, a 52px field at 19px, and eight of
seventeen rows under it — the list scrolls, which is the whole claim: typing is the interaction and
the list is there to confirm.

Two things it corrects rather than adds. There is **no second bar step** for the twelve panes that
need a cluster: `↵` opens the pane immediately with its header chip focused, because every pane
header carries that chip anyway and a bar step both duplicates and hides it. And the field is
**pane-scoped** — names and authored aliases, never descriptions and never objects — which is what
makes `shell` a teaching state rather than a no-match.

It is filed as a frame screen because that is what it is, and `06-panes` already holds seven others.
Its own filename was `02-Empty-Tab-Spec`, so **a `01-` of that round exists somewhere and is not
here** — the page cites "candidate A's named weaknesses", which is presumably it.

## The numbering

Folder numbers are **reading order**, not delivery order — the material arrived across six rounds
and was regrouped by subject in September 2026. Each page still states its own round in its banner.

Two things moved when it was regrouped: the widget catalogue (`04-controls`) had pages in two
different rounds, and the primitives sheet came from the handles round. Nothing was dropped.

## The conventions

- **Briefs are Hungarian** (`../brief/NN-tema.md`) — they are written to a designer who reads Hungarian.
- **Design output is English** — folder names, page titles, component names, CSS, comments.
- One subject per folder, one page per `.dc.html`, numbered within the folder.
