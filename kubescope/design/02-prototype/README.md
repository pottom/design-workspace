# Round 03 — the clickable prototype

`Prototype.dc.html` — the Instrument design, working. Real interaction, fake data. Nothing here is a
picture of a state; every state is produced by the same gestures the app will have.

Open it and try, in this order:

1. **Right-click a pod row.** Open logs / manifest / shell / events, previous instance, copy name,
   restart rollout, delete pod. The menu is titled with the row's phase and name, so you always know
   what you right-clicked.
2. **Open a log.** It opens as a new pane by splitting the pane you came from — or it fills an empty
   pane if there is one. Its scope is **pinned**: the header says `pinned <pod> · <container>`, and
   clicking the cluster chip tells you why it cannot be re-pointed.
3. **Right-click a pane header** (or the `⋮`). Split right / split down / duplicate / change cluster /
   change namespace / ask for nodes instead / pause watch / close. Closing gives the space back to a
   neighbour.
4. **Click a row, then press `L` `Y` `S` `E`.** The selection strip at the bottom of the pane does the
   same thing with the mouse.
5. **Click the cluster chip** on a question pane, or **click a cluster in the left rail**: the same
   question is asked of another cluster and the rows change. Object panes refuse, and say so.
6. **Type in the pane's filter**, or use `!running`. The count in the header is live (`6 of 41`).
7. **`break watch`** in the top-right marks the active pane stale: it says what it is showing is 47s
   old and offers a retry. This is what a pane looks like when it has lost the truth.
8. **`1 2 3 4`** switch workspaces. `3 · incident` is the four-pane, three-cluster layout.
9. **`⌘K`** opens the command palette; typing filters it; clicking a command runs it.
10. **`compact / roomy`** switches row height 22px ↔ 26px, live.
11. **Right-click a cluster in the rail** for pause / probe / open pods / reveal in kubeconfig.
12. **Restart or delete** opens a confirmation that names the cluster, shows the equivalent `kubectl`
    line, and says what happens to traffic. Delete actually removes the row.

## What is deliberately not simulated

- **Dragging split handles and pane/tab drag with live preview.** The gestures exist in the app and
  are staying; they are motion work, not layout work, and faking them badly would teach the wrong
  thing. Splitting is exercised through the menus instead.
- **Tabs in one pane slot** — the model allows them, the prototype always opens a new pane.
- **Real streaming.** Logs are a fixed 40-line fixture; the header claims a rate but nothing moves.
- **Time travel, cross-cluster diff, metrics, RBAC, certificates, OpenShift** — the palette and the
  row menu point at them so you can see where they land, and they answer with a note instead.

## The model, as built

- **Panes live in a 4×4 grid** — that is this prototype's stand-in for the split tree, and it enforces
  the same thing the tree does: a pane cannot be split below the minimum, and it says so.
- **Two pane families are real in the code.** A question pane (pods / nodes / events / fleet) has a
  settable cluster and namespace; an object pane (logs / manifest / shell) carries `obj` and refuses
  to change scope. This is the one rule the round-01 brief said must survive, and it is enforced by
  the state, not by the drawing.
- **Fake data is separated** from the components: `podsOf(cluster)` builds a deterministic pod list
  per cluster from a workload table plus a per-cluster trouble table, so nine clusters have their own
  believable contents and the same cluster always looks the same. Swap those functions for the Rust
  side and the components do not change.
