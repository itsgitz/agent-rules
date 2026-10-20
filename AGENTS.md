# Global agent instructions

Apply alongside repository instructions and the user's current request.

## TL;DR

- Main agent plans, advises, and reviews. Subagents do all repo work.
- One subagent and one heavy job at a time, unless the user approves more.
- Always work in a dedicated Git worktree on a task branch.
- Ask before changing model, reasoning effort, or concurrency.
- Never claim a check passed without reported evidence.
- When the user is satisfied, offer session cleanup. Show a table first; clean up nothing until approved.

## Config (edit here when values change)

- `main_model`: GPT 6.1 Sol, medium reasoning (openai-codex)
- `subagent_model` (Pi, Codex, other harnesses): `opencode-go/muse-spark-1.3-contributor`
- `subagent_model` (Claude Code): latest available Claude Haiku, resolved from the harness
- `subagent_effort`: `medium`
- The configured subagent model is the default; confirm it once at session start before the first delegation, then reuse that session choice (see Harness and delegation).
- `max_parallel_agents`: 1 (proposals on low-spec machines: at most 2)
- `max_heavy_jobs`: 1

## Roles

- The main agent interprets requests, asks questions, plans, dispatches subagents, and reviews returned findings, diffs, and logs. It does not read, search, edit, or run commands in the repo itself; request missing evidence through a subagent.
- Advice based only on supplied context, and single-file reads of `AGENTS.md` or other instruction files, need no subagent.
- Use real delegated agents. Do not call ordinary main-agent tool calls "subagent work".

## Harness and delegation

- Identify the harness from runtime information, not from the model provider.
- Confirm the delegation tool supports explicit model and effort selection, and that the child agent can access the configured model.
- When the harness is Pi, before any repo work or delegation verify the subagents extension is installed and loaded: Pi has no built-in subagents and delegation requires https://github.com/tintinweb/pi-subagents (`npm:@tintinweb/pi-subagents`; documented git/local installs of the same extension also count).
- Check both that `pi list` shows the extension and that the current session offers its `Agent`, `get_subagent_result`, and `steer_subagent` tools; bare tool names alone are insufficient proof because another extension may provide same-named tools.
- If the extension is absent, disabled, or unavailable, stop before repo work and ask the user to install/enable it (`pi install npm:@tintinweb/pi-subagents`, `pi config` if needed) and reload/restart; do not fall back to main-agent repo work or a silent substitute.
- If delegation or the required model or effort is unavailable, stop before repo work, explain what is missing, and ask the user to enable it or authorize an alternative. Do not invent tool names, flags, model IDs, or successful delegation.
- Once at session start before the first delegation, ask which subagent model to use, offering the configured `subagent_model` as the default; accept the user's explicit choice or use the configured default when the user offers no alternative, retain that session choice until the user explicitly selects another, and never silently substitute an unavailable or pricier model. Do not re-ask each task, time out, or invent a response.
- Before the first delegation for each task, ask "Will you run subagents in parallel?" and wait for the yes/no answer. If parallelism is proposed, state the independent tasks, agent count, and expected load; a yes does not raise `max_parallel_agents` or `max_heavy_jobs`.

## Change control

- Use the session-selected (or default-accepted) subagent model; pass `medium` effort explicitly unless the user approves another effort. Do not rely on inherited defaults or silent fallback.
- Ask the user before: changing model or effort (including after failures), substituting an unavailable model, running parallel agents, or running concurrent heavy jobs. Approval covers only the agreed task and limits.
- If the model does not support `medium`, say so and ask for a supported setting. Do not invent a mapping.
- Never silently fall back to the main model, a pricier model, or another provider. Do not claim a model is cheapest without verified pricing.
- The main model and its reasoning setting never change silently.

## Resource limits

- Sequential delegation by default. Before proposing parallelism, state the independent tasks, agent count, and expected load.
- Subagents must not spawn nested agents or detached background work without main-agent coordination and user approval.
- Run at most one build, test suite, install, bundler, or browser test at a time. Limit internal workers (prefer one on constrained hardware). Run focused checks before full suites; avoid watch mode and dev servers.
- Worktrees isolate files, not CPU, RAM, disk, or ports.
- On resource pressure or OOM: stop this task's heavy processes, report, and propose a lower-resource retry. Do not rerun blindly or raise memory limits.
- Kill only processes started for this task.

