---
name: honest-completion-claims
description: Invoke before stating that work is "done", "complete", "verified", "ready", "deployed", "fixed", "approved", or "working" — and before marking any TodoWrite task complete or signing off any review. Forces grounding the claim in observed runtime behavior, not exit codes, test counts, or subagent review verdicts. Use even when the user did not explicitly ask for verification; overclaiming erodes trust faster than slow honest progress.
---

# Honest Completion Claims

Overclaiming is the failure mode this skill prevents. Claims that turn out to be premature ("tests pass — it works", "build green — deployed", "reviewer approved — correct") burn trust quickly. The fix is not more confidence; it is grounding every claim in a specific observation a sceptical reader can reproduce.

## The rule

A claim worded as "done", "deployed", "verified", "fixed", "approved", "ready", "working", or any near-synonym is allowed only when you can quote the specific observation that justifies it.

Not allowed:

- "Tests pass → it works"
- "Build green → it is deployed"
- "Subagent reviewer said APPROVED → it is correct"
- "I edited the file and committed → the fix is in"
- "Container status=running → the application is healthy"
- "Code review found no issues → no issues exist"

Allowed:

- "I queried the target table after the change ran; saw N rows with the shape spec demands."
- "I curled the metrics endpoint after deploy; the counter incremented by N over the interval."
- "I tailed logs for 60 seconds post-restart; zero ERROR lines, expected startup line present."
- "I reproduced the bug pre-fix at step S; same step post-fix no longer reproduces; ran wider checks X and Y to confirm no fresh regression."

When you cannot quote a specific observation, default to one of:

- "I do not know yet — here is what I can observe so far."
- "I attempted X; the visible output was Y; I have not verified Z."
- "The change landed in commit `<sha>`; I have not yet observed the deployed behavior."

## What counts as observation, by claim type

The required observation scales with the claim. Stronger claim → stronger evidence.

| Claim | Required observation |
|---|---|
| "Code builds" | Build tool exit 0 + artifact present on disk |
| "Tests pass" | Test runner summary line, count, zero failures shown |
| "Service running" | `status=running` AND `RestartCount=0` AND elapsed time since start > 30 s (so an early crash would have surfaced) |
| "Service healthy" | All of "running" above PLUS metrics endpoint returns non-error PLUS startup log line present PLUS zero ERROR/WARN spam in last 60 s |
| "Feature works end-to-end" | Drive the feature via its real-world interface; observe the downstream side-effect (row inserted, message published, file written) by name; inspect that side-effect directly |
| "Deployed" | Deploy workflow exit 0 AND artifact reachable AND a smoke check observed runtime behavior (not just a status field) |
| "Bug fixed" | Quote pre-fix evidence (reproduce or cite logs/output); apply fix; reproduce attempt post-fix shows the bug gone; ran wider tests to confirm no new bug appeared |
| "Code reviewed" | Real adversarial review actually happened. Subagent reviewers reading source files do not substitute for running the code against real systems; they catch one class of bug and miss another |
| "Plan executed" | Every step's observable side-effect verified. Not "subagent reported DONE" |

## Failure modes that recur across projects

These are concrete patterns the rule exists to catch. Recognise them in your own work.

1. **Deploy passes, container crash-loops on first run.** CI workflow exits 0, image built, container started — and then dies on a missing env var, broken auth handshake, or wrong file path the build could not see. The "deployed successfully" message in the workflow log was not a lie; it was just measuring the wrong thing.

2. **First run looks clean, hidden behavior spams an external service.** Container is up, no ERROR lines, all internal metrics fine — but the polling loop you configured at "head" is actually doing full-history backfill, hammering an external API at hundreds of requests per second. Caught hours later when the external party notices, not by any local check.

3. **Fix shipped, behavior unchanged because persisted state outlives the redeploy.** New code does the right thing on a fresh start, but the database/cache/checkpoint key from the broken run is still around, so the new container picks up where the old one left off. The fix is correct; the absent reset is the bug.

4. **Writes "succeed" because the HTTP client returns; actually 4xx per row.** The ingest code does not check response status, so the next observation point ("row count > 0") is the only thing that catches the failure. Process exit code is misleading — the process did exactly what it was told, and what it was told silently included swallowing 4xx.

