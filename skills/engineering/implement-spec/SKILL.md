---
name: implement-spec
description: "Implement the result of /to-spec and /to-tickets in code."
disable-model-invocation: true
---

You have been provided a spec. This spec should have tickets associated with it, describing how to implement the spec.

The issue tracker should have been provided to you. If not, tell the user to run `/setup-matt-pocock-skills`.

The goal is the entire spec implemented on a single **integration branch**, with every ticket resolved the way the issue tracker closes work.

The tickets are not a list of steps. They are a **task graph** with blocking relationships between them. This means there is always a **frontier** of tickets which are ready to be grabbed.

You build the spec yourself, in this session. Your context is the asset: when the spec was planned in this session, you already hold the reasoning behind every ticket, and billion-context keeps it recoverable as it folds what is consumed. Don't hand reading or tickets to sub-agents that would have to rebuild that context. If `search_context` and `decompress` are missing, billion-context isn't running: tell the user.

## Steps

1. Understand the task graph from the spec and tickets. If they were written in this session, recover them from context (`search_context`, then `decompress` what you need) and read only what isn't there. Otherwise read them from the tracker.

2. Create the integration branch. If the issue tracker closes work through PRs, or the user asks for one, open a draft PR after the first ticket lands (a branch with no commits ahead of main can't open one), marked as closing the spec and tickets.

3. Work the frontier one ticket at a time, in this session. For each ticket:
   - Recover what you already know: its decisions from the grilling and the spec, and any earlier reading of the code it touches. `search_context` searches the fold summaries, so start there. When you need exact wording, `decompress` just that range (`startId`/`endId`), or a whole block with `toFile` and read only the lines you need. A restore lands in this ticket's part of the session, so the ticket's fold covers it; don't fold it separately.
   - Read only what is missing or stale. A file that changed since you read it (`git diff`, `git log`) is read again, never edited from a folded copy.
   - Load the `tdd` skill and build the ticket with it.
   - Commit to the integration branch.
   - Fold the finished ticket with `compress`: one summary naming the commit, what changed, decisions made while building, and gotchas. Don't fold in the middle of a ticket unless billion-context nudges you to.

4. Run tickets in parallel only when the frontier holds two or more sizable tickets that are independent and touch disjoint files. Small or overlapping tickets stay sequential in this session. For a parallel batch:
   - Build one of the tickets yourself, and hand each of the others to `acp_delegate` with agent `worker`.
   - Give each worker its own git worktree on its own branch, cut from the integration branch tip, and pass the worktree as `cwd`.
   - Brief each worker with findings, not pointers: the ticket, its decisions, the `file:line` locations and conventions it needs. A worker should read only the files it edits. Ask it to build with the `tdd` skill and to report its branch, commit, test status, and anything surprising.
   - Don't touch a worker's ticket or files while it runs.
   - When a worker reports, merge its branch into the integration branch yourself, resolve any conflicts, and fold the ticket as in step 3.

5. Once all tickets are complete, load the `code-review` skill and run it on the integration branch. Fix the issues it raises yourself.

6. If a draft PR exists, mark it ready for review. Otherwise, resolve each ticket the way the issue tracker closes work, and report the integration branch.

7. Remove any worker worktrees.
