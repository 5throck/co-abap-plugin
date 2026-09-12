# CLAUDE.md

**Claude Code (CLI & Desktop App)** configuration for the vsp/SAP ABAP Harness Engineering project.

> **Doc intent:** This file is Claude Code-specific. Shared project context (build, codebase map, ABAP rules, Harness workflow) lives in [docs/context.md](docs/context.md). Agent roles live in [AGENTS.md](AGENTS.md). Per-session skills live in [skills/abap-dev/SKILL.md](skills/abap-dev/SKILL.md).

---

## Session Start

At the start of every Claude Code session, run this checklist:
*(Ref: `docs/context.md` -> `Initial Context Files`)*

```
0. git config core.hooksPath .githooks         # activate hooks (run once per clone)
1. Read docs/context.md                        # full architecture map, ABAP rules, workflow
2. Read AGENTS.md                              # canonical agent roster
3. Read memory/MEMORY.md                       # recent session history (skip if absent)
4. Read skills/abap-dev/SKILL.md               # SAP development workflows
5. Read skills/post-write-chain/SKILL.md       # mandatory QA chain after any write
```

---

---

## Claude Code: CLI vs Desktop App

Both the CLI and the Desktop App share the same configuration files and MCP server setup. Key differences, especially regarding hook behavior and UI features, are detailed in [docs/tooling-matrix.md](docs/tooling-matrix.md).

> **Hook limitation**: `PostToolUse` hooks configured in `.claude/settings.json` do **not** fire in the Desktop App. After any `WriteSource` / `EditSource`, run the Post-Write Mandatory Chain manually (see [skills/post-write-chain/SKILL.md](skills/post-write-chain/SKILL.md)) and sync via `bun scripts/dev-sync.ts`.

> **Linux developers**: Use CLI only —the Desktop App is not available on Linux.

> **Recommended split**: Use CLI for automated ABAP workflows (hook-driven, multi-agent orchestration). Use Desktop App for visual diff review, PR monitoring, and parallel sessions.

---



## Setup

