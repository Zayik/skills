# Phase boundaries

A **phase** is a chunk of work inside a session: the grilling, the implementation, the QA. The definition is fuzzy on purpose: a phase ends when you think *"ok, we're done with that"*.

The **phase boundary** is the gap between two phases, and it is the only place this decision belongs. Mid-phase there is no decision to make: continue, and let billion-context fold what is consumed.

This assumes pi with billion-context loaded. billion-context folds consumed context into summaries and keeps the originals recoverable with `search_context` and `decompress`, and it cancels pi's automatic compaction. So a session stays useful far past the ~150k [smart zone](https://www.aihero.dev/ai-coding-dictionary/smart-zone) a raw context window would allow, and the window's size is rarely the reason to leave it.

## The five options

| Option       | What it does                                                    |
| ------------ | --------------------------------------------------------------- |
| **Continue** | Stay in the session. No context switch at all.                    |
| **`/new`**   | Start a new pi session from nothing. The old one stays resumable with `/resume`. |
| **`/handoff`** | Write a portable markdown file and seed a session anywhere with it. |
| **Delegate** | Send a tightly scoped task to an `acp_delegate` delegate and get a compact reply back. |
| **Fold**     | Compress the phase you just finished with billion-context's `compress`, then continue. Recoverable. |

pi's own `/compact` is not on the list. Folding already does its job, range by range, and keeps each range recoverable; `/compact` replaces the whole live history with one summary instead, which is the "confidently wrong about a decision the summary flattened" failure.

## The tree

Work top to bottom at the boundary. The first **yes** wins.

**1. Can you continue in this session?** Almost always yes. The next phase usually needs this one as a **primary source**: grilling → spec → tickets → implementation is the standard case, because each step wants the reasoning verbatim, not a summary of it. billion-context keeps the window lean as you go, so continuing costs nothing and loses nothing. Rule it out before anything else.

**2. Is the context irrelevant to what comes next?** Is everything in this session (the exploration, the decisions, the dead ends) disposable, or do you want the next phase isolated from it? If so, **`/new`**. It is the cheapest clean slate: it takes no time and the old session stays resumable.

The cost of getting this wrong is one-way. Leave a *relevant* session and you lose the **why** behind what you built, and no amount of reading the diff back gets it returned.

**3. Do you need to hand off?** `/handoff` is narrow. You need it only when you are:

- swapping to a **new harness**,
- moving to a **new directory** or repo,
- sending the work to a **colleague**,
- or forking a side task you found **mid-phase** without derailing what you're doing.

That list is the whole clause. What `/handoff` buys is **portability**: a file that travels. If nothing is travelling, you don't need it.

**4. Can the task be done in a fresh context and reported back?** Delegate only when one of these holds: an **independent review** (the reviewer must not share your assumptions), a **large lookup** whose raw material you will not need afterwards, or **parallel edits** on disjoint files. Brief the delegate with what you already know, so it does not re-read it. Anything you will need to understand yourself, do yourself.

**5. Otherwise, fold and continue.** Relevant context, same harness, same directory, you stay in the loop: fold the finished phase with `compress`, writing the summary for what comes next ("we're going to QA this area"), and carry on. billion-context will nudge you to fold as the session grows anyway; folding at the boundary just picks the cleaner cut.

## Primary and secondary sources

`/new` and `/handoff` turn a **primary source** (the session as it happened) into a **secondary source** (a summary of it) or into nothing. A fold summarises in place but keeps the primary source one `search_context` away. The trade is always the same shape:

| Source                            | Information | Noise | Room to move |
| --------------------------------- | ----------- | ----- | ------------ |
| Primary (Continue, Fold)          | Full (recoverable) | Less after folding | Lots |
| Secondary (`/handoff`)            | Lossy       | Less  | Lots         |
| None (`/new`)                     | None        | None  | All          |

This is why question 1 comes first. With billion-context, staying is rarely the expensive option.

## These are judgement calls

The questions are not objective: each has taste in it, and the same boundary can go two ways on two days. The value is in asking them **in order**, at the boundary rather than in the middle of the work.
