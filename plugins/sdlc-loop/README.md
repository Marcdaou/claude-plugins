# sdlc-loop

The AI-native SDLC playbook as a Claude Code plugin. One specialized agent per stage, each driven by a skill, each committing a file the next stage reads:

```
Jira sprint ─► intent.md ─► Figma + spec.md ─► plan.md ─► branch + PR ─► test-report.md ─► Vercel preview ─► incident ─► new Jira issue
  planner    [you approve]     designer       builder [you approve]   builder       tester           deployer        maintainer
```

## What you get

| Command | Stage | Agent | What it does |
|---|---|---|---|
| `/sdlc-loop:setup` | – | – | One-time: discovers your Jira/Vercel/Supabase/Figma ids and writes `.claude/sdlc-loop.json` |
| `/sdlc-loop:plan` | 1 Plan | `planner` | Picks the top-priority To Do issue in the active sprint, drafts `intent.md`. **You approve**, issue → In Progress |
| `/sdlc-loop:design` | 2 Design | `designer` | Designs the screens in a new Figma file, writes `design.md` + `spec.md` |
| `/sdlc-loop:build` | 3 Build | `builder` | Writes `plan.md` (**you approve**), then implements on a feature branch and opens a PR |
| `/sdlc-loop:test` | 4 Test | `tester` | Typecheck, lint, unit tests (Vitest), build → `test-report.md` |
| `/sdlc-loop:deploy` | 5 Deploy | `deployer` | Vercel **preview** deploy + smoke check, issue → In Review |
| `/sdlc-loop:maintain` | 6 Maintain | `maintainer` | Reads Vercel/Supabase logs and advisors, writes incident records, files follow-up Jira issues |
| `/sdlc-loop:run` | all | you're the lead | Runs the whole loop, resuming from `status.json` |

Work for each ticket lives in `docs/work/<ISSUE-KEY>-<slug>/` – the files are the audit trail.

## The production gate
Agents stop at a Vercel preview. A `PreToolUse` hook blocks pushes to `main`/`master`, `vercel --prod`/promote, production deployments via the Vercel MCP, and Supabase migrations or write SQL. Promoting to production and applying migrations stay human decisions.

## Requirements
- Claude Code with the **Atlassian** and **Vercel** MCP servers connected; **Figma** (official Figma plugin) and **Supabase** MCP are optional.
- `gh` (GitHub CLI) logged in, `git`, `jq` (used by the hook), Node.js.
- A Jira project with sprints, a GitHub repo, a Vercel project. Tuned for JS/TS web apps (React/Vite/Next-style).

## Quick start
```
/plugin marketplace add Marcdaou/claude-plugins
/plugin install sdlc-loop@marcdaou-claude-plugins
```
Restart Claude Code, open your repo, then:
```
/sdlc-loop:setup
/sdlc-loop:run
```

## Config: `.claude/sdlc-loop.json`
```json
{
  "jira":     { "site": "acme.atlassian.net", "cloudId": "<uuid>", "projectKey": "PROJ" },
  "figma":    { "teamName": "Acme Design" },
  "vercel":   { "projectName": "acme-web", "projectId": "prj_...", "teamId": "team_...", "rootDirectory": "web", "gitIntegration": false },
  "supabase": { "projectName": "acme", "projectRef": "<ref>" },
  "app": { "dir": "web", "stack": "React + Vite + Tailwind + TypeScript", "packageManager": "npm",
           "migrationsDir": "supabase/migrations", "functionsDir": "supabase/functions" },
  "git": { "defaultBranch": "main" }
}
```
Ids only, no secrets – commit it so your team shares it.
