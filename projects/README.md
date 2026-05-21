# Projects

One directory per study clone. Each `projects/<name>/` is an independent upstream repo tracked as a **git submodule** of this workspace, pinned to a specific commit. Treat as **read-only** — do not commit upstream without explicit instruction.

## How clones are added

```bash
git submodule add <git-url> projects/<name>
git commit -m "projects: add <name> study clone"
```

A fresh workspace clone needs `git clone --recurse-submodules` (or `git submodule update --init` afterward) to populate this directory. Submodule pointers live in [`../.gitmodules`](../.gitmodules).

## Conventions

- **Read-only by default.** These are study clones, not forks — build/test/lint scripts are run locally, but don't `git push` upstream without being asked.
- **Per-project commands.** Build and test instructions live inside each sub-repo (`README.md`, `CLAUDE.md`, `package.json`, `Makefile`, etc.). There is no workspace-level toolchain.
- **Orientation summaries.** Before re-reading an upstream repo, check `knowledge/projects/<name>.md` for a one-page summary of what the project is and why it's here.
- **Pin discipline.** Updating a submodule pointer (`git submodule update --remote`) is an intentional act — commit the bump with a note about what changed.

## Index

| Path | Upstream | What it is | Summary |
|------|----------|------------|---------|
| [`hermes-agent/`](hermes-agent/) | [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) | Self-improving Python agent with a built-in learning loop (skill creation + conversation mining + persistent user model via Honcho). | [knowledge](../knowledge/projects/hermes-agent.md) |
| [`maw-js/`](maw-js/) | [Soul-Brews-Studio/maw-js](https://github.com/Soul-Brews-Studio/maw-js) | Multi-agent runner — each agent lives in a tmux window, addressable across machines via HMAC-signed federation. Bun + Claude Code. | [knowledge](../knowledge/projects/maw-js.md) |
| [`openclaude/`](openclaude/) | [Gitlawb/openclaude](https://github.com/Gitlawb/openclaude) | Claude Code UX with the provider swapped out — OpenAI-compatible / Gemini / Codex OAuth / Ollama / GitHub Models under one shell. | [knowledge](../knowledge/projects/openclaude.md) |
| [`openclaw/`](openclaw/) | [openclaw/openclaw](https://github.com/openclaw/openclaw) | Personal assistant reachable through ~23 messaging channels (WhatsApp, Telegram, Slack, Discord, iMessage, …) via a local Gateway daemon. | [knowledge](../knowledge/projects/openclaw.md) |
| [`oracle-framework/`](oracle-framework/) | [Soul-Brews-Studio/oracle-framework](https://github.com/Soul-Brews-Studio/oracle-framework) | Philosophy + `ψ/` filesystem architecture for sustainable AI-human collaboration. Append-only memory, query-not-command. | [knowledge](../knowledge/projects/oracle-framework.md) |
| [`thClaws/`](thClaws/) | [thClaws/thClaws](https://github.com/thClaws/thClaws) | Native-Rust agent workspace; one binary, four surfaces (desktop GUI / CLI REPL / single-turn / webapp). Standards-first: MCP, `AGENTS.md`, `SKILL.md`. | [knowledge](../knowledge/projects/thclaws.md) |

## See Also

- [`../knowledge/projects/`](../knowledge/projects/) — orientation summary per clone (read first).
- [`../CLAUDE.md`](../CLAUDE.md) — workspace-level layout and conventions.
- [`../.gitmodules`](../.gitmodules) — canonical submodule manifest.
