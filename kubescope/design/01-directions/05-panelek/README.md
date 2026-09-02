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
| `10-Certificates.dc.html` | what expires, fleet-wide | `SEC-03` |
| `11-Helm.dc.html` | releases, and where they drift | `HELM-01…07` |
| `12-Metrics-Browser.dc.html` | what metrics exist, and the query builder | `MET-09/10/11` |
| `13-Waste.dc.html` | requested against used, namespace to fleet | `DIAG-09` |
| `14-Kubeconfig.dc.html` | contexts, edited safely | `CONN-01/09/10` |
| `15-Watch-Rules.dc.html` | your own alerts over the live streams | `APP-12/13` |

## Object panes — scope is inherited and fixed

| file | pane | capabilities |
|---|---|---|
| `16-Details.dc.html` | one object: status, conditions, owners | `RES-04/05` |
| `17-Describe.dc.html` | the readable summary | `RES-10` |
| `18-YAML.dc.html` | the stored object, and editing it | `RES-06`, `ACT-02/03` |
| `19-Terminal.dc.html` | exec, attach, node debug | `ACT-05/09/11` |
| `20-Metrics.dc.html` | one object’s curves, events on them | `MET-02/04/05` |
| `21-Diff.dc.html` | two clusters, two times, two revisions | `XC-03`, `TIME-09`, `HELM-04` |
| `22-Impact-Preview.dc.html` | what this write would break | `ACT-16` |
| `23-Drain-Simulation.dc.html` | where the pods would go | `DIAG-08` |
| `24-Form-Editor.dc.html` | limits, env, probes, ConfigMap, Ingress | `ACT-12/13/14/15` |

## Feature screens — the frame, not a pane

| file | screen | capabilities |
|---|---|---|
| `25-First-Run.dc.html` | no clusters, no layout | `CONN-01/04/05` |
| `26-Time-Travel.dc.html` | the whole interface, half an hour ago | `TIME-02/03/09` |
| `27-Cross-Cluster.dc.html` | “is the same thing running everywhere?” | `XC-03/04/07/08` |
| `28-Bulk-Action.dc.html` | eight objects, three clusters, one write | `RES-11`, `XC-06`, `ACT-17` |
| `29-OpenShift.dc.html` | update, operators, quotas, routes | `OCP-07…13` |
| `30-Incident.dc.html` | the four-pane layout, end to end | the lot |

Each file ends with a short note: what the pane’s primary element is, how dense it is, and what it
gives up.
