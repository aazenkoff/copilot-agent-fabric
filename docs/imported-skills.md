# Imported Skills from `mattpocock/skills`

This repository ports selected skills from [`mattpocock/skills`](https://github.com/mattpocock/skills) pinned at commit `383b6a06d59c4ce0ffcb14112bfd91265a86cf91`.

The upstream content was adapted into this repo's flat `.github/skills/<slug>.skill.md` format and Copilot CLI wording. Executable scripts and personal/repo-specific workflows were not installed automatically.

## Import Decisions

| Upstream skill | Decision | Reason |
|---|---|---|
| `caveman` | Imported | General communication-mode skill with no external dependency. |
| `design-an-interface` | Imported | Useful architecture/API design workflow; adapted subagent wording to local conventions. |
| `diagnose` | Imported | Strong debugging loop that complements existing code-analysis/testing skills. |
| `domain-model` | Imported | Useful planning/domain-language workflow; adapted docs references to local project conventions. |
| `edit-article` | Imported | Fits documenter workflows and markdown-generation without conflicting with existing skills. |
| `git guardrails hook skill` | Skipped | Editor-specific hook installation and bundled shell behavior are tool-specific and conflict with this repo's Copilot CLI boundary. |
| `github-triage` | Imported | Fits GitHub issue management workflows; adapted to require maintainer approval before state mutation. |
| `grill-me` | Imported | General plan stress-testing workflow with no dependency. |
| `improve-codebase-architecture` | Imported | Complements code review/refactor planning by focusing on deep modules, seams, and locality. |
| `migrate-to-shoehorn` | Skipped | Narrow TypeScript dependency migration that would introduce a third-party package recommendation not broadly useful here. |
| `obsidian-vault` | Skipped | Contains a personal absolute vault path and Obsidian-specific conventions outside this repo's agent-config scope. |
| `qa` | Imported | Fits tester/orchestrator issue-filing workflows; adapted to avoid unconditional issue creation without repo authorization. |
| `request-refactor-plan` | Imported | Useful planning workflow for safe incremental refactors. |
| `scaffold-exercises` | Skipped | Course/exercise repository conventions and `ai-hero-cli` linting are repo-specific and not applicable here. |
| `setup-pre-commit` | Skipped | Installs Husky/lint-staged and modifies executable hook behavior; this repo should not add third-party tooling by default. |
| `tdd` | Imported | Distinct behavior-first TDD workflow that complements testing infrastructure. |
| `to-issues` | Imported | Useful for converting plans into parallelizable GitHub issue slices. |
| `to-prd` | Imported | Useful product-planning workflow for orchestrator/documentation tasks. |
| `triage-issue` | Imported | Useful investigation-to-issue workflow; adapted for durable issue bodies. |
| `ubiquitous-language` | Imported | Useful DDD glossary workflow for planning and documentation agents. |
| `write-a-skill` | Imported | Directly supports agent-creator skill work in this repository. |
| `zoom-out` | Imported | Lightweight context-building skill for unfamiliar code areas. |

## License Notice

The imported/adapted skill content is based on upstream MIT-licensed material:

```text
MIT License

Copyright (c) 2026 Matt Pocock

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
