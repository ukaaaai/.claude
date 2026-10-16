# Claude Code User Settings

Claude Codeのuser-level設定ファイル集。プロジェクト横断で使用するワークフロー、スキル、コマンドを定義。

## 使い方

```bash
# ~/.claude/ にクローン
git clone <this-repo> ~/.claude

# または既存の~/.claude/にマージ

# token最適化ツールのrtkをインストール
brew install rtk
```

## 構成

```
~/.claude/
├── CLAUDE.md              # グローバル設定（ワークフロー、変数）
├── agents/                # サブエージェント・teammate定義
│   ├── advisor.md         # レビュー担当（fable5）。advisor reviewでスポーン
│   ├── cursor-codegen.md  # cursor agentによるコード生成
│   ├── cursor-refactor.md # cursor agentによるリファクタ
│   └── cursor-test-gen.md # cursor agentによるテスト生成
├── commands/              # ユーザー実行コマンド
│   ├── commit.md          # /commit
│   └── pr.md              # /pr
├── skills/                # 自動トリガースキル
│   ├── codebase-review/
│   ├── create-skill/
│   ├── database-migration/
│   ├── documentation/
│   ├── large-task/
│   ├── pr-review/
│   ├── project-init/
│   ├── project-sync/
│   ├── ui-ux-design/
│   └── update-inst/
└── templates/             # プロジェクト初期化テンプレート
    └── project/
```

## スキル一覧

| スキル | 説明 | トリガー |
|--------|------|----------|
| **codebase-review** | 6観点（perf/sec/test/arch/cq/docs）で並列レビュー | `/codebase-review`、品質監査依頼時 |
| **pr-review** | Claude + advisor(fable5)のマルチモデルレビュー | PRレビュー依頼時 |
| **project-init** | CLAUDE.md・.claude/の初期設定 | PJ初期化依頼時 |
| **project-sync** | CLAUDE.mdとcontext/の整合性確保 | ドキュメント整理依頼時 |
| **documentation** | コード変更に伴うドキュメント更新 | API/環境変数追加検出時 |
| **database-migration** | ORM検出、マイグレーション作成支援 | スキーマ変更依頼時 |
| **large-task** | 複数セッションにまたがる大規模タスク分割 | 大規模実装時 |
| **create-skill** | 既存設定と整合したスキル自動作成 | `/create-skill <内容>` |
| **update-inst** | 間違いの再発防止ルール追加 | `/update-inst <間違えた内容>` |
| **ui-ux-design** | プロダクショングレードのUI/UX生成 | UI構築依頼時 |

## コマンド一覧

| コマンド | 説明 | 引数 |
|----------|------|------|
| `/commit` | git-cz形式でコミット | `--push`: コミット後push |
| `/pr` | Draft PR作成 | `[base-branch]`: マージ先 |

## ワークフロー

CLAUDE.mdで定義された6フェーズワークフロー:

1. **Phase 0: 準備** - メモリディレクトリ作成、過去タスク検索
2. **Phase 1: 調査** - context7/WebSearch必須、既存コード確認
3. **Phase 2: 計画** - advisor reviewで検証（指摘なくなるまで）
4. **Phase 3: 実装** - 調査→計画→実行→レビューの4ステップ
5. **Phase 4: 品質確認** - lint/format/typecheck/test + advisor review
6. **Phase 5: 完了報告**

## メモリディレクトリ

各タスクの作業ログを `.local/memory/YYMMDD_<task_name>/` に保存:

```
.local/
├── memory/
│   └── YYMMDD_<task>/
│       ├── 05_log.md      # 作業ログ（必須）
│       ├── 30_plan.md     # 実装計画
│       └── ...
└── issues/                # codebase-reviewで生成
```

## advisor review（agent teams）

別モデル（fable5）の **advisor** をagent teamsのteammateとしてスポーンしてレビューを実施:

```
Agent(name: "advisor-code-1", subagent_type: "advisor", model: "fable", prompt: "<レビュー依頼>")
```

- 定義: `agents/advisor.md`（`model: fable`。指摘は must/recommend/nit 付き）
- レビュー対象は**絶対パスで渡す**（advisorはメインセッションの会話履歴を継承しない）
- 修正すべき点がなくなるまでループ
- 再レビューは `SendMessage(to: "advisor-code-1", ...)` でコンテキスト継続
- 前提: `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`
- 詳細: `context/advisor-review-guide.md`

## プロジェクト設定

新規プロジェクトでは `/project-init` を実行、または `templates/project/CLAUDE.md` をコピー:

```markdown
# <プロジェクト名>

## 変数
MEMORY_DIR=.local/
BASE_BRANCH=develop

## 品質チェック
npm run lint
npm run format
npm run typecheck
npm test

## 特記事項
- [PJ固有のルール]
```

## ライセンス

MIT
