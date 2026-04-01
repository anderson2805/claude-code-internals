---
title: Home
layout: home
nav_order: 1
---

# Claude Code Internals

A technical deep-dive into how Claude Code actually works under the hood, based on source code analysis following the March 31, 2026 source map leak (59.8 MB `.map` file accidentally included in npm package v2.1.88, containing ~512,000 lines of TypeScript across 1,900+ files). This site is an educational resource for Claude Code power users who want to understand the system they're working with and use it more effectively.

{: .warning }
> This is an independent educational resource based on publicly available analysis. It is not affiliated with or endorsed by Anthropic.

## What You'll Learn

This guide covers Claude Code's internal architecture and engineering decisions - the kind of knowledge that helps you write better prompts, structure your projects for optimal AI assistance, and understand why Claude Code behaves the way it does.

### [Architecture Overview](docs/architecture/)
How Claude Code is built: the 35+ subsystem architecture, the agent loop, the React/Ink terminal UI, and how 1,900+ TypeScript source files are organized.

### [Tool System Deep-Dive](docs/tools/)
The 20+ built-in tools, their multi-layer security model, how permissions work, and the tool-per-directory pattern that makes each tool self-contained.

### [Prompt Engineering Techniques](docs/techniques/)
The system prompt strategies, context window management, memory systems, and prompt construction techniques used internally.

### [Practical Tips for Power Users](docs/practical-tips/)
Actionable advice derived from understanding the internals - how to work *with* the system rather than against it.

---

## Key Stats from the Source

| Metric | Count |
|:-------|:------|
| Total TypeScript files | 1,902 |
| Lines of code | ~512,000 |
| Subsystems | 35+ |
| Command entries | 207 |
| Tool entries | 184 |
| Hook modules | 104 |
| Service modules | 130 |
| Utility modules | 564 |
| Skills | 20+ bundled |
| Runtime modes | 6 |
| Security checks (Bash) | 23 |
| Cache-break vectors | 14 |

---

*Last updated: April 2026*
