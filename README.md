# 🩺 claude-config-doctor

[日本語](./README.ja.md)

A [skill](https://code.claude.com/docs/en/skills) that health-checks your Claude Code configuration files.

It goes beyond structural linting — detecting semantic conflicts across CLAUDE.md, rules, commands, skills, hooks, and settings. Also supports [plugin](https://code.claude.com/docs/en/plugins) projects and [marketplace](https://code.claude.com/docs/en/plugin-marketplaces) repositories (manifest validation, directory structure, cross-component consistency, and more).

## Why config-doctor

Typical linters check file existence, frontmatter syntax, and naming conventions.
config-doctor uses a Claude Code skill to inspect what they can't.

| | Linters | claude-config-doctor |
| --- | --- | --- |
| Frontmatter & JSON syntax | ✅ | ✅ |
| File/directory existence | ✅ | ✅ |
| **Semantic consistency** (e.g. hooks vs permissions conflicts) | ❌ | ✅ |
| **Cross-file contradiction detection** (CLAUDE.md ↔ rules ↔ settings) | ❌ | ✅ |
| **Insights integration** (friction patterns → config recommendations) | ❌ | ✅ |
| **Best practices sync** (fetches latest official docs at runtime) | ❌ | ✅ |

### Difference from claude-md-management

Anthropic's official [claude-md-management](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/claude-md-management) is a plugin dedicated to improving the quality of CLAUDE.md files.

| | config-doctor | claude-md-management |
| --- | --- | --- |
| Scope | **All** config files (CLAUDE.md / rules / commands / skills / agents / hooks / settings) | **CLAUDE.md only** |
| Purpose | Health check & cross-file consistency validation | Quality audit, improvement & session learning capture |
| Read / Write | **Read-only** (report output only) | **Read + Edit** (modifies files after user approval) |
| Components | 1 skill (`/config-doctor:check`) | 1 skill + 1 command (`/revise-claude-md`) |

The two are complementary, not competing. Use config-doctor to **diagnose** your entire configuration, and claude-md-management to **improve** your CLAUDE.md files.

---

For security reasons, this skill is strictly read-only — it only outputs a report and never modifies any files.

If the findings look reasonable, just type something like "update my config based on these findings" into the prompt and Claude Code will optimize your configuration for you. Easy!

### Sample output

```shell
> /config-doctor:check

Running full config-doctor health check. Let me gather the initial context.

...

# Claude Code Configuration Health Check

**Date**: 2026-03-30
**Project**: my-app
**Reviewer**: Claude Code /config-doctor:check
**Review iterations**: 0
**Mode**: Full

## Summary

| Section                  | Status       | Issues |
|--------------------------|--------------|--------|
| 0. Global Config         | ✅ PASS      | 0      |
| 1. CLAUDE.md             | ⚠️ WARN      | 2      |
| 2. Rules                 | ✅ PASS      | 0      |
| 3. Commands & Skills     | ❌ FAIL      | 1      |
| 4. Agents                | ⏭️ SKIPPED   | 0      |
| 5. Hooks                 | ⚠️ WARN      | 1      |
| 6. Cross-file Consistency| ❌ FAIL      | 2      |
| 7. Best Practices        | ℹ️ ADVISORY  | 3      |
| 8. Insights Integration  | ℹ️ ADVISORY  | 1      |

## Recommended Actions
1. [❌ FAIL] Section 3: Command `deploy.md` references non-existent agent `deployer.md`
2. [❌ FAIL] Section 6: `rm -rf` is in permissions.allow but blocked by PreToolUse hook
3. [⚠️ WARN] Section 1: CLAUDE.md is 247 lines — consider splitting into rules
4. [⚠️ WARN] Section 5: Orphan script `.claude/hooks/old-lint.sh` not wired in settings
```

---

## Installation

Add the marketplace and install the plugin.

```shell
# add marketplace
/plugin marketplace add tyabu12/claude-config-doctor

# install
/plugin install config-doctor@tyabu12-claude-config-doctor

# reload to activate
/reload-plugins
```

<details>
<summary>Manual installation (without plugin system)</summary>

Copy the skill files from [`skills/check/`](./skills/check/) directly into your project:

```bash
mkdir -p .claude/skills/config-doctor
for f in SKILL.md plugin.md project.md reference.md; do
  curl -fsSL "https://raw.githubusercontent.com/tyabu12/claude-config-doctor/main/skills/check/$f" \
    -o ".claude/skills/config-doctor/$f"
done
```

When installed manually, the command is `/config-doctor` instead of `/config-doctor:check`.

</details>

## Usage

> **Tip**: Run `/insights` before a full check to refresh session data for the most accurate results.

```shell
# Recommended monthly: full check (best practices search, `/insights` report analysis)
/config-doctor:check

# Light mode: structural checks only
/config-doctor:check light
```

<details>
<summary>Dogfooding: /self-check (Phase 1 only, via `/config-doctor:check`)</summary>

```shell
❯ /self-check

  Claude Code Plugin Health Check

  Date: 2026-04-01
  Plugin: config-doctor
  Reviewer: Claude Code /config-doctor:check
  Review iterations: 0

  Summary

  ┌────────────────────────────────┬─────────────┬────────┐
  │            Section             │   Status    │ Issues │
  ├────────────────────────────────┼─────────────┼────────┤
  │ 0. Manifest                    │ ✅ PASS     │ 0      │
  ├────────────────────────────────┼─────────────┼────────┤
  │ 1. Directory Structure         │ ⚠️  WARN     │ 1      │
  ├────────────────────────────────┼─────────────┼────────┤
  │ 2. Skills                      │ ✅ PASS     │ 0      │
  ├────────────────────────────────┼─────────────┼────────┤
  │ 3. Commands                    │ ⏭️  SKIPPED  │ 0      │
  ├────────────────────────────────┼─────────────┼────────┤
  │ 4. Agents                      │ ⏭️  SKIPPED  │ 0      │
  ├────────────────────────────────┼─────────────┼────────┤
  │ 5. Hooks                       │ ⏭️  SKIPPED  │ 0      │
  ├────────────────────────────────┼─────────────┼────────┤
  │ 6. MCP & LSP                   │ ⏭️  SKIPPED  │ 0      │
  ├────────────────────────────────┼─────────────┼────────┤
  │ 7. Cross-Component Consistency │ ✅ PASS     │ 0      │
  ├────────────────────────────────┼─────────────┼────────┤
  │ 8. Best Practices              │ ℹ️  ADVISORY │ 1      │
  └────────────────────────────────┴─────────────┴────────┘

  Section Details

  0. Manifest Validation — ✅ PASS

  plugin.json:
  - JSON syntax: valid
  - Required field name: present, non-empty string ("config-doctor")
  - Name format: kebab-case — valid
  - Version "1.1.0": valid semver
  - Description: present, non-empty string
  - Author: object with name field — valid
  - Optional metadata: repository (valid URL string), license (string), keywords (array of strings) — all valid
  - No component path overrides, userConfig, or channels defined
  - No settings.json at plugin root
  - No unknown top-level fields

  marketplace.json:
  - JSON syntax: valid
  - Required fields: name, owner, plugins — all present
  - Name format "tyabu12-claude-config-doctor": kebab-case — valid
  - Owner: has name field — valid
  - Metadata: description present (string) — valid
  - Plugin entry: name "config-doctor" (kebab-case), source "./" (starts with ./, resolves to existing directory) — valid
  - Description: present, non-empty string — valid
  - Plugin name consistency: marketplace entry "config-doctor" matches plugin.json "config-doctor"; descriptions match
  - No unknown top-level fields

  1. Directory Structure — ⚠️  WARN (1)

  - Anti-pattern check: .claude-plugin/ contains only plugin.json and marketplace.json — no component directories inside. PASS
  - Component directories: skills/ exists at root with skill content. No commands/, agents/, hooks/, output-styles/, .mcp.json, or .lsp.json — expected for this plugin's scope
  - README.md: exists at plugin root. PASS
  - Path traversal: no component files reference paths outside the plugin root. PASS
  - ⚠️  WARN: Stray .claude/ directory exists alongside .claude-plugin/. Contains settings.local.json (local dev permissions) and skills/self-check/ (development-only
  self-diagnostics). CONTRIBUTING.md (lines 29–33) documents this as intentional. Documented — WARN acknowledged as expected.

  2. Skills Validation — ✅ PASS

  Skill: skills/check/ (self-skip rule overridden per /self-check instructions)

  - SKILL.md exists: yes
  - Frontmatter syntax: well-formed YAML
  - description: present, ~195 characters (under 250 limit)
  - allowed-tools: Read, Glob, Grep, WebSearch, WebFetch, Bash, Agent — all valid per reference.md
  - model: opus — valid
  - disable-model-invocation: true — valid boolean
  - argument-hint: "[light | full]" — valid string
  - Supporting files: plugin.md, project.md, reference.md — all referenced and all exist in the skill directory
  - $ARGUMENTS usage: argument-hint is defined and $ARGUMENTS is referenced in SKILL.md and delegated procedure files
  - Script references: none (no scripts to check)

  3. Commands — ⏭️  SKIPPED

  No commands/ directory exists.

  4. Agents — ⏭️  SKIPPED

  No agents/ directory exists.

  5. Hooks — ⏭️  SKIPPED

  No hooks/hooks.json, no inline hooks in plugin.json, no hooks/ directory.

  6. MCP & LSP — ⏭️  SKIPPED

  No .mcp.json, .lsp.json, or inline MCP/LSP config in plugin.json.

  7. Cross-Component Consistency — ✅ PASS

  - Skill→Agent references: none (no agents referenced)
  - Command→Agent references: no commands
  - Agent→Skill references: no agents
  - Hook→Script references: no hooks
  - Channel→MCP references: no channels
  - Manifest→Component consistency: no custom component paths in manifest; default skills/ directory contains expected skill files
  - Namespace consistency: only skills/ exists, no commands/ — no naming conflicts
  - settings.json alignment: no settings.json at plugin root

  8. Best Practices (Advisory) — ℹ️  ADVISORY (1)

  Compared against official documentation from code.claude.com (plugins, plugins-reference, skills, hooks pages).

  - Plugin structure: follows standard layout. PASS
  - Skill vs command preference: uses skills/ directory (recommended). PASS
  - Portability: no scripts or hardcoded paths — not applicable. PASS
  - Validation support: claude plugin validate . is documented in CONTRIBUTING.md step 5. PASS
  - ℹ️  ADVISORY: Official docs document newer plugin features including output styles (output-styles/), LSP servers (.lsp.json), channels, userConfig, and settings.json with
  agent key for default agent activation. These are not relevant to this plugin's current scope but could be adopted if future development warrants them.

  Sources: https://code.claude.com/docs/en/plugins-reference, https://code.claude.com/docs/en/plugins, https://code.claude.com/docs/en/skills,
  https://code.claude.com/docs/en/hooks

  Recommended Actions

  1. [⚠️  WARN — Section 1] .claude/ directory exists alongside .claude-plugin/. Documented as intentional in CONTRIBUTING.md — no action needed unless the project structure
  changes.
  2. [ℹ️  ADVISORY — Section 8] Consider evaluating newer plugin features (output styles, LSP servers, channels, userConfig) if future development warrants them.
```

</details>

## What it checks

The skill automatically detects the project type (standard project, plugin, or marketplace) and runs the appropriate diagnostics.

### Standard projects (`.claude/` configuration)

| Section | What it checks |
| --- | --- |
| 0. Global Config | `~/.claude/` globals, `.claudeignore` relevance |
| 1. CLAUDE.md | Line count, encoding, path accuracy, tech stack drift, staleness |
| 2. Rules | Frontmatter, glob patterns, content accuracy, CLAUDE.md consistency |
| 3. Commands & Skills | Syntax, tool permissions, procedure accuracy, supporting files, agent cross-refs |
| 4. Agents | Required fields, tool lists, model values, evaluation criteria |
| 5. Hooks | Orphan scripts, matcher correctness, script logic, exit codes, frontmatter hooks |
| 6. Cross-file | JSON validity, permission conflicts, hook-permission alignment |
| 7. Best Practices | Fetches latest Anthropic docs at runtime and compares against your config |
| 8. Insights | Aggregates `/insights` friction data into actionable config fixes |

### Plugin projects (`.claude-plugin/plugin.json`)

| Section | What it checks |
| --- | --- |
| 0. Manifest | JSON syntax, required fields, name format, version, metadata, userConfig, channels, marketplace.json |
| 1. Directory Structure | Anti-patterns, component directories, README, path traversal, stray `.claude/` |
| 2. Skills | SKILL.md existence, frontmatter, description, tool permissions, `$ARGUMENTS`, `${CLAUDE_PLUGIN_ROOT}` |
| 3. Commands | Frontmatter, description, tool permissions, legacy overlap with skills |
| 4. Agents | Required fields, supported fields, model, tools, isolation, security restrictions |
| 5. Hooks | hooks.json syntax, event names, handler structure, script portability, orphan scripts |
| 6. MCP & LSP | JSON syntax, server entries, portability, channel references, extension format |
| 7. Cross-Component | Skill↔Agent refs, hook→script refs, channel→MCP refs, namespace conflicts |
| 8. Best Practices | Fetches latest Anthropic plugin docs at runtime and compares against your config |

### Marketplace repositories (`.claude-plugin/marketplace.json` without `plugin.json`)

Marketplace repos are automatically detected and each local plugin listed in `marketplace.json` receives the full plugin diagnostic above. Remote plugin entries are validated for source structure only.

FAIL items are cross-reviewed by a sandboxed subagent (max 1 iteration) before the final report.

## Security

Because this skill reads local session data, it is designed with security in mind while maintaining practical utility.

- **Strictly read-only** — never modifies any files
- **Prompt injection defense** — all config file content and web content is treated as data to analyze, never as instructions to follow
- **Scoped web access** — WebSearch and WebFetch are restricted to a hardcoded allowlist of official Anthropic domains only. No project-specific information ever leaves your machine via search queries
- **Output sanitization** — findings reference file paths and line numbers, never quoting content verbatim, preventing second-order injection if the report is shared
- **Subagent sandboxing** — cross-review subagents are limited to `Read, Glob, Grep` only
- **Insights privacy** — session data is aggregated into abstract recommendations; per-session behavioral details are never included in the report

## Requirements / Limitations

- **Model**: The skill automatically uses the Opus model to ensure check accuracy
- Full mode also searches for the latest best practices, so it uses `WebSearch`, `WebFetch`, and `Agent` tools (sections 7-8). `light` mode uses only `Read`, `Glob`, `Grep`, `Bash`, and `Agent`
- **Read-only** — never modifies any configuration files. No auto-fix functionality
- **Execution time**: Full review takes about 5 minutes. `light` mode is faster but skips best practices search and `/insights` report analysis
- **Uninstall**: `/plugin uninstall config-doctor@tyabu12-claude-config-doctor` (or delete `.claude/skills/config-doctor/` if installed manually)

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines.

## License

[MIT](./LICENSE)
