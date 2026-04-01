# Common Reference for config-doctor

This file contains shared valid value lists referenced by the procedure files (project.md, plugin.md).
Safety Constraints, Initialization, Report Format, and Review Loop are defined in the main SKILL.md.

## Valid Tool Names

Valid tools for `allowed-tools` (skills/commands) and `tools` (agents): Read, Grep, Glob, Bash, Write, Edit, Agent, WebSearch, WebFetch, Skill, NotebookEdit. Entries may use `ToolName(pattern)` syntax (e.g., `Bash(python *)`); validate only the tool name portion before the parenthesis. Tools prefixed with `mcp__` are also valid. If an unrecognized tool name is encountered that is not `mcp__`-prefixed, flag as WARN (possible typo or newly added tool).

## Valid Model Values

Known valid model values: `opus`, `sonnet`, `haiku`. Flag unrecognized values.

## Valid Effort Values

Known valid effort values: `low`, `medium`, `high`, `max`. Flag unrecognized values.

## Valid Hook Event Names

Known hook event names: `SessionStart`, `UserPromptSubmit`, `PreToolUse`, `PermissionRequest`, `PostToolUse`, `PostToolUseFailure`, `Notification`, `SubagentStart`, `SubagentStop`, `TaskCreated`, `TaskCompleted`, `Stop`, `StopFailure`, `TeammateIdle`, `InstructionsLoaded`, `ConfigChange`, `CwdChanged`, `FileChanged`, `WorktreeCreate`, `WorktreeRemove`, `PreCompact`, `PostCompact`, `Elicitation`, `ElicitationResult`, `SessionEnd`. Flag unrecognized event names as WARN (possible typo or newly added event).