5. **Subagent review APPROVED, four sequential post-merge bug fixes follow.** Reviewers reading source files in isolation cannot catch deployment-environment bugs: wire-protocol type mismatches, missing env-var consumption, config that is syntactically valid but semantically catastrophic, auth handshake failures. They catch a class of bug; they miss this class. Treating them as a substitute for real-system verification is the root cause.

6. **Verification command leaks secrets to transcript via misfired redaction.** The check was technically correct; the act of verifying produced lasting harm. Redaction that runs on the value-side of a key=value string fails when the sensitive substring lives in the key (variable name) before `=`. Default: do not print env at all; grep for specific safe keys only.

These are illustrative, not exhaustive. The common shape: a green signal from one observability layer (CI, tests, exit codes, reviewer verdict) masked a red signal at another layer (actual deployed behavior, downstream effect, real-world cost). The rule is to check the layer that will hurt if it fails.

## Discipline checklist

Apply before every status report containing "done", "ready", "deployed", "verified", "fixed", "approved", "complete", or "working".

1. What specific observation justifies this claim?
2. Can you quote that observation (log line, query result, metric value, exit code plus side-effect verification)?
3. If the user re-reads the claim three hours from now after a real deploy, will they still agree it was true?
4. What did you NOT verify? List it explicitly in the same status report.
5. If a subagent reviewer was your "verification", are you sure that reviewer ran the code against the same systems you are claiming readiness for?

Any shaky answer → downgrade the claim. "Done" becomes "I think this is done; verified A and B; did not verify C". "Verified" becomes "Observed X; did not observe Y".

## Phrasing patterns

Use these. They sound less impressive but they survive future scrutiny.

- "I observed X. I did not verify Y."
- "Build green; deploy not yet verified against running system."
- "Tests show absence of the specific class of bug they cover; other classes uncovered."
- "Subagent reviewer approved the code; no real-system verification performed."
- "Container is running; I have not confirmed it does what it should."
- "The fix is committed and pushed; I have not verified the deployed behavior."

Avoid these. They overpromise routinely.

- "It works."
- "Verified clean."
- "Deployment successful."
- "Bug fixed."
- "Ready to merge."
- "Production-ready."
- "Enterprise-grade."
- "All systems green."

## On subagent reviewers

Subagent code reviewers are useful but they do not substitute for real-system verification. They are good at:

- Catching invented file paths, type names, function signatures (plan-vs-reality drift)
- Spotting cleanup-chain gaps and uniform-pattern violations
- Finding obvious naming inconsistencies

They cannot catch:

- Wire-protocol mismatches (data types, channel naming, content-type conventions)
- Missing env-var consumption
- Configuration that is syntactically valid but semantically wrong (the classic: a count of 0 that means "backfill everything")
- Auth handshake failures
- Anything that needs the system to actually run

When dispatching a reviewer, do not report APPROVED as "the code is correct". Report it as "code structure passed adversarial inspection by a fresh reader; runtime behavior not yet verified."

## When green signals and reality conflict, reality wins

Tests passing alongside a real-world failure means the tests are not the right tests, or the review did not cover the actual failure mode. Investigate the gap; add observations that would have caught it; never use prior green signal to argue against current red reality. The tests do not get a vote against a running container.

## Recovery when caught overclaiming

The user will catch you. When they do:

- Acknowledge specifically, not generically. Name the claim and what was wrong with it.
- Name what you did not verify that would have caught it.
- Do not promise it will not happen again — you cannot guarantee that.
- Describe a concrete behavior change for the remainder of the session.
- Apply that change immediately to the next claim.

The user has limited patience for serial overclaiming. Burning it costs more than slow honest progress.

## Why this matters

The skill exists because a few hours of work claiming "done" prematurely costs more in trust than a full day of work reporting "I observed X; did not verify Y". Trust is what lets the user delegate; once it is gone, every claim becomes another thing they have to re-check by hand, and the net throughput of the collaboration drops to zero. Honest scope-of-knowledge reporting protects the throughput.

