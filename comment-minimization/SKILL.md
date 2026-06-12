---
name: comment-minimization
description: Use after writing or editing code, and before declaring the work complete. Aggressively deletes comments that do not carry a non-obvious WHY. Applies to code, YAML, TOML, shell, and other comment-bearing files. Does not change behaviour.
---

# Comment Minimization

Default to zero comments. **When uncertain, delete.** The burden of proof is on keeping a comment, never on removing it. Deleting a useful comment costs little — it lives in git history and the PR. Keeping a useless one taxes every future reader.

**Violating the letter of this rule is violating its spirit.** Trimming a comment to one line is not a substitute for deleting it. "Mostly restates the code but adds a little" is a deletion.

## The decision is binary: delete or keep

For every comment, default to **delete**. It survives only if it passes the keep test. There is no third "borderline" or "trim it just in case" option — **borderline means delete.**

### The keep test

A comment is kept ONLY if BOTH are true:
1. You can state its non-obvious WHY in one sentence — a forced upstream constraint, a workaround for a specific bug, a subtle invariant, a contract the code cannot express.
2. A competent reader of this code would plausibly get it **wrong** without the comment.

If you cannot do (1) in one sentence, or the best you can say is "it's useful context" / "it labels the block" / "it's a helpful caveat" / "it's a TODO" — **delete it.** None of those is a WHY.

## Always delete

- Comments restating what the code does. Well-named identifiers already do that.
- Comments that *label* or *summarise* the block/resource/policy below them (`# Read the bearer secret`, `# The MCP server`, `# Optional IP allowlist`). The name and structure already say it.
- "Used by X", "added for the Y flow", "handles issue #123" — belongs in the PR description; it rots in code.
- TODOs and caveats: "starting point", "reconcile before prod", "for now", aspirational follow-ups. Track them in an issue, not a comment.
- Multi-line preambles, URLs (unless they point to a specific still-relevant bug/issue), apologetic or hedging phrasing.
- File-level and function docstrings that restate the name or signature. A docstring is a comment; it gets the same keep test.
- Section-banner decoration (`# ─────`, `# ====`, ASCII separators).
- Test docstrings restating the assertion.
- TOML / YAML section comments that just name the section.

## Trim survivors

Only after a comment has passed the keep test — never as a way to avoid deleting. Make it one short line. Drop URLs/references unless load-bearing, "Note:"/"Important:" preambles, rhetoric, hedging, multi-paragraph explanations. A multi-paragraph kept docstring almost always collapses to one line.

## Red flags — STOP, you are about to keep something you should delete

- "I'll trim it instead of deleting it." → Trim is only for comments that already passed the keep test. Delete.
- "It's borderline / I'm not sure." → Uncertainty means delete.
- "It's a useful caveat / nice context / labels the block." → Not a WHY. Delete.
- "It's a security note, so keep it." → Security is not exempt. If the code already says it (e.g. `hmac.compare_digest` *is* the constant-time compare), delete.
- "It's a TODO for before prod." → Issue tracker, not a comment. Delete.
- "I'll keep it but flag it to the user as maybe-removable." → If you'd flag it, it's already a delete. Make the call.

## Rationalization table

| Excuse | Reality |
|--------|---------|
| "Trim to one line is safer than deleting." | Trim is only for comments that already passed the keep test. A restatement trimmed is still a restatement. Delete. |
| "It's borderline, I'll keep it." | Borderline is a delete. The bar is high; the burden is on keeping. |
| "It adds a little context." | "A little" is below the bar. Delete. |
| "It documents a deliberate choice / caveat." | If the reader can't get it **wrong** without it, delete. Deliberate ≠ non-obvious. |
| "Removing it might lose information." | It's in git and the PR. Whoever needs it finds it there. |

## Process

1. Read the file.
2. For each comment, default to delete. Apply the keep test; only a pass survives. Trim the survivors.
3. Do not change code behaviour, identifiers, structure, or unrelated formatting.
4. If removing a comment leaves consecutive blank lines, collapse to one.
5. Re-run tests if any code-adjacent files were edited.

## Output

Per-file tally: deleted N, trimmed M, kept K. For each keep, one line stating the non-obvious WHY — the thing a reader would get wrong without it. If you cannot state it, you should have deleted the comment. Do not present a "borderline / you may want to remove" list — make the call yourself.

## Do not over-apply

- Markdown documentation is not code; do not strip prose from `README.md`, design docs, or user-facing instructions.
- CFN `Description:`, YAML/OpenAPI `description:` fields are not code comments — keep, trim only if visibly verbose.
- A docstring expressing a genuinely non-obvious contract is worth keeping — but it must pass the keep test like any comment; a docstring that restates the name or signature is deleted.
- Do not remove license headers, copyright notices, or `SPDX-License-Identifier:` tags.
