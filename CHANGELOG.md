# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [1.1.1] - 2026-04-01

### Security

- Harden prompt injection defenses across multiple input surfaces
- Strengthen subagent launch constraints
- Tighten output sanitization rules for untrusted data sources

## [1.1.0] - 2026-04-01

### Added

- Plugin project diagnostics — automatically detects `.claude-plugin/plugin.json` and runs plugin-specific checks (manifest, directory structure, skills, agents, hooks, MCP/LSP, cross-component consistency)
- Marketplace diagnostics — validates `marketplace.json` manifest and supports marketplace-only repos with per-plugin sub-checks
- Best Practices sections now include agents documentation search

## [1.0.0] - 2026-03-31

- Initial release
