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

なおこのスキルはセキュリティ上の観点から、レポートを出力するだけなので読み取り専用になっています。
ファイルは一切変更されません。

指摘事項を修正するには、Recommended Actions を Claude Code に渡すか、手動で対応してください。

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

```bash
claude plugin add github:tyabu12/claude-config-doctor
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

```
# 月1回想定: フルチェック（ベストプラクティス検索、`/insights` レポート分析）
/config-doctor:check

# ライトモード: 構造チェックのみ
/config-doctor:check light
```

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
- **アンインストール**: `claude plugin remove config-doctor`（手動インストールの場合は `.claude/skills/config-doctor/` を削除）

## コントリビュート

[CONTRIBUTING.md](./CONTRIBUTING.md) を参照してください。

## ライセンス

[MIT](./LICENSE)
