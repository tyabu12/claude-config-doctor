---
name: self-check
description: Self-check for config-doctor's own project. Use when modifying config-doctor itself.
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Bash
model: opus
---

# /self-check

Read `skills/check/SKILL.md` and execute its review procedure against this project's directory.

**Override**: The self-skip rule in Skills Validation does not apply.

All other safety constraints in the SKILL.md (read-only, prompt injection defense, output quoting, etc.) remain in effect.
