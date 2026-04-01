---
title: Hidden & Surprising Features
layout: default
parent: Prompt Engineering
nav_order: 3
---

# Hidden & Surprising Features

The source code reveals several features and design decisions that aren't obvious from normal usage. These range from quirky to genuinely useful.

## The Companion System ("Buddy")

Claude Code has a built-in companion sprite system in `buddy/`:

| Module | Purpose |
|:-------|:--------|
| `companion.ts` | Companion logic |
| `CompanionSprite.tsx` | React component for sprite rendering |
| `sprites.ts` | Sprite definitions |
| `prompt.ts` | Companion personality prompt |
| `types.ts` | Type definitions |
| `useBuddyNotification.tsx` | Notification integration |

The default companion is **Quartz**, a penguin that appears beside the input box. It has its own personality prompt and can respond when addressed by name. The system prompt explicitly tells Claude Code to "stay out of the way" when the user talks to Quartz.

{: .insight }
> Quartz is a separate "watcher" entity with its own prompt and personality. When you address it by name, the main Claude Code instance is instructed to respond in one line or less.

## Built-in Vim Mode

Claude Code includes a complete vim keybinding emulation in `vim/`:

| Module | Purpose |
|:-------|:--------|
| `motions.ts` | Cursor movement (w, b, e, 0, $, etc.) |
| `operators.ts` | Operations (d, c, y, etc.) |
| `textObjects.ts` | Text objects (iw, aw, i", a(, etc.) |
| `transitions.ts` | Mode transitions (i, a, v, etc.) |
| `types.ts` | Vim state types |

This is a full vim implementation for the input area - not just basic vi keybindings.

## Voice Mode

The `voice/voiceModeEnabled.ts` module reveals voice input support. While currently minimal (1 module), the infrastructure exists for voice-driven coding.

## Terminal Output as Images

Two utility modules convert terminal output to images:
- `utils/ansiToPng.ts` - ANSI terminal output to PNG
- `utils/ansiToSvg.ts` - ANSI terminal output to SVG
- `utils/asciicast.ts` - Terminal session recording

These enable features like sharing terminal output as images in briefs and handoffs.

## Agent Swarms

The `utils/agentSwarmsEnabled.ts` module reveals an experimental "agent swarm" capability - multiple agents working in parallel on related tasks. This goes beyond the standard single sub-agent model.

## The "Stuck" Skill

The bundled `stuck` skill (`skills/bundled/stuck.ts`) is a meta-skill that helps when Claude Code gets stuck in a loop or can't make progress. It's essentially a self-rescue mechanism.

## Project Onboarding

The `projectOnboardingState.ts` module tracks whether a project has been "onboarded" - i.e., whether Claude Code has analyzed the project structure and created initial context. This is why the first interaction in a new project often involves codebase exploration.

## Activity Manager

The `utils/activityManager.ts` module tracks user activity patterns, which may inform:
- When to show notifications
- Session timeout behavior
- Resource allocation decisions

## API Preconnection

`utils/apiPreconnect.ts` establishes an API connection before you even type your first message, reducing perceived latency. This is why the first response often feels fast.

## Argument Substitution

`utils/argumentSubstitution.ts` implements template variable substitution in commands and skills, enabling dynamic content like `${CLAUDE_PLUGIN_ROOT}` in plugin paths.

## The Doctor Screen

`screens/Doctor.tsx` is a diagnostic screen that can check Claude Code's health:
- Configuration validity
- API connectivity
- Plugin status
- Environment issues

{: .tip }
> If Claude Code is behaving unexpectedly, the Doctor screen can diagnose common issues. Access it through the CLI diagnostics.

## The Brief Tool

The `BriefTool` (`tools/BriefTool/`) is designed for creating handoff documents:
- `BriefTool.ts` - Core brief creation
- `attachments.ts` - Attach files and context
- `upload.ts` - Upload briefs for sharing
- `prompt.ts` - Instructions for brief generation

This enables structured handoffs between sessions or team members.

## Analytics and Telemetry

The `services/analytics/` subsystem (7+ modules) reveals a comprehensive analytics system:

| Module | Purpose |
|:-------|:--------|
| `config.ts` | Analytics configuration |
| `datadog.ts` | Datadog integration |
| `growthbook.ts` | Feature flag system (GrowthBook) |
| `firstPartyEventLogger.ts` | Event logging |
| `sink.ts` | Analytics data sink |
| `sinkKillswitch.ts` | Kill switch for analytics |
| `metadata.ts` | Event metadata |

{: .insight }
> The `sinkKillswitch.ts` module provides a kill switch for analytics collection. The `growthbook.ts` integration reveals that feature flags (via GrowthBook) control which features are enabled for which users - explaining why some features roll out gradually.

## Model Migration Notifications

`hooks/notifs/useModelMigrationNotifications.tsx` handles notifications when models are updated or deprecated, ensuring users are aware of changes that might affect their workflows.

## Rate Limit Intelligence

`hooks/notifs/useRateLimitWarningNotification.tsx` provides proactive warnings when approaching rate limits, rather than waiting for failures.
