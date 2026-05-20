---
name: learning
description: Read and write learning summaries — capture what was learned in a session, or recall past learnings on a topic
when_to_use: User says "learning", "log learning", "what did I learn", "summarize learning", "recall learning on X", or finishes studying a repo/topic and wants the takeaways persisted
user-invocable: true
---

# Learning Skill

Capture and recall learning summaries for this workspace. Storage is split by purpose:

- **Dated learnings** — `notes/YYYY-MM-DD-{title}.md` (chronological journal). Indexed by `notes/README.md`.
- **Topic consolidations** — `knowledge/{topic-kebab}.md` (durable reference). Indexed by `knowledge/README.md`.

## Modes

The user's phrasing decides the mode. If ambiguous, ask.

### Write a new learning (default for "log…", "summarize…", "I learned…")

1. **Pick the target.** Single session / one topic studied today → `notes/`. Cross-session consolidation of an established topic → `knowledge/`.
2. **Pick a filename.**
   - Notes: `YYYY-MM-DD-{title-kebab}.md` using today's date.
   - Knowledge: `{topic-kebab}.md` (no date).
3. **Check for duplicates** — `ls` the target dir and grep for similar titles. If one exists, offer to update it instead of creating a sibling.
4. **Write the summary** using the template below.
5. **Update the index** — append a one-line entry to `notes/README.md` or `knowledge/README.md` under the right section. Replace `_No notes yet._` / `_No entries yet._` placeholder if still present.

#### Write template (notes)

```markdown
# {Title}

**Date:** YYYY-MM-DD
**Source:** {repo / URL / conversation}

## TL;DR
One-paragraph takeaway.

## What I Learned
- Concrete fact / mechanism / pattern (1)
- …

## Why It Matters
How this changes a decision, unlocks a project, or connects to existing knowledge.

## Open Questions
- …

## See Also
- [[related-note-or-knowledge-file]]
```

#### Write template (knowledge)

Skip the `Date` line. Lead with a one-sentence definition, then sections that make sense for the topic (Concepts, Protocols, Gotchas, References). Knowledge files are durable — edit them over time rather than dating them.

### Read / recall (for "what did I learn about X", "recall…", "show learnings on…")

1. **Start with the indexes** — read `notes/README.md` and `knowledge/README.md` first; they list every entry.
2. **If a topic is named**, grep both `notes/` and `knowledge/` for the term (filename and content). Surface the most recent match plus any durable topic entry.
3. **If asked for a date range / "recent"**, list notes by filename date prefix in reverse chronological order.
4. **Report**: TL;DRs of each matching entry plus the file path. Don't paste the full file unless the user asks.

## Rules

- Today's date is whatever the current session date is — never guess; use the date from environment context.
- Convert relative dates in the user's wording ("yesterday", "last week") to absolute before writing.
- Filenames are kebab-case ASCII. No spaces, no underscores.
- One topic per file. If a session spans two topics, write two notes and link them in `See Also`.
- Don't write the summary into `notes/README.md` or `knowledge/README.md` directly — those are indexes only.
- If a learning is just a code pattern derivable from the repo, don't write it — point at the source file instead. Save what's non-obvious: motivations, tradeoffs, decisions, gotchas.
