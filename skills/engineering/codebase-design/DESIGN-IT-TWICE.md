# Design It Twice

When the user wants to explore alternative interfaces for a chosen deepening candidate, use this pattern of parallel delegates. Independence is the point: each design comes from a fresh context that hasn't seen the others, which is why this runs as delegates rather than in your own loop. Based on "Design It Twice" (Ousterhout): your first idea is unlikely to be the best.

Uses the vocabulary in [SKILL.md](SKILL.md): **module**, **interface**, **seam**, **adapter**, **leverage**.

## Process

### 1. Frame the problem space

Before launching the delegates, write a user-facing explanation of the problem space for the chosen candidate:

- The constraints any new interface would need to satisfy
- The dependencies it would rely on, and which category they fall into (see [DEEPENING.md](DEEPENING.md))
- A rough illustrative code sketch to ground the constraints, not a proposal, just a way to make the constraints concrete

Show this to the user, then immediately proceed to Step 2. The user reads and thinks while the delegates work in parallel.

### 2. Launch the delegates

In the same turn, make three or four `acp_delegate` calls with `agent: "researcher"` and the default `async: true` (extra launches queue). The `researcher` is read-only, so each one replies with its design as text and writes no code. If `acp_delegate` is missing, billion-context isn't running: tell the user. Each must produce a **radically different** interface for the deepened module.

Brief each delegate with a separate technical brief built from findings you already have (`file:line` locations, coupling details, dependency category from [DEEPENING.md](DEEPENING.md), what sits behind the seam), not pointers it must re-read. The brief is independent of the user-facing problem-space explanation in Step 1. Give each agent a different design constraint:

- Agent 1: "Minimize the interface: aim for 1–3 entry points max. Maximise leverage per entry point."
- Agent 2: "Maximise flexibility: support many use cases and extension."
- Agent 3: "Optimise for the most common caller: make the default case trivial."
- Agent 4 (if applicable): "Design around ports & adapters for cross-seam dependencies."

Include both [SKILL.md](SKILL.md) vocabulary and GLOSSARY.md vocabulary in the brief so each delegate names things consistently with the architecture language and the project's domain language.

Each delegate replies with:

1. Interface (types, methods, params, plus invariants, ordering, error modes)
2. Usage example showing how callers use it
3. What the implementation hides behind the seam
4. Dependency strategy and adapters (see [DEEPENING.md](DEEPENING.md))
5. Trade-offs: where leverage is high, where it's thin

### 3. Present and compare

Present designs sequentially so the user can absorb each one, then compare them in prose. Contrast by **depth** (leverage at the interface), **locality** (where change concentrates), and **seam placement**.

After comparing, give your own recommendation: which design you think is strongest and why. If elements from different designs would combine well, propose a hybrid. Be opinionated: the user wants a strong read, not a menu.
