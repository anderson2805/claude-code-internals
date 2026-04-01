---
title: Memory System
parent: Architecture Overview
nav_order: 2
---

# Memory System

Claude Code has a sophisticated multi-layered memory system that persists information across sessions. Understanding how it works helps you use it effectively and avoid common pitfalls.

## The `memdir` Subsystem

The file-based memory system lives in `~/.claude/projects/<project-hash>/memory/` and consists of 8 modules:

| Module | Purpose |
|:-------|:--------|
| `memdir.ts` | Core memory directory management |
| `findRelevantMemories.ts` | Retrieves memories relevant to current context |
| `memoryScan.ts` | Scans memory files for loading |
| `memoryAge.ts` | Tracks memory freshness and staleness |
| `memoryTypes.ts` | Defines memory categories (user, feedback, project, reference) |
| `paths.ts` | Memory file path resolution |
| `teamMemPaths.ts` | Team-shared memory paths |
| `teamMemPrompts.ts` | Prompts for team memory operations |

## Memory Types

Memories are stored as individual Markdown files with YAML frontmatter:

```markdown
---
name: user_prefers_tdd
description: User follows TDD workflow for all new features
type: feedback
---

Always write tests before implementation code.

**Why:** User corrected approach when implementation was written first.
**How to apply:** When implementing features, create test file first.
```

### Four Memory Categories

1. **User** - Who you are, your role, expertise, preferences
2. **Feedback** - Corrections and confirmations about how to work
3. **Project** - Ongoing work context, decisions, deadlines
4. **Reference** - Pointers to external resources

## Memory Index (`MEMORY.md`)

The `MEMORY.md` file acts as an index - a table of contents loaded into every conversation. Each entry is a one-line pointer:

```markdown
- [TDD Preference](feedback_tdd.md) - User always wants tests first
- [Senior Go Dev](user_role.md) - Deep Go expertise, new to React
```

{: .warning }
> Lines after 200 in `MEMORY.md` are truncated. Keep the index concise.

## How Memory Retrieval Works

1. `MEMORY.md` is loaded at conversation start
2. Based on the user's message, `findRelevantMemories.ts` identifies which memories to load
3. Relevant memory files are read and injected into the system prompt
4. The model uses these memories to inform its responses

## Session Memory vs Persistent Memory

There are two distinct memory systems:

| Feature | Session Memory | Persistent Memory |
|:--------|:---------------|:------------------|
| Location | `services/SessionMemory/` | `memdir/` |
| Lifetime | Current conversation only | Across all conversations |
| Storage | In-memory | File system |
| Purpose | Track current task context | Long-term preferences |

### Session Memory (`services/SessionMemory/`)
- `sessionMemory.ts` - Core session memory logic
- `sessionMemoryUtils.ts` - Utility functions
- `prompts.ts` - Instructions for when/how to use session memory

{: .tip }
> **Practical implication:** If Claude Code "forgets" something within a session, it's likely a context window issue, not a memory issue. If it forgets across sessions, check your memory files in `~/.claude/projects/`.

## Memory Staleness

The `memoryAge.ts` module tracks how old memories are. Before acting on a memory, Claude Code is instructed to verify it's still accurate by checking current file state. This prevents acting on outdated information like renamed functions or deleted files.

## Team Memory

The `teamMemPaths.ts` and `teamMemPrompts.ts` modules support shared team memories - a way for team members to share context through a common memory store. This is used in team/organization settings where multiple people work on the same project.
