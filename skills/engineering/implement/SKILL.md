---
name: implement
description: "Implement a piece of work based on a spec or set of tickets."
disable-model-invocation: true
---

Implement one unit of work described by the user: a single ticket, or small work agreed in this session.

If the user passes a ticket reference, fetch it from the issue tracker and state its title before starting. If the reference is ambiguous, ask.

Work from what this session already holds. If the planning happened here, its decisions and your earlier reading of the code may now show only as summaries of earlier work: recover the detail from there instead of re-reading or re-asking. Read a file again only when it is missing from context or has changed since you read it (`git diff`, `git log`).

Build it yourself, in this session; don't hand any of the build to `acp_delegate`. (The review at the end runs its own reviewers.)

Load the `tdd` skill and use it where possible, at pre-agreed seams.

Run typechecking regularly, single test files regularly, and the full test suite once at the end.

Once done, load the `code-review` skill to review the work.

Commit your work to the current branch.
