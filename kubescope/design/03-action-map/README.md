# Round 04 — the action map, the rules, the eight questions

Answers to `brief/03-kepernyolista.md`. Four files, all React-ready markup with tokens as CSS custom
properties (`--ks-*`) declared on each file's root element.

| file | brief section |
|---|---|
| `01-Action-Map.dc.html` | §4 — 34 kinds × every action × what it opens; the six decisions behind the set; **the mouse-gesture map**; a recommended **eleven-phase build order** |
| `02-Rules.dc.html` | §5 — the fourteen rules, each with a live specimen |
| `03-Eight-Questions.dc.html` | §3 — the eight questions, answered |
| `04-Overlays.dc.html` | popups, dialogs, confirmations, toasts, palette, editor sheet, progress, first run |
| `05-Controls.dc.html` | **the widget catalogue** the scope document asks for as phase M-2: 9 controls, 9 indicators, 6 charts, each carrying the capability ID it exists for |
| `kubescope-logo.png` | the supplied dark logo, used as delivered — brand blues `#2f6bff → #24c8ff`. It appears in exactly two places: first run and the window icon. |

Everything is interactive where interaction is the argument: the action map switches between one row
and an eight-row selection, and between a writable and a read-only cluster; the truncation rule has a
width slider; per-pane zoom has five live levels; the drag demo produces the real drop result.

## On the capability scope

`01-kepesseg-scope.md` was read in full for this round, and it changed two things. First, its **M-2
phase is exactly this catalogue** — "token sheet + widget catalogue, not screen designs" — so
`05-Controls.dc.html` is written to that definition and every item names its capability ID. Second,
its technology section (egui, `egui_plot`, `Theme` struct) is **superseded** by
`00-kepernyo-brief.md`: Tauri + React + TypeScript, uPlot, CodeMirror, xterm.js, TanStack Virtual.
Everything else in the scope document still stands, and the catalogue is checked against it.

Capabilities that had no home in rounds 01–03 and now have one: `MET-09/10/12`, `ACT-13/15/16/17`,
`SEC-07/08/09/10`, `DIAG-08/09`, `TIME-07/08/09`, `OCP-07…13`, `APP-12/13/14/15`, `HELM-06/07`,
`LOG-04/08/09`, `RES-10/11/12/13`, `CONN-09/10`.

## The decisions, in one page

**The action set.** Three ways in (right-click, letter key, palette), one registry behind them, four
groups always in the same order: look / go / change / destroy. The primary is per kind and is what
`⏎` and double-click do — a Pod's is its overview, a Build's is its log, an Event's is the object it
points at, an InstallPlan's is approval.

**Where new panes go.** An empty pane wins; otherwise the source pane splits along its longer axis.
`⌥` makes a tab, `⇧` replaces. No dialog ever asks.

**Multi-selection.** The set shrinks to what is true for all of them; the total lives in the frame
because it spans panes; the confirmation groups by cluster, because eight pods in three clusters is
three writes.

**Read-only clusters.** Writes stay visible and greyed with the reason in place of the shortcut.
Hiding them teaches nothing and sends people back to kubectl.

**Destructive grammar.** Three tiers by what cannot be undone: undo toast / one confirmation /
type the name. A production cluster promotes every action one tier.

**Completeness.** One field on every read — what answered, and when — with five presentations:
complete, partial, stale, frozen, not now. Amber `◐` means "true, but incomplete or old" and never
takes content away. Missing sources are named, never summarised, and keep their row.

**Time travel** is frame-level, striped amber, undismissable, timestamped in every pane header — and
the terminal refuses to pretend, staying live and green while the frame is amber.

**Sizes.** 1280×800 to 5K; pane minimum 320×180; content maximum 1180px per pane, so a wide screen
becomes more panes rather than longer rows. Pane internals are sized in `em` from the first commit,
because per-pane zoom cannot be retrofitted.

## What we gave up

- **The gesture map assumes a mouse with three buttons and a wheel.** Trackpad-only users lose middle
  click (open in a new pane) and get a two-finger-tap fallback for right click. We did not design a
  trackpad-first path; if that is a real population, say so and it changes the middle-click rules.
- **No submenus for actions**, only for lists. That keeps the menu learnable but makes the Pod menu
  nineteen items tall — about 300px. On a 180px-tall pane the menu will overlap other panes. Accepted:
  menus are allowed to leave their pane.
- **The action map is data, not yet a component.** It ships as the registry the menus, key map,
  palette and confirmations all read from — but the four renderers are phase 4 of the build order,
  not this round.
- **Twelve of the fourteen rules are decided, two are proposals**: the exact list of mapped error
  strings (rule 2) needs your real cluster errors, and the persistence set (rule 12) is a field list
  we expect you to argue with.

## Open questions

1. **Cluster colour with forty clusters.** One reserved hue per cluster runs out around twelve. Is a
   user-assigned colour (a kubeconfig annotation) acceptable, with generated fallbacks?
2. **Does `⏎` on a Node open its overview or its pod list?** We chose overview for consistency; the
   pod list is the more common intent. Your call.
3. **Trackpad population** — see above.
4. **Audit lines.** We write one when a secret value is copied. Is there a place those should go
   besides the local log?
