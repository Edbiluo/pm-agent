# PM-Agent Skill for Claude Code

> **用贵模型当产品经理做决策，便宜模型当码农写代码，通过模式库积累经验实现越用越省。**

A multi-agent architecture skill for [Claude Code](https://claude.com/claude-code) that separates decision-making (expensive model) from code execution (cheap model), reducing token costs without sacrificing code quality.

## Installation

```bash
npx skills add Edbiluo/pm-agent
```

Then type `/pm-agent` in any Claude Code session to activate.

## How It Works

```
User Requirement
  |
PM Agent (Sonnet / Opus)
  |-- Read project-structure.md (global context)
  |-- Read PATTERNS.md (pattern matching)
  |     |-- Hit -> Skip exploration, implement directly (cheapest)
  |     '-- Miss |
  |-- Phase 1: Dispatch Coding Agent to explore -> returns report
  |-- PM reviews report, picks approach, adds constraints
  |-- Phase 2: Dispatch Coding Agent to implement (with Phase 1 context)
  '-- Post: New operation type -> write Pattern for future reuse
```

### Role Separation

| Role | Model | Strength | Responsibility |
|------|-------|----------|----------------|
| PM Agent | Sonnet / Opus (expensive) | Reasoning, judgment | Understand intent, review reports, pick approach, add constraints |
| Coding Agent | Haiku (cheap) | Mechanical execution | Read code, search, write code, maintain docs |

**PM never reads source code. Coding never makes architecture decisions.**

### Two-Phase Dispatch

**Complex tasks (default):**

1. **Phase 1 - Explore**: PM dispatches Coding Agent to read code and return a structured report (files involved, existing implementation summary, multiple approach options, risks)
2. **PM Decision**: PM reviews the report, selects approach, scopes down changes, identifies risks
3. **Phase 2 - Implement**: PM dispatches Coding Agent with Phase 1 context bundled in, so Coding doesn't re-read files

**Simple tasks** (small, clear scope): Single-phase dispatch directly.

### Pattern Library

The skill accumulates reusable patterns over time:

| Scenario | Action |
|----------|--------|
| Pattern hit, validated within 30 days | Skip Phase 1, go straight to implementation |
| Pattern hit, older than 30 days | Quick validation check, then implement |
| No pattern match | Normal two-phase flow |
| After implementation | Auto-create or update pattern for next time |

Pattern files contain: trigger keywords, key files, implementation checklist, common variants, gotchas, and reference implementations.

## 5 Ways It Saves Tokens

| # | Method | How |
|---|--------|-----|
| 1 | Model tiering | Decisions use expensive model (few tokens), execution uses cheap model (many tokens but cheap) |
| 2 | PM never reads source | PM only reads 3 structured docs, no expensive-model tokens wasted on source code |
| 3 | Pattern reuse | Matched patterns skip entire Phase 1 exploration |
| 4 | Context passing | Phase 2 carries Phase 1 results, Coding doesn't re-read files |
| 5 | Standard accumulation | User corrections auto-saved as coding standards, same mistakes never repeat |

## Quality Assurance

- PM has global project view via `project-structure.md`
- Coding standards enforced on every dispatch
- `project-structure.md` force-updated after every implementation
- User feedback auto-captured as coding standard entries

## What Gets Generated on First Use

When you activate `/pm-agent` in a project for the first time, the skill will:

1. Ask you to select models for PM and Coding roles
2. Scan the project and generate `.claude/project-structure.md`
3. Append architecture config to `CLAUDE.md`
4. Create `.claude/patterns/` directory (grows with usage)

These files are **project-level** and live in the project's `.claude/` directory.

## Project Files

| File | Purpose |
|------|---------|
| `.claude/project-structure.md` | Full project structure doc - PM's only source of codebase knowledge |
| `.claude/coding-standards.md` | Coding standards, filtered and attached to every dispatch |
| `.claude/pm-config.json` | Runtime config (model selection, token display toggle) |
| `.claude/patterns/PATTERNS.md` | Pattern library index |
| `.claude/patterns/{name}.md` | Individual pattern files |

## CLI Commands

| Command | Description |
|---------|-------------|
| `npx skills add Edbiluo/pm-agent` | Install |
| `npx skills update` | Update to latest version |
| `npx skills remove pm-agent` | Uninstall |
| `npx skills list` | List installed skills |

## License

MIT
