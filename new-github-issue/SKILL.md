---
name: new-github-issue
description: Use when the user wants to investigate a problem, bug, or feature idea and turn it into a GitHub issue — e.g. "create a ticket", "file an issue", "write this up for the backlog". Not for implementing the work itself.
---

# New GitHub Issue

Pair with the developer to investigate a problem and file a GitHub issue with enough technical detail that another developer — or an agent — can pick it up cold and implement it without repeating the investigation.

**The only output is a filed GitHub issue. Do not implement the fix, create branches, spec files, or open PRs.**

## 1. Investigate together — loop until aligned

The most important step. Do **not** rush to a draft. Stay here until you and the developer share the same mental model of the problem.

Use /superpowers:brainstorming

- Ask one focused question at a time; prefer concrete options over open-ended prompts.
- If the developer doesn't know an answer, research the codebase yourself and present findings, then confirm your reading is correct before moving on.
- After each round, play back your current understanding of the problem, scope, and constraints in your own words, and ask the developer to correct anything that's off.
- Surface assumptions, unknowns, and edge cases explicitly — disagreement now is cheaper than a wrong issue.
- Only proceed once the developer confirms your restated understanding. When in doubt, do another round.

## 2. Ground the issue in the code

The next person should not have to redo this research. Collect into the issue:

- Exact locations: `file/path.js:line`, function names, lambda names.
- Relevant database tables/columns and any queries already identified.
- 1-2 similar existing endpoints/handlers that show the convention to follow.
- Consult `CLAUDE.md` for architecture, auth, `dbUtils`, and error-handling patterns — reference it, don't restate it. Note only project-specific utilities it doesn't cover.

## 3. Draft the issue

Use the template below. Include only the sections that apply — delete the rest. Every issue must have at minimum a **Summary / Problem** and **Acceptance criteria**.

```markdown
## Summary / Problem


## Motivation
<!-- Why now? Link the incident, PR, or Slack thread that surfaced this. -->

## Repro
<!-- Bug reports only. CLI commands or steps, with sample output. -->

## Affected accounts / data
<!-- Bug reports only. Scope of impact with sanitised UIDs, keys, or counts. -->

## Scope
<!-- Feature/refactor only. -->
### In scope
-
### Out of scope
-

## Current wiring
<!-- Feature/refactor only. ASCII diagram of the existing data/event flow. -->

## Proposed wiring
<!-- Feature/refactor only. Target state diagram. -->

## Proposed fix
<!-- Bug reports only. Which file/function/field changes and how. -->

## Migration plan
<!-- Feature/refactor only. Step-by-step rollout with feature flags and rollback triggers. -->

## Why this matters
<!-- Bug reports only. Operational or legal consequences of leaving it unfixed. -->

## Technical pointers
<!-- file:line references, tables, similar endpoints, patterns to follow. -->

## Acceptance criteria
- [ ]
- [ ]

## Risks / open questions
- **[risk]** — [owner]

## Related
- PR #
- Slack:
```

## 4. Confirm and file

Show the drafted issue to the developer and ask for approval or edits. Once confirmed, file it:

```bash
gh issue create \
  --repo <org>/<repo> \
  --title "<title>" \
  --body "<body>"
```

Stop after filing. Implementation is a separate task for whoever picks the issue up.
