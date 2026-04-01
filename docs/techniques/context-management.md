---
title: Context Window Management
parent: Prompt Engineering
nav_order: 1
---

# Context Window Management

Claude Code operates within a finite context window, and how it manages this constraint is one of its most important engineering challenges. Understanding this helps you structure conversations for maximum effectiveness.

## The Context Budget

Every API call includes:
- **System prompt** - Dynamically assembled, varies by tool availability
- **Conversation history** - All previous turns
- **Tool results** - Output from executed tools
- **Active context** - Current file content, git state

The `constants/apiLimits.ts` module defines the boundaries, and `QueryEngine.ts` manages the budget.

## Compression Strategy

When the conversation approaches context limits, Claude Code automatically compresses prior messages. This is mentioned in the system prompt:

> "The system will automatically compress prior messages in your conversation as it approaches context limits."

### What Gets Compressed
- Earlier conversation turns are summarized
- Tool results from completed operations are condensed
- The most recent turns are preserved in full

### What Never Gets Compressed
- The system prompt (always present in full)
- The current turn's context
- Active tool definitions

{: .insight }
> This is why Claude Code might "forget" details from early in a long conversation - they've been compressed. If you need to reference something from earlier, re-state it explicitly.

## Tool Result Management

### MCP Tool Collapse (`tools/MCPTool/classifyForCollapse.ts`)

MCP tool results are classified for potential collapsing:
- Large results may be summarized
- Repeated similar results can be deduplicated
- Results are categorized to determine compression strategy

### File Read Limits (`tools/FileReadTool/limits.ts`)

File reading has built-in limits:
- Default: reads up to 2,000 lines from the start
- Supports `offset` and `limit` parameters for targeted reading
- Large files trigger a warning suggesting partial reads
- Token count is checked against a maximum (10,000 tokens)

{: .tip }
> When working with large files, always tell Claude Code which section you're interested in. "Read lines 100-200 of file.ts" is much more context-efficient than "Read file.ts".

## Circular Buffer (`utils/CircularBuffer.ts`)

Claude Code uses a circular buffer for efficient memory management of conversation history. This data structure:
- Has a fixed capacity
- Overwrites oldest entries when full
- Provides O(1) insertion and access
- Prevents unbounded memory growth

## Query Guard (`utils/QueryGuard.ts`)

The `QueryGuard` module protects against:
- Excessive API calls in a single session
- Runaway agent loops
- Context window exhaustion from rapid tool use

## API Preconnection (`utils/apiPreconnect.ts`)

Before the user even types their first message, Claude Code preconnects to the API. This reduces perceived latency for the first response by establishing the connection during startup.

## Practical Implications

### Long Conversations
- After ~20-30 tool-heavy turns, expect compression to kick in
- Critical context should be re-stated if needed
- Consider starting a new conversation for unrelated tasks

### Large Codebases
- Claude Code reads files on-demand, not all at once
- It uses Glob and Grep to find relevant files first
- CLAUDE.md files provide persistent context that survives compression

### Multi-File Operations
- Each file read consumes context budget
- Targeted reads (specific line ranges) are more efficient
- The sub-agent system helps by isolating exploration from the main context
