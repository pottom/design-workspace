# Round 05 — every pane, worked out

One file per pane type and per feature screen. Each file is a **complete 1600×1000 screen** in the
chosen direction (A · Instrument), plus the same pane at 320px-narrow and in its awkward states.
Nothing here is a rule or a component sheet — this is what it looks like assembled.

Built on: `02-screens/A-Instrument-base.dc.html` (the visual system),
`01-directions/04-cselekves-terkep/05-Controls.dc.html` (the widgets),
`…/01-Action-Map.dc.html` (what each pane can do).

## Question panes — scope is settable

| file | pane | capabilities |
|---|---|---|
| `01-Resource-List.dc.html` | resource list — any kind, any cluster | `RES-02/03/07/08/11/12/13` |
| `02-Log.dc.html` | log stream, one container to N clusters | `LOG-01…09` |
| `03-Events.dc.html` | event stream | `RES-04`, `TIME-04` |
| `04-Fleet.dc.html` | ten clusters, one screen | `XC-02`, `CONN-03` |
| `05-Global-Search.dc.html` | one question, N clusters | `XC-01` |
| `06-Change-Stream.dc.html` | what changed, and when | `TIME-07/08` |
| `07-Resource-Monitor.dc.html` | who is eating what, live | `MET-12`, `DIAG-03` |
| `08-Diagnostics.dc.html` | why Pending, why not ready | `DIAG-01/02/04/06` |
| `09-RBAC.dc.html` | what can this account do, both directions | `SEC-01/02/07/08/09/10` |
| `12-Settings.dc.html` | everything about the app, as a pane | `APP-16…19` |
| `18-OpenShift.dc.html` | operators, upgrades, builds, routes | `OCP-07…13` |
| `19-Certificates.dc.html` | what expires, fleet-wide | `SEC-03` |
| `20-Waste.dc.html` | requested against used, namespace to fleet | `DIAG-09` |
| `21-Kubeconfig.dc.html` | contexts, edited safely | `CONN-01/09/10` |
| `22-Watch-Rules.dc.html` | your own alerts over the live streams | `APP-12/13` |

## Object panes — scope is inherited and fixed

| file | pane | capabilities |
|---|---|---|
| `10-Details.dc.html` | one object: status, conditions, owners | `RES-04/05` |
| `11-Assistant.dc.html` | an assistant that cites its evidence | `ASK-01…08` |
| `13-YAML-and-Editor.dc.html` | the stored object and the form, one pane | `RES-06`, `ACT-02/03/12` |
| `14-Terminal.dc.html` | exec, attach, node debug | `ACT-05/09/11` |
| `15-Metrics.dc.html` | one object’s curves, events on them, the builder | `MET-02/04/05/10` |
| `16-What-Would-Happen.dc.html` | impact, drain simulation, forecast — one idea | `XC-03`, `ACT-16`, `DIAG-08` |
| `17-Helm.dc.html` | releases, values per cluster, where they drift | `HELM-01…08` |
| `23-Describe.dc.html` | the readable summary | `RES-10` |
| `24-Node.dc.html` | one node: promised against used, pod by pod | `RES-05`, `DIAG-04`, `ACT-06/07/11` |

## Feature screens — the frame, not a pane

| file | screen | capabilities |
|---|---|---|
| `25-Layouts.dc.html` | tabs, templates, and what persists | `APP-04` · brief §6.6 |
| `26-First-Run.dc.html` | no clusters, no layout | `CONN-01/04/05/08` |
| `27-Time-Travel.dc.html` | the whole interface, half an hour ago | `TIME-01…06/09` |
| `28-Cross-Cluster.dc.html` | “is the same thing running everywhere?” | `XC-03/04/07/08` |
| `29-Cluster-Rail.dc.html` | three widths, one of them zero | `CONN-02/03` · brief §6.4 |
| `30-Bulk-Action.dc.html` | eight objects, three clusters, one write | `RES-11`, `XC-06`, `ACT-17`, `ACT-07` |
| `31-Incident.dc.html` | the four-pane layout, end to end — nine moments, 14:02 → 14:31 | the lot |

All 31 are built. The frame brief’s region map (§7/1) lives in `04-fogantyuk/07-Region-Map.dc.html`.

Each file ends with a short note: what the pane’s primary element is, how dense it is, and what it
gives up.

> **Corrected 2026-09-02.** The tables above previously listed a numbering that never reached disk —
> `10-Certificates` through `24-Form-Editor`, fifteen names for files that do not exist — while
> omitting Assistant, Settings, OpenShift, What-Would-Happen and Node, which do. Diff, Impact
> Preview, Drain Simulation, Form Editor and Metrics Browser were folded into other screens during
> the round rather than dropped: they live in `16`, `13`, `15` and `28`.
