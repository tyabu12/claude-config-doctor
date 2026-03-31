# 🩺 claude-config-doctor

[English](./README.md)

Claude Code の設定ファイル群のヘルスチェックを行う[スキル](https://code.claude.com/docs/ja/skills)です。

構造的な lint にとどまらず、CLAUDE.md / rules / commands / skills / hooks / settings 間の意味的な矛盾を検出します。

## なぜ config-doctor か

一般的な Linter はファイルの存在、frontmatter の構文、命名規則をチェックします。
config-doctor は ClaudeCode のスキルを使うことで、その先を検査します。

| | Linter | claude-config-doctor |
| --- | --- | --- |
| Frontmatter・JSON 構文 | ✅ | ✅ |
| ファイル・ディレクトリの存在 | ✅ | ✅ |
| **意味的整合性**（hooks と permissions の矛盾など） | ❌ | ✅ |
| **ファイル間の矛盾検出**（CLAUDE.md ↔ rules ↔ settings） | ❌ | ✅ |
| **Insights 統合**（friction パターン → 設定改善提案） | ❌ | ✅ |
| **ベストプラクティス同期**（実行時に最新の公式ドキュメントを取得） | ❌ | ✅ |

### claude-md-management との違い

Anthropic 公式の [claude-md-management](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/claude-md-management) は CLAUDE.md の品質改善に特化したプラグインです。

| | config-doctor | claude-md-management |
| --- | --- | --- |
| 対象範囲 | 設定ファイル**全体**（CLAUDE.md / rules / commands / skills / agents / hooks / settings） | **CLAUDE.md のみ** |
| 目的 | 健全性チェック・クロスファイル整合性検証 | 品質監査・改善・セッション学習の反映 |
| 読み取り/書き込み | **読み取り専用**（レポート出力のみ） | **読み取り+編集**（ユーザー承認後に修正） |
| 提供機能 | スキル 1 つ（`/config-doctor:check`） | スキル 1 つ + コマンド 1 つ（`/revise-claude-md`） |

両者は競合ではなく補完関係にあります。config-doctor で設定全体を**診断**し、claude-md-management で CLAUDE.md を**改善**する使い方が効果的です。

---

なおこのスキルはセキュリティ上の観点から、レポートを出力するだけなので読み取り専用になっています。
ファイルは一切変更されません。

診断結果が問題なさそうであれば、「指摘を元に設定を更新して」とそのままプロンプトに打てばあとは Claude Code がいい感じに設定を最適化してくれます。簡単ですね！

### 出力例

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
1. [❌ FAIL — Section 3] コマンド `deploy.md` が存在しないエージェント `deployer.md` を参照している
2. [❌ FAIL — Section 6] `rm -rf` が permissions.allow に含まれているが、PreToolUse hook でブロックされている
3. [⚠️ WARN — Section 1] CLAUDE.md が 247 行 — rules への分割を検討
4. [⚠️ WARN — Section 5] 孤立スクリプト `.claude/hooks/old-lint.sh` が settings に未接続
```

---

## インストール

marketplace を追加してプラグインをインストールしてください。

```shell
/plugin marketplace add tyabu12/claude-config-doctor
/plugin install config-doctor@tyabu12-claude-config-doctor
```

<details>
<summary>手動インストール（プラグイン方式を使わない場合）</summary>

[`skills/check/SKILL.md`](./skills/check/SKILL.md) をプロジェクトに直接コピーしてください:

```bash
mkdir -p .claude/skills/config-doctor
curl -fsSL https://raw.githubusercontent.com/tyabu12/claude-config-doctor/main/skills/check/SKILL.md \
  -o .claude/skills/config-doctor/SKILL.md
```

手動でインストールした場合、コマンドは `/config-doctor:check` ではなく `/config-doctor` になります。

</details>

## 使い方

> **Tip**: フルチェックの前に `/insights` を実行して最新のセッションデータに更新してください。より正確な結果が得られます。

```shell
# 月1回想定: フルチェック（ベストプラクティス検索、`/insights` レポート分析）
/config-doctor:check

# ライトモード: 構造チェックのみ
/config-doctor:check light
```

<details>
<summary>Dogfooding: /self-check (`/config-doctor:check` と同じ)</summary>

```shell
❯ /self-check

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

⏺ Found skills to review. Reading skill files.

  Read 2 files (ctrl+o to expand)

⏺ All skill frontmatter and content validated. Now I have everything needed for the report.

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
  │ 3. Commands & Skills      │ ✅ PASS     │ 0      │
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

  Commands: Skipped — .claude/commands/ does not exist.

  Skills:

  .claude/skills/self-check/SKILL.md
  - SKILL.md exists: ✅ PASS
  - Frontmatter syntax: ✅ PASS
  - Tool permissions: ✅ PASS — Read, Glob, Grep, Bash
  - description length: ✅ PASS — 74 characters
  - model: ✅ PASS — opus
  - Procedure accuracy: ✅ PASS

  skills/check/SKILL.md (plugin skill, reviewed per override)
  - SKILL.md exists: ✅ PASS
  - Frontmatter syntax: ✅ PASS
  - Tool permissions: ✅ PASS — Read, Glob, Grep, WebSearch, WebFetch, Bash, Agent
  - description length: ✅ PASS — 95 characters
  - model: ✅ PASS — opus
  - Encoding: ✅ PASS — UTF-8
  - Procedure accuracy: ✅ PASS
  - Review Loop pattern: ✅ PASS

  Status: ✅ PASS — All skills pass frontmatter, tool permissions, and content checks.

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

## チェック内容

スキルは下記の9つのレビューセクションからなります。

| セクション | チェック内容 |
| --- | --- |
| 0. Global Config | `~/.claude/` のグローバル設定、`.claudeignore` の妥当性 |
| 1. CLAUDE.md | 行数、エンコーディング、パス正確性、Tech Stack の乖離、陳腐化 |
| 2. Rules | Frontmatter、glob パターン、内容の正確性、CLAUDE.md との整合性 |
| 3. Commands & Skills | 構文、ツール権限、手順の正確性、サポートファイル、Agent の相互参照 |
| 4. Agents | 必須フィールド、ツールリスト、モデル値、評価基準 |
| 5. Hooks | 孤立スクリプト、matcher の正確性、スクリプトロジック、exit code、frontmatter hooks |
| 6. Cross-file | JSON の妥当性、権限の矛盾、hook と permission の整合性 |
| 7. Best Practices | 実行時に最新の Anthropic 公式ドキュメントを取得し、設定と比較 |
| 8. Insights | `/insights` の friction データを集約し、設定改善を提案 |

FAIL 項目はサンドボックス化されたサブエージェントによるクロスレビュー（最大1回）を経て、最終レポートに反映されます。

## セキュリティ

ユーザーのローカルのセッションデータを読み取るため、
カスタムスキルは実用性を担保しつつ、可能な限りセキュリティに配慮しています。

- **完全に読み取り専用** — ファイルを一切変更しない
- **プロンプトインジェクション防御** — 設定ファイルや Web コンテンツはすべて分析対象のデータとして扱い、指示として実行しない
- **Web アクセスの制限** — WebSearch と WebFetch は Anthropic 公式ドメインのハードコードされた許可リストに限定。プロジェクト固有の情報が検索クエリに含まれることはない
- **出力のサニタイズ** — 指摘事項はファイルパスと行番号で参照し、内容をそのまま引用しない。レポートを共有した際の二次インジェクションを防止
- **サブエージェントのサンドボックス** — クロスレビュー用サブエージェントは `Read, Glob, Grep` のみに制限
- **Insights のプライバシー** — セッションデータは抽象的な推奨事項に集約され、セッション固有の詳細は含まれない

## 要件・制限事項

- **使用モデル**: チェックの精度を確保するため、スキルでは自動的に Opus モデルを使用します
- Full モードは最新のベストプラクティスも検索するため `WebSearch`、`WebFetch`、`Agent` ツールも使用します（セクション 7-8）。`light` モードは `Read`、`Glob`、`Grep`、`Bash`、`Agent` のみです
- **読み取り専用** — 設定ファイルを一切変更しません。自動修正機能もありません
- **実行時間**: Full レビューは5分程度かかります。`light` モードはより高速ですが最新のベストプラクティスの検索と `/insights` レポートの分析をスキップします
- **アンインストール**: `/plugin uninstall config-doctor@tyabu12-claude-config-doctor`（手動インストールの場合は `.claude/skills/config-doctor/` を削除）

## コントリビュート

[CONTRIBUTING.md](./CONTRIBUTING.md) を参照してください。

## ライセンス

[MIT](./LICENSE)
