# Design: Sync co-abap workspace content into co-abap-plugin (2026-09-26)

- **Spec ID**: 2026-09-26-co-abap-sync
- **Date**: 2026-09-26
- **Status**: implemented
- **Source**: manual (user request: reflect `co-abap` project content into this plugin repo)

## Problem

This plugin repo last synced with the `co-abap` workspace on 2026-09-20
(template 0.6.0). The workspace moved forward six days of waves afterwards
(#144–#160): W4 thin-dispatcher AGENTS.md regeneration (ADR-0090), L1 hermes
platform delivery (ADR-0088, fifth skill mirror), country_config variant.json
bootstrap (ADR-0091), project-review remediation (ADR-0002), deliverables
05/06 stage promotion, i18n-specialist agent, design-foundation skill,
handbook sub-deliverables, governance/agents reference docs, script-library
growth (`scripts/lib/platforms.ts`, `upgrade-policy.ts`, `dependency-guard.ts`,
`regenerate-agents-md.ts`, `review-baseline.ts`), plus security hardening
(`.env.sample` password redaction, SAP_FEATURE_* safe defaults, LF-enforcing
`.gitattributes`). All of it was missing here.

## Decision

Hand-port the 2026-09-21..2026-09-26 delta from the `co-abap` working tree,
category by category:

- **Wholesale port**: `agents/` (21 defs incl. new `i18n-specialist.md`),
  `skills/` (47 dirs incl. new `design-foundation`, handbook sub-dirs,
  explain-me sub-dirs), `deliverables/templates/` (incl. new
  `05_unit_test_plan.md`, `06_release_report.md`), `scripts/` overlay,
  `AGENTS.md`, `GEMINI.md`, `CODEX.md`, `SECURITY.md`, `.gitattributes`,
  `.env.sample`, `.mcp.json.sample`, `.codex/config.toml`,
  `.gemini/settings.json`, docs tree (lifecycle, adr 0001+0002,
  governance/agents, country-profiles, design-foundation, user guides,
  context pair, skill-graph sources), platform command/prompt wrappers
  (`.claude/commands`, `.codex/prompts`, `.gemini/commands/meeting.md`),
  portable `.claude/helpers/graft-*.cjs`, and a new `.hermes/` mirror.
- **Platform skill mirrors**: overlay `.agents/.claude/.codex/.gemini/skills`
  + new `.hermes/skills`; platform-specific extra skills (graft, meeting,
  validate-docs-links) preserved untouched.
- **Hand-merged**: `CLAUDE.md` (co-abap body + plugin Setup/Consumer
  Integration sections restored before "Claude Code Settings"),
  `.claude/settings.json` (co-abap base + this repo's `sync-md.ts`
  PostToolUse hook restored, empty hook entry dropped),
  `.gitignore` (co-abap + plugin's `.mcp.json`/`bun.lockb`),
  `.gitleaks.toml` (co-abap base; upstream memory-scanning posture adopted
  after verifying `memory/` has no secret-like strings; co-abap-specific
  `DEVELOPER:Down1oad` stopword not carried).
- **Kept plugin-local (not overwritten)**: `.claude-plugin/`, `commands/`,
  `config/`, `hooks` reference in plugin.json, root `README*.md` (counts
  refreshed instead), `docs/CONTRIBUTING.md`, `docs/PR_BODY.md`,
  `docs/plugin-setup.md` (plugin-localized), `scripts/install-*.{sh,ps1}`,
  `scripts/package.json`, `scripts/helpers/generate-variant.ts`,
  `agents/README*.md`, plugin `docs/designs/*`, `docs/specs/registry.json`
  (plugin seed format kept), `.codex/hooks.json` (plugin SessionStart
  githooks design), root `CHANGELOG.md` (append-only entry),
  `package.json`/`bun.lock`, `opencode.json`, `memory/`.
- **Excluded (co-abap project artifacts)**: `deliverables/REQ-001-*`,
  `deliverables/index.md`, `docs/decisions/`, `docs/reports/`,
  `docs/superpowers/`, `docs/upstream-fix-list.md`, `variant.json`,
  `memory/`, `graft/`, `scratch/`, `.env`, `vsp.exe`.
- **Regenerated**: `docs/VERSION_MANIFEST.md`, `docs/skill-graph.json`,
  `docs/skill-graph.md` via the project scripts.
- **Post-port metadata**: SAP_FEATURE_* flipped to hardened `off` defaults in
  committed `.claude/settings.json`, `.codex/config.toml`,
  `.gemini/settings.json` (upstream safe-default posture; consumers of the
  *plugin* are unaffected — `.claude-plugin/plugin.json` still ships its own
  feature flags). `.claude/template-version.txt` `upgraded=` bumped.

## Accessibility

Non-UI infrastructure change — no accessibility impact (explicit statement per
ADR-0065).

## Preview Verification

Non-UI change — no rendered-preview verification required (explicit statement
per ADR-0070).

## Verification

- `bun scripts/audit.ts` — expected exit 0.
- `bun scripts/typecheck.ts` — expected clean.
- `gitleaks detect` — expected no findings (memory scanning posture adopted).
- Spot checks: new files present (`agents/i18n-specialist.md`,
  `skills/design-foundation/SKILL.md`,
  `deliverables/templates/05_unit_test_plan.md`,
  `docs/governance/agents/workflows.md`, `.hermes/skills/`), plugin-only
  keepers intact (`scripts/install-bun.sh`, `docs/plugin-setup.md`,
  `commands/`, `.claude-plugin/plugin.json`).
