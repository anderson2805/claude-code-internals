---
title: Practical Tips
layout: default
nav_order: 5
has_children: true
---

# Practical Tips for Power Users

These tips are derived directly from understanding Claude Code's internals. Each tip maps back to a specific architectural feature or design decision.

## Quick Reference

| Tip | Why It Works |
|:----|:-------------|
| [Read the file before asking for edits](#always-read-before-editing) | FileEditTool enforces this - saves a round trip |
| [Use specific line ranges for large files](#use-targeted-file-reads) | FileReadTool has a 10K token limit per read |
| [Re-state critical context in long sessions](#re-state-critical-context) | Compression removes early conversation turns |
| [Write good CLAUDE.md files](#invest-in-claudemd) | Injected into every system prompt, never compressed |
| [Use sub-agents for exploration](#leverage-sub-agents) | Keeps main context clean, isolated exploration |
| [Trust the permission system](#trust-the-security-system) | Multi-layer security catches real issues |

---

## Always Read Before Editing
{: #always-read-before-editing }

The FileEditTool will **error** if you ask Claude Code to edit a file it hasn't read in the current session. This is enforced in code, not just a suggestion.

**What this means for you:**
- If you say "change function X in file.ts", Claude Code must first read the file
- You can save a turn by saying "read file.ts and change function X to..."
- If an edit fails, it might be because the file wasn't read recently

**Source:** `tools/FileEditTool/FileEditTool.ts`

---

## Use Targeted File Reads
{: #use-targeted-file-reads }

The FileReadTool has limits:
- Default: 2,000 lines from the start
- Maximum: ~10,000 tokens per read
- Large files trigger warnings

**What this means for you:**
- Say "read lines 50-100 of large-file.ts" instead of "read large-file.ts"
- For files over 2,000 lines, always specify the section you need
- Each unnecessary file read wastes context budget

**Source:** `tools/FileReadTool/limits.ts`

---

## Re-State Critical Context
{: #re-state-critical-context }

The context compression system summarizes older turns to fit within the context window. Important details from early in the conversation may be lost.

**What this means for you:**
- After 20+ turns, re-state important constraints
- If Claude Code seems to "forget" an earlier decision, it was likely compressed
- For multi-step tasks, re-summarize the plan periodically

**Source:** `QueryEngine.ts`, system prompt compression

---

## Invest in CLAUDE.md
{: #invest-in-claudemd }

CLAUDE.md files are:
1. Loaded into every system prompt
2. Given high priority in the instruction hierarchy
3. Never compressed during conversation

**What this means for you:**
- Put project conventions, coding standards, and preferences in CLAUDE.md
- This is the most reliable way to persist instructions across sessions
- Use project-level and directory-level CLAUDE.md files for scoped instructions

**Source:** `constants/systemPromptSections.ts`, system prompt priority hierarchy

---

## Leverage Sub-Agents
{: #leverage-sub-agents }

Sub-agents run with isolated context, meaning:
- Exploration doesn't pollute your main conversation
- Multiple agents can run in parallel
- Each agent gets a fresh context window

**What this means for you:**
- For broad codebase research, let Claude Code spawn an explore agent
- For complex tasks, encourage breaking work into parallel sub-agents
- Sub-agent results are summarized before returning to the main context

**Source:** `tools/AgentTool/`, `services/AgentSummary/`

---

## Trust the Security System
{: #trust-the-security-system }

Claude Code has 6 layers of security (see [Security Model](../tools/security.md)). When it asks for permission, it's because a real security check flagged the operation.

**What this means for you:**
- Don't blindly approve everything - read what it's asking to do
- Permission prompts indicate the security system is working correctly
- If a command seems safe but is flagged, it might have subtle risks (like `sed -i` being a write operation)

**Source:** `tools/BashTool/bashSecurity.ts`, permission handlers
