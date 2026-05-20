---
name: learning
description: Read and write learning summaries — capture what was learned in a session, or recall past learnings on a topic. Always stores to knowledge/.
when_to_use: User says "learning", "log learning", "what did I learn", "summarize learning", "recall learning on X", or finishes studying a repo/topic and wants the takeaways persisted
user-invocable: true
---

# Learning Skill

Capture and recall learning summaries for this workspace. **All writes go to `knowledge/`** — this workspace treats learnings as durable reference material that grows over time, not session-bound diary entries.

## Storage rule (workspace-specific)

- **All learnings → `knowledge/{topic-kebab}.md`** (durable topic-organized reference). Indexed by `knowledge/README.md`.
- **Never write learning output to `notes/`.** `notes/` is reserved for manual journal-style entries; do not route automated skill output there.
- Recall mode (read-only) MAY still scan `notes/` to surface legacy or manually-written entries, but new writes always land in `knowledge/`.

## Modes

The user's phrasing decides the mode. If ambiguous, ask.

### Write a new learning (default for "log…", "summarize…", "I learned…")

1. **Pick a filename** — `knowledge/{topic-kebab}.md`. Kebab-case ASCII. **No date prefix.** One topic per file.
2. **Check for duplicates** — `ls knowledge/` and grep for similar titles or aliases. If an entry on the topic already exists, **edit it in place** (add new sections, refine existing ones) rather than creating a sibling file.
3. **Pick the section/subdirectory** — if the topic is a workspace `projects/<name>/` study clone, write to `knowledge/projects/<name>.md`. Otherwise, write to `knowledge/{topic-kebab}.md` at the knowledge root.
4. **Write the summary** using the knowledge template below.
5. **Update the index** — append a one-line entry to `knowledge/README.md` under the right section (e.g. "Projects", "External platforms / vendors"). If the section doesn't exist yet, add it. Replace `_No entries yet._` placeholder if present.

#### Write template (knowledge)

```markdown
---
title: {Topic}
aliases:
  - {alternate name 1}
  - {alternate name 2}
tags:
  - topic/{primary-topic}
  - {other tags as relevant}
---

# {Topic}

{One-sentence definition — what this thing is.}

## {First conceptual section}

{Body — concepts, mechanisms, decisions, rationale.}

## {Additional sections as needed}

Common sections: Concepts, Architecture, Protocols, Surfaces, Gotchas, Pricing,
Relation to This Workspace, References, See Also.

## References

- [Primary source title](URL)
- ...

## See Also

- [[related-knowledge-entry]]
- [[../notes/2026-MM-DD-related-note]] (if a manual dated note also exists)
```

Knowledge files are **durable** — edit them over time rather than dating them. No `Date:` line. Lead with a one-sentence definition; then sections that make sense for the topic.

### Read / recall (for "what did I learn about X", "recall…", "show learnings on…")

1. **Start with the indexes** — read `knowledge/README.md` first, then `notes/README.md`. Both list every entry.
2. **If a topic is named**, grep both `knowledge/` and `notes/` for the term (filename and content). Surface the durable knowledge entry first; mention any dated notes that exist as supporting context.
3. **If asked for a date range / "recent"**, list notes by filename date prefix in reverse chronological order (notes/ is the only place dates live).
4. **Report**: one-line summary of each matching entry plus the file path. Don't paste full file contents unless the user asks.

## Rules

- **All writes → `knowledge/`.** Never create `notes/YYYY-MM-DD-*.md` from this skill.
- Today's date is whatever the current session date is — only used inside knowledge content (e.g. "launched on 2026-05-19"), never in filenames.
- Convert relative dates in the user's wording ("yesterday", "last week") to absolute before writing.
- Filenames are kebab-case ASCII. No spaces, no underscores, no date prefix.
- One topic per file. If a session spans two topics, write two knowledge entries and link them in `See Also`.
- Don't write the summary into `knowledge/README.md` directly — that's an index only.
- If a learning is just a code pattern derivable from the repo, don't write it — point at the source file instead. Save what's non-obvious: motivations, tradeoffs, decisions, gotchas.
- Prefer **updating an existing knowledge entry** over creating a new one. The whole point of `knowledge/` is that entries grow over time.
