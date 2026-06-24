---
description: Pull the top unblocked slice off the GitHub kanban backlog, build it end-to-end, review it, and move it to done. Replaces /run-phase.
argument-hint: [optional: issue number to grab a specific slice; defaults to the next unblocked one]
---

# Grab Issue — Kanban Execution Loop

You execute ONE vertical-slice issue end-to-end, moving it across the kanban columns
(GitHub labels): `status:backlog` → `status:in-progress` → `status:review` → `status:done`.
This replaces `/run-phase`. One slice = one focused commit. Use the GitHub MCP tools
(`mcp__github__*`, via ToolSearch) against the repo in session scope.

## Step 0 — Prime context

Read `CLAUDE.md`, `plans/prd.md` (its Technical Foundation is the architecture context —
there is no separate master plan), and relevant `reference/` docs. Confirm the working tree
is clean (`git status`); if not, stop and tell the user to commit/stash first.

**Budget:** you are the single subagent for this slice, with a **120k-token ceiling**
(Rule 6). The issue's "Build budget" line is your target. If you approach the ceiling
before the acceptance criteria are met, do NOT silently overrun — stop, move the issue back
to `status:backlog`, comment what's done and what's left, and split the remainder into a
follow-on issue. A slice that won't fit is a slicing finding, not a reason to blow the budget.

## Step 1 — Pick the slice

- If `$ARGUMENTS` names an issue number, grab that one.
- Otherwise, list open issues labeled `slice` + `status:backlog` + `ready-for-agent`, and
  pick the **top unblocked** one: every issue in its "Blocked by" must be closed /
  `status:done`. Skip anything still `blocked`.
- If nothing is grabbable, say so and stop — report what's blocked and on what.

Read the chosen issue's full body and comments. Restate the slice and its acceptance
criteria back in one or two lines (Rule 10 checkpoint).

## Step 2 — Claim it

Move the card: remove `status:backlog`, add `status:in-progress`. Assign yourself if the
tracker supports it. Post a one-line comment: "Picked up — building this slice."

## Step 3 — Build the slice end-to-end

Implement the **complete vertical path** the issue describes (data → logic → UI → test),
and nothing beyond it (Rule 2, Rule 3 — surgical; do not gold-plate adjacent code).

- Follow `reference/conventions.md` and existing patterns (Rule 8, Rule 11). Read exports
  and immediate callers before adding code.
- **If the slice touches UI:** consult the remote design library before writing from
  scratch — read `reference/design-language.md` for tokens; fetch relevant
  `skills/` and `components/` from
  `https://raw.githubusercontent.com/ashesh2621/design-references/main/` and adapt to the
  project's tokens (credit original creators in comments).
- Verify as you go: imports resolve, types check.

## Step 4 — Self-review and fix

Run `git diff` on your own changes. Review for: logic errors, security, performance, and
`CLAUDE.md` adherence. Note severity. Fix all critical/high; fix medium/low if cheap, else
note them in the issue.

## Step 5 — Validate

Run the project's lint / typecheck / test / build (discover via `CLAUDE.md` or
`package.json` / `pyproject.toml`). On failure, fix and re-run (max 2 attempts). If still
failing, do NOT mask it (Rule 12) — move the issue back to `status:backlog`, comment the
failure, and stop.

## Step 6 — Slop scan (AI-cruft check, on the slice diff)

Re-read the diff and flag — fix all before proceeding (or surface one explicit exception
with a one-line justification):

- Vacuous comments restating code · defensive try/catch that swallows errors with no
  rethrow/log/handling · `any` / `as any` casts without a `// Reason:` · one-shot
  abstractions used in exactly one place · generated-marketing voice in docs
  ("seamlessly", "robust", "leverages") · dead code (commented blocks, unused imports,
  half-renamed identifiers) · mock/stub leftovers (`TODO`, `FIXME`, hardcoded `localhost`,
  stray `console.log`).

## Step 7 — CSO (lite security pass, on the slice diff)

Scope to what changed — don't audit the whole repo:

- Secrets in code (keys, tokens, connection strings — even in fixtures)
- Auth changes: trace where new code is called from; confirm the auth boundary still holds
- Input validation at every new boundary (Zod / Pydantic / explicit checks)
- SQL/command-injection surface from new string interpolation
- New dependencies: not abandoned (>18mo), not deprecated, not a typosquat
- Logging hygiene: no secret / token / PII logged

Critical & high MUST be fixed before commit. Medium/low: fix if cheap, else log to
`.agents/cso-findings/issue-<N>.md`. If the diff has no security surface, say so explicitly
— don't fake an audit.

## Step 8 — Acceptance-criteria check (definition of done)

Verify each acceptance-criteria checkbox in the issue actually holds, with a real check
(not "it compiles"). Per Rule 9 / Rule 12, do not declare done if any criterion fails.

## Step 9 — Commit + move to review

Produce ONE commit for the slice (`/commit` style; stage only files you changed, never
`-A`):

```
feat(<scope>): <slice title>

Closes #<N>

Acceptance criteria: PASS
Slop scan: PASS (or "PASS — k fixed")
CSO: PASS (or "PASS — no security surface" / "PASS — k fixed, m logged")
```

Move the card: remove `status:in-progress`, add `status:review`. Then run `/code-review`
on the diff (and `/codex` if the slice is risky or you want adversarial pushback). Address
findings — re-commit fixes as needed.

If the user works via PRs, open one for the slice instead of committing to the branch
directly, and let CI + review drive it (offer to subscribe to PR activity).

## Step 10 — Close out

Once review is clean and validation is green:
- Move the card: remove `status:review`, add `status:done`.
- Close the issue (the `Closes #<N>` in the commit will, once merged).
- **Unblock dependents:** find issues whose "Blocked by" lists this one; if this was their
  last blocker, remove `blocked` and add `ready-for-agent`. Comment on each that it's now
  grabbable.

## Step 11 — Hand off

End with:
> "Slice #<N> shipped → `status:done`. Unblocked: #x, #y.
> **Next:** run `/grab-issue` for the next slice, or `/office-hours` if the backlog is empty."

## Rules

- Per Rule 4, success criteria = the issue's acceptance criteria. Loop until verified.
- Per Rule 10, the issue + its labels ARE your checkpoint. If interrupted, re-read the
  in-progress issue before resuming.
- Per Rule 12, never move a slice to `status:done` with a failing criterion, a skipped
  test, or an unfixed critical/high finding. "Mostly works" is failure — move it back to
  `status:backlog` with a comment instead.
- Per Rule 3, one slice at a time — do not pull a second issue until this one is done or
  explicitly parked.
