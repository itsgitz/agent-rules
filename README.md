# agent-rules

Shared global agent instructions in [`AGENTS.md`](AGENTS.md): the main agent plans and reviews, while subagents perform repo work in dedicated Git worktrees.

## What’s inside AGENTS.md

- Roles and delegation: main agent plans and reviews; subagents do repo work, with harness and prerequisite checks before delegation.
- Models and effort: session model and reasoning choices need explicit approval; no silent fallback or substitution.
- Concurrency: main agent chooses sequential or parallel for independent work within user constraints and resources; heavy tasks run sequentially on low-spec machines with one heavy job by default.
- Safety: dedicated worktrees and Conventional Commits; sensitive actions need approval.
- Verification: evidence-based review with explicit failure, retry, and done criteria.
- Cleanup: limited to the currently running session and its subagents, by inventory table and approval.

`AGENTS.md` is authoritative; this summary omits exact rules.

## Install

```sh
git clone https://github.com/itsgitz/agent-rules.git
cd agent-rules
```

## Customize

Edit the Config section in `AGENTS.md` before copying, so your available
models, provider, reasoning effort, and resource limits apply to the
installed files. If the pinned subagent model is unavailable, delegation
stops until you configure a supported alternative.

## Pi prerequisite

Pi has no built-in subagents. Delegation requires the
[pi-subagents extension](https://github.com/tintinweb/pi-subagents).

```sh
pi install npm:@tintinweb/pi-subagents
pi list
```

If needed, enable it via `pi config`, then reload/restart Pi. This
installed-and-loaded prerequisite is checked before work; a missing package
blocks delegation.

## Copy to your tools

Warning: these files affect all projects. `cp -i` prompts before replacing
existing instructions. If you already have custom rules, merge instead of
replacing.

```sh
mkdir -p ~/.claude
cp -i AGENTS.md ~/.claude/CLAUDE.md

mkdir -p ~/.pi/agent
cp -i AGENTS.md ~/.pi/agent/AGENTS.md

mkdir -p ~/.codex
cp -i AGENTS.md ~/.codex/AGENTS.md
```

Note: the Pi path is singular (`agent`, not `agents`).

## Overrides

`CODEX_HOME` and `PI_CODING_AGENT_DIR` override the Codex and Pi home
directories. For Pi/Codex, `AGENTS.override.md` can take precedence over
`AGENTS.md`.

## Sources

- Claude Code: https://code.claude.com/docs/en/memory
- Codex: https://learn.chatgpt.com/docs/agent-configuration/agents-md
- Pi: https://github.com/earendil-works/pi

Start a new session after installing so the new instructions take effect.
