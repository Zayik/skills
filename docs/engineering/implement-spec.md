## What it does

`implement-spec` takes a [spec](https://www.aihero.dev/ai-coding-dictionary/spec) and its [tickets](https://www.aihero.dev/ai-coding-dictionary/ticket) and lands the whole thing in one run, on a single **integration branch**. One [agent](https://www.aihero.dev/ai-coding-dictionary/agent) builds every ticket itself, in the same [session](https://www.aihero.dev/ai-coding-dictionary/session) that planned the spec, then runs [code-review](https://aihero.dev/skills-code-review) over the result and resolves the tickets.

It builds in **one main loop**, because that session already holds the reasoning behind every ticket. Handing tickets to fresh subagents would make each of them re-read what the main agent already knows. billion-context keeps the long run lean: each finished ticket is folded into a summary, and the decisions and earlier reading of the code stay recoverable with `search_context` when a later ticket needs them. It still reads the tickets as a **task graph**, not a list: blocking edges decide what can start, so there is always a **frontier** of tickets whose blockers have landed.

## When to reach for it

You invoke this by typing `/implement-spec`, and the agent won't reach for it on its own.

| Your situation | Reach for |
| --- | --- |
| A spec, split into tickets with blocking edges, that you want landed in one run | `/implement-spec`, in the session that planned it |
| The planning session is gone (another day, another machine, someone else's spec) | `/implement-spec` in a fresh session, pointed at the spec |
| One ticket at a time, with you checking each one | [implement](https://aihero.dev/skills-implement) |
| A spec that isn't split into tickets yet | [to-tickets](https://aihero.dev/skills-to-tickets) first |
| A small piece of work with no real graph to it | [implement](https://aihero.dev/skills-implement) directly |

## Prerequisites

- **An issue tracker.** The skill reads the tickets from, and resolves them on, the tracker [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) configured. If none has been configured, it stops and tells you to run that first rather than guessing.
- **Tickets with blocking edges**, as [to-tickets](https://aihero.dev/skills-to-tickets) writes them.
- **pi with billion-context loaded.** The fold-and-recover loop depends on billion-context's `compress`, `search_context` and `decompress` tools, and parallel tickets use its `acp_delegate` `worker` role.

## One loop, ticket by ticket

For each ticket on the frontier, the agent recovers what it already knows (the ticket's decisions, and any earlier reading of the files it touches), reads only what is missing or has changed since, builds the ticket with [tdd](https://aihero.dev/skills-tdd), commits it to the integration branch, and folds it into one summary: the commit, what changed, the decisions made while building, and the gotchas. A file that changed since it was read is always read again, never edited from a folded copy.

Parallelism is the exception. Only when two or more sizable frontier tickets are independent and touch disjoint files does the agent hand all but one to `worker` delegates, each in its own git worktree. Each worker is briefed with findings, not pointers, so it reads only the files it edits. The agent builds the remaining ticket itself, stays off the workers' files, and merges their branches when they report.

The tracker decides whether a pull request exists at all. If your tracker closes work through PRs, or you ask for one, the skill opens a draft PR after the first ticket lands and marks it ready at the end. Otherwise the run stops on the integration branch with every ticket resolved the way your tracker closes work, which works fully offline against a local markdown tracker.

## Common questions

**How is this different from running `/implement` on each ticket myself?**

With `implement` you are the dispatcher: you start each ticket, check it, and keep track of which tickets are unblocked. `implement-spec` walks the graph for you in one session and lands everything on one branch. The price is that you no longer read each ticket's work as it lands; you review the integration branch at the end. For a small change with no real graph, skip it and use `implement` directly.

**Won't one session run out of context on a big spec?**

That was the reason upstream gave every ticket to a subagent. With billion-context the session doesn't hit that wall: finished tickets are folded as the run goes, so the live context stays small while the originals stay recoverable. Nothing in the skill hands off or compacts on a token count. If the agent visibly degrades (re-asking settled decisions, repeating a failed fix, losing track of which tickets are done), stop it and start a fresh session from the spec and the integration branch.

**Should I run it in the planning session or a fresh one?**

The planning session, whenever you can. It holds the grilling, the spec's reasoning and the reading done for the tickets, so nothing has to be re-read. Start fresh only when that session is unavailable or unrelated.

**Does it need GitHub? I want it to stop at the branch.**

No. The run ends on the integration branch. A PR opens only when the configured tracker closes work through PRs or you ask for one, so on a local markdown tracker the run ends with every ticket resolved and the work merged on the branch.

**Its review and fix loop ran for hours, or kept "fixing" tickets that hadn't been built yet.**

Both happen when `code-review` runs anywhere other than the one point the skill gives it. It compares the code against the whole spec, so it only makes sense once every ticket has landed. If it runs mid-run, every unbuilt ticket reads as a failure. At the end, the skill runs `code-review` once and fixes the findings itself, but it doesn't yet say when to stop after that fix. If you see a second broad review start, tell it to run focused checks for the fixed findings and stop. Expect that first review to find real problems.

**Two parallel workers collided on the same file, or picked different names for the same thing.**

Worktrees don't remove collisions; they postpone them to merge time. That is why the skill only runs tickets in parallel when they touch disjoint files, and briefs each worker with the exact names and conventions it needs. If two frontier tickets share a message catalogue, a config registry or a type, they are not disjoint: they run one after the other in the main session.

**Blocked tickets never start, even after their blocker has merged.**

This is a known problem on GitHub. The tracker's blocked-by count only drops when a blocker *closes*, and tickets typically close when the PR merges, which is the end of the run. Tell the agent to track which tickets have landed on the integration branch itself and compute the frontier from that.

**Does this replace Sandcastle or an AFK script?**

No. `implement-spec` puts an agent in charge of the build inside one harness session, which needs no infrastructure and lets you watch and steer. For work that is truly [AFK](https://www.aihero.dev/ai-coding-dictionary/afk), a deterministic loop ([Sandcastle](https://github.com/mattpocock/sandcastle), a shell script, a CI job) is faster, cheaper, and more reliable, because no agent makes the orchestration decisions.

**A worker's key test was skipped inside its worktree, and it reported green.**

A worktree holds only what git tracks. Tests that read gitignored fixtures, local databases, or credentials can silently skip there. Keep a ticket whose verification depends on untracked material in the main session.

## It's working if

- The agent reads a file once; later tickets recover that reading from folds instead of reading it again.
- Each finished ticket shows up as one fold naming its commit.
- Workers appear only for sizable tickets on disjoint files, and their briefs carry findings, not "go read the spec".
- Every ticket's trace shows `tdd` running, with a failing test before the code.
- The run ends on one branch with every ticket resolved, and a PR only if your tracker wanted one.

## Where it fits

`implement-spec` is the build step of the main chain, as the one-run alternative to running [implement](https://aihero.dev/skills-implement) once per ticket:

```txt
grill-with-docs → to-spec → to-tickets → implement-spec → code-review
```

Its neighbours are [to-tickets](https://aihero.dev/skills-to-tickets), which declares the blocking edges it reads as a task graph, and [code-review](https://aihero.dev/skills-code-review), which it runs over the integration branch before closing out. [ask-matt](https://aihero.dev/skills-ask-matt) is the router over the whole set when you are not sure which flow you are in.
