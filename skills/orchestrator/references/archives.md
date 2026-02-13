# 資料館（Archives）システム

## 概要

各チームメンバーの調査結果・設計ドキュメント・決定事項を永続化し、将来のコアチームでも活用できるようにする仕組み。

## ナレッジマネージャー(Librarian)

資料館には**ナレッジマネージャー(Librarian)**が常駐し、資料の検索・インデックス管理を担当。
**コアチームに属さない独立した存在**であり、全てのロールにサービスを提供する。

呼び出し: `Task(subagent_type="general-purpose", model="haiku", prompt="$(cat roles/librarian.md)\n\nJWTに関する過去の調査はありますか？")`

詳細: roles/librarian.md

## ディレクトリ構造

```yaml
directory: "${MEMORY_DIR}/"
structure:
  archives/:
    index.json: メインインデックス（ナレッジマネージャーが毎回自動更新）
    synonyms.json: 同義語辞書（ナレッジマネージャーが自動蓄積）
    research/: 調査レポート
    design/: 設計ドキュメント
    decisions/: 意思決定記録（ADR）
    patterns/: 発見したパターン
    reviews/: レビュー結果
    docs/: 文書担当の成果物
    ops/: 運用手順
    audit/: 監査レポート
    deprecated/: 統合済み・古い資料
```

## 命名規則

**CRITICAL: 常に新規ファイルとして出力。既存ファイルへの上書きは禁止。**

```
{トピック}-{YYMMDD}-{HHMMSS}.md
例: research/jwt-authentication-260128-143052.md
```

| ルール | 説明 |
|-------|------|
| 常に新規作成 | 同じトピックでも新規ファイルとして出力 |
| タイムスタンプ必須 | HHMMSS（時分秒）を含めてユニーク性を保証 |
| 上書き禁止 | 既存ファイルを編集・上書きしない |
| 整理はナレッジマネージャー | ファイルの整理・統合はナレッジマネージャーが担当 |

## 全チームメンバーのナレッジ出力ルール（CRITICAL）

**チームメンバーは常に新規ファイルとして出力する。既存ファイルの上書き・編集は禁止。**

| チームメンバー | 保存先カテゴリ |
|---------------|-------------|
| 調査担当(Researcher) | archives/research/ |
| 設計担当(Architect) | archives/design/ |
| 実装担当(Engineer) | archives/patterns/ |
| QA担当(QA Reviewer) | archives/reviews/ |
| 文書担当(Tech Writer) | archives/docs/ |
| 運用担当(DevOps) | archives/ops/ |
| 監査担当(Auditor) | archives/audit/ |

**出力判断: 「迷ったら出力する」が原則。**

出力しなくてよいケース: 既存資料を参照しただけで新しい知見がない場合のみ。

## オーケストレーターの役割

1. **タスク開始時**: ナレッジマネージャーに問い合わせて関連資料を検索
2. **チームメンバーへの依頼時**: 関連資料を `ref` で共有
3. **タスク完了後**: ナレッジマネージャーに整理を依頼

## 資料館の保守

- EMが定期的に資料館をレビュー（古い情報の更新、重複統合）
- 資料館は `.local/` 内にあり、通常gitignoreされる
- 重要な資料はプロジェクトのドキュメントに昇格させる
