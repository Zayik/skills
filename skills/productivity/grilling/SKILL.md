---
name: grilling
description: Grill the user relentlessly about a plan, decision, or idea. Use when the user wants to stress-test their thinking, or uses any 'grill' trigger phrases.
---

Interview the user relentlessly until you reach a shared understanding. Map this as a **design tree**: every decision branches into the decisions that hang off it.

Work the tree in **rounds**. The **frontier** is every decision whose prerequisites are already settled: the questions you can ask _now_ without guessing at answers you haven't heard yet. Ask the whole frontier in one round: number each question and give your recommended answer. Then wait for the user's answers before the next round.

Format a round like so:

```
❓ **Q1** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>

---

❓ **Q2** - **<question title>**: <question body, might be multiple paragraphs, including multiple choices>

➡️ <your recommended answer>
```

Word each question so "yes" accepts your recommended answer.

Each round the user answers reshapes the tree: settled decisions push the frontier outward and unblock questions that depended on them. Recompute the frontier and ask the next round. A question whose answer depends on another question still open in this round belongs to a _later_ round, not this one.

Finding _facts_ is your job, never the user's. When a frontier question needs a fact from the environment (filesystem, tools, etc.), look it up yourself; don't ask the user for anything you could look up. That reading is what `/to-spec` and `/to-tickets` build on later, and billion-context folds it once it is consumed, so doing it here is not wasted. Hand a lookup to an `acp_delegate` `researcher` only when it is large and its raw material is disposable, such as scanning a big external doc or dependency for one answer. Don't block on that: a running lookup is an unsettled prerequisite, so only the questions downstream of it wait for its reply; ask the rest of the frontier now. The _decisions_ are the user's: put each to them and wait.

A decision settled earlier in a long session may have been folded by billion-context. Recover it with `search_context`, which searches the fold summaries, instead of asking the user again; for its exact wording, `decompress` just that range. If those tools are missing, billion-context isn't running: tell the user.

The session is done when the frontier is empty: every branch of the design tree visited, nothing left silently assumed. Do not act on it until the user confirms you have reached a shared understanding.
