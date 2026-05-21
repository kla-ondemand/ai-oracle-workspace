---
name: hermes-agent
description: Drive an external Hermes Agent (Nous Research's self-improving agent) — programmatic one-shot queries via `hermes chat -q -Q`, session resume, gateway service that bridges chat platforms (Telegram, Discord, Slack, WhatsApp, Signal, IRC, Teams, Google Chat, LINE, Weixin), MCP/skills/providers surface, and cron-scheduled delivery. Covers both the local CLI and the cross-process gateway chat protocol.
when_to_use: User mentions "hermes", "hermes-agent", "nous agent", or asks to control / talk to / schedule / set up gateway for an external Hermes. Also triggers when the workflow needs to send a query to a Hermes running on another host, reach a Hermes from a messaging platform, manage the gateway service, or orchestrate Hermes as a subagent / parallel worker. Also when a user says they want to "message" or "tell" an agent that is actually a Hermes instance.
user-invocable: true
---

# Hermes Agent Skill

Nous Research's self-improving Python agent. Runs anywhere — laptop, $5 VPS, GPU cluster, or serverless (Modal / Daytona / Vercel Sandbox) that hibernates idle — and is reachable through (a) its own `hermes` CLI, or (b) a messaging gateway that bridges to Telegram, Discord, Slack, WhatsApp, Signal, IRC, Teams, Google Chat, LINE, and Weixin. This skill teaches another agent to **control** a Hermes that is already installed somewhere, not to install Hermes itself.

This skill is **verified against the local clone at `projects/hermes-agent/` and its `hermes_cli/_parser.py` / `hermes_cli/gateway.py` as of 2026-05-21** — flag names and behaviours below are what the code actually exposes today, not extrapolated from README. Hermes is the explicit successor to OpenClaw and imports its state via `hermes claw migrate`; see `knowledge/projects/hermes-agent.md` for the orientation summary.

## When To Use

- The user wants to send a prompt to an external Hermes agent — locally over its CLI, or remotely over its messaging gateway.
- The user wants the gateway service running so Telegram/Discord/Slack/WhatsApp/etc. can reach a long-running Hermes.
- The user wants to schedule recurring jobs that report into a chat platform (`hermes cron` + gateway delivery).
- The user wants to orchestrate Hermes as a sub-worker — one-shot RPC-style turns, parallel subagents, or a multi-LLM comparison.
- The user is migrating from OpenClaw and needs the state-import path (`hermes claw migrate`).

Do **not** invoke this skill for:
- Building a Hermes plugin / adapter from scratch — that's a code-authoring task inside `projects/hermes-agent/`; this skill is about driving it as a black box.
- Anything that needs Hermes's runtime internals (Honcho dialectic model, FTS5 mining loop). Those are studied via [[hermes-agent]] knowledge, not driven via this skill.

## Two Control Surfaces

| Surface | When to reach for it | Entry point |
|---------|---------------------|-------------|
| **CLI direct** | You can shell out on the host where Hermes is installed. Best for sync RPC-style turns, scripting, CI. | `hermes chat -q "..." -Q` |
| **Gateway chat protocol** | Hermes runs on another host, or you want a human-in-the-loop conversation that survives sessions and works from a phone. | Send a message on the configured chat platform; Hermes replies in-thread. |

The two surfaces share the same Hermes session store — a conversation started over Telegram can be resumed on the CLI by ID, and vice versa.

## CLI Direct — `hermes chat` (Programmatic)

`hermes chat` is interactive by default; the `-q/--query` flag makes it one-shot, and `-Q/--quiet` strips the banner/spinner/tool-preview chrome so only the final response and a session-info footer go to stdout.

```bash
# One-shot query, minimal output (designed for programmatic capture):
hermes chat -q "summarize today's incidents" -Q

# Attach an image:
hermes chat -q "what's wrong with this screenshot?" --image ./bug.png -Q

# Pick model / provider / toolsets / skills for this run:
hermes chat -q "..." -Q \
  -m anthropic/claude-sonnet-4 \
  --provider openrouter \
  -t shell,python \
  -s skill-a -s skill-b

# Cap tool iterations (default 90 or agent.max_turns):
hermes chat -q "..." -Q --max-turns 12

# Resume / continue a session (shared with gateway-side sessions):
hermes chat --resume <session-id>
hermes chat --continue                       # most recent
hermes chat --continue "incident-2026-05-21" # by name

# Tag the session as a tool integration so it doesn't pollute the user's list:
hermes chat -q "..." -Q --source tool

# Isolated / reproducible run (no user config, no auto-injected memory or skills):
hermes chat -q "..." -Q --ignore-user-config --ignore-rules

# Auto-approve any unseen shell hooks declared in config.yaml (no TTY prompt):
hermes chat -q "..." -Q --accept-hooks

# Filesystem checkpoints before destructive ops (use /rollback to undo):
hermes chat --checkpoints

# Worktree-isolated parallel run on the same repo:
hermes chat --worktree -q "..." -Q

# Bypass dangerous-command approval prompts — only in disposable envs:
hermes chat --yolo -q "..." -Q
```

The session ID is printed at end of the response (and in the `--quiet` footer); capture it if you need to resume.

> [!warning] `--quiet` is the contract for programmatic use
> Without `-Q`, the stdout stream includes a banner, spinner ANSI, and tool-call previews — fine for humans, terrible for parsing. Always pair `-q` with `-Q` when scripting.

## Gateway Chat Protocol

Hermes ships a **messaging gateway** — a long-running service that bridges chat platforms to the agent. Once configured and running, sending a DM (or mentioning the bot in an authorized channel) on the configured platform delivers the message to Hermes; Hermes replies back in the same thread.

### Supported platforms (gateway adapters)

Telegram, Discord, Slack, WhatsApp, Signal, IRC, Microsoft Teams, Google Chat, LINE, Weixin. New platforms can be added as plugins (`~/.hermes/plugins/`) — see `gateway/platforms/ADDING_A_PLATFORM.md` in the repo.

### Gateway lifecycle

```bash
# Configure (interactive wizard — picks platforms, gathers tokens/auth):
hermes gateway setup

# Run in the foreground (good for WSL / Docker / Termux / debugging):
hermes gateway run
hermes gateway run -v               # -v = INFO, -vv = DEBUG
hermes gateway run --force          # replace any existing gateway instance

# Install as a systemd / launchd background service:
hermes gateway install              # per-user
sudo hermes gateway install --system --run-as-user $USER  # Linux system-level

# Manage the service:
hermes gateway start
hermes gateway stop
hermes gateway restart
hermes gateway status                # add --deep for thorough probe
hermes gateway list                  # all profiles + status
hermes gateway uninstall

# Migrate legacy hermes.service units from pre-rename installs:
hermes gateway migrate-legacy
```

`hermes gateway status` is the canonical health check — it surfaces installed service state, live PIDs, and platform connectivity. If it reports "no live adapter for platform <X>", the gateway is up but that adapter failed to connect (almost always credentials).

### How a message turns into a Hermes turn

1. Platform adapter (e.g. Telegram) receives a message.
2. Gateway resolves the chat → home-channel binding and the authorized-user set.
3. Message is dispatched to the Hermes agent with the matching session.
4. Hermes runs (model + tools + memory + skills) and emits a reply.
5. Adapter sends the reply back through the platform, with typing indicators / mid-flight bubbles for platforms that enforce a hard reply window (LINE 60s, WhatsApp 24h, etc.).

You don't author this loop — you configure it and observe it. The skill's job is to know that **a message into the chat platform == a turn into Hermes**, and to use that primitive when you can't shell out on the Hermes host.

### Out-of-process delivery (cron / external triggers)

Cron jobs scheduled with `hermes cron add ... --deliver <platform-name>` route their output through the configured chat platform, even when the cron runner is a separate process from the live gateway. Each platform plugin declares a `standalone_sender_fn` for this path; without it, you'll see `No live adapter for platform '<name>'` and the cron job will silently fail to deliver.

When wiring an automation that pushes into a chat from outside the gateway process:
- Check `hermes cron list` to confirm the `deliver=` target is recognized.
- For first-party platforms (`telegram`, `discord`, `slack`, `whatsapp`), out-of-process delivery is built in.
- For plugin platforms, verify `cron_deliver_env_var` and `standalone_sender_fn` are set in the plugin.

## Sessions, Memory & Continuity

- **Session store is shared** across CLI and gateway. A `--source=tool` query and a Telegram thread can both resume the same session ID if you ask them to.
- **`--source=tool`** suppresses the session from the user's interactive session list; use it for third-party integrations that shouldn't clutter the human's history.
- **Honcho** powers the user dialectic model. Every turn (regardless of surface) refines it. Don't try to fake a different user via env hacks — the model belongs to whoever Hermes thinks is talking.
- **Skills** can be preloaded for a session via `-s/--skills <name>` (repeat for multiple).
- **`--ignore-rules`** drops auto-injection of `AGENTS.md`, `SOUL.md`, `.cursorrules`, memory, and preloaded skills — combine with `--ignore-user-config` for a fully isolated run (CI, reproductions, third-party integrations).

## MCP & Provider Surface

Hermes is provider-agnostic — Nous Portal, OpenRouter, NovitaAI, NVIDIA NIM, Xiaomi MiMo, z.ai/GLM, Kimi/Moonshot, MiniMax, Hugging Face, OpenAI, or any custom OpenAI-compatible endpoint.

```bash
hermes model                         # interactive picker
hermes auth login / logout
hermes proxy start --upstream nous   # local OpenAI-compatible proxy to OAuth providers
hermes proxy providers               # available upstreams
hermes proxy status
```

`hermes proxy` is useful when you want a third-party tool that speaks OpenAI's protocol to authenticate against Nous / xAI behind the scenes.

MCP servers are first-class — same binaries that work for Claude Code / Codex / Antigravity work for Hermes. Use `hermes` config/setup to register them.

## Common Workflows

### One-shot RPC-style turn from a script

```bash
ANSWER=$(hermes chat -q "$PROMPT" -Q --source tool --max-turns 8)
echo "$ANSWER"
```

`--max-turns` prevents runaway loops; `--source tool` keeps the session out of the user's chat history.

### Multi-agent comparison (Hermes vs Claude Code vs Codex)

```bash
hermes chat -q "$(cat task.md)" -Q --source tool > /tmp/hermes.md
claude    -p   "$(cat task.md)"                  > /tmp/claude.md
codex     exec "$(cat task.md)"                  > /tmp/codex.md
diff3 /tmp/hermes.md /tmp/claude.md /tmp/codex.md
```

### Reach Hermes from a phone

1. `hermes gateway setup` on the host running Hermes — pick Telegram (or Signal, WhatsApp, …) and supply the bot token / credentials.
2. `hermes gateway install && hermes gateway start` so it survives reboots.
3. DM the bot from the phone — replies come back in-thread.
4. To resume that same conversation from the CLI later: `hermes chat --resume <id>` (the gateway prints session IDs in its delivery logs).

### Scheduled report to a chat platform

```bash
hermes cron add "daily-incidents" "0 9 * * *" \
  --deliver telegram \
  --prompt "summarize today's incidents and post the digest"
```

The cron runner uses `standalone_sender_fn` to push the result through Telegram even if it runs in a separate process from the live gateway.

### Migrating from OpenClaw

```bash
hermes claw migrate
```

Detects `~/.openclaw` and imports SOUL.md, memories, skills, command allowlist, messaging settings, API keys, TTS assets, and AGENTS.md. The OpenClaw lineage ends here.

## Gotchas

- **Always pair `-q` with `-Q` for programmatic use.** Otherwise stdout includes a banner and ANSI spinner — unparsable.
- **`--source` matters for housekeeping.** Forget `--source tool` and every scripted call shows up in the user's session list.
- **Session sharing is bidirectional.** A gateway-side conversation is resumable from the CLI by ID and vice versa — don't assume "I started this on the CLI so it's CLI-only".
- **Cron `--deliver` needs `standalone_sender_fn`.** For plugin platforms, missing this is the #1 reason scheduled jobs run successfully but never reach the chat.
- **`hermes gateway status` is mandatory before "the bot isn't replying".** It distinguishes service-down, adapter-disconnected, and authorization issues — don't restart blindly.
- **Approval / sandbox model differs from Codex.** Hermes uses `--accept-hooks` and `--yolo`, plus per-config `hooks_auto_accept:` — there is no `--ask-for-approval × --sandbox` matrix. Don't carry Codex muscle memory.
- **`--ignore-user-config` does NOT drop credentials.** `.env` is still loaded; it's only `~/.hermes/config.yaml` that's bypassed. Useful, but read the flag literally.
- **Plugin gateway adapters need both env-enablement and YAML hooks** to surface in `hermes gateway status` cleanly — missing the `env_enablement_fn` hook is the most common "adapter exists but isn't visible" bug.
- **Termux has a curated install.** `pip install hermes-agent[termux]` (not `[all]`) — `[all]` pulls Android-incompatible voice deps.
- **OpenClaw migration is one-way.** `hermes claw migrate` reads from `~/.openclaw` and writes into `~/.hermes`; there's no reverse path. Back up first if you might want to revert.

## See Also

- [[../../knowledge/projects/hermes-agent]] — workspace orientation summary (Honcho, learning loop, OpenClaw lineage, Termux extra).
- [[../../knowledge/projects/openclaw]] — predecessor; state is imported into Hermes via `hermes claw migrate`.
- [[../codex-cli/SKILL]] — sibling CLI; comparison point for the safety-model surface (different shape).
- [[../claude-cli/SKILL]] — sibling CLI; the `-p` one-shot mode is the analogue of `hermes chat -q -Q`.
- `projects/hermes-agent/gateway/platforms/ADDING_A_PLATFORM.md` — authoritative reference if you ever need to author or debug a platform adapter.
- `projects/hermes-agent/hermes_cli/_parser.py` — source of truth for current CLI flags (re-verify before assuming a flag still exists).
