# プロンプトテンプレート集

PM（メインセッション）がTeamCreateでPhase Managerをteam memberとしてspawnし、Phase Managerが自律的にサブロールを管理するためのテンプレート。

**CRITICAL: プロンプト圧縮原則**
サブエージェントでは複雑なプロンプトによりMCPツールが利用不可になるバグがある（[#13605](https://github.com/anthropics/claude-code/issues/13605), [#14496](https://github.com/anthropics/claude-code/issues/14496)）。
そのため、ロール定義はインライン展開せず「ファイルパスを渡して読ませる」方式を使用する。

## Phase Manager起動テンプレート（PM専用）

PMはTeamCreateでチームを作成し、Phase Managerをteam memberとしてspawnする。Phase Managerは自律的にフェーズを完遂し、サマリーをSendMessageでPMに送信する。

### Phase A EM（偵察・戦略: Steps 0-1）

```
# Step 1: チーム作成
TeamCreate(team_name="orch-phase-a")

# Step 2: Phase Manager spawn
Task(
  subagent_type="general-purpose",
  team_name="orch-phase-a",
  name="phase-a-em",
  description="Phase A EM: 偵察・戦略",
  prompt="あなたはPhase A EM（偵察・戦略）です。

## 最初にやること
1. `~/.claude/skills/orchestrator/roles/phase-em.md` を読み、ロール定義に従う
2. `~/.claude/teams/orch-phase-a/config.json` を読み、PM（リーダー）の名前を確認

## チーム情報
- チーム名: orch-phase-a
- あなたの名前: phase-a-em

## 担当: Phase A（偵察・戦略）Steps 0-1
- Step 0: librarian→scout×N（並列）→結果統合
- Step 1: strategist×N（並列）→グループ分割決定
- ロール定義は `~/.claude/skills/orchestrator/roles/` から読む

## 完了時
1. `{task_memory}/phase-a-summary.md` にフル結果書き込み
2. `{task_memory}/60_decisions.md` に意思決定追記
3. サマリーをSendMessageでPMに送信
4. shutdown_requestまたはRedo指示を待機

## タスク
{ユーザーからのタスク全文}

## メモリディレクトリ
{task_memory}

## MCP
{MCP一覧。なければ「なし」。MCPが利用不可の場合は標準ツールにフォールバック}

## 過去資料
{ナレッジマネージャー結果。初回なら「なし」}"
)

# Step 3: PMはPhase Managerからのサマリーメッセージを待つ（自動配信）
# Go判断後:
SendMessage(type="shutdown_request", recipient="phase-a-em", content="Phase A complete")
# Phase Managerのshutdown_response（approve）を待つ → 承認後にTeamDelete
TeamDelete()
# Redo判断の場合:
SendMessage(type="message", recipient="phase-a-em", content="Redo: {フィードバック}", summary="Phase A Redo")
```

### Phase B EM（調査・設計: Steps 2-5）

```
# Step 1: チーム作成
TeamCreate(team_name="orch-phase-b")

# Step 2: Phase Manager spawn
Task(
  subagent_type="general-purpose",
  team_name="orch-phase-b",
  name="phase-b-em",
  description="Phase B EM: 調査・設計",
  prompt="あなたはPhase B EM（調査・設計）です。

## 最初にやること
1. `~/.claude/skills/orchestrator/roles/phase-em.md` を読み、ロール定義に従う
2. `~/.claude/teams/orch-phase-b/config.json` を読み、PM（リーダー）の名前を確認

## チーム情報
- チーム名: orch-phase-b
- あなたの名前: phase-b-em

## 担当: Phase B（調査・設計）Steps 2-5
- Steps 2-4: TeamCreate(サブチーム)→researcher+architect→協調実行→結果収集→TeamDelete
- Step 5: Pre-Assessor（one-shot）で事前評価
- ロール定義は `~/.claude/skills/orchestrator/roles/` から読む

## 完了時
1. `{task_memory}/phase-b-summary.md` にフル結果書き込み
2. `{task_memory}/60_decisions.md` に意思決定追記
3. サマリーをSendMessageでPMに送信
4. shutdown_requestまたはRedo指示を待機

## タスク
{ユーザーからのタスク全文}

## メモリディレクトリ
{task_memory}

## 前フェーズ
Phase A サマリー: {task_memory}/phase-a-summary.md（読んで偵察結果を把握）

## MCP
{MCP一覧。なければ「なし」。MCPが利用不可の場合は標準ツールにフォールバック}"
)

# Step 3: PMはPhase Managerからのサマリーメッセージを待つ（自動配信）
# Go判断後:
SendMessage(type="shutdown_request", recipient="phase-b-em", content="Phase B complete")
# Phase Managerのshutdown_response（approve）を待つ → 承認後にTeamDelete
TeamDelete()
# Redo判断の場合:
SendMessage(type="message", recipient="phase-b-em", content="Redo: {フィードバック}", summary="Phase B Redo")
```

### Phase C EM（実装・QA: Steps 6-7）

```
# Step 1: チーム作成
TeamCreate(team_name="orch-phase-c")

# Step 2: Phase Manager spawn
Task(
  subagent_type="general-purpose",
  team_name="orch-phase-c",
  name="phase-c-em",
  description="Phase C EM: 実装・QA",
  prompt="あなたはPhase C EM（実装・QA）です。

## 最初にやること
1. `~/.claude/skills/orchestrator/roles/phase-em.md` を読み、ロール定義に従う
2. `~/.claude/teams/orch-phase-c/config.json` を読み、PM（リーダー）の名前を確認

## チーム情報
- チーム名: orch-phase-c
- あなたの名前: phase-c-em

## 担当: Phase C（実装・QA）Steps 6-7
- TeamCreate(サブチーム)→engineer+qa-reviewer(+tech-writer,devops)→実装・QA反復→品質チェック→結果収集→TeamDelete
- ロール定義は `~/.claude/skills/orchestrator/roles/` から読む

## 完了時
1. `{task_memory}/phase-c-summary.md` にフル結果書き込み
2. `{task_memory}/60_decisions.md` に意思決定追記
3. サマリーをSendMessageでPMに送信
4. shutdown_requestまたはRedo指示を待機

## タスク
{ユーザーからのタスク全文}

## メモリディレクトリ
{task_memory}

## 前フェーズ
Phase A サマリー: {task_memory}/phase-a-summary.md
Phase B サマリー: {task_memory}/phase-b-summary.md
（読んで調査結果と設計方針を把握）

## MCP
{MCP一覧。なければ「なし」。MCPが利用不可の場合は標準ツールにフォールバック}"
)

# Step 3: PMはPhase Managerからのサマリーメッセージを待つ（自動配信）
# Go判断後:
SendMessage(type="shutdown_request", recipient="phase-c-em", content="Phase C complete")
# Phase Managerのshutdown_response（approve）を待つ → 承認後にTeamDelete
TeamDelete()
# Redo判断の場合:
SendMessage(type="message", recipient="phase-c-em", content="Redo: {フィードバック}", summary="Phase C Redo")
```

---

## Redo フィードバックテンプレート（PM専用）

Phase Managerのredoが必要な場合、PMがSendMessageで使用するテンプレート。

```
# 同一チーム内のPhase ManagerにRedo指示を送信
SendMessage(
  type="message",
  recipient="{phase-em-name}",
  content="## Redo指示: Phase {X}

### 前回のサマリーファイル
{task_memory}/phase-{x}-summary.md（このファイルを読み、前回の結果を把握してください）

### 問題点
- {具体的な問題1}
- {具体的な問題2}

### 修正方針
- {全再実行 / 部分再実行（対象ステップ: ...）}

### PMからのフィードバック
{具体的な指示・方向性}",
  summary="Phase {X} Redo指示"
)
```

---

## PM直接実行テンプレート（Steps 9-12）

### 進化（Steps 9-10、条件付き）

```
# ncr処理（ncrがある場合）
Task(
  subagent_type="general-purpose",
  model="inherit",
  description="NCR評価",
  prompt="あなたはEng-Managerです。
`~/.claude/skills/orchestrator/roles/eng-manager.md` を読み、ロール定義に従ってください。

以下のncrを評価してください:
{ncr一覧}"
)

# 構造レビュー（5ロール以上関与/新グループ設置時）
Task(
  subagent_type="general-purpose",
  model="inherit",
  description="構造レビュー",
  prompt="あなたはEng-Managerです。
`~/.claude/skills/orchestrator/roles/eng-manager.md` を読み、ロール定義に従ってください。

以下のタスクで使用されたチーム構造をレビューしてください:
{チーム構造の概要}"
)
```

### 監査（Step 11、条件付き）

```
Task(
  subagent_type="general-purpose",
  model="inherit",
  description="独立監査",
  prompt="あなたは監査担当(Auditor)です。
`~/.claude/skills/orchestrator/roles/auditor.md` を読み、ロール定義に従ってください。

以下のタスクを監査してください。

## タスク概要
{ユーザーのタスク}

## フェーズサマリー
Phase A: {task_memory}/phase-a-summary.md
Phase B: {task_memory}/phase-b-summary.md
Phase C: {task_memory}/phase-c-summary.md

## 意思決定ログ
{task_memory}/60_decisions.md

上記ファイルを読み、監査レポートを作成してください。"
)
```

### 自己改善（Step 12、条件付き）

```
# 定義ファイル更新
Task(
  subagent_type="general-purpose",
  model="sonnet",
  description="定義ファイル更新",
  prompt="あなたはEngineerです。
`~/.claude/skills/orchestrator/roles/engineer.md` を読み、ロール定義に従ってください。

監査担当からの自己改善勧告に基づき、定義ファイルを更新してください。

## 勧告内容
{definition勧告}

## 更新対象
{対象ファイルパス}"
)

# 更新レビュー
Task(
  subagent_type="general-purpose",
  model="inherit",
  description="定義更新レビュー",
  prompt="あなたはQA Reviewerです。
`~/.claude/skills/orchestrator/roles/qa-reviewer.md` を読み、ロール定義に従ってください。

自己改善による定義ファイル更新をレビューしてください。

## 更新内容
{変更差分}"
)
```

---

## Phase Manager内部で使用するテンプレート

Phase Managerが自律的に使用する。PMは関与しない。

### サブロール起動（one-shot）

```
Task(
  subagent_type="general-purpose",
  model="{model}",
  prompt="あなたは{ロール名}です。
`~/.claude/skills/orchestrator/roles/{role}.md` を読み、ロール定義に従ってください。

---

{タスク固有の指示}

## MCP
{MCP一覧。なければ「なし」。MCPが利用不可の場合は標準ツールにフォールバック}"
)
```

### サブロール起動（team member / サブチーム）

```
# Phase B EMの場合:
TeamCreate(team_name="orch-phase-b-investigate")
Task(
  subagent_type="general-purpose",
  model="{model}",
  team_name="orch-phase-b-investigate",
  name="{role_name}",
  prompt="あなたは{ロール名}です。
以下のファイルを読み、ロール定義とチームプロトコルに従ってください:
1. `~/.claude/skills/orchestrator/roles/{role}.md`
2. `~/.claude/skills/orchestrator/roles/_team-protocol.md`

---

{タスク固有の指示}

## MCP
{MCP一覧。なければ「なし」。MCPが利用不可の場合は標準ツールにフォールバック}"
)

# Phase C EMの場合:
TeamCreate(team_name="orch-phase-c-implement")
Task(
  subagent_type="general-purpose",
  model="{model}",
  team_name="orch-phase-c-implement",
  name="{role_name}",
  prompt="..."  # 同様の構造
)
```

### MCP情報テンプレート

チームメンバーへの依頼時に含める:

```
## MCP（利用可能な場合は標準ツールより優先）
{利用可能なMCPツール一覧。なければ「なし」}

| MCP      | 主要機能                                         | 用途                               | 代替標準ツール      |
| -------- | ------------------------------------------------ | ---------------------------------- | ------------------- |
| serena   | find_definition, find_references, get_hover_info | シンボル定義ジャンプ、参照箇所特定 | Grep, Glob          |
| context7 | resolve-library-id, get-library-docs             | ライブラリ公式ドキュメント取得     | WebSearch, WebFetch |
| devin    | (タスク実行)                                     | 外部リサーチ                       | WebSearch           |

MCPが利用可能な場合は優先して使用する。
**ただし、サブエージェント環境ではMCPが利用不可になる場合がある（既知のバグ）。**
MCPが利用不可の場合は標準ツールにフォールバックすること（理由明記不要）。
```

### 3軸検索指示テンプレート（Researcher向け）

```
## 調査方法（必須）
以下の3軸で調査してください:
1. **問題軸**: 「{問題の説明}」で検索
2. **機能軸**: 「{欲しい機能}」で検索
3. **カタログ軸**: {フレームワーク}の公式Widget/APIカタログを網羅的に確認

## 影響範囲分析（共通コンポーネント変更時は必須）
変更対象が複数箇所で使用されている場合:
- **全使用箇所の一覧**を報告してください
```

### 事前評価依頼テンプレート（Pre-Assessor向け）

```
## コスト評価依頼
- 新規コード行数: {行数}
- 新規クラス/enum数: {数}
- 低レベルAPI使用: {あり/なし}
- 新規ユーティリティ: {あり/なし}
- 共通コンポーネント変更: {あり/なし}
- QA対象箇所数: {箇所数}
```

### QAレビュー依頼テンプレート（QA Reviewer向け）

```
## レビュー基準（必須確認）
1. **簡潔性**: もっとシンプルに書ける方法はないか
2. **公式API活用**: フレームワークの公式Widget/APIで代替できないか
3. **コード量**: 要件に対してコードが多すぎないか
4. **リグレッション防止**: バグ修正の場合、テストが追加されているか
```

### フェーズ間ハンドオフテンプレート

```
## シャットダウン前の報告依頼

TeamDeleteの前に、以下を報告してください:
1. **decisions**: 意思決定の一覧（id, summary, rationale, alternatives）
2. **arc.saved**: 資料館に保存したファイルパスの一覧
3. **未解決事項**: 次フェーズに引き継ぐべき課題

上記を報告後、shutdown_requestに応答してください。
```
