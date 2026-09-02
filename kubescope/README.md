# KubeScope

A multi-cluster Kubernetes and OpenShift console. Native desktop application (Tauri), tiling pane
layout, Svelte and TypeScript interface.

**The design is finished and being built from.** Start at [`design/README.md`](design/README.md) —
eight folders in reading order, with the six settled decisions at the front.

## The briefs — what was asked for, in order

| | |
|---|---|
| [`brief/00-kepernyo-brief.md`](brief/00-kepernyo-brief.md) | the original brief: three directions first, then one design |
| [`brief/01-kepesseg-scope.md`](brief/01-kepesseg-scope.md) | the capability map — the contract about what the product does |
| [`brief/02-hol-tartunk.md`](brief/02-hol-tartunk.md) | the behaviour model: tiling, the two pane families, the real data, the sizes |
| [`brief/03-kepernyolista.md`](brief/03-kepernyolista.md) | the same capabilities arranged **by pane** |
| [`brief/04-fogantyuk-kijeloles.md`](brief/04-fogantyuk-kijeloles.md) | the handles, the drag gestures, and the grammar of selection |
| [`brief/05-design-atnezes.md`](brief/05-design-atnezes.md) | **the review**: where the redundancy is, and the five consolidations |
| [`brief/06-epitesi-katalogus.md`](brief/06-epitesi-katalogus.md) | **the build catalogue**: pane → body → decided rules → what is missing |
| [`brief/plans/`](brief/plans/) | the two earlier plans the round-04 brief cited. **Six of their prose lines are superseded** — see `design/05-frame/04-Decisions.dc.html` |

## The design

| | |
|---|---|
| [`design/`](design/) | 8 folders · 50 pages · the whole product drawn |

## What is open

1. **Narrow states.** Everything is drawn at 1600 × 1000; thirteen of the 31 pane screens have no
   narrow treatment. The widths that actually occur are 958 · 638 · 572 · 382.
2. **Five consolidations** proposed in the review: nineteen pane kinds down to eleven, with no
   entrance to the product removed.
3. **Six prose lines** in `brief/plans/` now contradict what was decided.

## The conventions

- **Briefs are Hungarian.** They are written to a designer who reads Hungarian.
- **Design output is English** — folder names, page titles, component names, CSS, comments.
- Folder numbers in `design/` are **reading order**, not delivery order. Each page states its round.

There is deliberately **no screenshot of the interface that preceded this design** in here. One was
built, and it came out badly; showing it would have anchored the attempt to it. What was learned
from it is written out in the briefs instead, as pitfalls rather than pictures.
