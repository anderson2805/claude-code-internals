---
title: Key Patterns for Developers
layout: default
nav_order: 7
---

# Key Patterns for Developers

These are the most impactful patterns from Claude Code's internals that directly affect how effective you are as a developer using it. Each pattern is grounded in source code analysis and explains both *what* happens and *what you should do about it*.

---

## 1. The Prompt Cache Shapes Everything You Do

Claude Code's prompt cache system isn't just an optimization -- it's a core architectural constraint with **14 tracked cache-break vectors**. Understanding it directly impacts your costs and response speed.

**How the cache works:**
- The system prompt (including CLAUDE.md content) is sent with every API call
- If the prompt prefix is identical to the last call, Anthropic serves it from cache -- **90% cheaper** and faster
- Anything that changes the prefix invalidates the cache and forces a full re-read

**What breaks the cache:**
- Changing permissions mid-conversation (approving new tool categories)
- Toggling modes frequently (plan mode in/out)
- Modifying CLAUDE.md during a session

**What this means for you:**
- **Set permissions up front.** Approve tool categories at the start of a session, not one at a time. Each new permission approval can shift the system prompt.
- **Keep CLAUDE.md stable during sessions.** Edit it between sessions, not during. It's reinserted every turn (up to 40,000 chars) and stays cached only if unchanged.
- **Sub-agents are cheap.** They share cache prefixes with the parent, making parallel agent spawning "basically free" in token cost.
- **Long, focused sessions beat many short ones.** Cache hits accumulate over a session -- restarting means rebuilding the cache from scratch.

---

## 2. CLAUDE.md Is Your Highest-Leverage File

CLAUDE.md files occupy a privileged position in the instruction hierarchy:

1. Loaded into **every** system prompt
2. **Never compressed** during conversation (unlike your earlier messages)
3. Supports **three scopes**: `~/.claude/CLAUDE.md` (global), project root, and subdirectories
4. Can hold up to **40,000 characters**

**High-impact things to put in CLAUDE.md:**
- Project coding conventions (naming, patterns, formatting)
- Test commands and how to run them
- Architecture decisions ("we use X for Y, not Z")
- Common pitfalls specific to your codebase
- Preferred libraries and why

**What NOT to put in CLAUDE.md:**
- Things that change frequently (sprint goals, current bugs)
- Anything easily derived from the code itself
- Verbose documentation -- be concise, every character costs tokens

**Directory-scoped CLAUDE.md example:**
```
# frontend/CLAUDE.md
- Use React Query for all data fetching, never raw useEffect
- Component files use PascalCase, hooks use camelCase with "use" prefix
- All new components need a Storybook story
```

This scoped file only loads when Claude Code is working in the `frontend/` directory, keeping the prompt lean elsewhere.

---

## 3. Context Window Is Finite -- Manage It Deliberately

Claude Code auto-compresses older conversation turns as you approach context limits. Understanding the compression mechanics helps you avoid the "Claude forgot what I said" problem.

**What gets compressed:** Earlier conversation turns and tool results from completed operations.

**What never gets compressed:** System prompt, CLAUDE.md content, current turn context, active tool definitions.

**Practical patterns:**

| Situation | What To Do |
|:----------|:-----------|
| Conversation over ~20 turns | Re-state critical constraints explicitly |
| Working with large files | Use targeted reads: "read lines 50-100" not "read the file" |
| Multi-step refactoring | Summarize the plan periodically, don't rely on turn 1 |
| Exploring a new codebase | Let Claude Code spawn explore sub-agents (keeps main context clean) |
| Context feels degraded | Use `/compact` to manually trigger compaction with a fresh summary |

**The sub-agent isolation pattern:** When Claude Code spawns a sub-agent for research, that agent gets its own context window. The results are summarized before returning. This means exploration doesn't eat your main conversation's context budget.

---

## 4. Everything Is a Tool -- Use the Right One

Claude Code implements nearly everything as a tool the model invokes. Understanding this dispatch model helps you prompt more effectively.

**Key tools and when to nudge toward them:**

| Goal | Tool | Why It Matters |
|:-----|:-----|:---------------|
| Architecture planning | `EnterPlanMode` | Disables code edits, focuses on design |
| Risky changes | `EnterWorktree` | Isolated git worktree, easy to discard |
| Parallel research | `Agent` (multiple) | Independent context windows, concurrent |
| Track complex work | `TaskCreate` / `TaskUpdate` | Visible progress tracking |
| Recurring automation | `CronCreate` | Scheduled autonomous operations |
| Manual compaction | `/compact` | Structured context summary when conversation gets long |

**The "plan before code" pattern:** For non-trivial tasks, explicitly ask Claude Code to plan first. Plan mode is a state machine -- the model must call a tool to enter and exit it, preventing accidental drift into implementation before the plan is approved.

---

## 5. The Edit-Before-Read Enforcement

Claude Code's `FileEditTool` will **error** if you ask it to edit a file it hasn't read in the current session. This is enforced in code, not a suggestion.

**Save round trips by combining reads and edits:**

| Instead of | Do this |
|:-----------|:--------|
| "Change the return type in utils.ts" | "Read utils.ts and change the return type of functionX to string" |
| "Fix the bug in handler.ts" | "Read handler.ts lines 40-80, the bug is in the error handling" |
| "Update the config" | "Read config.ts -- change the timeout from 30 to 60" |

