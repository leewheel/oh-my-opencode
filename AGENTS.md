# PROJECT KNOWLEDGE BASE

**Generated:** 2026-01-26T14:50:00+09:00
**Commit:** 9d66b807
**Branch:** dev

---

## **IMPORTANT: PULL REQUEST TARGET BRANCH**

> **ALL PULL REQUESTS MUST TARGET THE `dev` BRANCH.**
>
> **DO NOT CREATE PULL REQUESTS TARGETING `master` BRANCH.**
>
> PRs to `master` will be automatically rejected by CI.

---

## OVERVIEW

OpenCode plugin: multi-model agent orchestration (Claude Opus 4.5, GPT-5.2, Gemini 3 Flash, Grok Code). 32 lifecycle hooks, 20+ tools (LSP, AST-Grep, delegation), 10 specialized agents, full Claude Code compatibility. "oh-my-zsh" for OpenCode.

## STRUCTURE

```
oh-my-opencode/
├── src/
│   ├── agents/        # 10 AI agents - see src/agents/AGENTS.md
│   ├── hooks/         # 32 lifecycle hooks - see src/hooks/AGENTS.md
│   ├── tools/         # 20+ tools - see src/tools/AGENTS.md
│   ├── features/      # Background agents, Claude Code compat - see src/features/AGENTS.md
│   ├── shared/        # 55 cross-cutting utilities - see src/shared/AGENTS.md
│   ├── cli/           # CLI installer, doctor - see src/cli/AGENTS.md
│   ├── mcp/           # Built-in MCPs - see src/mcp/AGENTS.md
│   ├── config/        # Zod schema, TypeScript types
│   └── index.ts       # Main plugin entry (672 lines)
├── script/            # build-schema.ts, build-binaries.ts
├── packages/          # 7 platform-specific binaries
└── dist/              # Build output (ESM + .d.ts)
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Add agent | `src/agents/` | Create .ts with factory, add to `builtinAgents` in index.ts |
| Add hook | `src/hooks/` | Create dir with `createXXXHook()`, register in index.ts |
| Add tool | `src/tools/` | Dir with index/types/constants/tools.ts, add to `builtinTools` |
| Add MCP | `src/mcp/` | Create config, add to index.ts |
| Add skill | `src/features/builtin-skills/` | Create dir with SKILL.md |
| LSP behavior | `src/tools/lsp/` | client.ts (connection), tools.ts (handlers) |
| AST-Grep | `src/tools/ast-grep/` | napi.ts for @ast-grep/napi binding |
| Config schema | `src/config/schema.ts` | Zod schema, run `bun run build:schema` after changes |
| Claude Code compat | `src/features/claude-code-*-loader/` | Command, skill, agent, mcp loaders |
| Background agents | `src/features/background-agent/` | manager.ts (1165 lines) for task lifecycle |
| Skill MCP | `src/features/skill-mcp-manager/` | MCP servers embedded in skills |
| CLI installer | `src/cli/install.ts` | Interactive TUI (462 lines) |
| Doctor checks | `src/cli/doctor/checks/` | 14 health checks across 6 categories |
| Orchestrator | `src/hooks/atlas/` | Main orchestration hook (771 lines) |

## TDD (Test-Driven Development)

**MANDATORY for new features and bug fixes.** Follow RED-GREEN-REFACTOR:

| Phase | Action | Verification |
|-------|--------|--------------|
| **RED** | Write test describing expected behavior | `bun test` → FAIL (expected) |
| **GREEN** | Implement minimum code to pass | `bun test` → PASS |
| **REFACTOR** | Improve code quality, remove duplication | `bun test` → PASS (must stay green) |

**Rules:**
- NEVER write implementation before test
- NEVER delete failing tests - fix the code
- Test file: `*.test.ts` alongside source (100 test files)
- BDD comments: `//#given`, `//#when`, `//#then`

## CONVENTIONS

- **Package manager**: Bun only (`bun run`, `bun build`, `bunx`)
- **Types**: bun-types (not @types/node)
- **Build**: `bun build` (ESM) + `tsc --emitDeclarationOnly`
- **Exports**: Barrel pattern via index.ts
- **Naming**: kebab-case dirs, `createXXXHook`/`createXXXTool` factories
- **Testing**: BDD comments, 100 test files
- **Temperature**: 0.1 for code agents, max 0.3

## ANTI-PATTERNS (THIS PROJECT)

