# Run the workflow skills on pi with billion-context, as one main loop

This fork targets one environment: the **pi** harness with **billion-context** (bili) loaded. Upstream's skills assume Claude Code: a Skill tool, a subagent tool, `/clear`, `/compact`, and a context window that runs out near the ~150k smart zone. None of that holds here, so the skills on the main paths are rewritten for pi and for bili. Portability to other harnesses is deliberately given up.

The full design, with evidence and measurements, is the spec in the billion-context fork: `docs/pi-workflow-spec.md` (https://github.com/Zayik/billion-context/blob/2026-10-29_pi-fold-subagents/docs/pi-workflow-spec.md).

## The rule

Anthropic's "Building agents: When to fold subagents into your main agent loop": one main loop does the work, procedure lives in skills, capabilities live in tools. A subagent is kept only when one of three conditions holds: measurable degradation from context bloat; parallel tasks needing fresh, discardable contexts; or an agent that struggles with tool choice or has conflicting instructions. Kept subagents are called as tools and hand back a compact result.

## Decision

- **Pi conventions.** Skills are invoked as `/skill:<name>`, loaded by reading their `SKILL.md` ("load the `tdd` skill"), and a fresh context is `/new`. The only subagent mechanism is bili's `acp_delegate`, with three roles: `reviewer`, `researcher` (large, disposable lookups only) and `worker` (parallel, disjoint edits only).
- **Folding belongs to bili, not the skills.** bili decides when to fold (its nudge) and documents how to recover (its `search_context` and `decompress` tool descriptions). Skills never tell the agent to fold, and never repeat bili's recovery recipe: two sources of fold rules drift apart and contradict each other, the conflicting-instructions failure the article warns about. A skill keeps one bili-neutral line where its own steps would otherwise waste work: recover what the session already holds, summaries included, before re-reading files or re-asking the user.
- **Where each phase runs.**
  - `/grill-with-docs`, `/to-spec`, `/to-tickets`: one session. Facts are looked up by the main agent.
  - `/implement`: one unit of work. Continuing in the same session is the default; `/new` is for isolation.
  - `/implement-spec`: continues in the planning session by default. One main loop walks the ticket frontier. A `worker` is used only for two or more sizable, disjoint frontier tickets, briefed with findings instead of pointers. No exploration or merger subagents.
  - `/code-review`: two `reviewer` delegates in parallel.
  - `/handoff`: unchanged, for portability only.
- **No new triggers.** Nothing hands off or compacts automatically. bili handles context growth.
  - `/research`: one async `researcher` delegate reads; the main session writes the file, since the role is read-only. Reading the agent itself must understand stays in the main loop.
  - `/wayfinder`: research tickets run as parallel `researcher` delegates.
  - `/improve-codebase-architecture`: the codebase walk runs in the main loop. Its friction notes feed the report and the grilling, so a subagent summary of them would be lossy.
  - `codebase-design`'s design-it-twice: parallel `researcher` delegates, kept because each design needs a context that has not seen the others.
- **Phase boundaries.** Four options: Continue, `/new`, `/handoff`, Delegate. Fold is not one; bili folds on its own.
- **Briefs.** A delegate is briefed only with findings already in hand; the main agent never reads files just to write a brief. Reviewers are told `AGENTS.md` and `CLAUDE.md` are already in their context.

## Scope

Adapted: `grill-with-docs`, `grill-me`, `grilling`, `to-spec`, `to-tickets`, `implement`, `implement-spec`, `tdd`, `code-review`, `handoff`, `ask-matt` (with `PHASE-BOUNDARIES.md`), `research`, `wayfinder`, `improve-codebase-architecture`, `codebase-design`, `triage`, `retro`, and their docs pages. `domain-modeling` needed no change.

Not adapted: `writing-for-agents` (it mentions subagent dispatch only as an example of a context boundary), and the `in-progress/` and `misc/` skills. `in-progress/chief-of-staff` still says to do all work in subagents, the opposite of this ADR.

Amended 2026-11-01: the rest of the promoted skills were adapted, and fold instructions were moved out of the skills (see "Folding belongs to bili").

## Consequences

- These skills no longer work as written in Claude Code or Codex.
- Syncing with upstream means re-applying these changes to the adapted skills by hand.
- Whether models actually stop duplicating reads, still delegate when they should, and load the skills at the right time can only be judged by the live test plan in the billion-context fork (`docs/pi-workflow-live-test-plan.md`).
