---
description: One-time setup - discover your Jira, Figma, Vercel and Supabase ids via MCP and write .claude/sdlc-loop.json
---
Create `.claude/sdlc-loop.json` for this repo (schema in the `sdlc-loop:sdlc-conventions` skill). Do the discovery yourself and only ask the user to choose when there's more than one option.

1. **Check MCPs.** The loop needs the Atlassian (Jira) and Vercel MCP servers; Figma and Supabase are optional. If one is missing, tell the user how to add it (e.g. `claude mcp add --transport http atlassian https://mcp.atlassian.com/v1/sse`, `claude mcp add --transport http vercel https://mcp.vercel.com`, Figma via the official Figma plugin, `claude mcp add --transport http supabase https://mcp.supabase.com/mcp`) and continue with what's available.
2. **Jira:** `getAccessibleAtlassianResources` -> site + cloudId (ask if several). List projects (via `discover` "list jira projects") and ask which project key the loop should use.
3. **App layout:** find the app folder (where package.json with a dev/build script lives), the package manager (lockfile), the stack (deps), migrations/functions folders, and the default branch (`git symbolic-ref refs/remotes/origin/HEAD`).
4. **Vercel:** `list_projects`; match by repo/app name and confirm. Record projectId, teamId, rootDirectory (the app folder relative to repo root, or "" at root) and whether the project has a Git integration (`get_project` -> link).
5. **Supabase (optional):** `list_projects`; match `supabase/config.toml` project_id or the URL in env example files; confirm.
6. **Figma (optional):** `whoami` -> list teams; ask which team new design files go in.
7. Show the resulting JSON, ask for confirmation (AskUserQuestion), write `.claude/sdlc-loop.json`, and remind the user to commit it so teammates share the same config. It holds ids, not secrets.
8. Finish with: "Ready - run `/sdlc-loop:run` to pick up the next sprint ticket."
