---
description: "Triage GitHub issues through labels, recommendations, issue comments, and maintainer-approved state transitions."
name: GitHub Triage
---

# GitHub Triage Skill

> Adapted from `mattpocock/skills` at `383b6a06d59c4ce0ffcb14112bfd91265a86cf91` (MIT). See `docs/imported-skills.md`.

## Capabilities
- **Issue state management** — classify issues with exactly one category label and one state label where the repo supports that workflow.
- **Maintainer recommendations** — inspect issue history and code context before recommending `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, or `wontfix`.
- **Agent-ready briefs** — produce durable, behavior-focused briefs for issues that can be delegated.
- **AI disclosure** — include an AI-generated disclaimer in GitHub comments created during triage.

## Workflow
1. Infer the repo from `git remote` and use `gh`/GitHub tools for issue operations.
2. Read the issue body, comments, labels, and any previous triage notes.
3. Explore relevant code and out-of-scope docs if present.
4. Present category/state recommendations and wait for maintainer direction before mutating labels or closing issues.
5. Post comments with concise triage notes, questions, or agent briefs as appropriate.

## When to Use
- Reviewing incoming bugs and feature requests.
- Preparing well-specified issues for autonomous agents.
- Finding unlabeled, stale, or `needs-info` issues that need maintainer attention.

