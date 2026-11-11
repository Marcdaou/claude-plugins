---
name: sdlc-conventions
description: Shared rules for the sdlc-loop plugin - work-folder layout, status.json, file chain (intent -> spec -> plan -> code -> test-report -> deploy-log), human gates, and the per-repo config file `.claude/sdlc-loop.json` (Jira, Figma, Vercel, Supabase ids and app layout). Load this whenever running any sdlc-loop stage or agent, or when resuming an SDLC work item.
---

# SDLC loop conventions

Every stage reads the previous stage's file and writes its own. That chain is the point: any agent (or human) can pick up a work item cold, and every line of code traces back to a Jira issue. So never skip a file, and never write a later file before the earlier one is approved.

## Work folder
`docs/work/<ISSUE-KEY>-<kebab-slug>/` (e.g. `docs/work/PROJ-11-stripe-webhook-confirm/`)

| File | Written by | Gate |
|---|---|---|
| `intent.md` | planner | **human approves** |
| `design.md` (Figma links + screenshots list) and `spec.md` | designer | none - flows on |
| `plan.md` | builder | **human approves** |
| code on branch `<ISSUE-KEY>-<slug>` + PR | builder | none |
| `test-report.md` | tester | must be PASS to deploy |
| `deploy-log.md` | deployer | preview only |
| `docs/incidents/<date>-<slug>.md` | maintainer | becomes a new Jira issue / intent |

## status.json
Keep `status.json` in the work folder current - it is how `/sdlc-loop:run` resumes:
```json
{ "issue": "PROJ-11", "slug": "stripe-webhook-confirm", "stage": "plan",
  "approved": { "intent": false, "plan": false },
  "figma_url": null, "branch": null, "pr_url": null, "preview_url": null, "updated": "<ISO time>" }
```
`stage` is the stage that runs **next**: plan, design, build, test, deploy, maintain, done (build covers both writing plan.md and implementing; `approved.plan` tells which). Create it in the Plan stage exactly like the example (unknown fields `null`, `updated` = current UTC ISO time) and update it as the last step of every stage.

## Who asks the human
Subagents cannot talk to the user. Agents write their file, then return a short summary ending in `NEEDS_APPROVAL: <file>` when a gate applies. The lead (the slash command) shows the file and asks the user. Only the lead flips `approved.*` to true.

## Project config: `.claude/sdlc-loop.json`
Every project-specific value comes from this file in the repo root. **Read it first.** If it's missing, stop and return `BLOCKED: run /sdlc-loop:setup first` - guessing ids leads to acting on the wrong Jira project or Vercel/Supabase project.

```json
{
  "jira":     { "site": "acme.atlassian.net", "cloudId": "<uuid>", "projectKey": "PROJ" },
  "figma":    { "teamName": "Acme Design", "planKey": "<from whoami, optional>" },
  "vercel":   { "projectName": "acme-web", "projectId": "prj_...", "teamId": "team_...", "rootDirectory": "web", "gitIntegration": false },
  "supabase": { "projectName": "acme", "projectRef": "<ref>" },
  "app": {
    "dir": "web",
    "stack": "React + Vite + Tailwind + TypeScript",
    "packageManager": "npm",
    "migrationsDir": "supabase/migrations",
    "functionsDir": "supabase/functions"
  },
  "git": { "defaultBranch": "main" }
}
```
In the skills, `<config:jira.projectKey>`, `<app.dir>` etc. mean "the value from this file". `figma` and `supabase` are optional: without `figma` the designer writes the spec only; without `supabase` skip database steps. Never use a Supabase or Vercel project other than the one configured, even if the account has several.

## Production gate
Agents stop at a Vercel **preview**. Never promote to production, push to the default branch, or apply migrations to the production Supabase project - a human does that. The plugin hook blocks the obvious ways; do not look for others.
