# Plugin Diagnostic Procedure

This file defines the diagnostic procedure for Claude Code plugin projects.
It is read by the main SKILL.md after project type detection.
Valid tool, model, and effort lists are defined in reference.md.

This check always runs in full mode — there is no `light` option.

## Pre-Procedure Setup

Read `.claude-plugin/plugin.json` to understand the plugin's metadata and component configuration.

## Procedure

Run the following reviews sequentially and collect findings as you go.
If a section's target files do not exist, mark it SKIPPED and move on.

### 0. Manifest Validation

Read `.claude-plugin/plugin.json` and check:

- [ ] **JSON syntax**: Verify the file is valid JSON. Malformed JSON prevents the plugin from loading. Mark as FAIL.
- [ ] **Required fields**: `name` is the only required field when a manifest is present. Verify it exists and is a non-empty string.
- [ ] **Name format**: Verify `name` is kebab-case (lowercase letters, digits, and hyphens only). Flag uppercase, spaces, or special characters as FAIL — the official validator rejects non-kebab-case names.
- [ ] **Version format**: If `version` is present, verify it follows semantic versioning (`MAJOR.MINOR.PATCH`). Flag invalid formats as WARN.
- [ ] **Description**: If `description` is present, verify it is a non-empty string. If absent, flag as WARN (recommended for marketplace discoverability).
- [ ] **Author**: If `author` is present, verify it is an object with at least `name` (string). Optional fields: `email`, `url`.
- [ ] **Optional metadata fields**: If present, verify types — `homepage` (string/URL), `repository` (string/URL), `license` (string), `keywords` (array of strings).
- [ ] **Component path overrides**: If any of `commands`, `agents`, `skills`, `hooks`, `mcpServers`, `outputStyles`, `lspServers` are specified, verify the referenced paths exist on disk. Note: these override the default directory locations.
- [ ] **userConfig**: If present, verify it is an object where each key maps to an object with at least `description` (string). Check `sensitive` field is boolean if present.
- [ ] **channels**: If present, verify each entry has a `server` field that matches a key in the plugin's MCP server configuration.
- [ ] **settings.json**: If `settings.json` exists at the plugin root, note its presence — this provides default configuration when the plugin is enabled. Verify it is valid JSON. If an `agent` key is present, verify the referenced agent exists in the plugin's `agents/` directory (or custom agents path from manifest). Flag unknown keys as WARN (currently only `agent` is supported; unknown keys are silently ignored).
- [ ] **Unknown fields**: Flag any top-level fields not in the known schema as WARN (possible typo or future field).

### 1. Directory Structure Validation

Check that the plugin follows the standard directory layout:

- [ ] **Anti-pattern check**: Verify that `commands/`, `agents/`, `skills/`, `hooks/`, `output-styles/` directories are NOT inside `.claude-plugin/`. Only `plugin.json` belongs in `.claude-plugin/`. This is a common mistake explicitly warned against in official documentation. Flag as FAIL.
- [ ] **Component directories at root**: For each component type, check if the directory exists at the expected location (root level or custom path from manifest):
  - `skills/` — skill directories with `SKILL.md`
  - `commands/` — command markdown files (legacy; `skills/` preferred for new skills)
  - `agents/` — agent markdown files
  - `hooks/` — hook configuration (`hooks.json`)
  - `output-styles/` — output style definition files
  - `.mcp.json` — MCP server definitions (root level)
  - `.lsp.json` — LSP server definitions (root level)
- [ ] **README.md**: Verify a `README.md` exists at the plugin root. Flag absence as WARN (recommended for documentation).
- [ ] **Path traversal**: Check that no component file references paths outside the plugin root (e.g., `../shared-utils`). These paths break after plugin installation because only the plugin directory is cached. Flag as FAIL.
- [ ] **Stray `.claude/` directory**: If a `.claude/` directory exists alongside `.claude-plugin/`, check whether `README.md` or `CONTRIBUTING.md` documents its presence as intentional (e.g., for development tooling or dogfooding). If documented, report as PASS and note the justification. If not documented, flag as WARN — this may indicate confusion between standard project config and plugin structure. Plugin components should be at root level, not under `.claude/`.

