---
title: Tool System
layout: default
nav_order: 3
has_children: true
---

# Tool System Deep-Dive

Claude Code's tool system is the mechanism through which it interacts with your filesystem, runs commands, and manages code. With 184 tool-related modules, it's a carefully engineered system with multiple layers of security.

## Tool Inventory

Claude Code has 20+ built-in tools, each following the tool-per-directory pattern:

### File Operations
| Tool | Directory | Key Files |
|:-----|:----------|:----------|
| **FileReadTool** | `tools/FileReadTool/` | FileReadTool.ts, imageProcessor.ts, limits.ts |
| **FileWriteTool** | `tools/FileWriteTool/` | FileWriteTool.ts |
| **FileEditTool** | `tools/FileEditTool/` | FileEditTool.ts, utils.ts, types.ts |
| **GlobTool** | `tools/GlobTool/` | GlobTool.ts |
| **GrepTool** | `tools/GrepTool/` | GrepTool.ts |
| **NotebookEditTool** | `tools/NotebookEditTool/` | NotebookEditTool.ts |

### Shell Execution
| Tool | Directory | Key Files |
|:-----|:----------|:----------|
| **BashTool** | `tools/BashTool/` | BashTool.tsx, bashSecurity.ts, bashPermissions.ts, destructiveCommandWarning.ts, sedValidation.ts, shouldUseSandbox.ts |
| **PowerShellTool** | `tools/PowerShellTool/` | PowerShellTool.tsx, powershellSecurity.ts, gitSafety.ts, clmTypes.ts |

### Agent & Planning
| Tool | Directory | Key Files |
|:-----|:----------|:----------|
| **AgentTool** | `tools/AgentTool/` | AgentTool.tsx, forkSubagent.ts, runAgent.ts, resumeAgent.ts, agentMemory.ts |
| **EnterPlanModeTool** | `tools/EnterPlanModeTool/` | EnterPlanModeTool.ts |
| **ExitPlanModeTool** | `tools/ExitPlanModeTool/` | ExitPlanModeV2Tool.ts |
| **EnterWorktreeTool** | `tools/EnterWorktreeTool/` | EnterWorktreeTool.ts |
| **ExitWorktreeTool** | `tools/ExitWorktreeTool/` | ExitWorktreeTool.ts |

### External Integration
| Tool | Directory | Purpose |
|:-----|:----------|:--------|
| **MCPTool** | `tools/MCPTool/` | Model Context Protocol integration |
| **McpAuthTool** | `tools/McpAuthTool/` | MCP authentication |
| **LSPTool** | `tools/LSPTool/` | Language Server Protocol integration |
| **ReadMcpResourceTool** | `tools/ReadMcpResourceTool/` | Read MCP resources |
| **ListMcpResourcesTool** | `tools/ListMcpResourcesTool/` | List MCP resources |
| **RemoteTriggerTool** | `tools/RemoteTriggerTool/` | Remote agent triggers |

### Session Management
| Tool | Directory | Purpose |
|:-----|:----------|:--------|
| **AskUserQuestionTool** | `tools/AskUserQuestionTool/` | Prompt user for input |
| **BriefTool** | `tools/BriefTool/` | Brief/handoff creation with attachments |
| **ConfigTool** | `tools/ConfigTool/` | Settings management |
| **ScheduleCronTool** | `tools/ScheduleCronTool/` | Cron job scheduling (Create/Delete/List) |

## The Tool-Per-Directory Pattern

Every tool follows a consistent structure:

```
tools/ToolName/
├── ToolName.ts      # Main logic: parameter validation, execution
├── UI.tsx           # React/Ink component for terminal display
├── prompt.ts        # System prompt text injected when tool is available
├── constants.ts     # Tool-specific constants
└── [extras].ts      # Tool-specific modules (security, validation, etc.)
```

{: .insight }
> Each tool's `prompt.ts` file contains the exact instructions that Claude sees about how to use that tool. These prompts are dynamically injected into the system prompt, which is why adding/removing tools changes Claude's behavior.

## Tool Registration

Tools are registered in `tools.ts` at the root level. The registration system:
1. Discovers tool directories
2. Loads each tool's definition (name, parameters, handler)
3. Loads each tool's prompt
4. Makes tools available to the agent loop
5. Applies permission rules per tool
