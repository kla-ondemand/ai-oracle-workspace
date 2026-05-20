# Oracle Open Framework

A philosophy + filesystem architecture for sustainable AI-human collaboration. Not code-first — it is a documented set of principles, a `ψ/` directory structure for AI memory, and a small set of tools/patterns that emerged from ~8 months and 2,000+ commits of practice.

**Repo:** [Soul-Brews-Studio/oracle-framework](https://github.com/Soul-Brews-Studio/oracle-framework) · **Local clone:** `projects/oracle-framework/`

## Differentiating Angle

The slogan — **"The Oracle Keeps the Human Human"** — is load-bearing. Oracle is explicitly designed to **amplify** human consciousness rather than replace it, and every architectural decision (append-only memory, no auto-actions, query systems instead of command systems) follows from that.

## The Three Principles

| Principle | What it means | How it shows up |
|---|---|---|
| **Nothing is Deleted** | Append-only; timestamps are truth. | Git history, SQLite, trace logs. |
| **Patterns Over Intentions** | Observe behavior, not promises. | Retrospectives, learnings, search. |
| **External Brain, Not Command** | Mirror reality, don't decide. | Query systems, dashboards, no auto-actions. |

Each principle is sourced to a documented pain point from the AlchemyCat predecessor (June 2025): context loss → Nothing is Deleted; never knowing if a session was satisfactory → Patterns Over Intentions; purely transactional collaboration → External Brain.

## The ψ/ Structure (5 pillars + 2 incubation)

```
ψ/
├── active/    — current research (ephemeral, gitignored)
├── inbox/     — current task / handoffs / external agents (tracked)
├── writing/   — drafts / book / blog queue (tracked)
├── lab/       — projects in progress (tracked)
├── incubate/  — cloned repos for active dev (gitignored)
├── learn/     — cloned repos for study (gitignored)
└── memory/    — resonance/learnings/retrospectives/logs (tracked)
```

**Knowledge flow:**
`active/context → memory/logs → memory/retrospectives → memory/learnings → memory/resonance`

Commands implementing the flow: `/snapshot` → `rrr` → `/distill`.

## Key Insight: Shared Soul

> "Symlink = Identity, NOT Sync. One soul, multiple bodies."

Multi-agent coordination is solved by **symlinking** multiple agent worktrees to the same `ψ/` — they share principles, so they coordinate naturally with no command hierarchy. This is the architectural answer to the multi-agent orchestration problem.

## Why It Matters For This Workspace

Oracle is the **philosophical layer** that [[maw-js]] implements operationally (its "oracles" are agents that follow this framework). If this workspace ever grows a memory architecture, this is the documented prior art to copy or critique.

## Source Pointers

- `README.md` — full philosophy, ψ/ structure, three layers, repo map.
- Mentions an `AlchemyCat` predecessor at [`AI-HUMAN-COLLAB-CAT-LAB`](https://github.com/alchemycat/AI-HUMAN-COLLAB-CAT-LAB) — read for the *pain* that motivated Oracle.

## See Also

- [[maw-js]] — the operational counterpart: how oracles actually run.
- [[hermes-agent]] — a different take on the same problem ("self-improving memory loop") with its own architecture.
