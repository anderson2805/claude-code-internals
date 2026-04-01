---
title: Anti-Distillation & Undercover
parent: Prompt Engineering
nav_order: 4
---

# Anti-Distillation & Undercover Mode

Two of the most surprising findings from the source code reveal Anthropic's defensive strategies against competitive threats and their approach to public repository work.

## Anti-Distillation Mechanisms

"Distillation" is when a competitor records API traffic from a product to train their own models on the outputs. Claude Code implements two countermeasures:

### 1. Fake Tool Injection

When the `ANTI_DISTILLATION_CC` flag is enabled:
- API requests include `anti_distillation: ['fake_tools']`
- The server injects **decoy tool definitions** into system prompts
- If a competitor records this traffic, their training data is poisoned with fake capabilities

### 2. Connector-Text Summarization

- Server-side summarization of assistant reasoning between tool calls
- Returned with **cryptographic signatures** for reconstruction
- Hides full reasoning chains from traffic recordings
- The genuine client can reconstruct the full chain; an eavesdropper cannot

Both mechanisms are heavily feature-flagged and gated by user type.

{: .insight }
> These mechanisms reveal that Anthropic considers API traffic recording by competitors a real threat. The fake tool injection is particularly clever -- it doesn't just hide information, it actively introduces noise that would degrade a competitor's training.

## Undercover Mode

The `undercover.ts` module implements a mode specifically for when Claude Code operates in public or open-source repositories.

The system prompt explicitly states:

> *"You are operating UNDERCOVER in a PUBLIC/OPEN-SOURCE repository. Your commit messages, PR titles, and PR bodies MUST NOT contain ANY Anthropic-internal information. Do not blow your cover."*

### What This Means

When working on public repos, Claude Code:
- Strips all traces of Anthropic internals from outputs
- Avoids mentioning internal tool names, codenames, or processes
- Generates generic commit messages and PR descriptions
- Does not reference internal documentation or systems

### Internal Codenames (Hex-Encoded)

The codebase uses codenames that were hex-encoded to bypass internal scanning:

| Codename | Refers To |
|:---------|:----------|
| **Tengu** | Claude Code project |
| **Capybara** | Claude 4.6 variant |
| **Fennec** | Opus 4.6 |
| **Numbat** | Unreleased model (in testing) |

{: .tip }
> If you notice Claude Code behaving slightly differently in public vs. private repos, undercover mode may be active. It's designed to prevent information leakage, not to limit functionality.

## Binary Attestation

API requests contain placeholder strings (`cch=d89d6`) that are replaced with computed hashes by Bun's native Zig HTTP stack before transmission. This cryptographically proves the request came from a genuine Claude Code binary, rather than a modified client or API proxy.

Analysts noted a vulnerability: if conversation content happens to mention the sentinel strings, the attestation can be corrupted.

## Frustration Detection

A regex in `userPromptKeywords.ts` monitors user sentiment by matching patterns like "wtf," "ffs," "horrible," "this sucks." While the exact behavioral response isn't fully documented, it presumably adjusts Claude Code's tone, approach, or level of caution in response to user frustration.

{: .insight }
> This is a rare example of sentiment-aware behavior in a coding tool. Most AI assistants treat all interactions uniformly regardless of user emotional state.
