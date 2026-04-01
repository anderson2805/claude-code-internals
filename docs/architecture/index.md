---
title: Architecture Overview
layout: default
nav_order: 2
has_children: true
---

# Architecture Overview

Claude Code is a surprisingly large and complex TypeScript application. With 1,902 source files organized into 35+ subsystems, it's far more than a simple "chat wrapper" around an API. Understanding its architecture helps you predict its behavior and work with it more effectively.

## The Big Picture

```
                    ┌─────────────────────────┐
                    │     Entry Points         │
                    │  CLI / SDK / MCP / IDE   │
                    └────────────┬────────────┘
                                 │
                    ┌────────────▼────────────┐
                    │    Bootstrap & Setup     │
                    │  Auth, Config, State     │
                    └────────────┬────────────┘
                                 │
                    ┌────────────▼────────────┐
                    │   Coordinator / REPL     │
                    │    (Agent Loop Core)     │
                    └────────────┬────────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
    ┌─────────▼─────────┐ ┌─────▼──────┐ ┌────────▼────────┐
    │   Tool System      │ │  Services  │ │  Bridge (IDE)   │
    │ 20+ built-in tools │ │ 130 modules│ │  31 modules     │
    └─────────┬─────────┘ └─────┬──────┘ └────────┬────────┘
              │                  │                  │
    ┌─────────▼─────────┐ ┌─────▼──────┐ ┌────────▼────────┐
    │   Hooks & Perms    │ │  Memory    │ │  Remote Sessions│
    │   104 modules      │ │  System    │ │  WebSocket      │
    └────────────────────┘ └────────────┘ └─────────────────┘
```

## Subsystem Inventory

The source code is organized into these major subsystems:

| Subsystem | Modules | Purpose |
|:----------|:--------|:--------|
| **utils** | 564 | Shared utilities - the largest subsystem by far |
| **services** | 130 | Business logic services (analytics, API, memory, docs) |
| **hooks** | 104 | React hooks for UI state, notifications, permissions |
| **bridge** | 31 | IDE integration layer (VS Code, JetBrains) |
| **constants** | 21 | Configuration constants, prompt sections, API limits |
| **skills** | 20 | Bundled skill definitions (debug, simplify, verify, etc.) |
| **memdir** | 8 | File-based persistent memory system |
| **entrypoints** | 8 | CLI, SDK, MCP entry points |
| **state** | 6 | Application state management |
| **buddy** | 6 | Companion sprite system (the Quartz penguin) |
| **vim** | 5 | Built-in vim keybinding emulation |
| **remote** | 4 | Remote session management |
| **screens** | 3 | Terminal UI screens (REPL, Doctor, Resume) |
| **server** | 3 | Direct-connect session server |
| **plugins** | 2 | Plugin loading and management |
| **coordinator** | 1 | Agent mode orchestration |
| **bootstrap** | 1 | Startup state initialization |
| **voice** | 1 | Voice input mode |

## Key Architectural Patterns

### 1. React/Ink Terminal UI
Claude Code's terminal interface is built with [Ink](https://github.com/vadimdemedes/ink), a React renderer for the terminal. This means the entire CLI uses React components, hooks, and state management - the same patterns used in web apps, but rendered to the terminal.

### 2. Tool-Per-Directory Pattern
Every tool lives in its own directory with a consistent file structure:
- `ToolName.ts/tsx` - Main tool logic
- `UI.tsx` - Display/rendering component
- `prompt.ts` - System prompt instructions for the tool
- `constants.ts` - Tool-specific constants

### 3. Multi-Entry Architecture
Claude Code can be invoked in four different ways:
- **CLI** (`entrypoints/cli.tsx`) - Direct terminal usage
- **SDK** (`entrypoints/sdk/`) - Programmatic API
- **MCP** (`entrypoints/mcp.ts`) - Model Context Protocol server mode
- **Bridge** (`bridge/`) - IDE extension integration

### 4. Agent-Based Subprocesses
The `AgentTool` supports spawning specialized sub-agents (explore, plan, general-purpose, verification) that run as isolated processes with their own context.