> **Parent project**: [abap_vibe_coding](https://github.com/5throck/abap_vibe_coding)

1. **Install vsp binary**: Download from [vsp releases](https://github.com/5throck/vsp/releases) and place in your project root.
2. **Configure SAP credentials**: Copy `.env.sample` to `.env` and fill in your SAP connection details.
3. **Enable MCP servers**: Ensure `enableAllProjectMcpServers: true` is set in `.claude/settings.local.json`.
4. **Activate hooks**: Run `git config core.hooksPath .githooks` (once per clone).

### Consumer Project Integration

When this plugin is installed in a consumer project:
- **Marketplace install**: SAP credentials are configured via `userConfig` in `.claude-plugin/plugin.json`
- **Manual install**: Copy `.mcp.json.sample` to your project root and configure manually
- **Hooks**: PostToolUse hooks fire in Claude Code CLI only (not Desktop App)

---

## Claude Code Settings

- `.claude/settings.json` —shared team permissions (committed to repo; note that `.claude/` is a hidden dot-folder and may not show in standard listing tools by default)
- `.claude/settings.local.json` —personal write permissions + git operations (gitignored)
- `.claude/commands/` —slash commands (`/sync`, `/memlog`, `/new-task`, `/triage`, `/transport`, `/post-write`, `/celebrate`, `/project-review`, `/meeting`)

Both files are loaded automatically. `enableAllProjectMcpServers: true` is set in the local file to activate the abap MCP server.


---

## Hooks

A `PostToolUse` hook fires after every `Write` or `Edit` tool call and runs `scripts/sync-md.ts` (cross-platform via Bun). This hook is defined in `.claude/settings.json`.

| Environment | Hook fires? | Notes |
|-------------|:-----------:|-------|
| Claude Code CLI | —| Automatic on every WriteSource/EditSource |
| Claude Code Desktop App | —| Known issue —run Post-Write chain manually |
| Gemini CLI | —| Automated hooks disabled —run Post-Write chain manually |
| Antigravity | —| No hook support in VS Code extension |

`sync-md.ts` updates the `memory/MEMORY.md` index after every edit, ensuring session logs stay discoverable. It accepts optional `$DATE` and `$SUMMARY` arguments (defaults to today's date and `"update"`).


### Desktop App Manual Post-Write Chain

When using Claude Code Desktop App, PostToolUse hooks do not fire. After any `WriteSource` or `EditSource`, run this chain manually:

```
1. bun scripts/sync-md.ts           # update memory index
2. SyntaxCheck(<object_url>)        # verify ABAP syntax
3. RunUnitTests(<object_url>)       # run unit tests
4. RunATCCheck(<object_url>)        # ATC quality check
5. bun scripts/dev-sync.ts "fix: description"     # full sync: audit → commit → PR
```

See `skills/desktop-app-fallback/SKILL.md` for the complete fallback workflow.


---

*Last Updated: 2026-09-12*


### Optimal Interaction Guidelines
- **XML Tagging**: Utilize XML tags like `<thought>`, `<plan>`, and `<execution>` to structure complex reasoning and plans before generating final responses.
- **Tone**: Maintain an objective, highly analytical tone. Focus on systematic execution.

## Subagent Dispatch & 3-Tier Model Mapping

To fully leverage the 3-tier cost optimization strategy during execution plan creation and subagent dispatch, you must explicitly set the `model` parameter in `Agent()` calls using the correct short alias. 

**Registry Model ID to Short Alias Translation:**
- **High-tier (Design/Planning)**: Registry ID `claude-opus-*` → `model = "opus"`
- **Medium-tier (Review/QA)**: Registry ID `claude-sonnet-*` → `model = "sonnet"`
- **Low-tier (Execution/Coding)**: Registry ID `claude-haiku-*` → `model = "haiku"`

**Example `Agent()` Call:**
```javascript
Agent(
  model = "haiku", // Use short alias: opus, sonnet, or haiku
  description = "Code-writer for serial implementation",
  prompt = "..."
)
```
When dispatching subagents defined in `agents/*.md`, translate their configured tier into the corresponding short alias above.


<!-- COMMON-CLAUDE:START -->
#### teammateMode (Claude Code Agent Teams execution mode)

**teammateMode** specifies the parallel execution mode when Agent Teams is enabled in Claude Code.

**Values**:
- `in-process` — Parallel execution within the same process (applies to both Claude Code CLI and Desktop App)
- `tmux` — Parallel execution using tmux split-pane (Claude Code CLI only, not supported in Desktop App)
- `null` — Default value (auto-selects based on environment)

**Configuration location**: `.claude/settings.json` → `teammateMode`

**Note**: Antigravity does not have an equivalent to Agent Teams, so teammateMode is a Claude Code-specific setting. Antigravity 2.0+ uses Agent Manager to manage multiple workspace shards.

**Relationship to execution plan table**: teammateMode controls parallel execution mode. The execution plan table defines the multi-agent task dispatch.
<!-- COMMON-CLAUDE:END -->

<!-- COMMON-CLAUDE:START -->
### 4. Language Policy for Documentation

All `.md` files you create or modify MUST be in English, except in `ko/` or `locales/ko/` directories (Korean translation zones) or when explicitly declared as a Korean legal/regulatory content exception.

- README.md, CLAUDE.md, GEMINI.md, AGENTS.md, context.md, CHANGELOG.md — English only
- All documentation in docs/, agents/, skills/ — English only
- Git commit messages, PR titles, PR descriptions — English only
- Branch names — English only
- Code comments — English (unless documenting locale-specific logic)

#### Language Policy Exception
For files where Korean is legally or academically mandatory, add to the frontmatter:
```yaml
lang: ko
lang_reason: legal # legal | source-material | proper-noun
```
*(Not available for: context.md, CLAUDE.md, GEMINI.md, AGENTS.md, or any variant context.md)*

#### Korean Plain-Language Preference (`순우리말`-First)
When writing Korean documentation or Korean translation output, prefer native Korean words (`순우리말`) over loanwords (`외래어`) whenever a natural, widely-understood native equivalent exists — e.g. prefer `만들기` over `크리에이션`, `알림` over `노티피케이션`. Loanwords effectively settled in Korean (`컴퓨터`, `데이터`, `소프트웨어`, `파일`) and established technical terms remain permitted; clarity takes precedence over forced nativization. New Korean content applies this immediately; existing Korean documents are nativized incrementally (touched sections only, no bulk rewrites).
<!-- COMMON-CLAUDE:END -->

<!-- COMMON-CLAUDE:START -->
## Execution Plan Boilerplate

The execution plan table format, the Design Gate (Row 0) rule, exemption categories, and the `/sync`-as-final-step rule are the Single Source of Truth in **[AGENTS.md §5.1 Standard Execution Plan Template](AGENTS.md#51-standard-execution-plan-template)** and **[§5.1.1 Design Gate Exemptions](AGENTS.md#511-design-gate-exemptions)** — do not restate them here.

> **Note (Claude Code-specific)**: The `Model` column shows the Claude Code short alias (`sonnet`/`opus`/`haiku`/`fable`) actually passed to the `Agent()` tool's `model` parameter — not the registry ID (e.g. `claude-sonnet-5-0`). See §6 (Native Sub-agents) below for the registry-ID → alias translation table. On Gemini/Antigravity, use the literal model ID instead (see GEMINI.md's equivalent note).
<!-- Note: `fable` is a forward-looking alias not yet registered in docs/workspace-schema.json; do not use until added to the schema -->

**Claude Code execution**: Use the native `Agent` tool for specialist dispatch. See §6 (Native Sub-agents) and §7 (Native Plan Mode) in this file.
<!-- COMMON-CLAUDE:END -->

<!-- COMMON-CLAUDE:START -->
Use the native `Agent` tool to spawn sub-agents for parallel or isolated tasks. Sub-agents load their role-based configurations from `agents/<name>.md`.

> **Agent Architecture**: See [docs/context.md](docs/context.md) for governance rules.
> **Agent Roster**: See [AGENTS.md](AGENTS.md) for the canonical index of all available agents.
> **docs-writer tier**: Medium (claude-sonnet-5-0) — upgraded from Low per 2026-05-28 team restructuring.

**Agent Dispatch** - use the `Agent` tool (not a bash CLI command):
```
Agent(
  description   = "Implement automation script",
  prompt        = "You are an automation engineer. [paste agents/automation-engineer.md content here]\n\nTask: Implement the script per the approved plan.",
  subagent_type = "claude",  // platform agent type; embed the agents/<name>.md role definition in the prompt
  model         = "haiku"    // automation-engineer is Low-tier (registry: claude-haiku-4-5) — see registry→model mapping below
)
```

> **Registry name → `model` parameter mapping**: `docs/workspace-schema.json` and the tables above name models by full registry ID (e.g. `claude-opus-5-0`) for cross-platform documentation. The native `Agent` tool's `model` parameter only accepts the short aliases `sonnet | opus | haiku | fable`. <!-- Note: `fable` is a forward-looking alias not yet registered in workspace-schema.json --> When dispatching, translate the agent's tier to its registry model, then to the matching alias: High → `claude-opus-5-0` → `model = "opus"`; Medium → `claude-sonnet-5-0` → `model = "sonnet"`; Low → `claude-haiku-4-5` → `model = "haiku"`. Omitting `model` lets the subagent fall back to its frontmatter (`model: inherit`), which inherits the parent session's model instead of the tier-appropriate one — always set `model` explicitly to actually get the cost-tier benefit.
>
> **Automated enforcement (Claude Code CLI only)**: `scripts/hooks/agent-model-gate.ts` runs as a `PreToolUse` hook (matcher: `Agent`) and asks for confirmation whenever an `Agent()` call dispatches one of the 8 workspace-root agents (`pm`, `architect`, `auditor`, `lifecycle-manager`, `automation-engineer`, `docs-writer`, `scaffolding-expert`, `security-expert`) without a valid `model` alias — this is the check that catches the High/Low-tier (`opus`/`haiku`) silent-fallback bug described above before it happens. L0-only; not propagated to `templates/common/` since variant projects have their own agent rosters without a workspace-wide tier registry.

Each implementation task follows the **Phase 4 execution loop** (see [AGENTS.md - Subagent Roster](AGENTS.md#subagent-roster)):
1. **automation-engineer** implements the changes (or code-writer for project-specific agents).
2. **PM** verifies against acceptance criteria by running `bun scripts/audit.ts` directly.
3. **Quality gate (audit script)** validates compliance.

> Loop and correct if review errors are flagged - maximum **3 iterations** before escalating to the user.
<!-- COMMON-CLAUDE:END -->

<!-- COMMON-CLAUDE:START -->
#### Cost Optimization (3-Tier Model Strategy)
The High/Medium/Low tier concept and its usage rules are the Single Source of Truth in [AGENTS.md §3.6 3-Tier Strategy](AGENTS.md#36-3-tier-strategy). Claude Code's model-ID mapping (overridden per agent invocation when appropriate):
- **High-tier** → `claude-opus-5-0`
- **Medium-tier** → `claude-sonnet-5-0`
- **Low-tier** → `claude-haiku-4-5`
<!-- COMMON-CLAUDE:END -->

<!-- COMMON-CLAUDE:START -->
### 7. Native Plan Mode (`EnterPlanMode`)
Enter native plan mode using the `EnterPlanMode` tool when:
- The user requests a new feature or significant refactor.
- The change modifies more than 2 files.
- The correct approach is unclear or requires clarifying assumptions.

Once in plan mode:
1. Draft the implementation plan and present it for user review.
2. Obtain explicit user approval before modifying any code.
3. Track progress using the native `TaskCreate` / `TaskUpdate` toolset.
4. After completion, summarize outcomes in the active `memory/YYYY-MM-DD.md` daily log.
<!-- COMMON-CLAUDE:END -->

<!-- COMMON-CLAUDE:START -->
### 8. Task Tracking (`TaskCreate` / `TaskUpdate`)
When working in a plan-mode session:
- Call `TaskCreate` before starting any multi-step execution.
- Set status `in_progress` prior to beginning each atomic step.
- Update status to `completed` immediately upon verification of the step.
- Never leave tasks `in_progress` at the end of a session.
<!-- COMMON-CLAUDE:END -->

<!-- COMMON-CLAUDE:START -->
### 9. Project Boundary Policy

- **Strict Scope**: Work only within the current project directory.
- **No Cross-Project Modification**: Modifying files outside the project root during a session is forbidden.

> For lifecycle management rules, see [docs/context.md — Lifecycle Management](docs/context.md#lifecycle-management).
<!-- COMMON-CLAUDE:END -->

<!-- COMMON-CLAUDE:START -->
### 10. Custom Command Error Recovery
If a custom slash command or background script returns a non-zero exit code:
* **Don't bypass hooks**: Never attempt to run git commands with `--no-verify` to bypass the hook system unless under explicit, written user instruction.
* **Code Page / UTF-8 Issues (Windows)**: If broken Korean characters or Unicode errors appear in CLI output, the Windows terminal code page (CP949) is likely the cause. Ensure `$OutputEncoding = [Console]::OutputEncoding = [System.Text.Encoding]::UTF8;` or `chcp 65001` is prepended to scripts.
* **Diagnostic Audit**: Immediately read the failure stdout log. Common errors include:
  * Missing staged `CHANGELOG.md` edits (caught by `pre-commit`). Fix by running `/changelog` and staging the file.
  * Direct push attempt to `main` (caught by `pre-push`). Fix by executing the `/sync` pipeline script which handles target branch generation and PR staging automatically.
<!-- COMMON-CLAUDE:END -->

<!-- COMMON-CLAUDE:START -->
### 11. Windows Platform Requirement

**Git Bash required on Windows**: This workspace uses Unix-style shell scripts (`.sh`) for `.githooks/` hook files. Windows users must have Git Bash installed and configured as the default shell for git hooks.

- Git Bash ships with [Git for Windows](https://gitforwindows.org/) — install if not present.
- Verify: `git config core.hooksPath` should point to `.githooks/`
- All `scripts/` operational scripts are TypeScript (`.ts`) — run via `bun scripts/<name>.ts`. No `.sh/.ps1` counterparts (ADR-0036).
- If a hook fails on Windows with "command not found", run it via Git Bash: `"C:\Program Files\Git\bin\bash.exe" .githooks/pre-commit`
<!-- COMMON-CLAUDE:END -->

<!-- COMMON-CLAUDE:START -->
## Git & PR Additions (Claude Code)

All shared Git/PR rules are in [docs/context.md](docs/context.md). Claude Code-specific additions:

- **PR Language**: Governed by [docs/context.md](docs/context.md). All PR titles, bodies, and review comments must be written in English - no exceptions.

*Last Updated: 2026-09-12 — removed redundant N-1/N boilerplate rows; /sync already covers lifecycle + audit + commit + push + PR; previous: 2026-06-21 inlined N-1/N rows*
<!-- COMMON-CLAUDE:END -->
