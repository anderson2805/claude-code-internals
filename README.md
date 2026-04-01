# Claude Code Internals

**[Read the site →](https://anderson2805.github.io/claude-code-internals/)**

A technical deep-dive into Claude Code's architecture and internals, based on analysis of the March 31, 2026 source map leak (~512,000 lines of TypeScript across 1,900+ files). An educational resource for Claude Code power users who want to understand the system and use it more effectively.

> **Disclaimer:** This is an independent educational resource. It is not affiliated with or endorsed by Anthropic.

## What's Covered

### Architecture (5 pages)
- **Agent Loop** — TAOR pattern, 7-stage bootstrap, tool concurrency model, silent model downgrade
- **Memory System** — Three-layer architecture (MEMORY.md → topic files → transcripts), AutoDream consolidation, skeptical memory
- **IDE Bridge** — 31-module bridge system, JWT auth, capacity wake, 6 runtime modes
- **Services Layer** — 130 modules: MagicDocs, PromptSuggestion, analytics, GrowthBook feature flags
- **Unreleased Features** — KAIROS daemon, ULTRAPLAN, Buddy pet system, Agent Teams

### Tool System (2 pages)
- **Tool Inventory** — 20+ built-in tools, tool-per-directory pattern, concurrent vs serialized classification
- **Security Model** — 6-layer security, 23 bash security checks, sed edit parsing, PowerShell CLM, sandbox mode

### Prompt Engineering (4 pages)
- **System Prompt** — Dynamic assembly from `systemPromptSections.ts`, behavioral anchoring, anti-pattern docs
- **Context Management** — 5 compaction strategies, prompt cache economics (14 cache-break vectors), file read limits
- **Skills & Plugins** — 20+ bundled skills, plugin marketplace, MCP skill builders
- **Anti-Distillation** — Fake tool injection, connector-text summarization, undercover mode, frustration detection

### Practical Tips (2 pages)
- Power user tips mapped to specific source code evidence
- 10 advanced workflows (worktrees, cron agents, LSP, parallel sub-agents, hooks)

### Analysis (2 pages)
- **Community Insights** — Synthesis of 17+ sources with full citations
- **Unique Findings** — 20 deeper technical insights missed by most analysis

## Key Stats

| Metric | Count |
|:-------|:------|
| TypeScript files | 1,902 |
| Lines of code | ~512,000 |
| Subsystems | 35+ |
| Commands | 207 |
| Tools | 184 |
| Hooks | 104 |
| Services | 130 |
| Utilities | 564 |
| Runtime modes | 6 |
| Bash security checks | 23 |
| Cache-break vectors | 14 |

## Built With

- [Jekyll](https://jekyllrb.com/) — Static site generator (from the [GitHub Pages examples](https://github.com/collections/github-pages-examples) collection)
- [Just the Docs](https://just-the-docs.com/) — Documentation theme
- [GitHub Pages](https://pages.github.com/) — Hosting

## Sources

Community analysis synthesized from: [Alex Kim](https://alex000kim.com/posts/2026-03-31-claude-code-source-leak/), [VentureBeat](https://venturebeat.com/technology/claude-codes-source-code-appears-to-have-leaked-heres-what-we-know), [Layer5](https://layer5.io/blog/engineering/the-claude-code-source-leak-512000-lines-a-missing-npmignore-and-the-fastest-growing-repo-in-github-history/), [Latent.Space](https://www.latent.space/p/ainews-the-claude-code-source-leak), [Vrungta/Substack](https://vrungta.substack.com/p/claude-code-architecture-reverse), [Superframeworks](https://superframeworks.com/articles/claude-code-source-code-leak), [DEV Community](https://dev.to/kolkov/we-reverse-engineered-12-versions-of-claude-code-then-it-leaked-its-own-source-code-pij), [ClaudeFast](https://claudefa.st/blog/guide/mechanics/claude-code-source-leak), [Sabrina.dev](https://www.sabrina.dev/p/reverse-engineering-claude-code-using), and others. Full source list on the [Community Insights](https://anderson2805.github.io/claude-code-internals/docs/community-insights.html) page.
