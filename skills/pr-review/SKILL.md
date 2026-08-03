---
name: pr-review
description: PRレビュー。PR番号・ブランチ名指定時またはレビュー依頼時に使用。Claude Codeとadvisor(fable5)のマルチモデルレビューでCritical/High/Medium分類の指摘を報告。
---

# PRレビュー

## トリガー条件

- PRレビューを依頼された場合
- PR番号またはブランチ名が指定された場合

## 実行手順

### 1. PR情報の取得

```bash
# メモリディレクトリを用意（advisorに渡すファイルの置き場）
mkdir -p ${MEMORY_DIR}/memory/YYMMDD_pr-review-<番号>

# PR詳細を取得
gh pr view <番号> --json title,body,author,headRefName,baseRefName,files \
  > ${MEMORY_DIR}/memory/YYMMDD_pr-review-<番号>/pr-<番号>.json

# diffを取得
gh pr diff <番号> > ${MEMORY_DIR}/memory/YYMMDD_pr-review-<番号>/pr-<番号>.diff
```

### 2. PJルールの確認

CLAUDE.mdを読み、以下を把握:
- アーキテクチャルール
- 命名規則
- コーディング規約

### 3. Claude Codeによるレビュー

変更された各ファイルについて:
1. Readツールでファイル全体を読む（diffだけでなく）
2. 既存パターンとの整合性を確認
3. 問題点を特定

### 4. advisorによるレビュー

別モデル（fable5）の観点を追加。agent teams の teammate としてスポーンする。
詳細: @context/advisor-review-guide.md

**前提**: agent teamsが有効であること。無効なら `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` の設定を案内して停止する。

```
Agent(
  name: "advisor-pr-<番号>-1",
  subagent_type: "advisor",
  model: "fable",
  description: "PR #<番号> のレビュー",
  prompt: "以下のPRをレビューしてください:
- コードの品質（バグ、セキュリティ、パフォーマンス）
- 設計の妥当性
- テストの十分性
- ドキュメントの必要性

レビュー結果は必ず SendMessage で lead に送ってください。指摘がなければ「指摘なし」とだけ回答してください。

## 読むべきファイル（絶対パス）
- PR diff: ${MEMORY_DIR}/memory/YYMMDD_pr-review-<番号>/pr-<番号>.diff
- PR情報（タイトル・本文・変更ファイル）: ${MEMORY_DIR}/memory/YYMMDD_pr-review-<番号>/pr-<番号>.json
- プロジェクト規約: <リポジトリ>/CLAUDE.md

diffだけで判断できない場合は、変更されたファイルの全体を読んで既存パターンとの整合性を確認してください。"
)
```

**CRITICAL: diffの全文をプロンプトに貼らない。絶対パスで渡してadvisorに読ませること。**

### 5. 結果の統合

Claude Codeとadvisorの両方の指摘を統合し、以下の形式でレポート:

```markdown
# PRレビュー: #<番号>

## 対象
- PR: #<番号> - <タイトル>
- ブランチ: <ブランチ> → <ベース>

## Claude Codeによる指摘

### Critical
- [指摘内容]

### High
- [指摘内容]

## advisor (fable5) による指摘

### Critical
- [指摘内容]

### High
- [指摘内容]

## 統合サマリ

| 観点 | Claude Code | advisor | 統合判断 |
|------|------------|---------|---------|
| セキュリティ | OK | OK | OK |
| パフォーマンス | 指摘あり | OK | 要確認 |

## 推奨アクション
1. [アクション1]
2. [アクション2]

## マージ推奨: [Yes/No/条件付き]
```

## 問題の分類

| 分類 | 説明 | 対応 |
|-----|------|------|
| Critical | バグ、セキュリティ、破壊的変更 | マージ前に必須修正 |
| High | アーキテクチャ違反、テスト不足 | 強く推奨 |
| Medium | 命名、コメント | 改善推奨 |
| Good | 良い実装 | 賞賛 |
