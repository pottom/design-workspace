# Design workspace

Where design work is commissioned and delivered. One directory per product; inside each, what was
asked for and what came back.

```
<product>/
  brief/     what we ask for — read this first
  design/    what the design produces — write here
```

## How a round works

1. A brief lands in `<product>/brief/`. It says what to design, what is fixed, and what is free.
   It is the whole input: if something is not in there, it is not a constraint.
2. The design writes into `<product>/design/`, in its own directory per round —
   `design/01-directions/`, `design/02-screens/`, and so on. Nothing is overwritten; a later round
   sits beside the earlier one, so a decision can be traced back to what it replaced.
3. Whatever a round produces, it also leaves a `README.md` beside it saying **what was decided and
   what was given up**. The pictures age; the reasoning is what the next round needs.

## What the output is for

**It is not a specification to be reimplemented — it is the starting code.** The applications here
are built in React and TypeScript, so components and CSS come across as they are. Write them as if
you were writing the implementation, because you are.

Tokens as CSS custom properties. Screens as components. Fake data separated from the components that
draw it, so the same component can later be handed real data.

## Products

| | |
|---|---|
| [`kubescope/`](kubescope/) | A multi-cluster Kubernetes and OpenShift console. Native desktop app, tiling panes. |
