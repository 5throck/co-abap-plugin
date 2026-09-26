# Design: Cross-platform plugin packaging fixes (Windows/macOS/Linux)

- **Spec ID**: 2026-09-26-cross-platform-plugin-fixes
- **Date**: 2026-09-26
- **Status**: implemented
- **Source**: manual (follow-up to PR #100 cross-platform assessment)

## Problem

A platform-support audit of the plugin packaging layer found defects:

1. `.claude-plugin/plugin.json` referenced `"hooks": "./hooks/hooks.json"` —
   the file was deleted on 2026-07-08 (d945b00, hooks moved to consumer-side
   settings) leaving a dangling manifest reference on every platform.
2. The abap MCP server used `"command": "vsp"` (PATH lookup), but neither
   documented install location puts vsp on PATH: the README/install scripts
   install into the plugin root, and `docs/plugin-setup.md` prescribed
   `~/abap/vsp`. Consumers following either flow got no working MCP server.
3. `.claude/settings.json` hook commands used POSIX-only shell syntax —
   `${CLAUDE_PROJECT_DIR:-.}` expansion (graft hooks) and `2>/dev/null || true`
   (sync-md hook) — both break under cmd.exe/PowerShell on Windows.
4. `.mcp.json.sample` shipped three identical enabled server entries
   (abap-windows/abap-macos/linux); copying the file as-is launches three
   duplicate vsp processes.
5. CI ran the audit gate on ubuntu only, so Windows/macOS packaging
   regressions were invisible.

## Decision

- **plugin.json**: drop the `"hooks"` key; abap server command becomes
  `${CLAUDE_PLUGIN_ROOT}/vsp` (documented plugin-bundle resolution; libuv
  appends `.exe` on Windows).
- **plugin-setup.md**: prerequisite now says to install vsp into the plugin
  root via `scripts/install-vsp.{sh,ps1}` — consistent with the manifest.
- **settings.json**: graft hooks use the plain relative command
  `node .claude/helpers/graft-hooks.cjs …` (cwd is the project root; the
  helper resolves `CLAUDE_PROJECT_DIR` internally); sync-md hook drops the
  POSIX redirect/suppression.
- **.mcp.json.sample**: header note explains the three entries are per-OS
  alternatives (keep exactly one) and that `./vsp` expects the binary at the
  project root.
- **ci.yml**: audit job runs a `fail-fast: false` matrix over
  ubuntu/macos/windows (`runs-on: ${{ matrix.os }}`); secret-scan stays
  ubuntu-only (docker-based gitleaks).

## Accessibility

Non-UI infrastructure change — no accessibility impact (explicit statement per
ADR-0065).

## Preview Verification

Non-UI change — no rendered-preview verification required (explicit statement
per ADR-0070).

## Verification

- JSON validity re-checked for `plugin.json`, `settings.json`,
  `.mcp.json.sample`; workflow parses with a YAML loader (matrix + runs-on
  verified).
- `bun scripts/audit.ts` expected PASS; `bun test scripts/tests/` expected
  all pass.
- Runtime confirmation of `${CLAUDE_PLUGIN_ROOT}` MCP resolution on a real
  consumer install is a manual follow-up (first Windows-latest CI run will
  exercise the audit gate on Windows for the first time).
