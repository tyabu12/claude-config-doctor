# Contributing to claude-config-doctor

Contributions are welcome — thank you.
This project is a single skill file, so contributing is straightforward.

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

### Project structure note

This project is a **plugin** (has `.claude-plugin/`), but also has a `.claude/` directory. This is intentional:

- `.claude/settings.local.json` — local development permissions (gitignored)
- `.claude/skills/self-check/` — a project-level skill that runs the plugin's own `/config-doctor:check` against itself. This is a development tool, not part of the distributed plugin.

The `/self-check` diagnostic flags this as WARN because `.claude/` alongside `.claude-plugin/` can indicate confusion. In this project it is expected.

### Submitting changes

1. Fork the repository
2. Create a branch (`git checkout -b improve-section-5`)
3. Make your changes to `SKILL.md`
4. Run `/self-check` and confirm all checks PASS
5. Run `claude plugin validate .` to verify the plugin manifest and component structure
6. Test by running `claude --plugin-dir /path/to/claude-config-doctor` in a real project and executing `/config-doctor:check`
7. Test against a plugin project as well (any repo with `.claude-plugin/plugin.json`) to verify plugin diagnostics
8. Open a PR with a clear description of what changed and why

## Guidelines

### Keep it simple

The entire tool is one skill file. This is intentional. Changes should maintain this simplicity.

### Security constraints matter

The safety constraints (read-only, prompt injection defense, scoped web access, output sanitization) are core to this project's design. PRs that weaken these will not be merged.

### Test with real projects

Since this is a Claude Code skill, automated testing isn't straightforward. Please verify your changes by running the skill against at least one real project before submitting.
