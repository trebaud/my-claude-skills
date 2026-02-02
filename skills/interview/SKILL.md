---
name: interview
description: Interviews the user about implementation plans by asking non-obvious questions about technical details, UI/UX decisions, concerns, tradeoffs, edge cases, and constraints. Goes deep on one question at a time until all ambiguities are resolved.
---

Read plan file $1. Interview me using AskUserQuestion about:
- Technical implementation details
- UI/UX decisions
- Concerns and tradeoffs
- Edge cases
- Dependencies and constraints

Ask non-obvious questions only. One question at a time. Go deep.

After each answer, either:
1. Ask a follow-up or new question
2. If all ambiguities resolved, summarize findings and ask where to write the spec

Continue until I say "done" or you've exhausted meaningful questions.