| Category | Forbidden |
|----------|-----------|
| **Package Manager** | npm, yarn - use Bun exclusively |
| **Types** | @types/node - use bun-types |
| **File Ops** | mkdir/touch/rm/cp/mv in code - agents use bash tool |
| **Publishing** | Direct `bun publish` - use GitHub Actions workflow_dispatch |
| **Versioning** | Local version bump - managed by CI |
| **Date References** | Year 2024 - use current year |
| **Type Safety** | `as any`, `@ts-ignore`, `@ts-expect-error` |
| **Error Handling** | Empty catch blocks `catch(e) {}` |
| **Testing** | Deleting failing tests to "pass" |
| **Agent Calls** | Sequential agent calls - use `delegate_task` for parallel |
| **Tool Access** | Broad tool access - prefer explicit `include` |
| **Hook Logic** | Heavy PreToolUse computation - slows every tool call |
| **Commits** | Giant commits (3+ files = 2+ commits), separate test from impl |
| **Temperature** | >0.3 for code agents |
| **Trust** | Trust agent self-reports - ALWAYS verify independently |

## UNIQUE STYLES

- **Platform**: Union type `"darwin" | "linux" | "win32" | "unsupported"`
- **Optional props**: Extensive `?` for optional interface properties
- **Flexible objects**: `Record<string, unknown>` for dynamic configs
- **Agent tools**: `tools: { include: [...] }` or `tools: { exclude: [...] }`
- **Hook naming**: `createXXXHook` function convention
- **Factory pattern**: Components created via `createXXX()` functions

## AGENT MODELS

| Agent | Default Model | Purpose |
|-------|---------------|---------|
| Sisyphus | anthropic/claude-opus-4-5 | Primary orchestrator with extended thinking |
| oracle | openai/gpt-5.2 | Read-only consultation, high-IQ debugging |
| librarian | opencode/glm-4.7-free | Multi-repo analysis, docs, GitHub search |
| explore | opencode/grok-code | Fast codebase exploration (contextual grep) |
| multimodal-looker | google/gemini-3-flash | PDF/image analysis |
| Prometheus (Planner) | anthropic/claude-opus-4-5 | Strategic planning, interview mode |
| Metis (Plan Consultant) | anthropic/claude-sonnet-4-5 | Pre-planning analysis |
| Momus (Plan Reviewer) | anthropic/claude-sonnet-4-5 | Plan validation |

## COMMANDS

```bash
bun run typecheck      # Type check
bun run build          # ESM + declarations + schema
bun run rebuild        # Clean + Build
bun test               # 100 test files
```

## DEPLOYMENT

**GitHub Actions workflow_dispatch only**

1. Never modify package.json version locally
2. Commit & push changes
3. Trigger `publish` workflow: `gh workflow run publish -f bump=patch`

**Critical**: Never `bun publish` directly. Never bump version locally.

## CI PIPELINE

- **ci.yml**: Parallel test/typecheck → build → auto-commit schema on master → rolling `next` draft release
- **publish.yml**: Manual workflow_dispatch → version bump → changelog → 8-package OIDC npm publish → force-push master

## COMPLEXITY HOTSPOTS

| File | Lines | Description |
|------|-------|-------------|
| `src/features/builtin-skills/skills.ts` | 1729 | Skill definitions |
| `src/features/background-agent/manager.ts` | 1377 | Task lifecycle, concurrency |
| `src/agents/prometheus-prompt.ts` | 1196 | Planning agent |
| `src/tools/delegate-task/tools.ts` | 1070 | Category-based delegation |
| `src/hooks/atlas/index.ts` | 752 | Orchestrator hook |
| `src/cli/config-manager.ts` | 664 | JSONC config parsing |
| `src/index.ts` | 672 | Main plugin entry |
| `src/features/builtin-commands/templates/refactor.ts` | 619 | Refactor command template |

## MCP ARCHITECTURE

Three-tier MCP system:
1. **Built-in**: `websearch` (Exa), `context7` (docs), `grep_app` (GitHub search)
2. **Claude Code compatible**: `.mcp.json` files with `${VAR}` expansion
3. **Skill-embedded**: YAML frontmatter in skills (e.g., playwright)

## CONFIG SYSTEM

- **Zod validation**: `src/config/schema.ts`
- **JSONC support**: Comments and trailing commas
- **Multi-level**: Project (`.opencode/`) → User (`~/.config/opencode/`)
- **CLI doctor**: Validates config and reports errors

## NOTES

- **Testing**: Bun native test (`bun test`), BDD-style, 83 test files
- **ClaudeCode**: Requires >= 1.0.150
- **Multi-lang docs**: README.md (EN), README.ko.md (KO), README.ja.md (JA), README.zh-cn.md (ZH-CN)
- **Config**: `~/.config/opencode/oh-my-opencode.json` (user) or `.opencode/oh-my-opencode.json` (project)
- **Trusted deps**: @ast-grep/cli, @ast-grep/napi, @code-yeongyu/comment-checker
- **Claude Code Compat**: Full compatibility layer for settings.json hooks, commands, skills, agents, MCPs
- **Flaky tests**: 2 known flaky tests (ralph-loop CI timeout, session-state parallel pollution)
