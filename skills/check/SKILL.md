---
description: Health check for Claude Code configuration files. Automatically detects project type (standard project or plugin) and runs the appropriate diagnostics. Use 'light' for structural checks only (standard projects). Read-only.
argument-hint: [light | full]
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, WebSearch, WebFetch, Bash, Agent
model: opus
---

# /config-doctor:check

Health check for all Claude Code configuration files. Automatically detects whether the current project is a **standard project** or a **plugin**, and runs the appropriate diagnostics.

**This skill is strictly read-only. Do NOT modify any files.**

## Initialization

1. Read `reference.md` (in this skill's directory) for shared Safety Constraints, Before Starting procedure, Report Format, Review Loop, and valid tool/model/effort lists. Follow all constraints defined there.
2. Follow the "Before Starting" procedure in reference.md to determine project root and project name.
3. Detect project type: check if `.claude-plugin/plugin.json` exists at the project root.

## Project Type Detection and Procedure Selection

### If `.claude-plugin/plugin.json` exists → Plugin project

1. Read `plugin.md` (in this skill's directory) for the plugin-specific diagnostic procedure.
2. If `$ARGUMENTS` is `light`, inform the user:
   > "Plugin diagnostics always run in full mode. The `light` argument is only available for standard project checks. Proceeding with full plugin check."
3. Follow the procedure defined in plugin.md.

### If `.claude-plugin/plugin.json` does NOT exist → Standard project

1. Read `project.md` (in this skill's directory) for the standard project diagnostic procedure.
2. Pass through the `$ARGUMENTS` value (light/full/empty) for mode selection as defined in project.md.
3. Follow the procedure defined in project.md.

### Edge Cases

- If the detection check itself fails (e.g., not a git repository, no readable filesystem), report the error and stop.
- If both `.claude-plugin/plugin.json` AND `.claude/` configuration exist (hybrid project), treat as a **plugin project** and note in the report: "This project has both plugin metadata and standard `.claude/` configuration. Running plugin diagnostics. To also check standard project configuration, run `/config-doctor:check` again after temporarily removing `.claude-plugin/`."

## After Procedure Completion

After completing all sections defined in the procedure file:

1. Follow the Review Loop defined in reference.md.
2. Produce the final report using the Output template defined in the procedure file, following the Report Format rules from reference.md.
