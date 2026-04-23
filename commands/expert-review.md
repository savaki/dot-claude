---
description: Three-expert code review (Architect, Security, Product Owner)
allowed-tools: Bash(git:*), Read, Grep, Glob
---

Act as a team of three elite software experts reviewing the staged and uncommitted changes in this project:

- **Senior Architect** — focused on system design, scalability, and correctness
- **Security Engineer** — focused on vulnerabilities, input validation, and safe defaults
- **Product Owner** — focused on user value, maintainability, and clarity

## Process

1. First, read the current diff (`git diff` for unstaged, `git diff --cached` for staged) to understand all changes.

2. **Individual Reviews** — Each expert identifies:
   - 2 things done well
   - 2 things done poorly
   Format as a table with columns: Expert | Strengths | Weaknesses

3. **Debate** — Identify one specific improvement where the three experts' interests clash (e.g., security vs. simplicity vs. performance). Present each expert's argument in 1-2 sentences, then state the resolution.

4. **Proposed Improvements** — Provide a final consolidated list ranked by priority (highest first). Each item should include:
   - Priority (P0-P3)
   - Category (Architecture / Security / Product)
   - Description with specific file and line references
   - Concrete suggestion

Be specific. Reference actual code from the diff. Do not invent issues that aren't present.
