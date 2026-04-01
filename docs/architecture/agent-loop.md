---
title: The Agent Loop
parent: Architecture Overview
nav_order: 1
---

# The Agent Loop (TAOR Pattern)

The core of Claude Code is an agent loop following the **Think-Act-Observe-Repeat (TAOR)** pattern. The runtime is deliberately minimal -- "the runtime is dumb; the model is the CEO." The LLM decides the next action at every step rather than following hard-coded orchestration. Understanding this loop is key to using Claude Code effectively.

## Core Components

### Coordinator Mode (`coordinator/coordinatorMode.ts`)
The coordinator orchestrates how Claude Code operates in different modes. It manages transitions between interactive use, autonomous execution, and multi-agent coordination.

### REPL Screen (`screens/REPL.tsx`)
The main interactive loop. This is the React component that:
1. Renders the input prompt
2. Sends messages to the Claude API
3. Processes tool calls from the response
4. Renders results back to the terminal
5. Loops back to step 1

### Query Engine (`QueryEngine.ts`)
Manages the actual API calls, including:
- Context assembly (system prompt + conversation history)
- Token counting and context window management
- Streaming response handling
- Error recovery and retries

## How a Turn Works

```
User Input
    │
    ▼
┌──────────────────┐
│ Assemble Context  │ ◄── System prompt + history + tools
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  API Call         │ ◄── Streaming response
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Parse Response   │
└────────┬─────────┘
         │
    ┌────┴────┐
    │         │
    ▼         ▼
 Text      Tool Calls
 Output    ┌──────────────┐
           │ Permission   │
           │ Check        │
           └──────┬───────┘
                  │
           ┌──────┴───────┐
           │ Execute Tool  │
           └──────┬───────┘
                  │
           ┌──────┴───────┐
           │ Append Result │
           │ to History    │
           └──────┬───────┘
                  │
                  ▼
           Next API Call
           (loop continues)
```

## Context Assembly

The system prompt is not a single static string. It's assembled from multiple sections defined in `constants/systemPromptSections.ts`:

1. **Base instructions** - Core behavior rules
2. **Tool definitions** - Each tool's prompt from its `prompt.ts`
3. **Environment info** - OS, shell, git status, working directory
4. **CLAUDE.md content** - Project-specific instructions
5. **Memory** - Relevant memories from the `memdir` system
6. **Active context** - Current files, git state, etc.

{: .insight }
> The system prompt is dynamically assembled each turn, not static. This means it adapts to your project, your preferences, and the current state of the conversation.

## Sub-Agent Architecture

Claude Code can spawn sub-agents via the `AgentTool`. Each sub-agent:

- Gets its own conversation context (fresh start)
- Has access to a subset of tools
- Runs in the same process or in an isolated git worktree
- Returns a single result message back to the parent

### Built-in Agent Types

| Agent | Purpose | Key Restriction |
|:------|:--------|:----------------|
| `generalPurposeAgent` | Multi-step tasks, research | Full tool access |
| `exploreAgent` | Codebase exploration | No Edit/Write tools |
| `planAgent` | Architecture planning | No Edit/Write tools |
| `clawCodeGuideAgent` | Claude Code help questions | Read-only tools |
| `verificationAgent` | Verify work is correct | Read-only + Bash |
| `statuslineSetup` | Configure status line | Read + Edit only |

{: .tip }
> When you ask Claude Code to "explore the codebase", it spawns an `exploreAgent` that deliberately cannot modify files. This is a safety feature, not a limitation. The explore agent can search broadly without risk.

## Cost Tracking

The `cost-tracker.ts` and `costHook.ts` modules track API usage in real-time:
- Token counts per turn
- Cumulative session costs
- Cost breakdowns by tool usage
- Rate limit monitoring

## Speculation / Prompt Suggestion

The `services/PromptSuggestion/speculation.ts` module implements speculative prompt completion - predicting what you might want to do next based on context. This is what powers the suggested follow-up actions you sometimes see.

## Tool Concurrency Model

Tools are classified as either **concurrent** (read-only, parallel-safe) or **serialized** (mutating, sequential execution). This separation enforces write consistency while enabling read parallelism -- multiple file reads can happen simultaneously, but edits are serialized.

## Silent Model Downgrade

After three consecutive 529 (overloaded) errors, the system silently switches from Opus to Sonnet **without user notification**. This is a resilience mechanism, but users should be aware their model may have been downgraded during high-traffic periods.

## Default Loop Bounds

The query engine has tuned defaults that reveal design intent:

| Parameter | Default | Purpose |
|:----------|:--------|:--------|
| `max_turns` | 8 | Prevents runaway agent loops |
| `max_budget_tokens` | 2,000 | Modest per-query token budget |
| `compact_after_turns` | 12 | Compaction triggers after 12 turns |
| `structured_retry_limit` | 2 | JSON parse failures get 2 retries |

Stop reasons include: `completed`, `max_turns_reached`, `max_budget_reached`.
