---
title: IDE Bridge System
layout: default
parent: Architecture Overview
nav_order: 3
---

# IDE Bridge System

The bridge subsystem is one of the largest in Claude Code (31 modules), handling the connection between Claude Code and IDE extensions (VS Code, JetBrains). Understanding this helps explain behavior differences between CLI and IDE usage.

## Bridge Architecture

```
┌──────────────┐         ┌──────────────┐
│  IDE Extension│◄───────►│  Bridge      │
│  (VS Code /   │  WebSocket│  Layer       │
│   JetBrains)  │         │  (31 modules)│
└──────────────┘         └──────┬───────┘
                                │
                         ┌──────▼───────┐
                         │  Claude Code  │
                         │  Core         │
                         └──────────────┘
```

## Key Bridge Modules

| Module | Purpose |
|:-------|:--------|
| `bridgeMain.ts` | Entry point for bridge mode |
| `bridgeConfig.ts` | Bridge connection configuration |
| `bridgeMessaging.ts` | Message protocol between IDE and CC |
| `bridgePermissionCallbacks.ts` | Permission handling in IDE context |
| `bridgeApi.ts` | API surface exposed to IDE |
| `bridgeUI.ts` | UI rendering adaptations for IDE |
| `jwtUtils.ts` | JWT authentication for bridge sessions |
| `capacityWake.ts` | Waking bridge on capacity availability |
| `replBridge.ts` | REPL adaptation for bridge mode |
| `remoteBridgeCore.ts` | Core remote bridge functionality |

## Session Management

| Module | Purpose |
|:-------|:--------|
| `createSession.ts` | Creates new bridge sessions |
| `codeSessionApi.ts` | API for code-specific sessions |
| `pollConfig.ts` | Polling configuration for bridge health |
| `pollConfigDefaults.ts` | Default polling intervals |
| `flushGate.ts` | Controls message flushing timing |

## Inbound Communication

| Module | Purpose |
|:-------|:--------|
| `inboundMessages.ts` | Handles messages from IDE to CC |
| `inboundAttachments.ts` | File attachments from IDE |

{: .insight }
> The bridge uses JWT-based authentication (`jwtUtils.ts`) for secure communication between the IDE extension and Claude Code process. This means each bridge session is authenticated, preventing unauthorized access to your Claude Code instance.

## How Bridge Mode Differs from CLI

When running in bridge mode (through an IDE):
1. **Permission handling** routes through `bridgePermissionCallbacks.ts` instead of the terminal prompt
2. **UI rendering** is adapted via `bridgeUI.ts` for the IDE's webview panel
3. **Message protocol** uses WebSocket instead of stdin/stdout
4. **Session persistence** is managed differently via `codeSessionApi.ts`

## Capacity Wake

The `capacityWake.ts` module implements an interesting pattern: when the bridge detects available capacity (the model isn't busy), it can proactively wake up to handle queued requests. This enables smoother IDE integration where requests might queue while Claude is processing.

## Remote Sessions

The `remote/` subsystem extends the bridge concept further:

| Module | Purpose |
|:-------|:--------|
| `RemoteSessionManager.ts` | Manages remote Claude Code sessions |
| `SessionsWebSocket.ts` | WebSocket connection for remote sessions |
| `remotePermissionBridge.ts` | Permission handling for remote |
| `sdkMessageAdapter.ts` | Adapts messages for SDK format |

{: .tip }
> If you experience different behavior between CLI and IDE usage, it's likely due to the bridge layer's permission handling or UI adaptations - not a different model or different capabilities.
