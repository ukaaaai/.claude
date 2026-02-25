# 資料館（Archives）システム仕様書

## 概要
チームの調査・設計・決定事項を永続化し、将来の活用を担保する仕組み。

## ナレッジマネージャー (Librarian)
- **役割**: 資料の検索・インデックス管理を行う独立エージェント。
- **特性**: 全ロールにサービスを提供。
- **呼出例**: `Task(subagent_type="general-purpose", model="haiku", prompt="$(cat roles/librarian.md)\n\n[検索クエリ]")`

## ディレクトリ構造 (${MEMORY_DIR}/archives/)
- `index.json`: メインインデックス（自動更新）
- `synonyms.json`: 同義語辞書（自動蓄積）
- `research/`: 調査レポート
- `design/`: 設計ドキュメント
- `decisions/`: 意思決定記録 (ADR)
- `patterns/`: 発見したパターン
- `reviews/`: レビュー結果
- `docs/`: 文書成果物
- `ops/`: 運用手順
- `audit/`: 監査レポート
- `deprecated/`: 旧資料・統合済み

## 命名・出力規則 (CRITICAL)
**原則: 既存ファイルの上書き・編集は一切禁止。常に新規ファイルで出力すること。**

- **形式**: `{トピック}-{YYMMDD}-{HHMMSS}.md`
- **例**: `research/jwt-auth-260128-143052.md`
- **保存先ルール**:
  - Researcher -> `research/`
  - Architect -> `design/`
  - Engineer -> `patterns/`
  - QA Reviewer -> `reviews/`
  - Tech Writer -> `docs/`
  - DevOps -> `ops/`
  - Auditor -> `audit/`
- **判断基準**: 「迷ったら出力」。新規知見がない参照のみの場合を除き、全成果物を保存。

## 運用フロー
1. **開始時**: Phase ManagerがLibrarianへ関連資料を照会。
2. **依頼時**: 関連資料を `ref` でメンバーに共有。
3. **完了後**: Librarianに情報の整理・インデックス更新を依頼。
4. **保守**: EMによる定期レビュー、重要資料のプロジェクトドキュメントへの昇格。