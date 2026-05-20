# Workspace Bootstrap

**Date:** 2026-05-20
**Source:** Claude Code session (workspace setup from empty tree to public submodule monorepo)

## TL;DR

Bootstrapped the workspace from a near-empty directory into a public submodule monorepo at [`kla-ondemand/ai-oracle-workspace`](https://github.com/kla-ondemand/ai-oracle-workspace). Cloned six third-party study repos, converted them to submodules pinned to upstream HEAD, abstracted the workspace name out of the docs, and stood up two skill locations (`.claude/skills/` for auto-loaded, `skills/` for shared library). Initial commit `23a6754`.

## What I Learned

- **`gh repo create --source=. --push` is the one-shot path** to (a) create a public repo on GitHub, (b) wire the remote, and (c) push the existing local commit — no separate `git remote add` / `git push -u origin main` needed. It picked up the existing main branch and tracking automatically.
- **Stale `.git/modules/<path>` blocks `git submodule add`** even when the working tree is gone. Symptom: error says "use `--force` … or `--name`". A prior interrupted session had left `.git/modules/projects/openclaw/` with `HEAD: refs/heads/.invalid` and no `main` ref. Fix: `rm -rf .git/modules/projects/openclaw` and retry `git submodule add`. The `--force` flag would reuse the broken state — destructive cleanup is safer.
- **Submodule pin choice is one-way-ish.** Picking "Pin to current HEAD commit" records the SHA in the superproject tree. To refresh later: `git submodule update --remote --merge`. The "track upstream branch" alternative writes a `branch =` entry to `.gitmodules` and changes how `--remote` resolves. Switching modes after the fact means editing `.gitmodules` and re-pinning.
- **Each submodule pinned to whatever upstream HEAD was at *clone time*, not at the original `git clone` time.** Between the initial `git clone` (this morning) and `git submodule add` (~30 min later), `openclaw` upstream advanced from `0c67dc7` to `110042d`. That's expected behavior but worth noting if anyone tries to verify the pin matches the local clone we did first.
- **`agents/*/` in `.gitignore` cleanly avoids the "nested git repo as gitlink" trap.** Without it, `git add` of the workspace would silently turn each agent dir into an unconfigured submodule pointer (no `.gitmodules` entry). With it, only `agents/README.md` is tracked; the agents stay independent.
- **The agent template's `incarnate.sh` is named for a reason** — copying the template by hand omits the symlink fixups (`CLAUDE.md`/`GEMINI.md`/`CODEX.md`/`KIMI.md` → `AGENTS.md`) and the config manifest. Renaming `template` → `_template` doesn't affect `incarnate.sh` invocation, but every workspace-doc reference to the path had to be updated in lockstep.
- **The `/learning` skill name (vs `/learn`) matters** — the workspace inherits a global `/learn` skill (parallel-Haiku codebase exploration). Local skill names that collide with globals are confusing; the underscore-like distinction (`learning` vs `learn`) is enough.

## Why It Matters

- The workspace is now versionable and shareable. Anyone with the URL can `git clone --recurse-submodules` and get the same six study repos at the exact same commits.
- The two-location skill split is now committed convention, not just a one-off. Future skills go to `.claude/skills/<name>/` if they're for this workspace, or to `skills/<name>/` if they should be portable to other surfaces.
- The `agents/_template` rename means the active-agent listing in `agents/` is just `agent-data-science/` (the `_` prefix sinks `_template` below it alphabetically). When more agents are incarnated, the template stops cluttering the visible set.

## Open Questions

- Should `agents/` also become submodules? Currently they're nested-but-ignored. The trade-off: convertibility-when-pushed vs avoiding the destructive re-clone we did for `projects/`. No need to decide until a second machine wants to sync agents.
- The `oracle-framework` philosophy (ψ/ shared-soul, three principles) could shape this workspace's own memory layout. Worth a pass after [[knowledge/projects/oracle-framework]] is fully digested.
- `incubate/` is gitignored but doesn't exist yet — placeholder for "clones for active development" per the [[oracle-framework]] ψ/ convention. May need a stub README or to drop from gitignore until actually used.

## See Also

- [[../knowledge/projects/openclaude]] · [[../knowledge/projects/openclaw]] · [[../knowledge/projects/maw-js]] · [[../knowledge/projects/oracle-framework]] · [[../knowledge/projects/hermes-agent]] · [[../knowledge/projects/thclaws]] — per-project orientation summaries from this session.
- [[../skills/README]] — shared-skill convention introduced today.
- [[../.claude/skills/learning/SKILL]] — the skill that produced this note.
