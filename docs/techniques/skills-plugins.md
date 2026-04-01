---
title: Skills & Plugin System
layout: default
parent: Prompt Engineering
nav_order: 2
---

# Skills & Plugin System

Claude Code's extensibility comes from two complementary systems: **Skills** (prompt-based capabilities) and **Plugins** (full extension packages). Understanding these reveals how Claude Code can be customized far beyond its defaults.

## Skills System

Skills are prompt-based extensions that inject specialized behavior into Claude Code. The `skills/` subsystem has 20 modules.

### Bundled Skills

| Skill | File | Purpose |
|:------|:-----|:--------|
| `batch` | `skills/bundled/batch.ts` | Batch processing operations |
| `clawApi` | `skills/bundled/clawApi.ts` | Claude API usage guidance |
| `clawApiContent` | `skills/bundled/clawApiContent.ts` | API content formatting |
| `clawInChrome` | `skills/bundled/clawInChrome.ts` | Chrome browser integration |
| `debug` | `skills/bundled/debug.ts` | Debugging workflows |
| `keybindings` | `skills/bundled/keybindings.ts` | Keybinding customization |
| `loop` | `skills/bundled/loop.ts` | Recurring task execution |
| `loremIpsum` | `skills/bundled/loremIpsum.ts` | Placeholder content generation |
| `remember` | `skills/bundled/remember.ts` | Memory management |
| `scheduleRemoteAgents` | `skills/bundled/scheduleRemoteAgents.ts` | Remote agent scheduling |
| `simplify` | `skills/bundled/simplify.ts` | Code simplification review |
| `skillify` | `skills/bundled/skillify.ts` | Turn prompts into skills |
| `stuck` | `skills/bundled/stuck.ts` | Unstuck assistance |
| `updateConfig` | `skills/bundled/updateConfig.ts` | Configuration management |
| `verify` | `skills/bundled/verify.ts` | Verification workflows |
| `verifyContent` | `skills/bundled/verifyContent.ts` | Content verification |

### Skill Loading (`skills/loadSkillsDir.ts`)

Skills are loaded from:
1. Bundled skills (shipped with Claude Code)
2. Plugin-provided skills
3. MCP-provided skills (`skills/mcpSkillBuilders.ts`)

### MCP Skill Builders (`skills/mcpSkillBuilders.ts`)

This module bridges MCP tools with the skill system - allowing MCP servers to provide skills that appear as native Claude Code capabilities.

{: .insight }
> The `skillify` skill is meta - it's a skill for creating new skills. It converts a natural language workflow description into a structured skill definition. This is how Claude Code enables user-created skills.

## Plugin System

Plugins are more comprehensive than skills. The `plugins/` subsystem manages:

| Module | Purpose |
|:-------|:--------|
| `builtinPlugins.ts` | Default plugins shipped with Claude Code |
| `bundled/index.ts` | Plugin bundle entry point |

### Plugin Capabilities

A plugin can provide:
- **Skills** - Prompt-based behavior extensions
- **Agents** - Specialized sub-agent definitions
- **Commands** - Slash commands (like `/commit`)
- **Hooks** - Event-driven automation (PreToolUse, PostToolUse, Stop)
- **MCP Servers** - Additional tool integrations

### Plugin Discovery

Plugins are discovered from:
1. Built-in plugins (bundled with Claude Code)
2. User-installed plugins (via package managers)
3. Project-local plugins (in project directories)

## Services That Power Skills

### MagicDocs (`services/MagicDocs/`)

| Module | Purpose |
|:-------|:--------|
| `magicDocs.ts` | Auto-generates documentation from code |
| `prompts.ts` | Prompts for documentation generation |

MagicDocs can analyze your codebase and generate contextual documentation - this powers features like auto-CLAUDE.md generation.

### Agent Summary (`services/AgentSummary/`)

The `agentSummary.ts` module creates summaries of sub-agent work, condensing multi-step agent operations into digestible results.

### Prompt Suggestion (`services/PromptSuggestion/`)

| Module | Purpose |
|:-------|:--------|
| `promptSuggestion.ts` | Suggests follow-up actions |
| `speculation.ts` | Speculative prompt completion |

This service predicts what you might want to do next and suggests it proactively.

{: .tip }
> You can create your own skills by using the `/skillify` command or by placing skill files in the appropriate plugin directory. Skills are just markdown files with YAML frontmatter - no code required.
