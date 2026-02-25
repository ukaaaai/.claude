# チームメンバー共通プロトコル

Phase ManagerがTeamCreateで作成したチームに参加するメンバー向けの共通ルール。
spawn時にこのファイルの内容がプロンプトに注入される。

## コミュニケーション

### メッセージ送信（SendMessage）
- **チームリーダー（Phase Manager）への報告**: `SendMessage(type: "message", recipient: "{leader-name}", content: "...", summary: "...")`
  - leader-nameはチームconfig `~/.claude/teams/{team_name}/config.json` を読み、リーダーの名前を確認して使用する（TeamCreateにより自動生成される）
- **チームメイトへの連絡**: `SendMessage(type: "message", recipient: "{teammate-name}", content: "...", summary: "...")`
- メッセージは自動配信される（ポーリング不要）
- 作業完了・ブロック発生・重要な発見時に必ず報告

### メッセージの内容ルール
- 構造化JSONステータスメッセージは送信しない（プレーンテキストで通信）
- 簡潔に要点を伝える
- 必要な場合のみ詳細を含める

## タスク管理

### タスクの確認と更新
- `TaskList` で利用可能なタスクを確認
- タスク開始時: `TaskUpdate(taskId, status: "in_progress")`
- タスク完了時: `TaskUpdate(taskId, status: "completed")`
- 完了後は `TaskList` で次のタスクを確認

### タスクの優先順位
- ID順（小さい順）で処理
- blockedByが空でないタスクは開始不可
- 依存タスクの完了を待つ

## シャットダウン

### shutdown_request への対応
1. `shutdown_request` を受信
2. 未完了の作業がなければ: `SendMessage(type: "shutdown_response", request_id: "{id}", approve: true)`
3. 未完了の作業がある場合: `SendMessage(type: "shutdown_response", request_id: "{id}", approve: false, content: "理由")`

### シャットダウン前のハンドオフ（必須）
shutdown承認前に以下を必ず報告:
1. **decisions**: 下した意思決定の一覧（id, summary, rationale, alternatives）
2. **arc.saved**: 資料館に保存したファイルパスの一覧
3. **未解決事項**: 次フェーズに引き継ぐべき課題

## ツール統制

### 推奨ツール/禁止ツール
各ロール定義に記載された「推奨ツール」「禁止ツール」に従うこと。
**禁止ツールを使用した場合、その作業は無効とみなされる。**

### MCP優先原則
利用可能なMCPツールを常に標準ツールより優先する。
ただしサブエージェント環境ではMCPが利用不可の場合がある（既知のバグ）。
利用不可の場合は標準ツールにフォールバックすること。

## 共有概念

| 概念        | 用途                                                                    | 報告方法       |
| ----------- | ----------------------------------------------------------------------- | -------------- |
| `arc`       | ナレッジ追跡。`arc.saved`=保存した資料パス、`arc.used`=参照した資料パス | テキストで報告 |
| `ncr`       | 新機能リクエスト。type=skill/role/group、name、reason                   | テキストで報告 |
| `decisions` | 意思決定。id、summary、rationale、alternatives                          | テキストで報告 |
