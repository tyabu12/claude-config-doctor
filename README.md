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

> **Tip**: Run `/insights` before a full check to refresh session data for the most accurate results.

```sh
# Recommended monthly: full check (best practices search, `/insights` report analysis)
/config-doctor:check

# Light mode: structural checks only
/config-doctor:check light
```

<details>
<summary>Dogfooding: /config-doctor:check</summary>

```sh
❯ /config-doctor:check

⏺ Bash(git rev-parse --path-format=absolute --git-common-dir 2>/dev/null | sed 's|/\.git$||')
  ⎿  /Users/tyabu12/Work/claude-config-doctor

⏺ Bash(basename /Users/tyabu12/Work/claude-config-doctor)
  ⎿  claude-config-doctor

⏺ Now let me read the key configuration files in parallel.

  Searched for 5 patterns, read 6 files (ctrl+o to expand)

⏺ Now let me check for insights data and run the best practices searches.

  Searched for 1 pattern (ctrl+o to expand)

⏺ Let me validate the insights schema, find matching sessions, and run the best practices searches in parallel.

  Searched for 2 patterns, read 1 file (ctrl+o to expand)

⏺ Web Search("site:code.claude.com CLAUDE.md")
  ⎿  Did 1 search in 7s

⏺ Web Search("site:code.claude.com hooks")
  ⎿  Did 1 search in 6s

⏺ Web Search("site:code.claude.com commands")
  ⎿  Did 1 search in 7s

⏺ Web Search("site:code.claude.com skills")
  ⎿  Did 1 search in 7s

⏺ Web Search("site:platform.claude.com Claude Code configuration")
  ⎿  Did 1 search in 6s

⏺ Web Search("site:anthropic.com/engineering Claude Code")
  ⎿  Did 1 search in 9s

⏺ No sessions matched this project for insights. Let me fetch a couple of key documentation pages for Section 7 advisory recommendations.

⏺ Fetch(https://code.claude.com/docs/en/memory)
  ⎿  Received 24.4KB (200 OK)

⏺ Fetch(https://code.claude.com/docs/en/best-practices)
  ⎿  Received 34.3KB (200 OK)

⏺ Let me check if there are any skills besides the config-doctor plugin.

  Searched for 1 pattern, read 1 file (ctrl+o to expand)

⏺ No .claude/ directory exists at all (it's a plugin-installed skill only). Now I have everything needed for the report.

  Claude Code Configuration Health Check

  Date: 2026-03-31
  Project: claude-config-doctor
  Reviewer: Claude Code /config-doctor
  Review iterations: 0
  Mode: Full

  Summary

  ┌───────────────────────────┬─────────────┬────────┐
  │          Section          │   Status    │ Issues │
  ├───────────────────────────┼─────────────┼────────┤
  │ 0. Global Config          │ ✅ PASS     │ 0      │
  ├───────────────────────────┼─────────────┼────────┤
  │ 1. CLAUDE.md              │ ⏭️  SKIPPED  │ 0      │
  ├───────────────────────────┼─────────────┼────────┤
  │ 2. Rules                  │ ⏭️  SKIPPED  │ 0      │
  ├───────────────────────────┼─────────────┼────────┤
  │ 3. Commands & Skills      │ ⏭️  SKIPPED  │ 0      │
  ├───────────────────────────┼─────────────┼────────┤
  │ 4. Agents                 │ ⏭️  SKIPPED  │ 0      │
  ├───────────────────────────┼─────────────┼────────┤
  │ 5. Hooks                  │ ⏭️  SKIPPED  │ 0      │
  ├───────────────────────────┼─────────────┼────────┤
  │ 6. Cross-file Consistency │ ⏭️  SKIPPED  │ 0      │
  ├───────────────────────────┼─────────────┼────────┤
  │ 7. Best Practices         │ ℹ️  ADVISORY │ 4      │
  ├───────────────────────────┼─────────────┼────────┤
  │ 8. Insights Integration   │ ⏭️  SKIPPED  │ 0      │
  └───────────────────────────┴─────────────┴────────┘

  0. Global Config

  - Global CLAUDE.md: ~/.claude/CLAUDE.md does not exist. No conflicts possible.
  - Global settings.json: ~/.claude/settings.json exists and is valid JSON. Contains permissions (1 allow entry), statusLine configuration, enabledPlugins, and effortLevel
  setting. No project-level settings exist to conflict with.
  - .claudeignore: Does not exist in project root.

  Status: ✅ PASS — Global settings are well-formed. No conflicts detected.

  1. CLAUDE.md

  Status: ⏭️  SKIPPED — No CLAUDE.md file exists in the project root or .claude/ directory.

  2. Rules

  Status: ⏭️  SKIPPED — .claude/rules/ directory does not exist.

  3. Commands & Skills

  Status: ⏭️  SKIPPED — No .claude/commands/ or .claude/skills/ directories exist (plugin-installed skills under config-doctor namespace are excluded from review).

  4. Agents

  Status: ⏭️  SKIPPED — .claude/agents/ directory does not exist.

  5. Hooks

  Status: ⏭️  SKIPPED — No hook configurations found in settings files, no .claude/hooks/ directory exists.

  6. Cross-file Consistency

  Status: ⏭️  SKIPPED — Both CLAUDE.md and .claude/settings.json are absent.

  7. Best Practices (Advisory)

  Based on current Claude Code documentation (https://code.claude.com/docs/en/memory, https://code.claude.com/docs/en/best-practices, https://code.claude.com/docs/en/skills,
  https://code.claude.com/docs/en/hooks):

  1. ℹ️  ADVISORY — No CLAUDE.md: This project has no CLAUDE.md. Running /init will generate a starter file based on the project structure. Even a minimal file with build commands
   and coding conventions improves session consistency. Official recommendation is to keep it under 200 lines and focus on information Claude cannot infer from the code itself.
  2. ℹ️  ADVISORY — No .claude/settings.json: No project-level settings file exists. Consider adding one to pre-allow common commands (build, test, lint) so team members don't
  need to approve them individually each session.
  3. ℹ️  ADVISORY — No rules or skills: The project uses no .claude/rules/ files or custom skills. For projects with specific coding conventions or repeatable workflows, rules and
   skills can improve consistency. Skills load on demand (unlike CLAUDE.md which loads every session), making them ideal for domain-specific workflows.
  4. ℹ️  ADVISORY — No hooks: Hooks provide deterministic enforcement of project rules (e.g., auto-formatting after edits, blocking writes to protected directories). Consider
  adding hooks for any actions that must happen every time without exception.

  Sources:
  - https://code.claude.com/docs/en/memory
  - https://code.claude.com/docs/en/best-practices
  - https://code.claude.com/docs/en/skills
  - https://code.claude.com/docs/en/hooks-guide
  - https://code.claude.com/docs/en/claude-directory

  8. Insights Integration (Advisory)

  Status: ⏭️  SKIPPED — No analyzed sessions for this project. No session-meta files matched project path /Users/tyabu12/Work/claude-config-doctor.

  Recommended Actions

  1. [ℹ️  ADVISORY — Section 7] Run /init to generate a CLAUDE.md file for this project with build commands and coding conventions.
  2. [ℹ️  ADVISORY — Section 7] Create .claude/settings.json with permission allowlists for commonly used commands to reduce approval prompts for team members.
  3. [ℹ️  ADVISORY — Section 7] Consider adding .claude/rules/ or .claude/skills/ as the project grows, to codify repeatable workflows and domain-specific conventions.
  4. [ℹ️  ADVISORY — Section 7] Evaluate whether any project operations (formatting, linting, protected paths) warrant deterministic enforcement via hooks.
```

