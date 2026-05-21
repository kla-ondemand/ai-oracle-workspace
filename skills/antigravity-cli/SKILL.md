---
name: antigravity-cli
description: Drive Google's Antigravity CLI (`agy`) — the Go-rewritten successor to Gemini CLI; flag-first invocation, plugin model with import-from-claude/import-from-gemini, sandboxing, AGENTS.md/GEMINI.md, and the 2026-06-18 consumer sunset of Gemini CLI.
when_to_use: User mentions "antigravity", "antigravity cli", "agy", "gemini cli successor", "google agent cli", or asks to install/migrate-to/configure Antigravity. Also triggers when invoking `agy` in shell, working under `~/Library/Application Support/Antigravity/`, importing plugins from Claude Code or Gemini CLI, or planning a Gemini CLI → Antigravity migration before the 2026-06-18 consumer deadline.
user-invocable: true
---

# Antigravity CLI Skill

Google's terminal surface for the **Antigravity** agent platform — the Go-rewritten **successor to Gemini CLI** for consumer tiers. Ships bundled with the Antigravity desktop app; shares state with it. Co-optimised with Gemini models.

This skill is **verified against `agy --help` output as of 2026-05-21**, not extrapolated from peer CLIs. The surface mirrors Claude Code more closely than Gemini CLI did — `-p`, `-c`, `--continue`, `--dangerously-skip-permissions`, `--sandbox` are recognisable to anyone coming from `claude`.

## When To Use

- The user asks to run, install, configure, or migrate **to** Antigravity CLI (`agy`).
- The user is on Gemini CLI today and the **2026-06-18 consumer sunset** is approaching — Pro / Ultra / free tier users must migrate.
- The user wants to invoke `agy` as a sub-tool inside a larger workflow (e.g. a reviewer agent that runs `agy -p` to author a patch, or a multi-LLM evaluation).
- The user is comparing Antigravity CLI to Claude Code / Codex CLI / Kimi Coder and needs concrete differences.
- A project's `AGENTS.md` needs adjusting because it will be read by `agy` alongside other tools.

Do **not** invoke this skill for:
- The Antigravity **desktop app** itself — that's the primary surface, not this CLI (see `knowledge/google-antigravity.md`).
- Generic Gemini API usage outside the CLI (use a generic Gemini API skill).
- AI Studio / Firebase Studio web flows.
- Standard / Enterprise license holders who plan to **stay on Gemini CLI** — that remains supported indefinitely (no forced migration).

## Invocation (verified surface)

```bash
# Interactive session in current directory:
agy

# One-shot non-interactive print (the equivalent of `codex exec` / `claude -p`):
agy -p "explain what the auth module does"
agy --print "explain what the auth module does"
agy --prompt "explain what the auth module does"   # alias for --print

# Interactive session that starts with a prompt and continues:
agy -i "let's refactor the auth module"
agy --prompt-interactive "let's refactor the auth module"

# Continue the most recent conversation:
agy -c
agy --continue

# Resume a specific previous conversation by ID:
agy --conversation <id>

# Pull additional directories into the workspace (repeatable):
agy --add-dir ../shared-lib --add-dir ../proto-defs

# Pipe a file as context:
agy -p "$(cat tasks/refactor.md)"

# Long-running one-shot — bump the wait timeout (default 5m):
agy -p "audit the entire codebase for OWASP top-10 issues" --print-timeout 30m
```

Subcommands (separate from top-level flags):

```bash
agy install            # configure PATH + shell profile aliases
agy plugin <action>    # see "Plugins" below
agy update             # update the CLI
agy changelog          # show release notes
```

## Permissions & Sandbox

Antigravity's safety model is **two flags**, not a tri-mode policy:

| Flag | Effect | When to use |
|------|--------|-------------|
| (default) | Each tool action prompts for approval. | Default for unfamiliar repos / production work. |
| `--sandbox` | Run with terminal/network restrictions enabled. | Untrusted prompts, autonomous spikes, or any time you wouldn't trust the agent's shell output. |
| `--dangerously-skip-permissions` | Auto-approve every tool call. **No prompts.** | Throwaway worktrees only. Same semantics as Claude Code's flag of the same name. |

> [!warning] `--dangerously-skip-permissions` + no worktree = lost work
> Always pair `--dangerously-skip-permissions` with a throwaway git worktree (`worktree` skill). The sandbox prevents network/filesystem damage outside the workspace, but it cannot undo destructive git operations inside it.

