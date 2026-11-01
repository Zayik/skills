---
"mattpocock-skills": minor
---

Fork only: the skills no longer tell the agent when or how to fold. billion-context owns that through its nudge and tool descriptions, and the skills keep one neutral line about recovering what the session already holds before re-reading or re-asking. `implement-spec` stops folding each finished ticket, and `ask-matt` drops Fold as a phase-boundary option. The remaining Claude Code subagent wording is adapted for pi: `research` hands the reading to a `researcher` delegate and writes the file itself, `wayfinder` runs research tickets as parallel `researcher` delegates, `improve-codebase-architecture` walks the codebase in the main loop, `codebase-design`'s design-it-twice uses parallel `researcher` delegates, and `triage`, `retro`, `wayfinder` and `improve-codebase-architecture` load skills the pi way. See `.agents/adr/0003-pi-and-billion-context.md`.