### 2. Skills Validation

If `skills/` does not exist or is empty (and no custom skills path in manifest), mark SKIPPED.

**Skip this plugin's own skill** — do not review skills under the `config-doctor` namespace.

For each skill directory:

- [ ] **SKILL.md exists**: Verify each skill directory contains a `SKILL.md` file. Flag absence as FAIL.
- [ ] **Frontmatter syntax**: Verify YAML frontmatter is well-formed and parseable.
- [ ] **Description**: Verify `description` field is present (required for plugin skills per official docs). Flag absence as FAIL. If `description` exceeds 250 characters, flag as WARN.
- [ ] **Name**: If `name` field is present, verify it contains only lowercase letters, numbers, and hyphens (max 64 characters). Note: for plugin skills, the folder name becomes the skill name prefixed with the plugin namespace.
- [ ] **Tool permissions**: If `allowed-tools` is present, verify it lists only valid tool names (see reference.md).
- [ ] **Model**: If `model` is present, verify it is a known value (see reference.md).
- [ ] **Effort**: If `effort` is present, verify it is a valid value (see reference.md).
- [ ] **Other frontmatter**: If present, verify:
  - `disable-model-invocation` is a boolean
  - `user-invocable` is a boolean
  - `argument-hint` is a string or array
  - `context` is `fork`
  - `shell` is `bash` or `powershell`
  - `paths` glob patterns match at least one existing file
  - `hooks` contains valid hook configuration structure
- [ ] **Supporting files**: If the skill's `SKILL.md` references supporting files (e.g., `reference.md`, `scripts/`, `examples/`), verify they exist within the skill directory.
- [ ] **$ARGUMENTS usage**: If `argument-hint` is defined, verify the skill content references `$ARGUMENTS`.
- [ ] **Script references**: If the skill references scripts, verify they use `${CLAUDE_PLUGIN_ROOT}` or `${CLAUDE_PLUGIN_DATA}` for paths (not hardcoded absolute paths). This ensures scripts work after plugin installation.

### 3. Commands Validation

If `commands/` does not exist or is empty (and no custom commands path in manifest), mark SKIPPED.

Note: `commands/` is the legacy location. Official docs recommend `skills/` for new skills.

For each `.md` file in `commands/`:

- [ ] **Frontmatter syntax**: Verify YAML frontmatter is well-formed and parseable.
- [ ] **Description**: If `description` is present, check length (WARN if over 250 characters).
- [ ] **Tool permissions**: If `allowed-tools` is present, verify valid tool names (see reference.md).
- [ ] **Content accuracy**: Verify that files and paths referenced in the command content actually exist.
- [ ] **Legacy notice**: If the plugin has both `commands/` and `skills/` for overlapping functionality, flag as WARN — potential confusion with duplicate slash commands.

### 4. Agents Validation

If `agents/` does not exist or is empty (and no custom agents path in manifest), mark SKIPPED.

For each `.md` file in `agents/`:

- [ ] **Frontmatter syntax**: Verify frontmatter is well-formed.
- [ ] **Required fields**: Check that `name` and `description` are present. Flag absence as FAIL.
- [ ] **Supported fields**: Verify frontmatter fields are from the supported set: `name`, `description`, `model`, `effort`, `maxTurns`, `tools`, `disallowedTools`, `skills`, `memory`, `background`, `isolation`. Flag unknown fields as WARN.
- [ ] **Model**: If `model` is present, verify it is a known value (see reference.md).
- [ ] **Tool list**: If `tools` is present, verify all entries are valid Claude Code tool names (see reference.md).
- [ ] **Disallowed tools**: If `disallowedTools` is present, verify entries are valid tool names.
- [ ] **Isolation**: If `isolation` is present, verify it is `worktree` (the only valid value).
- [ ] **Unsupported agent fields**: Plugin agents do NOT support `hooks`, `mcpServers`, or `permissionMode` in frontmatter (security restriction). Flag as FAIL if present.
- [ ] **maxTurns**: If present, verify it is a positive integer. Flag unreasonably high values (>100) as WARN.
- [ ] **Cross-references**: If the agent references skills, verify those skills exist in the plugin's `skills/` directory.

