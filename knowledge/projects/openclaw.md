# OpenClaw

A personal AI assistant that lives on your channels (~23 of them: WhatsApp, Telegram, Slack, Discord, iMessage, Signal, IRC, Matrix, Teams, Feishu, LINE, WeChat, QQ, etc.) and is reachable via voice on macOS/iOS/Android. Distributed as a Node CLI that installs a local Gateway daemon.

**Repo:** [openclaw/openclaw](https://github.com/openclaw/openclaw) · **Local clone:** `projects/openclaw/`

## Differentiating Angle

The README says it directly: **"The Gateway is just the control plane — the product is the assistant."** OpenClaw is not framed as a developer tool; it is framed as a single-user assistant that you reach through messengers you already use. The whole architecture flows from that.

## Mental Model

- **Gateway daemon** runs locally (launchd on macOS, systemd-user on Linux) and connects to every channel.
- **DM pairing as the default security posture** — unknown senders get a pairing code, and the bot ignores their message until you run `openclaw pairing approve <channel> <code>`. Treating inbound DMs as untrusted input is baked in, not optional.
- **Onboarding is interactive:** `openclaw onboard` walks through gateway → workspace → channels → skills. Not a config file you tweak.
- **Plugin / extension architecture is strict.** The root `AGENTS.md`/`CLAUDE.md` is a wall of maintainer policy: core must stay plugin-agnostic, plugins cross into core only via `openclaw/plugin-sdk/*`, channels live under `src/channels/**`, etc. If you read code here, read the scoped `AGENTS.md` for the subtree first.

## Operational Gotchas (from the maintainer CLAUDE.md)

- Package manager: **pnpm** with Bun lock alignment. Don't swap.
- Formatter is **oxfmt**, not Prettier. Typecheck is **tsgo**, not `tsc --noEmit`.
- Sharp/libvips can fail on Homebrew — use `SHARP_IGNORE_GLOBAL_LIBVIPS=1 pnpm install`.
- Versioning: `vYYYY.M.D-beta.N` for beta releases.
- Hermes Agent has an explicit **migration path away from OpenClaw** (`hermes claw migrate` reads `~/.openclaw`). Position OpenClaw as the predecessor in that lineage.

## Why It Matters For This Workspace

OpenClaw is the canonical example in this workspace of an agent whose primary surface is **chat platforms, not a terminal**. If you ever want to wire an agent to WhatsApp/Telegram/Signal, this is the reference.

## Source Pointers

- `README.md` — channels, security defaults, onboarding flow.
- `CLAUDE.md` (root) — maintainer policy: plugin-SDK boundaries, build/test/lint lanes, GitHub/PR rituals. **Telegraph style; read the scoped `AGENTS.md` for whatever subtree you actually touch.**
- `src/channels/**` — channel implementations.
- `src/plugin-sdk/*`, `src/plugins/*` — plugin contract.
- `extensions/**` — bundled plugins; "product/docs/UI/changelog wording: 'plugin/plugins'; `extensions/` is internal."

## See Also

- [[hermes-agent]] — Nous Research's successor agent; has `hermes claw migrate` to import OpenClaw state.
- [[openclaude]] — different shape: coding CLI, not a channel-based assistant.
- [[thclaws]] — another agent harness, but Rust + GUI surfaces instead of channel-first.
