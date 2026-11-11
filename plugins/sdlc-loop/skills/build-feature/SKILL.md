---
name: build-feature
description: Stage 3b (Build) - implement an approved plan.md on a feature branch in the app (the app in `<app.dir>/`, database in `<app.migrationsDir>`), commit, push and open a GitHub PR. Use only after plan.md is approved, or when the user says "build it", "implement the plan", or /sdlc-loop:build continues past plan approval.
---

# Implement the approved plan

1. Check `status.json` has `approved.plan: true`. If not, stop and return `BLOCKED: plan not approved` - building early defeats the gate.
2. `git checkout <git.defaultBranch> && git pull && git checkout -b <KEY>-<slug>` (or reuse the branch if it exists). Run `git status` first: unrelated uncommitted files travel with you onto the branch, so **stage with explicit paths only** (`<app.dir>/...`, database folders, the work folder) - never `git add -A` / `git add .`. Leave build artifacts like `*.tsbuildinfo` unstaged.
3. Follow plan.md step by step. After each step run its verify command (at minimum `<pm> --prefix <app.dir> run build`) and fix before moving on - small feedback loops catch errors while they're cheap. Verify steps that need a browser, a preview or a database you don't have (e.g. no local database stack) are not failures: list them under `## Deferred checks` in plan.md for the tester/deployer.
4. Match existing patterns: the styling approach and components already in `<app.dir>/src`, the existing data/client layer, and the existing layout of `<app.functionsDir>`.
5. New migrations go in `<app.migrationsDir>` with the next number. Do **not** apply them to the remote project - production is human-only.
6. Write the tests listed in plan.md and run them. If Vitest is missing, add it (`<pm> --prefix <app.dir> i -D vitest`, script `"test": "vitest run"`) - the builder owns test setup; the tester owns lint.
7. If reality forces a deviation from plan.md, update plan.md with a `## Deviations` section rather than silently diverging.
8. Commit in logical chunks: `<KEY>: <what>` and end each message with the repo's Co-Authored-By line if one is configured. Push the branch and open a PR with `gh pr create --base <git.defaultBranch>`:
   - Title `<KEY>: <summary>`; body links intent/spec/plan, Figma URL, Jira URL, and a checklist of spec requirement IDs.
9. Update status.json (`stage: "test"`, `branch`, `pr_url`) after the PR exists and commit it as the last commit. Return PR URL + files changed.