There is **no** `suggest` / `auto-edit` / `full-auto` tri-mode (that's Codex CLI's model, not Antigravity's).

## Plugins (consolidated extension model)

Gemini CLI's four extension primitives — **Agent Skills, Hooks, Subagents, Extensions** — are consolidated under the **Antigravity plugin** umbrella, managed via `agy plugin`:

```bash
agy plugin list                          # list imported/installed plugins
agy plugin install <target>              # install (supports plugin@marketplace syntax)
agy plugin install <name>@<marketplace>  # marketplace install
agy plugin uninstall <name>
agy plugin enable <name>
agy plugin disable <name>
agy plugin validate [path]               # validate a plugin manifest
agy plugin link <marketplace> <target>   # generate marketplace link
agy plugin import gemini                 # import existing Gemini CLI extensions
agy plugin import claude                 # import existing Claude Code skills
```

> [!note] Built-in migration paths
> `agy plugin import gemini` and `agy plugin import claude` are the **headline migration mechanic** — Google built import paths from both Gemini CLI **and Claude Code** into the tool. This is more generous than the launch announcement implied; the migration story is mechanical, not manual.

Migration is still **not 1:1 lossless** (Google's announcement: *"there won't be 1:1 feature parity right out of the gate."*), but `import` is the right first step before any hand-port.

## Config & State

On **macOS**, Antigravity stores all state — desktop app *and* CLI — under:

```
~/Library/Application Support/Antigravity/
├── bin/agy-node              # Node runtime wrapper for Node-based plugins
├── app_storage.json          # desktop-app preferences (notification, last project)
├── Cookies, Local Storage… # Chromium/Electron desktop-app state
└── …
```

The CLI binary itself lives at `~/.local/bin/agy` (Mach-O native binary, Go-compiled). Run `agy install` once after upgrade to refresh PATH and shell aliases:

```bash
agy install                    # default — configures PATH + shell aliases
agy install --dir <path>       # target a non-default directory
agy install --skip-aliases     # don't touch shell profile aliases
agy install --skip-path        # don't touch shell profile PATH
```

This workspace's root has a legacy `.antigravitycli/` (gitignored at `.gitignore:87`) containing a symlink into `~/.gemini/config/projects/…json`. That's **Gemini CLI v1 state**, not Antigravity v2 state — audit and delete it after `agy plugin import gemini` succeeds.

There is **no documented `--model` flag** in the current help output. Model selection is presumably managed via config file or in-session command; do not promise users a `--model` flag works without checking.

## AGENTS.md / GEMINI.md (shared convention)

Antigravity CLI reads project-root instructions on every run:

- **`AGENTS.md`** — the vendor-neutral standard adopted by Google, OpenAI, Anthropic, Factory, Sourcegraph, Cursor. This is what every agent in this workspace uses as its source of truth (`agents/<name>/AGENTS.md`).
- **`GEMINI.md`** — legacy Gemini CLI convention; still honoured by `agy` for migration compatibility. In this workspace it's a symlink to `AGENTS.md`.

When editing `AGENTS.md` to support `agy`:
- Keep it backend-neutral. Use `{{placeholder}}` for model IDs, tool names, paths.
- Antigravity respects the same headings/sections Claude does; no special markers needed.
- After editing, run the agent's `./scripts/check-agent-neutrality.sh` to verify no Gemini-specific lock-in slipped in.

## Dynamic Subagents

Antigravity's headline orchestration primitive — subagents spawn dynamically during a task and execute **in parallel** within one orchestration environment. From the CLI, you don't pre-declare them; the agent decides when to fan out based on the prompt:

```bash
agy -p "review the diff against main: look for security issues, performance regressions, and missing tests, then summarise"
```

`agy` may decompose that into parallel subagent calls. This is the architectural bet that drives Gemini 3.5 Flash co-optimisation — fan-out across many cheap subagent calls rather than a few expensive flagship reasoning calls.

**Cost shape implication:** plan for subagent fan-out as the default, not flagship-call counts. If you're budgeting `agy` usage, model the per-task call count higher than you would for Claude Code or Codex CLI on the same job. The **AI Ultra tier ($100/mo, 5× quota)** is the realistic floor for heavy multi-agent workflows.

## Managed Agents (Gemini API) — adjacency

`agy` is the **terminal surface**; **Managed Agents** is the **API primitive**. They share the harness but you invoke them differently:

| You want… | Use |
|-----------|-----|
| Interactive terminal session | `agy` |
| One-shot CLI run from a script | `agy -p "..."` |
| Sandboxed agent in your own backend, called via HTTPS | Managed Agents (`POST /v1/agents:run` in the Gemini API) |
| GUI orchestration with parallel subagent panes | Antigravity desktop app |

If a user is doing **backend automation that needs isolation**, point them at Managed Agents instead of shelling out to `agy`. If they want **a developer's terminal workflow**, `agy` is correct.

## Gemini CLI Sunset (2026-06-18)

> [!warning] Consumer-tier deadline
> On **2026-06-18**, Gemini CLI and IDE extensions stop serving requests for **Google AI Pro, Ultra, and free tier** users. Antigravity CLI is the replacement; install before the deadline.

| Tier | What Happens |
|------|--------------|
| Free / Pro / Ultra (consumer) | Gemini CLI deprecated 2026-06-18. Must migrate to `agy`. |
| Standard / Enterprise license | **Gemini CLI continues indefinitely** with model updates. No forced migration. |

## Common Workflows

### Migrating from Gemini CLI (the easy path)

```bash
# 1. Pull existing Gemini CLI extensions into Antigravity:
agy plugin import gemini

# 2. Verify what came over and what's still disabled:
agy plugin list

# 3. Validate any hand-ported plugin manifests:
agy plugin validate ~/.antigravity/plugins/<name>

# 4. Audit and remove the legacy v1 state directory:
ls .antigravitycli/    # workspace-root legacy
ls ~/.gemini/          # user-level legacy
# …then delete once you've confirmed the import worked.
```

### Migrating from Claude Code

```bash
agy plugin import claude
agy plugin list
```

Skills + custom agents from `~/.claude/skills/` and `.claude/skills/<name>/SKILL.md` are picked up by the import. Not all features map 1:1 — review `agy plugin list` and rework anything missing.

### One-shot patch with review

```bash
git worktree add /tmp/refactor-auth -b refactor/auth
cd /tmp/refactor-auth
agy -p "extract the token-refresh logic from auth.ts into a separate module; keep the public API unchanged"
git diff
```

### Long-running audit with a bumped timeout

```bash
agy -p "audit the auth module for OWASP top-10 issues; write findings to AUDIT.md" --print-timeout 30m
```

### Multi-LLM comparison

```bash
agy    -p "$(cat task.md)" > /tmp/agy-output.md
codex  exec "$(cat task.md)" > /tmp/codex-output.md
claude -p "$(cat task.md)" > /tmp/claude-output.md
diff3 /tmp/agy-output.md /tmp/codex-output.md /tmp/claude-output.md
```

### Spawning subagents from a single prompt

```bash
agy -p "review the diff against main: spawn one sub-agent to look for security issues, another for performance regressions, and report back as a single summary"
```

Fan-out is implicit — you describe the work, `agy` decides whether to use one agent or several based on the prompt's structure.

### Adding extra directories to the workspace

```bash
agy --add-dir ../shared-protos --add-dir ../infra-config -p "trace how `UserCreated` flows from proto definition through service code to infra config"
```

`--add-dir` is repeatable; useful when you have a polyrepo layout and want one `agy` session to see across repos.

## Gotchas

- **Provider lock-in.** `agy` is co-optimised for Gemini. Don't promise users it can swap providers like [[../codex-cli/SKILL]] / Claude Code. If they need provider portability, point them at [[../../projects/openclaude]].
- **`--dangerously-skip-permissions` + no worktree = lost work.** Always pair it with a throwaway git worktree. `--sandbox` prevents external damage but cannot undo destructive git operations inside the workspace.
- **No `--model` flag in current help.** Model selection appears to be config-driven, not flag-driven, at launch. Don't claim a `--model` flag works without re-checking `agy --help`.
- **Subagent fan-out inflates token cost.** Antigravity's default is parallel subagents; this multiplies token consumption. The AI Ultra tier ($100/mo, 5× quota) is the realistic floor for heavy multi-agent workflows — flag this if the user is on the Pro tier and complaining about hitting limits.
- **Plugin import ≠ feature parity.** `agy plugin import gemini|claude` gets the manifests across, but Google's announcement explicitly disclaims 1:1 parity. Expect rework on non-trivial extensions.
- **AGENTS.md is read every run.** Long `AGENTS.md` files inflate every prompt. Keep them tight; push lengthy context into linked files `agy` can read on demand.
- **CLI state lives under the desktop app dir on macOS.** `~/Library/Application Support/Antigravity/` is shared between the desktop app and the CLI. Deleting one resets the other. There is no separate `~/.antigravity/` on this machine.
- **The launch is fresh (2026-05-19).** Re-run `agy --help` rather than trusting this skill blindly if you suspect a flag has changed.

## See Also

- [[../../knowledge/google-antigravity]] — full product/strategic context: four surfaces, dynamic subagents, sunset timeline, pricing.
- [[../codex-cli/SKILL]] — OpenAI Codex CLI; same `AGENTS.md` convention, provider-portable, different approval model (tri-mode vs Antigravity's binary).
- [[../../notes/2026-05-20-google-antigravity-2]] — launch-day note on what changed at I/O 2026.
- `agents/_template/AGENTS.md` — workspace's neutral agent protocol; already Antigravity-compatible via the `GEMINI.md` → `AGENTS.md` symlink.
- `agents/_template/neutrality.md` — why backend-neutral placeholders matter when multiple agentic CLIs (`agy`, `codex`, `claude`, `kimi`) read the same `AGENTS.md`.