### 5. Hooks Validation

If neither `hooks/hooks.json` nor inline hooks in `plugin.json` exist, AND `hooks/` directory does not exist or is empty, mark SKIPPED.

- [ ] **hooks.json syntax**: If `hooks/hooks.json` exists, verify it is valid JSON.
- [ ] **Hook event names**: Verify all hook event keys are recognized event names (see reference.md for the valid set).
- [ ] **Handler structure**: For each hook handler, verify:
  - `type` field is present and is one of: `command`, `http`, `prompt`, `agent`
  - `matcher` field (if present) is a valid regex string
- [ ] **Command handlers**: For `type: command` handlers:
  - Verify the referenced script path exists
  - Check scripts use `${CLAUDE_PLUGIN_ROOT}` or `${CLAUDE_PLUGIN_DATA}` for paths (not hardcoded absolute paths) — critical for plugin portability
  - Verify scripts are executable (if on a Unix-like system)
- [ ] **HTTP handlers**: For `type: http` handlers, verify the URL is well-formed.
- [ ] **Inline hooks in plugin.json**: If `plugin.json` contains a `hooks` field (inline format), apply the same checks as above.
- [ ] **Orphan scripts**: If `hooks/` directory contains `.sh` or other script files not referenced by any hook configuration, flag as WARN.
- [ ] **Duplicate hooks**: If hooks are defined in BOTH `hooks/hooks.json` AND inline in `plugin.json`, flag as WARN — potential for confusion or conflicts.

### 6. MCP & LSP Validation

If neither `.mcp.json` nor `.lsp.json` exist (and no inline config in `plugin.json`), mark SKIPPED.

**MCP servers:**

If `.mcp.json` does not exist and `plugin.json` does not define `mcpServers`, skip this subsection.

- [ ] **JSON syntax**: Verify `.mcp.json` is valid JSON.
- [ ] **Server entries**: For each server entry, verify:
  - Server has a recognizable structure (command-based or URL-based)
  - For command-based servers: the `command` executable can reasonably be expected to exist (note: runtime dependencies may not be present during static analysis — flag only clearly broken paths)
  - Script paths use `${CLAUDE_PLUGIN_ROOT}` or `${CLAUDE_PLUGIN_DATA}` for portability
- [ ] **Channel references**: If `plugin.json` defines `channels`, verify each channel's `server` field matches a defined MCP server key.
- [ ] **Inline vs file**: If MCP servers are defined in both `.mcp.json` and inline in `plugin.json`, flag as WARN.

**LSP servers:**

If `.lsp.json` does not exist and `plugin.json` does not define `lspServers`, skip this subsection.

- [ ] **JSON syntax**: Verify `.lsp.json` is valid JSON.
- [ ] **Required fields**: Each server entry must have `command` (string) and `extensionToLanguage` (object mapping extensions to language identifiers).
- [ ] **Extension format**: Verify extension keys start with `.` (e.g., `.go`, `.py`).

### 7. Cross-Component Consistency

Verify that all plugin components reference each other correctly:

- [ ] **Skill→Agent references**: If any skill references an agent (by name or file), verify the agent exists in `agents/`.
- [ ] **Command→Agent references**: If any command references an agent, verify it exists in `agents/`.
- [ ] **Agent→Skill references**: If any agent's `skills` field references skills, verify they exist in `skills/`.
- [ ] **Hook→Script references**: All hook script paths resolve to existing files.
- [ ] **Channel→MCP references**: All channel `server` fields match MCP server keys.
- [ ] **Manifest→Component consistency**: If `plugin.json` specifies custom component paths, verify those paths contain the expected component files.
- [ ] **Namespace consistency**: Verify skill/command folder names don't conflict with each other (since both can create slash commands under the same plugin namespace).
- [ ] **settings.json alignment**: If `settings.json` exists at plugin root, verify any `permissions.allow` entries reference valid tools and any hooks defined there don't conflict with `hooks/hooks.json`.

