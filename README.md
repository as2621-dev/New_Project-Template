# Project Template

A slim Claude Code template. 11 commands, 14 rules. No fluff.

## Use this template for a new project

This repo is configured as a **GitHub Template Repository**. Don't fork — use the template button.

1. On GitHub: open the repo page → click **"Use this template" → Create a new repository** → name it (e.g. `acme-app`) and create.
2. In Cursor / VS Code: **New Window → Clone Repository →** paste the new repo's HTTPS URL.
3. In the new project root:
   ```bash
   cp .env.example .env   # fill in real secrets — .env is gitignored
   ```
4. Open Claude Code in that directory and run:
   ```
   /grill-me "<your rough idea>"
   /cmo
   /cto
   /to-issues
   /grab-issue
   ```

You now have a fresh repo with its own git history, all rules in `CLAUDE.md`, all 11 commands in `.claude/commands/`, and empty `plans/`, `reference/`, `documents/`, `.agents/` ready to fill.

> **Heads up:** `.claude/settings.local.json`, `.env`, and `.cursor/rules/openmemory.mdc` are all gitignored — they're machine-local. The shared, version-controlled config is `.claude/settings.json` (if present), `.env.example`, and everything in `.claude/commands/`.

## The 14 rules

See [`CLAUDE.md`](./CLAUDE.md). These apply to every task.

## The 11 commands

The first five are the core pipeline (top-to-bottom); the rest are support commands.

| Command | Use it to… |
|---|---|
| `/grill-me` | Pressure-test a raw idea with a relentless, **non-technical** interview before `/cmo`. Critical — pushes back, won't flatter. |
| `/cmo` | Refine the grilled idea into a product brief. Fills holes, sharpens scope. |
| `/cto` | Turn the product brief into a master plan + **PRD** + reference docs. |
| `/to-issues` | Slice the PRD into vertical-slice (tracer-bullet) issues on the GitHub kanban backlog. |
| `/grab-issue` | Pull the top unblocked slice and build it end-to-end: code → review → fix → validate → slop scan + CSO → single commit → done. |
| `/office-hours` | Run a weekly diagnostic. What's stuck, what's risky, what's the next call. |
| `/rca` | Root-cause analysis for a bug. Diagnoses + proposes a fix. Doesn't apply it. |
| `/debug` | Autonomous browser bug hunt. Reproduces with `browser-use`, diagnoses with Chrome DevTools, fixes, re-verifies in-browser, loops until gone. Applies the fix; hands off to `/commit`. |
| `/commit` | Conventional commit. Stages explicit files. Never amends. Never skips hooks. |
| `/codex` | Adversarial second opinion. The 200-IQ pedant. Use when stuck or want pushback that doesn't social-smooth. |
| `/handoff` | Compact the conversation into a handoff doc (saved to temp, not committed) so a fresh agent can continue. |

## Typical flow for a new initiative

```
/grill-me "rough idea"                 → documents/grilling-notes.md (non-technical stress-test)
/cmo                                    → documents/product-brief.md
/cto                                    → plans/master-plan.md + plans/prd.md + reference/*.md
/to-issues                              → vertical-slice issues on GitHub (status:backlog)
/grab-issue                             → build top slice → slop scan → CSO → 1 commit → status:done
/grab-issue                             → next unblocked slice → ...
/office-hours                           → weekly check-in
/rca "thing X broke"                    → .agents/rca/*.md (when bugs happen)
/debug "checkout button does nothing"   → .agents/debug/*.md (browser bugs, auto-fixed)
/codex challenge <diff>                 → when you want adversarial pressure
/handoff                                → hand off mid-stream to a fresh session
```

## The kanban board

Slices live on **GitHub Issues**, not in the repo. The board is just labels:

`status:backlog` → `status:in-progress` → `status:review` → `status:done`

Plus `ready-for-agent` (fully specced, grabbable), `blocked` (has an open blocker), and `slice` (a tracer-bullet issue). `/grab-issue` moves a card across columns as it works it, and unblocks dependents when a slice closes.

## What `/grab-issue` does before each commit

For the slice's diff, before the single commit:

1. **Acceptance check** — each acceptance-criteria checkbox on the issue actually holds (a real check, not "it compiles")
2. **Slop scan** — flags vacuous comments, `any` casts, defensive try/catch, dead code, marketing voice in docs, hardcoded `localhost`, leftover TODOs
3. **CSO lite** — secrets in diff, auth boundary changes, input validation gaps, injection surface, new dependency health, log hygiene

All must pass. Findings get fixed before commit (or for medium/low CSO, logged to `.agents/cso-findings/` for follow-up).

## Directory layout

```
CLAUDE.md                          # The 14 rules
.claude/commands/                  # The 11 slash commands
documents/                         # Grilling notes + product briefs (grill-me + CMO output)
plans/                             # Master plan + PRD (CTO output)
reference/                         # Stack notes, conventions, API contracts, design language (CTO output)
  └── browser-debug-playbook.md    # Tool routing + CLI cheat-sheets for /debug
design-references/                 # Pointer only — full library is remote
  └── RESOURCES.md                 # Points to github.com/ashesh2621/design-references
                                   # (86 skills + 511 design systems + 2,827 components
                                   #  + 20,660 templates, ~1 GB, fetch on demand)
.agents/
  ├── office-hours/                # Weekly diagnostic notes
  ├── rca/                         # Root-cause analyses
  ├── debug/                       # Browser debug reports from /debug
  ├── codex/                       # Codex transcripts
  └── cso-findings/                # Deferred medium/low security findings
```

Slice work itself lives on GitHub Issues. Handoff docs go to the OS temp / scratchpad, never committed.

## Notes

- `/grab-issue` and `/debug` are the only commands that touch feature code (`/grab-issue` builds slices; `/debug` applies a verified bug fix). Everything else writes docs, plans, or reports.
- One commit per slice. The issue + its labels are the checkpoint, so an interrupted slice resumes cleanly.
- Each slice is a **thin vertical path** through every layer (data → logic → UI → tests) — demoable on its own. Keep them scoped tightly enough that an agent can execute one given only the issue and `CLAUDE.md`.
- `/codex` is **user-triggered only**. `/grab-issue` does NOT auto-invoke Codex on findings — humans decide when to escalate.
