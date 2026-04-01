---
title: File Editing Strategy
layout: default
parent: Tool System
nav_order: 2
---

# File Editing Strategy

How Claude Code edits files is one of its most critical design decisions. Understanding the approach helps you write better edit instructions and avoid common frustration points.

## Three File Modification Tools

Claude Code has three distinct tools for modifying files, each with different use cases:

### 1. FileEditTool (String Replacement)

The primary editing tool uses **exact string matching**, not line numbers or diffs:

```
Parameters:
  - file_path: absolute path
  - old_string: exact text to find
  - new_string: replacement text
  - replace_all: boolean (default false)
```

**Key design decisions:**
- Requires reading the file first (enforced - the tool errors if you haven't read)
- `old_string` must be unique in the file (or use `replace_all`)
- Preserves exact indentation from the file
- No line number dependency (avoids off-by-one errors)

{: .insight }
> The "must read before editing" requirement isn't just a suggestion - it's enforced in the tool code. This prevents blind edits based on assumptions about file content, which is a major source of bugs in AI-generated code.

### 2. FileWriteTool (Full Rewrite)

For creating new files or complete rewrites:

```
Parameters:
  - file_path: absolute path
  - content: full file content
```

**When it's used:**
- Creating new files
- Complete file rewrites where editing would be more complex
- The system prompt explicitly says to prefer FileEditTool over FileWriteTool for existing files

### 3. NotebookEditTool

Specialized for Jupyter notebooks:
- Understands cell structure
- Can edit individual cells
- Preserves notebook metadata

## Why String Matching Over Diffs?

The string-matching approach has specific advantages for AI agents:

1. **No line number drift** - Line numbers change as edits accumulate. String matching is position-independent.
2. **Self-verifying** - If `old_string` isn't found, the edit fails explicitly rather than silently applying to the wrong location.
3. **Context-rich** - The `old_string` provides context about what's being changed, making edits reviewable.
4. **Idempotent-ish** - Running the same edit twice fails (old_string already replaced), preventing double-application.

## Sed Detection

Interestingly, Claude Code also monitors for file edits through the Bash tool:

- `sedEditParser.ts` - Parses sed commands to understand what they'd modify
- `sedValidation.ts` - Validates sed operations against permission rules

This means even if a user tries `sed -i 's/old/new/' file.txt` through the Bash tool, Claude Code understands this is a file modification and applies the same permission checks.

{: .tip }
> **For best results when requesting edits:**
> - Be specific about what to change ("change the return type of function X from Y to Z")
> - If Claude Code's edit fails with "old_string not unique", it means the text appears multiple times - provide more surrounding context
> - For large refactors, prefer multiple small edits over one massive rewrite
