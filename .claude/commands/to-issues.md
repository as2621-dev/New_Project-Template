---
description: Break the PRD into independently-grabbable vertical-slice issues and publish them to GitHub Issues as a kanban backlog. Run AFTER /cto.
argument-hint: [optional: path to PRD; defaults to plans/prd.md]
---

# To Issues — PRD → Vertical-Slice Kanban

Adapted from Matt Pocock's `to-issues`. Break the PRD into **tracer-bullet vertical
slices** and publish each as a **GitHub Issue**, forming a kanban backlog. This replaces
the old `/plan-phases`: no 4-sub-phase phases — thin end-to-end slices instead.

## Step 0 — Load inputs

1. Read the PRD at `$ARGUMENTS` (default `plans/prd.md`). If missing, stop and tell the
   user to run `/cto` (the PRD now ships from `/cto`).
2. Read `plans/master-plan.md`, `CLAUDE.md`, and relevant `reference/` docs for the domain
   glossary and conventions (Rule 11 — issue titles use the project's vocabulary).
3. If the repo has code, skim it for the current state and **prefactoring opportunities**:
   "make the change easy, then make the easy change."

## Step 1 — Ensure the kanban labels exist

The board is GitHub Issue **labels**. Before publishing, ensure these exist on the repo
(create any that are missing):

- `status:backlog` · `status:in-progress` · `status:review` · `status:done` — the columns
- `ready-for-agent` — triage: this slice is fully specced and an agent can grab it
- `blocked` — has an open blocker
- `slice` — marks a tracer-bullet issue from this command

Use the GitHub MCP tools (search ToolSearch for `mcp__github__` label/issue tools). The
target repo is the one in session scope.

## Step 2 — Draft vertical slices (tracer bullets)

Break the PRD into slices. **Each slice cuts through EVERY layer end-to-end** — not a
horizontal slice of one layer.

<vertical-slice-rules>
- Each slice delivers a narrow but COMPLETE path through every layer (data → logic → UI → tests)
- A finished slice is demoable / verifiable on its own
- Prefactoring slices come first
- Aim for the fewest seams — one is ideal
</vertical-slice-rules>

Bad (horizontal): "Build the database schema." / "Build all the API endpoints."
Good (vertical): "A user can sign up with email and see an empty dashboard" — touches
schema + endpoint + UI + one test, demoable end-to-end.

## Step 3 — Quiz the user before publishing

Present the breakdown as a numbered list. For each slice show:

- **Title** — short, in domain vocabulary
- **Blocked by** — which slices must finish first (if any)
- **User stories covered** — which PRD stories this delivers

Then ask:
- Does the granularity feel right (too coarse / too fine)?
- Are the dependencies correct?
- Merge or split any?

Iterate until the user approves. Do NOT publish before approval (Rule 1).

## Step 4 — Publish to GitHub Issues, in dependency order

Publish **blockers first** so you can reference real issue numbers in "Blocked by."
For each approved slice, open a GitHub Issue:

- **Labels:** `slice`, `status:backlog`, and `ready-for-agent` (unless it has an open
  blocker → add `blocked` and omit `ready-for-agent` until the blocker closes).
- **Body** — use this template:

```markdown
## What to build
[End-to-end behavior of this slice. Describe the demoable outcome, not layer-by-layer
implementation. No file paths or code snippets — they go stale. Exception: a tiny
decision-encoding snippet from a prototype (schema, type shape, state machine) is fine.]

## Acceptance criteria
- [ ] Criterion 1 (observable, testable — fails if business logic is wrong, per Rule 9)
- [ ] Criterion 2
- [ ] Criterion 3

## User stories covered
- PRD story #N: As a …, I want …, so that …

## Blocked by
- #<issue-number> (or "None — can start immediately")
```

After publishing, post one comment on the PRD issue (or print) listing the created issue
numbers and the dependency order.

## Step 5 — Hand off

End with:
> "Published [N] vertical-slice issues to the backlog (`status:backlog` + `ready-for-agent`).
> Dependency order: #a → #b → #c.
> **Next:** run `/grab-issue` to pull the top unblocked slice and build it end-to-end."

## Rules

- Per Rule 2, don't pad slices to hit a number. Ship the fewest that cover the PRD.
- Per Rule 3, each slice is surgically scoped — a thin path, not "refactor module X."
- Per Rule 9, acceptance criteria must be checkable and must fail when intent is violated,
  not just when code won't compile.
- Do NOT close or modify any parent/PRD issue here. Only create slice issues.
- Per Rule 12, if the PRD can't be sliced thinly (every path needs everything), say so —
  that's a finding about the PRD, not a reason to ship one giant issue.
