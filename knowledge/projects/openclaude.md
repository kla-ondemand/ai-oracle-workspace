# OpenClaude

An open-source coding-agent CLI that lets the Claude Code terminal workflow speak to non-Anthropic providers — OpenAI-compatible, Gemini, GitHub Models, Codex OAuth, Codex, Ollama, Atomic Chat, and other backends — under one shell.

**Repo:** [Gitlawb/openclaude](https://github.com/Gitlawb/openclaude) · **Local clone:** `projects/openclaude/`

## Differentiating Angle

It is not "another agent CLI." It is the Claude Code UX (prompts, tools, agents, MCP, slash commands, streaming output) **with the provider swapped out**. The whole point is that you keep the same workflow whether you call OpenAI, run a local Ollama, or use Codex via OAuth.

## Mental Model

- `/provider` is the central slash command — guided setup + saved profiles per backend.
- `CLAUDE_CODE_USE_OPENAI=1` plus `OPENAI_BASE_URL` is the universal escape hatch for any OpenAI-compatible service (OpenRouter, DeepSeek, Groq, Mistral, LM Studio, etc.).
- `ollama launch openclaude --model …` skips env-var setup entirely — Ollama sets `ANTHROPIC_BASE_URL` + routing automatically.
- A bundled VS Code extension covers launch integration and theme.
- Mirrored to GitLawb (a decentralized repo host) as well as GitHub.

## Why It Matters For This Workspace

If you ever want to drive a Claude-Code-style terminal agent against a local model or a cheap third-party gateway, OpenClaude is the closest fork to study — same surface, different plumbing.

## Source Pointers

- `README.md` — provider table, install paths.
- `docs/non-technical-setup.md`, `docs/quick-start-{windows,mac-linux}.md` — friendly setup.
- `docs/advanced-setup.md`, `ANDROID_INSTALL.md` — source-build + Android.
- `vscode-extension/` — companion extension.

## See Also

- [[openclaw]] — different shape: multi-channel personal assistant, not a coding CLI.
- [[hermes-agent]] — same provider-flexibility theme + adds a learning loop.
- [[thclaws]] — same "swap provider" theme implemented in Rust with a GUI.
