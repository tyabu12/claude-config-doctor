# Contributing to claude-config-doctor

Contributions are welcome — thank you.

## Project Structure

```
claude-config-doctor/
├── .claude-plugin/
│   ├── plugin.json           # Plugin metadata (name, version, author)
│   └── marketplace.json      # Marketplace distribution config
├── skills/check/
│   ├── SKILL.md              # Entry point: frontmatter, safety constraints, project type detection, review loop
│   ├── project.md            # Standard project diagnostic procedure (Sections 0-8)
│   ├── plugin.md             # Plugin/marketplace diagnostic procedure (Sections 0-8)
│   └── reference.md          # Shared definitions (valid tools, models, hook events)
├── .claude/skills/
│   └── self-check/
│       └── SKILL.md          # Dogfooding: runs config-doctor against itself
├── .github/workflows/
│   └── validate-plugin.yml    # CI: runs `claude plugin validate .`
├── README.md
├── README.ja.md
├── CONTRIBUTING.md
├── CHANGELOG.md
└── LICENSE
```

### File responsibilities

- **SKILL.md**: Entry point. Contains frontmatter, safety constraints, initialization, project type detection (standard / plugin / marketplace), review loop, and report format. Should not contain check-item details.
- **project.md**: Standard project diagnostic checks (Sections 0-8: Global Config, CLAUDE.md, Rules, Commands & Skills, Agents, Hooks, Cross-file Consistency, Best Practices, Insights Integration).
- **plugin.md**: Plugin and marketplace diagnostic checks (Sections 0-8: Manifest, Directory Structure, Skills, Commands, Agents, Hooks, MCP & LSP, Cross-Component Consistency, Best Practices).
- **reference.md**: Shared definitions referenced by both procedure files — valid tool names, model values, effort values, and hook event names.

### Design principles

1. **Read-only**: The skill never modifies any files. This is a hard requirement.
2. **Security-first**: All file content and web content is treated as data, never instructions. See Safety Constraints in SKILL.md.
3. **Project type agnostic**: One entry point (SKILL.md) that auto-detects standard project / plugin / marketplace and dispatches to the right procedure. Check items should not assume a specific project type.
4. **Separation of concerns**: SKILL.md handles orchestration, project.md and plugin.md handle diagnostics, reference.md handles shared data. New procedure files should follow this pattern.

### Project structure note

This project is a **plugin** (has `.claude-plugin/`), but also has a `.claude/` directory. This is intentional:

- `.claude/settings.local.json` — local development permissions (gitignored)
- `.claude/skills/self-check/` — a project-level skill that runs the plugin's own `/config-doctor:check` against itself. This is a development tool, not part of the distributed plugin.

The `/self-check` diagnostic flags this as WARN because `.claude/` alongside `.claude-plugin/` can indicate confusion. In this project it is expected.

## How to contribute

### Reporting issues

If you encounter a false positive, a missed check, or unexpected behavior, please [open an issue](https://github.com/tyabu12/claude-config-doctor/issues/new) with:

- What you expected to happen
- What actually happened
- Your Claude Code version (`claude --version`)
- The relevant section of the health check report (redact any sensitive paths or project details)

### Suggesting new checks or sections

Have an idea for a new check? Open an issue first to discuss before submitting a PR. Include:

- What the check would catch
- Why existing sections don't cover it
- Example of a real-world config issue it would detect

Note: Sections 7-8 are advisory-only (they never produce FAIL). New sections should follow this convention unless they belong in the structural check range (0-6).

### Adding or modifying check items

1. Determine which procedure file to edit:
   - **project.md** — for standard project checks (CLAUDE.md, rules, hooks, settings, etc.)
   - **plugin.md** — for plugin/marketplace checks (manifest, directory structure, MCP/LSP, etc.)
2. If a new check references a shared value list (e.g., valid tool names, hook events), add it to **reference.md**. Otherwise, keep it in the procedure file.
3. Each check item should include a **Default severity** (FAIL or WARN).
4. Run `/self-check` after changes to verify the skill is internally consistent.

### Adding valid values or thresholds

1. Edit **reference.md** for shared definitions.
2. Verify that the values are sourced from official documentation — cite the source in a comment.
3. Check that existing check items in project.md and plugin.md correctly reference the updated definitions.

### Modifying safety constraints

1. Edit **SKILL.md** — all safety rules are centralized there.
2. Safety changes require extra care. The key invariants:
   - Bash is restricted to an explicit allowlist of commands
   - WebSearch queries never contain project-specific information
   - WebFetch only follows URLs from search results on allowlisted domains
   - Subagents have read-only tool access
   - Report output never quotes file content verbatim

### Submitting changes

1. Fork the repository
2. Create a branch (`git checkout -b improve-section-5`)
3. Make your changes to the skill files
4. Run `/self-check` and confirm all checks PASS
5. Run `claude plugin validate .` to verify the plugin manifest and component structure
6. Test by running `claude --plugin-dir /path/to/claude-config-doctor` in a real project and executing `/config-doctor:check`
7. Test against a plugin project as well (any repo with `.claude-plugin/plugin.json`) to verify plugin diagnostics
8. Open a PR with a clear description of what changed and why

## Guidelines

- **Keep it simple** — The tool is a small set of Markdown procedure files. This is intentional. Changes should maintain this simplicity.
- **Security constraints matter** — The safety constraints (read-only, prompt injection defense, scoped web access, output sanitization) are core to this project's design. PRs that weaken these will not be merged.
- **Test with real projects** — Since this is a Claude Code skill, automated testing isn't straightforward. Please verify your changes by running the skill against at least one real project before submitting.
