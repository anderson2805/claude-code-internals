---
title: Unreleased Features
parent: Architecture Overview
nav_order: 5
---

# Unreleased Features Found in Source

The source code contains several features that are in development but not yet publicly available. These reveal Anthropic's roadmap for Claude Code.

## KAIROS: Always-On Background Daemon

Referenced **150+ times** in the codebase, KAIROS is the most substantial unreleased feature:

- **Always-on background daemon** that runs continuously
- Executes autonomous `<tick>` prompts with 15-second blocking budget per cycle
- **Append-only audit logging** for all daemon actions
- **GitHub webhook subscriptions** for real-time repository event handling
- **Cron-scheduled 5-minute refreshes** for monitoring
- Includes a `/dream` skill for nightly memory distillation

### What KAIROS Would Enable

Instead of invoking Claude Code when you need it, KAIROS would make it a persistent background process that:
- Monitors your repository for changes
- Responds to GitHub events (PRs, issues, CI failures)
- Proactively performs maintenance tasks
- Consolidates learning during idle time

{: .insight }
> KAIROS represents a fundamental shift from "tool you invoke" to "agent that's always present." The append-only audit log suggests Anthropic is thinking carefully about accountability for autonomous background actions.

## ULTRAPLAN: Remote Deep Thinking

ULTRAPLAN offloads complex planning to remote instances:

- Runs **Opus 4.6** remotely with **30-minute thinking windows**
- Local clients poll for updates every 3 seconds
- Designed for complex architectural decisions that benefit from extended reasoning time

This would enable Claude Code to take on problems that require much longer thinking than the current interactive response time allows.

## Buddy: Terminal Pet System

The companion system goes far beyond the current Quartz penguin:

- **Tamagotchi-style terminal pet** with persistent state
- **Deterministic gacha system** for obtaining new companions
- **Species rarity tiers** (common, rare, etc.)
- **Shiny variants** of each species
- **Procedurally generated stats** (CHAOS, SNARK)
- Descriptions written by Claude on first hatch
- Full sprite rendering in terminal

### Current Buddy Implementation

The current codebase has the foundation:

| Module | Purpose |
|:-------|:--------|
| `CompanionSprite.tsx` | React component for sprite rendering |
| `companion.ts` | Core companion logic |
| `sprites.ts` | Sprite definitions |
| `prompt.ts` | Companion personality |
| `types.ts` | Type definitions |
| `useBuddyNotification.tsx` | Notification integration |

## AutoDream: Memory Consolidation

The AutoDream system for background memory processing:

- Runs during idle periods with gate requirements:
  - 24 hours since last run
  - 5+ completed sessions since last run
- **Four phases:**
  1. **Orient** -- Scan recent sessions
  2. **Gather** -- Extract signal from recent work
  3. **Consolidate** -- Write to persistent memory
  4. **Prune** -- Keep under 200 lines / 25KB
- Runs as a **read-only forked subagent** to prevent corruption

{: .tip }
> AutoDream is essentially "dreaming" -- processing experiences during downtime to extract and consolidate learning. This is analogous to how biological memory consolidation works during sleep.

## Voice Mode & Computer Use

- **Voice Mode** (`voice/voiceModeEnabled.ts`) -- Voice-driven coding
- **Computer Use** integration -- Screenshot capture, click simulation, keyboard input for GUI interaction

## Agent Teams (Experimental)

Beyond individual sub-agents, the source contains infrastructure for **peer agent coordination**:
- Processes coordinating via shared task lists
- `TeamCreateTool` / `TeamDeleteTool` for managing agent teams
- Distinct from the hub-and-spoke sub-agent model
- True peer-to-peer agent collaboration

## Feature Flag Gating

All unreleased features are behind feature flags managed by **GrowthBook** (`services/analytics/growthbook.ts`). This enables:
- Progressive rollout to user segments
- A/B testing of different implementations
- Quick disable if issues arise
- Different feature sets per subscription tier