### 8. Best Practices (Advisory)

Use WebSearch to check for recent Claude Code plugin best practices. **Restrict searches to these domain-prefixed queries only:**
- `site:code.claude.com plugins` — plugin creation and structure guidance
- `site:code.claude.com plugins-reference` — plugin technical reference
- `site:code.claude.com plugin-marketplaces` — marketplace and distribution
- `site:code.claude.com skills` — skill configuration and frontmatter
- `site:code.claude.com hooks` — hook configuration patterns
- `site:code.claude.com agents` — agent configuration and frontmatter
- `site:anthropic.com/engineering Claude Code plugins` — engineering blog posts

If all searches return zero results, note: "Documentation domains may have changed.
Skipping best practices comparison." and mark SKIPPED.

**Deduplication rule**: Before reporting any advisory, check the project's existing documentation (`README.md`, `CONTRIBUTING.md`, `CHANGELOG.md`, etc.) and verify the item is not already addressed. Do not report advisories for items that are already present or documented in the project's workflow (e.g., do not suggest adding a `CHANGELOG.md` if one already exists; do not suggest running `claude plugin validate .` if it is already listed as a step in `CONTRIBUTING.md`).

Compare the plugin's configuration against documented recommendations:

- [ ] **Plugin structure**: Does the plugin follow the standard layout?
- [ ] **Skill vs command preference**: Are new skills using the `skills/` directory (recommended) rather than legacy `commands/`?
- [ ] **Portability**: Do all paths use `${CLAUDE_PLUGIN_ROOT}` or `${CLAUDE_PLUGIN_DATA}` environment variables?
- [ ] **Validation support**: Can the plugin be validated with `claude plugin validate .`?
- [ ] **New features**: Are there recently documented plugin features the plugin could adopt?

For search results that mention specific new features or configuration patterns but lack sufficient detail in the snippet, use WebFetch to read the full page. Limit to at most 3 page fetches. Prefer official documentation over blog posts.

**This section is advisory only.** Label all findings as informational, never FAIL. Include source URLs so the user can verify independently.

## Output

```markdown
# Claude Code Plugin Health Check

**Date**: YYYY-MM-DD
**Plugin**: (name from plugin.json, or directory name)
**Reviewer**: Claude Code /config-doctor:check
**Review iterations**: N

## Summary

| Section | Status | Issues |
|---------|--------|--------|
| 0. Manifest | ✅ PASS / ⚠️ WARN / ❌ FAIL | count |
| 1. Directory Structure | ✅ PASS / ⚠️ WARN / ❌ FAIL | count |
| 2. Skills | ✅ PASS / ⚠️ WARN / ❌ FAIL / ⏭️ SKIPPED | count |
| 3. Commands | ✅ PASS / ⚠️ WARN / ❌ FAIL / ⏭️ SKIPPED | count |
| 4. Agents | ✅ PASS / ⚠️ WARN / ❌ FAIL / ⏭️ SKIPPED | count |
| 5. Hooks | ✅ PASS / ⚠️ WARN / ❌ FAIL / ⏭️ SKIPPED | count |
| 6. MCP & LSP | ✅ PASS / ⚠️ WARN / ❌ FAIL / ⏭️ SKIPPED | count |
| 7. Cross-Component Consistency | ✅ PASS / ⚠️ WARN / ❌ FAIL / ⏭️ SKIPPED | count |
| 8. Best Practices | ℹ️ ADVISORY / ⏭️ SKIPPED | count |

## Recommended Actions
1. [❌ FAIL — Section N] Description of the issue and recommended fix
2. [⚠️ WARN — Section N] Description of the issue and recommended fix
3. [ℹ️ ADVISORY — Section N] Description of the suggestion
```
