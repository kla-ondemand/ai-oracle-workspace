# maw-js

A CLI for running multiple AI agents across machines: each agent lives in a tmux window, you `maw hey <agent> "…"` to send tasks, `maw peek` to watch its screen, and federation links nodes together with HMAC-SHA256-signed RPC. Built on Bun + Claude Code.

**Repo:** [Soul-Brews-Studio/maw-js](https://github.com/Soul-Brews-Studio/maw-js) · **Local clone:** `projects/maw-js/`

## Differentiating Angle

Other agent runners pretend the agent is a single process you chat with. **maw is honest that you'll end up running many of them**, possibly on different machines, and gives them addressable names + a tmux-based screen you can peek at. The federation lens UI (separate `maw-ui` repo) is a 2D mesh visualization.

## Mental Model

- **Wake / sleep / done** is the agent lifecycle: `maw wake neo` creates the tmux window, `maw sleep` stops it, `maw done` cleans worktree/branch.
- **Bare-name addressing** is local: `maw hey neo "…"` finds the local oracle named `neo` (errors on ambiguity). For remote: `white:neo:3` = node `white`, oracle `neo`, tmux window 3.
- **Federation = HMAC-signed HTTP** between nodes. Shared secret of ≥16 chars in `maw.config.json` under `federationToken`. Named peers point at remote URLs.
- **`maw bud`** spawns a new oracle. Stem-only naming — `maw bud fusion` → `fusion-oracle` repo; never include `-oracle` in the stem or you double the suffix.
- **UI ships separately.** `maw serve` runs API + UI on `:3456` only if you've run `maw ui install` (which pulls `dist.tar.gz` from a `maw-ui` GitHub release). Lens reads `?host=` at runtime — same backend can be viewed from any federation.

## Operational Gotchas

- Versioning: **CalVer** since 2026-04-18 (was SemVer alpha). Format: `v{yy}.{m}.{d}[-alpha.{HHMM}]`.
- `maw: command not found` is a known failure mode (npm name collision, see issue #531). Recovery: `bun add -g github:Soul-Brews-Studio/maw-js`, or `bunx -p … maw doctor`, or source `scripts/maw-heal.sh` from your shell rc.
- `maw update` stashes the binary to `~/.bun/bin/maw.prev` before destructive `bun remove`, so a broken update is recoverable.

## Why It Matters For This Workspace

If you ever want to coordinate multiple agents across multiple machines, **maw is the multi-agent control plane** counterpart to [[oracle-framework]]'s shared-soul philosophy. They are a paired stack from the same studio.

## Source Pointers

- `README.md` — full CLI surface, federation config, recovery runbook.
- `docs/install-recovery.md` — the `command not found` runbook.
- `docs/testing/coverage-gap-analysis.md` — what's tested, what isn't.

## See Also

- [[oracle-framework]] — the philosophy + ψ/ soul structure that maw's "oracles" implement.
- [[hermes-agent]] — different multi-agent model: subagent spawning inside one process, plus messaging gateway.
