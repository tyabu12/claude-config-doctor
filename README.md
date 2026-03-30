# 🩺 claude-config-doctor

[日本語](./README.ja.md)

A [skill](https://code.claude.com/docs/en/skills) that health-checks your Claude Code configuration files.

It goes beyond structural linting — detecting semantic conflicts across CLAUDE.md, rules, commands, skills, hooks, and settings.

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

For security reasons, this skill is strictly read-only. it only outputs a report and never modifies any files.

To act on findings, pass the Recommended Actions to Claude Code or fix them manually.

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
/plugin marketplace add tyabu12/claude-config-doctor
/plugin install config-doctor@tyabu12-claude-config-doctor
```

<details>
<summary>Manual installation (without plugin system)</summary>

Copy [`skills/check/SKILL.md`](./skills/check/SKILL.md) directly into your project:

```bash
mkdir -p .claude/skills/config-doctor
curl -fsSL https://raw.githubusercontent.com/tyabu12/claude-config-doctor/main/skills/check/SKILL.md \
  -o .claude/skills/config-doctor/SKILL.md
```

When installed manually, the command is `/config-doctor` instead of `/config-doctor:check`.

</details>

## Usage

```
# Recommended monthly: full check (best practices search, `/insights` report analysis)
/config-doctor:check

# Light mode: structural checks only
/config-doctor:check light
```

## What it checks

The skill consists of the following 9 review sections.

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