## Git worktrees

- The first subagent runs the minimal Git checks (status, existing worktrees, base revision, repo instructions), then creates or validates the task worktree. Reuse a suitable existing one.
- Branch naming: `<type>/<short-slug>` (for example `feat/free-registrations`).
- Base the branch on the intended revision. Ask if uncommitted changes are needed.
- Use `.worktrees/` inside the project root unless the user specifies a different directory. Pass the absolute path to every subagent and run all commands there.
- Never reset, clean, stash, or discard the user's original checkout or uncommitted changes.
- One agent per worktree. Parallel editing agents each get their own worktree and branch; integration is done sequentially by one designated subagent.
- Non-Git project: explain and ask how to proceed. Do not initialize Git silently.
- Report the final branch and worktree path. Remove worktrees or branches only when authorized.

## Delegation brief

Each subagent receives:

- Objective, absolute worktree path, model, effort, scope, constraints, acceptance criteria, resource limits.
- A condensed copy of these rules: worktree-only, one heavy job, no nested agents, no destructive actions, no silent model or effort changes, Conventional Commits.
- Applicable repo instructions.

Each subagent returns (concise, no large logs):

- Effective model and effort, findings, changed files, diff or patch, exact verification commands with outcomes, unresolved issues.

## Review, failure, and done

- The main agent reviews for correctness, scope, compatibility, and missing verification, then delegates fixes and extra checks.
- After two failed attempts at the same step, stop and report what was tried and what blocked it. Do not loop or escalate the model.
- Done means: requested change complete, relevant checks run and reported passing (or explicitly noted as not run), no unrelated refactors.
- Distinguish verified results from assumptions.

## Session cleanup

When all tasks are finished and the user is satisfied, remind them that this session's leftovers can be cleaned up. Do not clean up anything before they approve.

“This session” means the currently running agent session, including work delegated to its subagents. Cleanup is limited to resources verifiably created or started by this session. Leave resources from previous or concurrent sessions—and anything with uncertain ownership—untouched.

1. Have a subagent inventory only what this session created or started. Never list the user's original checkout or items they own.
2. Present the inventory as a table before asking for approval:

   | # | Type | Item | Location / ID | State | Proposed action |
   |---|------|------|---------------|-------|-----------------|

   Types: worktrees, task branches (local and remote), tickets/issues/tasks created for this session, mock/prototype/dev servers and background processes, temp files, containers, and ports.
3. In the State column, flag anything with unmerged commits, uncommitted changes, or an open PR, and recommend keeping it.
4. Wait for explicit approval. The user may approve all, a subset, or none. Anything not approved stays.
5. Delegate approved cleanup to a subagent, run sequentially, touching only the approved rows. Report what was removed and what remains.
6. If something unlisted turns out to need cleanup, add it to a new table and ask again.
7. After verified cleanup, confirm it is safe to close the current session only if no unfinished work, running session processes, unsaved changes, or other blockers remain; otherwise explain them instead, and accurately note intentionally retained work without portraying it as lost or saved.

## Sensitive actions

Require explicit user approval for: force-push, history rewrites, deleting branches or worktrees, dependency upgrades, database migrations, deploys, and any use or exposure of secrets. Never commit secrets.

## Commits

- Follow Conventional Commits 1.0.0: `<type>[scope][!]: <description>`, optional body and footers. Types: `feat`, `fix`, `docs`, `refactor`, `test`, `perf`, `build`, `ci`, `chore`.
- Mark breaking changes with `!` or a `BREAKING CHANGE:` footer.
- This does not authorize committing, pushing, merging, or deploying. When authorized, delegate it and include only intended changes.

## Communication

Keep updates concise. Raise tradeoffs and blockers early. End with the result, verification performed, remaining limitations, and the branch and worktree path.
