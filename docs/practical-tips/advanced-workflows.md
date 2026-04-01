---
title: Advanced Workflows
layout: default
parent: Practical Tips
nav_order: 1
---

# Advanced Workflows

Understanding Claude Code's internals enables workflows that go beyond basic "ask and receive" interactions.

## 1. Worktree-Isolated Development

Claude Code supports git worktrees natively through `EnterWorktreeTool` and `ExitWorktreeTool`. This enables:

```mermaid
graph TD
    A["Main workspace (stable)"] --> B["Worktree A<br/>(feature branch - agent working)"]
    A --> C["Worktree B<br/>(experiment - different agent)"]
```

**How to use it:**
- Ask Claude Code to work in an isolated worktree for risky changes
- Sub-agents can be spawned with `isolation: "worktree"` to work on separate branches
- If the changes work, merge back; if not, discard the worktree cleanly

**Source:** `tools/EnterWorktreeTool/`, `tools/ExitWorktreeTool/`

---

## 2. Plan Mode for Architecture

The `EnterPlanModeTool` switches Claude Code into a planning mode where:
- Code modifications are disabled
- The model focuses on architecture and design
- Plans are structured and reviewable
- You approve the plan before any code is written

**Best for:**
- Large feature design
- Refactoring strategy
- Understanding existing architecture before changes

**Source:** `tools/EnterPlanModeTool/`, `tools/ExitPlanModeTool/`

---

## 3. Cron-Scheduled Agents

The `ScheduleCronTool` (with CronCreate, CronDelete, CronList) enables scheduled autonomous operations:

```mermaid
flowchart LR
    A["⏰ Schedule:<br/>Every day at 9am"] --> B["🤖 Task:<br/>Check for dependency updates<br/>and create a PR if any found"]
```

**Use cases:**
- Automated dependency updates
- Daily code quality checks
- Scheduled documentation generation
- Regular test suite execution

**Source:** `tools/ScheduleCronTool/`

---

## 4. Remote Agent Triggers

The `RemoteTriggerTool` enables triggering Claude Code agents remotely:

```mermaid
flowchart LR
    A[External System] --> B[Trigger] --> C[Claude Code Agent] --> D[Result]
```

**Use cases:**
- CI/CD integration (trigger on PR creation)
- Webhook-driven automation
- Cross-system orchestration

**Source:** `tools/RemoteTriggerTool/`

---

## 5. LSP-Powered Code Intelligence

The `LSPTool` integrates Language Server Protocol, giving Claude Code:
- Go-to-definition
- Find references
- Symbol search
- Type information
- Code diagnostics

**How to leverage it:**
- Ask Claude Code about type relationships
- Request "find all usages of function X"
- LSP provides more accurate results than grep for code navigation

**Source:** `tools/LSPTool/`, `symbolContext.ts`

---

## 6. MCP Server Integration

Claude Code can connect to MCP servers for extended capabilities:

```mermaid
flowchart LR
    A[Claude Code] <--> B[MCP Server] <--> C["External Service<br/>(Database, API, etc.)"]
```

**Key modules:**
- `MCPTool` - Execute MCP tools
- `McpAuthTool` - Authenticate with MCP servers
- `ListMcpResourcesTool` - Discover available resources
- `ReadMcpResourceTool` - Read MCP-provided resources

**Source:** `tools/MCPTool/`, `tools/McpAuthTool/`, `skills/mcpSkillBuilders.ts`

---

## 7. Memory-Driven Personalization

Use the memory system strategically:

```mermaid
graph TD
    M[Memory Types] --> U["user — Your role, expertise, preferences"]
    M --> F["feedback — Corrections and confirmations"]
    M --> P["project — Current work context"]
    M --> R["reference — External resource pointers"]
```

**Advanced memory techniques:**
- Store team conventions as feedback memories
- Use project memories for sprint context
- Reference memories for linking to external docs, dashboards, tickets
- The `remember` skill (`skills/bundled/remember.ts`) provides a structured way to save memories

**Source:** `memdir/`, `skills/bundled/remember.ts`

---

## 8. Multi-Session Handoffs with Briefs

The `BriefTool` creates structured handoff documents:

```mermaid
flowchart TD
    A["Session 1: Research phase"] -->|"Brief with findings + attachments"| B["Session 2: Implementation phase"]
```

**What briefs include:**
- Context summary
- File attachments
- Decision log
- Next steps

**Source:** `tools/BriefTool/`, `attachments.ts`, `upload.ts`

---

## 9. Parallel Sub-Agent Execution

Launch multiple sub-agents simultaneously for independent tasks:

```mermaid
graph TD
    MA[Main Agent] --> A1["Agent 1: Research auth patterns<br/>(explore)"]
    MA --> A2["Agent 2: Find all API endpoints<br/>(explore)"]
    MA --> A3["Agent 3: Check test coverage<br/>(general-purpose)"]
    A1 -->|results| MA2["Main Agent: Synthesize and implement"]
    A2 -->|results| MA2
    A3 -->|results| MA2
```

**Source:** `tools/AgentTool/forkSubagent.ts`, `runAgent.ts`

---

## 10. Custom Hooks for Automation

The hook system (104 modules) enables event-driven automation:

| Event | Use Case |
|:------|:---------|
| `PreToolUse` | Validate before tool execution |
| `PostToolUse` | React to tool results |
| `Stop` | Actions when conversation ends |

**Example hooks:**
- Auto-format code after every file write
- Run linter after every edit
- Block certain commands in production directories
- Send notifications on task completion

**Source:** `hooks/`, `schemas/hooks.ts`
