---
name: comment-minimization
description: Use after writing or editing code, and before declaring the work complete. Aggressively removes comments that do not carry a non-obvious WHY, and trims survivors to one short line. Applies to code, YAML, TOML, shell, and other comment-bearing files. Does not change behaviour.
---

# Comment Minimization

Default to zero comments. The bar for a surviving comment is high.

## The rule

Keep a comment **only** when:
- The WHY is non-obvious from the code itself: a forced upstream constraint, a workaround for a specific bug, a subtle invariant that would surprise a reader, a contract that the type signature or naming cannot express.

Always delete:
- Comments restating what the code does. Well-named identifiers already do that.
- "Used by X", "added for the Y flow", "handles the case from issue #123" — that belongs in the PR description; it rots in code.
- Multi-line preambles, URLs (unless they point to a specific bug or issue that still matters), hedges like "for now", apologetic phrasing.
- File-level docstrings that just summarise what the module obviously does. Keep only if the WHY of the module's existence is non-obvious.
- Section-banner decoration (`# ─────`, `# ============`, ASCII art separators). They are decoration, not information.
- Test docstrings restating the assertion.
- TOML / YAML section comments that just name the section ("# secrets section" above `[secrets]`).

## Trim survivors

If a comment earns its keep, make it one short line. Drop:
- URLs and references unless load-bearing.
- "Note:" and "Important:" preambles.
- Rhetoric and hedging.
- Multi-paragraph explanations.

If a kept docstring is multi-paragraph, almost certainly cut it down to one paragraph or one line.

## Process

1. Read the file.
2. For each comment, decide: **delete / keep / trim**.
3. Apply edits. Do not change code behaviour, identifiers, structure, or formatting unrelated to the comment removal.
4. If removing a comment leaves consecutive blank lines, collapse to a single blank line.
5. Re-run tests if any code-adjacent files were edited — the change should not affect behaviour, but verify.

## Output

When invoked on a diff or a set of files, report:
- A per-file tally: deleted N, trimmed M, kept K.
- For each non-trivial keep: one line stating the WHY justification.
- Any file where you found nothing to change.

## Do not over-apply

- Markdown documentation is not code; do not strip prose from `README.md`, design docs, or user-facing instructions.
- CFN `Description:` fields, YAML `description:` properties, OpenAPI `description:` fields, function docstrings explaining a non-obvious contract — these are **not** code comments. Different rules apply, and they can be useful. Trim only if they are visibly verbose.
- Do not remove license headers, copyright notices, or `SPDX-License-Identifier:` tags.
