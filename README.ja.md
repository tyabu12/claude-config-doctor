# 🩺 claude-config-doctor

[English](./README.md)

Claude Code の設定ファイル群のヘルスチェックを行う[スキル](https://code.claude.com/docs/ja/skills)です。

構造的な lint にとどまらず、CLAUDE.md / rules / commands / skills / hooks / settings 間の意味的な矛盾を検出します。[プラグイン](https://code.claude.com/docs/en/plugins)プロジェクトや[マーケットプレイス](https://code.claude.com/docs/en/plugin-marketplaces)リポジトリ（マニフェスト検証、ディレクトリ構造、コンポーネント間整合性など）にも対応しています。

<div align="center">

https://github.com/user-attachments/assets/10ca5a0a-607f-4c48-9a26-40f028cb40a4

</div>

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
| 決定論的な出力 | ✅ | ❌（LLM ベース） |

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
# マーケットプレイスを追加
/plugin marketplace add tyabu12/claude-config-doctor

# プラグインをインストール
/plugin install config-doctor@tyabu12-claude-config-doctor

# プラグインをリロードして有効化
/reload-plugins
```

<details>
<summary>手動インストール（プラグイン方式を使わない場合）</summary>

[`skills/check/`](./skills/check/) 配下のスキルファイルをプロジェクトに直接コピーしてください:

```bash
mkdir -p .claude/skills/config-doctor
for f in SKILL.md plugin.md project.md reference.md; do
  curl -fsSL "https://raw.githubusercontent.com/tyabu12/claude-config-doctor/main/skills/check/$f" \
    -o ".claude/skills/config-doctor/$f"
done
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

## チェック内容

プロジェクトの種類（通常プロジェクト、プラグイン、マーケットプレイス）を自動検出し、適切な診断を実行します。

### 通常プロジェクト（`.claude/` 設定）

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

<details>
<summary>プラグインプロジェクト（`.claude-plugin/plugin.json`）</summary>

| セクション | チェック内容 |
| --- | --- |
| 0. Manifest | JSON 構文、必須フィールド、名前形式、バージョン、メタデータ、userConfig、channels、marketplace.json |
| 1. Directory Structure | アンチパターン、コンポーネントディレクトリ、README、パストラバーサル、`.claude/` の混在 |
| 2. Skills | SKILL.md の存在、frontmatter、description、ツール権限、`$ARGUMENTS`、`${CLAUDE_PLUGIN_ROOT}` |
| 3. Commands | Frontmatter、description、ツール権限、skills との重複 |
| 4. Agents | 必須フィールド、対応フィールド、model、tools、isolation、セキュリティ制約 |
| 5. Hooks | hooks.json 構文、イベント名、ハンドラ構造、スクリプトのポータビリティ、孤立スクリプト |
| 6. MCP & LSP | JSON 構文、サーバエントリ、ポータビリティ、channel 参照、拡張子形式 |
| 7. Cross-Component | Skill↔Agent 参照、hook→script 参照、channel→MCP 参照、名前空間の競合 |
| 8. Best Practices | 実行時に最新の Anthropic プラグインドキュメントを取得し、設定と比較 |

</details>


<details>
<summary>マーケットプレイスリポジトリ（`.claude-plugin/marketplace.json` のみ、`plugin.json` なし）</summary>

マーケットプレイスリポジトリは自動検出され、`marketplace.json` に記載された各ローカルプラグインに対して上記のプラグイン診断がフルで実行されます。リモートプラグインエントリはソース構造の検証のみ行われます。

</details>

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

## FAQ

### チェック結果が実行のたびに微妙に異なります。正常ですか？

はい。config-doctor は LLM ベースのため、結果は非決定的であり、実行ごとに表現の揺れは発生します。再実行で消える指摘はグレーゾーンの可能性が高いため、ヒント程度に捉えてください。

### `full` と `light`、どちらを使うべきですか？

- **`light`** は構造チェックのみ（`Read`、`Glob`、`Grep`、`Bash`、`Agent`）。高速で日常的な利用に最適です。
- **`full`**（デフォルト）はベストプラクティス検索と `/insights` 分析を追加（`WebSearch`、`WebFetch`、`Agent`）。約5分かかります。トークンを一定消費してしまうため、一定期間ごとや大きな設定変更の際の診断を推奨します。

チェックの精度を確保するため、どちらのモードでも自動的に **Opus モデル**を使用します。

### 見つかった問題を自動修正できますか？

いいえ。config-doctor はセキュリティの観点から厳しく**読み取り専用**になっており、ファイルを一切変更しません。レポートを確認した後、プロンプトに「この指摘を元に設定を更新して」と入力すれば、Claude Code が修正を適用してくれます。

### このプラグイン自体のメンテナンスや品質は大丈夫ですか？

はい。config-doctor は自分自身に対してもチェックを実行しています（dogfooding）。開発リポジトリには `/self-check` コマンドがあり、`/config-doctor:check` を自身のプラグイン構造に対して実行します。

<details>
<summary>セルフチェックの実行結果（Phase 1 のみ）</summary>

```shell
❯ /self-check

  Claude Code Plugin Health Check

  Date: 2026-04-01
  Plugin: config-doctor
  Reviewer: Claude Code /config-doctor:check
  Review iterations: 0

  Summary

  ┌────────────────────────────────┬─────────────┬────────┐
  │            Section             │   Status    │ Issues │
  ├────────────────────────────────┼─────────────┼────────┤
  │ 0. Manifest                    │ ✅ PASS     │ 0      │
  ├────────────────────────────────┼─────────────┼────────┤
  │ 1. Directory Structure         │ ⚠️  WARN     │ 1      │
  ├────────────────────────────────┼─────────────┼────────┤
  │ 2. Skills                      │ ✅ PASS     │ 0      │
  ├────────────────────────────────┼─────────────┼────────┤
  │ 3. Commands                    │ ⏭️  SKIPPED  │ 0      │
  ├────────────────────────────────┼─────────────┼────────┤
  │ 4. Agents                      │ ⏭️  SKIPPED  │ 0      │
  ├────────────────────────────────┼─────────────┼────────┤
  │ 5. Hooks                       │ ⏭️  SKIPPED  │ 0      │
  ├────────────────────────────────┼─────────────┼────────┤
  │ 6. MCP & LSP                   │ ⏭️  SKIPPED  │ 0      │
  ├────────────────────────────────┼─────────────┼────────┤
  │ 7. Cross-Component Consistency │ ✅ PASS     │ 0      │
  ├────────────────────────────────┼─────────────┼────────┤
  │ 8. Best Practices              │ ℹ️  ADVISORY │ 1      │
  └────────────────────────────────┴─────────────┴────────┘

  Section Details

  0. Manifest Validation — ✅ PASS

  plugin.json:
  - JSON syntax: valid
  - Required field name: present, non-empty string ("config-doctor")
  - Name format: kebab-case — valid
  - Version "1.1.0": valid semver
  - Description: present, non-empty string
  - Author: object with name field — valid
  - Optional metadata: repository (valid URL string), license (string), keywords (array of strings) — all valid
  - No component path overrides, userConfig, or channels defined
  - No settings.json at plugin root
  - No unknown top-level fields

  marketplace.json:
  - JSON syntax: valid
  - Required fields: name, owner, plugins — all present
  - Name format "tyabu12-claude-config-doctor": kebab-case — valid
  - Owner: has name field — valid
  - Metadata: description present (string) — valid
  - Plugin entry: name "config-doctor" (kebab-case), source "./" (starts with ./, resolves to existing directory) — valid
  - Description: present, non-empty string — valid
  - Plugin name consistency: marketplace entry "config-doctor" matches plugin.json "config-doctor"; descriptions match
  - No unknown top-level fields

  1. Directory Structure — ⚠️  WARN (1)

  - Anti-pattern check: .claude-plugin/ contains only plugin.json and marketplace.json — no component directories inside. PASS
  - Component directories: skills/ exists at root with skill content. No commands/, agents/, hooks/, output-styles/, .mcp.json, or .lsp.json — expected for this plugin's scope
  - README.md: exists at plugin root. PASS
  - Path traversal: no component files reference paths outside the plugin root. PASS
  - ⚠️  WARN: Stray .claude/ directory exists alongside .claude-plugin/. Contains settings.local.json (local dev permissions) and skills/self-check/ (development-only
  self-diagnostics). CONTRIBUTING.md (lines 29–33) documents this as intentional. Documented — WARN acknowledged as expected.

  2. Skills Validation — ✅ PASS

  Skill: skills/check/ (self-skip rule overridden per /self-check instructions)

  - SKILL.md exists: yes
  - Frontmatter syntax: well-formed YAML
  - description: present, ~195 characters (under 250 limit)
  - allowed-tools: Read, Glob, Grep, WebSearch, WebFetch, Bash, Agent — all valid per reference.md
  - model: opus — valid
  - disable-model-invocation: true — valid boolean
  - argument-hint: "[light | full]" — valid string
  - Supporting files: plugin.md, project.md, reference.md — all referenced and all exist in the skill directory
  - $ARGUMENTS usage: argument-hint is defined and $ARGUMENTS is referenced in SKILL.md and delegated procedure files
  - Script references: none (no scripts to check)

  3. Commands — ⏭️  SKIPPED

  No commands/ directory exists.

  4. Agents — ⏭️  SKIPPED

  No agents/ directory exists.

  5. Hooks — ⏭️  SKIPPED

  No hooks/hooks.json, no inline hooks in plugin.json, no hooks/ directory.

  6. MCP & LSP — ⏭️  SKIPPED

  No .mcp.json, .lsp.json, or inline MCP/LSP config in plugin.json.

  7. Cross-Component Consistency — ✅ PASS

  - Skill→Agent references: none (no agents referenced)
  - Command→Agent references: no commands
  - Agent→Skill references: no agents
  - Hook→Script references: no hooks
  - Channel→MCP references: no channels
  - Manifest→Component consistency: no custom component paths in manifest; default skills/ directory contains expected skill files
  - Namespace consistency: only skills/ exists, no commands/ — no naming conflicts
  - settings.json alignment: no settings.json at plugin root

  8. Best Practices (Advisory) — ℹ️  ADVISORY (1)

  Compared against official documentation from code.claude.com (plugins, plugins-reference, skills, hooks pages).

  - Plugin structure: follows standard layout. PASS
  - Skill vs command preference: uses skills/ directory (recommended). PASS
  - Portability: no scripts or hardcoded paths — not applicable. PASS
  - Validation support: claude plugin validate . is documented in CONTRIBUTING.md step 5. PASS
  - ℹ️  ADVISORY: Official docs document newer plugin features including output styles (output-styles/), LSP servers (.lsp.json), channels, userConfig, and settings.json with
  agent key for default agent activation. These are not relevant to this plugin's current scope but could be adopted if future development warrants them.

  Sources: https://code.claude.com/docs/en/plugins-reference, https://code.claude.com/docs/en/plugins, https://code.claude.com/docs/en/skills,
  https://code.claude.com/docs/en/hooks

  Recommended Actions

  1. [⚠️  WARN — Section 1] .claude/ directory exists alongside .claude-plugin/. Documented as intentional in CONTRIBUTING.md — no action needed unless the project structure
  changes.
  2. [ℹ️  ADVISORY — Section 8] Consider evaluating newer plugin features (output styles, LSP servers, channels, userConfig) if future development warrants them.
```

</details>

### アンインストール方法は？

```shell
/plugin uninstall config-doctor@tyabu12-claude-config-doctor
```

手動インストールの場合は `.claude/skills/config-doctor/` を削除してください。

## 関連ツール

### `/doctor`（組み込みコマンド）

Claude Code の組み込みコマンド `/doctor` はインストール状態や設定の基本診断を行います。config-doctor はその先にある**セマンティックな設定分析**（ファイル間の矛盾検出、ベストプラクティス比較、Insights 連携など）をカバーします。環境に問題がないか `/doctor` で確認してから、config-doctor で設定を最適化するのがおすすめです。

### claude-md-management

Anthropic 公式の [claude-md-management](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/claude-md-management) プラグインは CLAUDE.md の品質監査・改善を行います。config-doctor が設定全体を**診断**（読み取り専用）し、claude-md-management が CLAUDE.md を**編集**（ユーザー承認後）する補完関係です。

### rulesync

[rulesync](https://github.com/dyoshikawa/rulesync) は Claude Code、Cursor、Copilot、Gemini CLI など多数の AI コーディングアシスタントの設定を、単一のソースから一括生成する CLI ツールです。config-doctor は rulesync が生成した Claude Code の設定を検証し、仕様の変更や意味的な問題を検出できます。rulesync で**生成**し、config-doctor で**検証**する使い方が効果的です。

## コントリビュート

[CONTRIBUTING.md](./CONTRIBUTING.md) を参照してください。

## ライセンス

[MIT](./LICENSE)
