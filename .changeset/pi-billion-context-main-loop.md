---
"mattpocock-skills": minor
---

Fork only: the main-flow skills now target pi with billion-context. `implement-spec` builds the spec in one main loop and folds each finished ticket, using `worker` delegates only for tickets on disjoint files. `code-review` runs its axes as `reviewer` delegates. `grilling`, `to-spec`, `to-tickets`, `implement` and `handoff` recover folded decisions with `search_context` instead of re-reading or re-asking. `ask-matt` routes phase boundaries with `/new` and billion-context folds instead of `/clear` and `/compact`. See `.agents/adr/0003-pi-and-billion-context.md`.