Giving line numbers or function names alongside the edit request helps Claude Code read only what's needed, preserving context budget.

---

## 6. Security Layers Are Working When They Prompt You

Claude Code has **6 layers of security** including 23 Bash command checks. When you get a permission prompt, the system has flagged a real concern.

**Patterns that trigger prompts (and why):**

| Pattern | Why It's Flagged |
|:--------|:-----------------|
| `sed -i` in Bash | It's a file write operation via shell, bypassing FileEditTool |
| `curl \| bash` | Piping remote code to shell execution |
| Commands touching `.env` files | Potential credential exposure |
| `rm -rf` or `git reset --hard` | Destructive and hard to reverse |
| Network requests to non-whitelisted URLs | Data exfiltration risk |

**Trust mode matters:** Plugins, skills, MCP servers, and session hooks **only initialize in trusted mode** (verified at startup stage 5). If you clone an untrusted repo with a malicious CLAUDE.md or plugin config, the trust gate prevents it from executing automatically.

---

## 7. Parallel Sub-Agents for Speed

Claude Code can spawn multiple sub-agents simultaneously, each with their own context window. This is one of the most underused features for complex tasks.

**When to use parallel agents:**

```
"Research these three things in parallel:
1. How authentication works in this codebase
2. Find all API endpoints and their handlers  
3. Check test coverage for the payment module"
```

Each agent explores independently and returns a summary. The cost is minimal because sub-agents share cache prefixes with the parent.

**Agent types to know:**

| Type | Best For |
|:-----|:---------|
| `Explore` | Codebase research, finding files, understanding patterns |
| `general-purpose` | Multi-step tasks, running commands, writing code |
| `Plan` | Architecture design, implementation planning |
| `code-reviewer` | Reviewing completed work against a plan |

---

## 8. Hooks Turn Claude Code Into an Automated Pipeline

The hook system (104 modules) lets you attach shell commands to Claude Code events:

| Event | Practical Use |
|:------|:-------------|
| `PreToolUse` | Validate or block tool calls before execution |
| `PostToolUse` | Auto-format after writes, auto-lint after edits |
| `Stop` | Run tests when Claude Code finishes, send notifications |

**Example: Auto-format on every file write**
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "command": "prettier --write $CLAUDE_FILE_PATH"
      }
    ]
  }
}
```

This eliminates the "now format it" follow-up and ensures consistent code style without wasting conversation turns.

---

## 9. Memory System Has a Hierarchy -- Use It

Claude Code's memory persists across sessions through a file-based system. The four memory types serve different purposes:

| Type | What to Store | Lifespan |
|:-----|:-------------|:---------|
| **user** | Your role, expertise, preferences | Long-term |
| **feedback** | Corrections and confirmed approaches | Long-term |
| **project** | Sprint context, deadlines, who's doing what | Medium-term |
| **reference** | Links to external dashboards, ticket systems, docs | Long-term |

**The AutoDream system** consolidates memories during idle periods (24+ hours between sessions, 5+ completed sessions). It runs as a read-only sub-agent that scans recent sessions, extracts patterns, and prunes memories to stay under 200 lines / 25KB.

**What this means:** You don't need to manually manage all memories. But explicitly telling Claude Code "remember that we use PostgreSQL for all new services" creates a `feedback` memory that persists across every future session.

---

## 10. Useful Commands Most Developers Don't Know

Beyond the well-known slash commands, these are particularly valuable for development workflows:

| Command | What It Does | When to Use |
|:--------|:-------------|:------------|
| `/compact` | Manually triggers context compaction with optional focus instructions | When responses get confused in long sessions |
| `/effort` | Controls reasoning effort level (low/medium/high/max/auto) | Quick tasks don't need deep reasoning; complex tasks benefit from max |
| `/doctor` | Runs diagnostic self-checks on installation and settings | When Claude Code behaves unexpectedly |

---

## 11. Structure Projects for Optimal AI Assistance

Several architectural details reveal how Claude Code discovers and understands your project:

**Project onboarding:** The first time Claude Code encounters a project, it runs an onboarding scan (`projectOnboardingState.ts`). This analyzes project structure and creates initial context. A clean, conventional project structure makes this faster and more accurate.

**File discovery pattern:** Claude Code uses Glob for file finding and Grep for content search -- never scanning entire directories at once. This means:
- Standard file naming conventions help (Claude Code knows to look for `*.test.ts`, `*.spec.js`, etc.)
- Deeply nested or unconventionally named files may be missed on first pass
- Explicit paths in CLAUDE.md ("tests are in `src/__tests__/`") eliminate guesswork

**Git integration:** A centralized `gitOperationTracking.ts` module tracks all git operations across all tools, enabling conflict detection and rollback. Claude Code treats git state as ground truth -- keep your working tree clean for best results.

---

## 12. The Co-Evolution Principle

Perhaps the most important mental model: Claude Code is designed to **shrink** as models improve. The development philosophy is "delete code on model upgrade."

**What this means for you:**
- Features that feel like scaffolding (explicit plan steps, detailed tool instructions) are temporary
- The system adapts to model capabilities -- what works today may become unnecessary tomorrow
- Stay up to date with Claude Code versions; each update may simplify workflows
- If a pattern that used to work stops working, it may be because the model no longer needs the guardrail