</details>

<details>
<summary>Dogfooding: /config-doctor:check light</summary>

```sh
❯ /config-doctor:check light

⏺ Bash(git rev-parse --path-format=absolute --git-common-dir 2>/dev/null | sed 's|/\.git$||')
  ⎿  /Users/tyabu12/Work/claude-config-doctor

⏺ Bash(basename /Users/tyabu12/Work/claude-config-doctor)
  ⎿  claude-config-doctor

  Searched for 9 patterns, read 2 files (ctrl+o to expand)

⏺ Claude Code Configuration Health Check

  Date: 2026-03-31
  Project: claude-config-doctor
  Reviewer: Claude Code /config-doctor
  Review iterations: 0
  Mode: Light

  Summary

  ┌───────────────────────────┬────────────┬────────┐
  │          Section          │   Status   │ Issues │
  ├───────────────────────────┼────────────┼────────┤
  │ 0. Global Config          │ ✅ PASS    │ 0      │
  ├───────────────────────────┼────────────┼────────┤
  │ 1. CLAUDE.md              │ ⏭️  SKIPPED │ 0      │
  ├───────────────────────────┼────────────┼────────┤
  │ 2. Rules                  │ ⏭️  SKIPPED │ 0      │
  ├───────────────────────────┼────────────┼────────┤
  │ 3. Commands & Skills      │ ⏭️  SKIPPED │ 0      │
  ├───────────────────────────┼────────────┼────────┤
  │ 4. Agents                 │ ⏭️  SKIPPED │ 0      │
  ├───────────────────────────┼────────────┼────────┤
  │ 5. Hooks                  │ ⏭️  SKIPPED │ 0      │
  ├───────────────────────────┼────────────┼────────┤
  │ 6. Cross-file Consistency │ ⏭️  SKIPPED │ 0      │
  └───────────────────────────┴────────────┴────────┘

  0. Global Config

  - Global CLAUDE.md: Not present. ✅
  - Global settings.json: Present at ~/.claude/settings.json. Contains permissions.allow (1 entry: mcp__plugin_discord_discord__reply), statusLine configuration, enabledPlugins
  (clangd-lsp, discord), and effortLevel: "medium". Structure is valid. ✅
  - .claudeignore: Not present. ✅

  1. CLAUDE.md

  Status: ⏭️  SKIPPED — No CLAUDE.md file exists in the project root.

  2. Rules

  Status: ⏭️  SKIPPED — .claude/rules/ directory does not exist.

  3. Commands & Skills

  Status: ⏭️  SKIPPED — No .claude/commands/ or .claude/skills/ directories exist (plugin-installed skills excluded).

  4. Agents

  Status: ⏭️  SKIPPED — .claude/agents/ directory does not exist.

  5. Hooks

  Status: ⏭️  SKIPPED — No hook configurations found in project or global settings, and no .claude/hooks/ directory exists.

  6. Cross-file Consistency

  Status: ⏭️  SKIPPED — Neither CLAUDE.md nor .claude/settings.json exist at the project level.

  Recommended Actions

  No FAIL or WARN issues found. This project has minimal Claude Code configuration — most sections were skipped because the corresponding files don't exist yet. Consider:

  1. [ℹ️  INFO] Creating a CLAUDE.md at the project root to document conventions, tech stack, and development commands. This helps Claude Code understand your project better.
  2. [ℹ️  INFO] Creating .claude/settings.json to configure project-level permissions (e.g., allowing build/test commands) so collaborators share the same defaults.
```

</details>

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
