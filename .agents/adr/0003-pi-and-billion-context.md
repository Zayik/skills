# Run the workflow skills on pi with billion-context, as one main loop

This fork targets one environment: the **pi** harness with **billion-context** (bili) loaded. Upstream's skills assume Claude Code: a Skill tool, a subagent tool, `/clear`, `/compact`, and a context window that runs out near the ~150k smart zone. None of that holds here, so the skills on the main paths are rewritten for pi and for bili. Portability to other harnesses is deliberately given up.

The full design, with evidence and measurements, is the spec in the billion-context fork: `docs/pi-workflow-spec.md` (https://github.com/Zayik/billion-context/blob/2026-10-29_pi-fold-subagents/docs/pi-workflow-spec.md).

## The rule

Anthropic's "Building agents: When to fold subagents into your main agent loop": one main loop does the work, procedure lives in skills, capabilities live in tools. A subagent is kept only when one of three conditions holds: measurable degradation from context bloat; parallel tasks needing fresh, discardable contexts; or an agent that struggles with tool choice or has conflicting instructions. Kept subagents are called as tools and hand back a compact result.

## Decision

- **Pi conventions.** Skills are invoked as `/skill:<name>`, loaded by reading their `SKILL.md` ("load the `tdd` skill"), and a fresh context is `/new`. The only subagent mechanism is bili's `acp_delegate`, with three roles: `reviewer`, `researcher` (large, disposable lookups only) and `worker` (parallel, disjoint edits only).
- **bili awareness.** bili folds consumed context into summaries and keeps the originals recoverable through `search_context` and `decompress`. Skills recover folded decisions and still-valid earlier reading from there instead of re-reading files or re-asking the user. If those tools are missing, bili is not running, and the skill says so.
- **Where each phase runs.**
  - `/grill-with-docs`, `/to-spec`, `/to-tickets`: one session. Facts are looked up by the main agent.
  - `/implement`: one unit of work. Continuing in the same session is the default; `/new` is for isolation.
  - `/implement-spec`: continues in the planning session by default. One main loop walks the ticket frontier and folds each finished ticket. A `worker` is used only for two or more sizable, disjoint frontier tickets, briefed with findings instead of pointers. No exploration or merger subagents.
  - `/code-review`: two `reviewer` delegates in parallel.
  - `/handoff`: unchanged, for portability only.
- **No new triggers.** Nothing hands off or compacts automatically. bili handles context growth.

## Scope

Adapted: `grill-with-docs`, `grilling`, `to-spec`, `to-tickets`, `implement`, `implement-spec`, `tdd`, `code-review`, `handoff`, `ask-matt` (with `PHASE-BOUNDARIES.md`), and their docs pages. `domain-modeling` needed no change.

Not adapted, still carrying Claude Code wording: `research`, `retro`, `triage`, `wayfinder`, `codebase-design`, `improve-codebase-architecture`, `grill-me`, `writing-for-agents`, and the `in-progress/` and `misc/` skills. `ask-matt` still routes to `research`, which describes a background agent.

## Consequences

- These skills no longer work as written in Claude Code or Codex.
- Syncing with upstream means re-applying these changes to the adapted skills by hand.
- Whether models actually stop duplicating reads, still delegate when they should, and load the skills at the right time can only be judged by the live test plan in the billion-context fork (`docs/pi-workflow-live-test-plan.md`).
