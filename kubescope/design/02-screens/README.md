# Round 02 — the chosen direction

**Chosen: A — Instrument.**

`A-Instrument-base.dc.html` is the round-02 base: the same three screens as round 01, with your
decisions applied. This is what the rest of the round is built on.

`B-Ledger-dense.dc.html` is **superseded** — it was B re-cut to 41 rows before you chose A. Kept
beside, not deleted, because it holds one idea worth stealing later (a written sentence per pane) and
because it is the evidence for why the density decision hurts B and not A.

| decision | what it changed |
|---|---|
| Density: 41 rows | 22px rows, A's own number. At 1000px the chrome leaves room for **40 whole rows** — the 41st would be sliced, so the pane shows 40; at 1080p it is 46. |
| Column headers only on wide panes | The one-pane screen keeps its 20px column bar (1550px wide, it earns it). The four-pane incident screen has **none**: chrome per pane drops from 44px to **24px**, which is a whole extra row per pane at 1280×800. Rows carry their own units instead (`1/2`, `47`, `4d2h`). |
| Steal nothing from B or C | No serif sentences, no spine, no fleet matrix. Kept A's own vocabulary: one-line pane header, tinted unhealthy rows, two-letter cluster rail, the `L / Y / S / E / R` action strip. |

Also fixed from round 01: the incident pod pane no longer ends in empty space. Under the six pods
that need you and the `checkout-api` explanation, the pane continues with the rest of the cluster
under a hairline — the pane is full without inventing content.

## The rule, written down

A pane shows a column bar when it is **wider than 900px and taller than 400px**. Below that the bar
is dropped and the row labels itself. This is one rule, checked per pane, not per screen — so the
same pod list has headers when it owns the workspace and loses them when it is one of four.

## Chrome budget

| pane | round 01 | now |
|---|---|---|
| one pane, full workspace | 24 + 20 = 44px | 44px |
| four panes, 1600×1000 | 44px each = 176px total | 24px each = 96px total |
| four panes, 1280×800 | 44px each | 24px each — one extra data row per pane |

## Still to come in this round

1. First run — no clusters, no layout (`CONN`).
2. Time travel — the same screen half an hour ago (`TIME-01`), with field-level diff.
3. The cross-cluster question — "is the same thing running everywhere?" (`XC`).
4. Sketch level: metrics + events timeline, diagnostics, RBAC answer, certificates, a write action
   with confirmation, the command palette, OpenShift differences.
5. The clickable prototype (brief `6.2`): open a pane from empty, split it and fill the second, change
   a pane's cluster and namespace, filter with a live count, select a row and open logs / YAML / shell
   from it, and a pane that has lost its watch.
