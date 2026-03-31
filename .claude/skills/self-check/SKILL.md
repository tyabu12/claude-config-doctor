---
name: self-check
description: Self-check for config-doctor's own SKILL.md. Use when modifying config-doctor itself.
disable-model-invocation: true
allowed-tools: Read, Glob, Grep, Bash
model: opus
---

# /self-check

Read `skills/check/SKILL.md` and execute its review procedure against this project's `skills/` directory.

**Override**: The self-skip rule in Section 3 and 8 does not apply. Review `skills/check/SKILL.md` as a normal skill target.

All other safety constraints in the SKILL.md (read-only, prompt injection defense, output quoting, etc.) remain in effect.
