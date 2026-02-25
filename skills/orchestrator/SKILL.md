---
name: orchestrator
description: 階層型エージェントチームシステム。複雑なタスクをPhase Manager(PM-M)に委譲。実装・調査・多ステップ・品質重視タスクで自動発火。
---

# Orchestrator Skill

## 核心原則: PMコンテキスト最小化
* **PM役割**: プロジェクトマネージャー。意思決定(Go/Redo)と対話に専念。
* **分離**: PMはコードを直接触らない(`Read/Glob/Grep/Edit/Bash`禁止)。全操作をPM-Mへ委譲。
* **永続化**: 各Phaseの結果は`{task_memory}/phase-[a-c]-summary.md`に保存。PMは`SendMessage`の要約のみで判断。

## 実行フロー (Skip禁止)
1. **P0 (PM)**: MCP検出(`serena/context7/devin`), メモリ初期化, `05_log.md`記録。
2. **Phase A (偵察/戦略)**: `TeamCreate` -> PM-M起動。グループ分割案を決定。
3. **Phase B (調査/設計)**: `TeamCreate` -> PM-M+Sub-team(Res/Arc)。Pre-Assessor評価必須。
4. **Phase C (実装/QA)**: `TeamCreate` -> PM-M+Sub-team(Eng/QA)。品質チェック・テスト通過必須。
5. **Confirm (PM)**: `60_decisions.md`を精査。`AskUserQuestion`で全決定の承認を得る(必須)。
6. **Audit/Eval**: 3ロール以上関与で`Auditor`必須。勧告あれば`definition`更新。

## ロール定義 (Roles)
* **Phase Manager (PM-M)**: フェーズ実行責任者。サブチームを統括。
* **Fixed**: `Pre-Assessor` (事前評価), `Eng-Manager` (評価・構造レビュー)。
* **Aux (Haiku)**: `scout` (偵察), `strategist` (戦略), `evaluator` (評価), `librarian` (知識管理)。
* **Dynamic**: `researcher` (調査), `architect` (設計), `engineer` (実装/Sonnet), `qa-reviewer` (品質)。
* **Audit**: `auditor` (独立監査)。

## 判断基準 (Go/Redo)
* **Go**: ステータス完了、ブロッカー無し、意思決定記録、ハンドオフ十分。
* **Redo**: ステータス不備、品質未達、要修正。※同一フェーズ2回まで、以降は`AskUserQuestion`。

## MCP優先 & フォールバック
* **Priority**: MCPツール > 標準ツール。
* **Bug Fix**: サブエージェントでMCP利用不可時は、標準ツール(Grep/Web等)へ自動フォールバック。

## メモリ構成
`{task_memory} = {memory_dir}/memory/YYMMDD_<task>/`
* `05_log.md`: PMログ / `phase-[a-c]-summary.md`: 各フェーズ詳細
* `60_decisions.md`: 意思決定集約 / `99_history.md`: 判断記録

## PM-M サマリースキーマ (SendMessage)
1. **ステータス**: 完了/要redo/ブロッカー
2. **主要成果 & 意思決定**: (ID, 内容, 根拠)
3. **変更ファイル**: (Phase Cのみ)
4. **次へのハンドオフ**: ファイルパス、文脈、リスク
5. **推奨アクション**: Go/Redo/Ask