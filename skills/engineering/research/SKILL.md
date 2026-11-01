---
name: research
description: Investigate a question against high-trust primary sources and capture the findings as a Markdown file in the repo. Use when the user wants a topic researched, docs or API facts gathered, or reading legwork handed to a delegate.
---

Hand the reading to an `acp_delegate` delegate with agent `researcher` (the default `async: true`), so you keep working while it reads. If `acp_delegate` is missing, billion-context isn't running: do the research yourself in this session.

Brief it with the question, its scope, and what you already know, so it doesn't re-read that. Its job:

1. Investigate the question against **primary sources** (official docs, source code, specs, first-party APIs), not a secondary write-up of them. Follow every claim back to the source that owns it.
2. Reply with the findings, citing each claim's source (a URL, or `file:line`).

The `researcher` is read-only, so the file is yours to write. When its reply arrives:

1. Write the findings to a single Markdown file, keeping every citation.
2. Save it where the repo already keeps such notes; match the existing convention, and if there is none, put it somewhere sensible and say where.

When the answer is something you must understand yourself to do the work in front of you, not only record, do the reading yourself in this session instead: a delegate's reply is a summary, and you would end up reading the sources again.
