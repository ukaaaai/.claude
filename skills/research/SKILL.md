---
name: research
description: 調査用サブエージェントによる技術・コードベース調査。context7/devin/serena MCPを優先利用し、使えない場合はWebSearchで代替。大規模調査は複数サブエージェントを階層型オーケストレーションで実行。
---

# Research Skill

技術調査・コードベース調査を構造化されたプロセスで実行する。
MCPの利用可能状況を動的に検出し、最適なツールを選択する。

## 自動発動条件

**以下のキーワードを検出したら自動的にこのスキルを発動する:**

| カテゴリ | キーワード |
|----------|-----------|
| 日本語 | 調査, 調べて, リサーチ, 検索して, 探して, 確認して, 比較して |
| 英語 | research, investigate, look up, find out, search for, explore |
| 文脈 | ベストプラクティス, 最新の方法, どうやって, 仕様を確認, ドキュメント |

**発動例:**
- 「Reactの最新のベストプラクティスを調査して」→ 自動発動
- 「この実装方法を調べてほしい」→ 自動発動
- 「GraphQLの仕様を確認して」→ 自動発動
- 「Next.js 15の変更点をリサーチ」→ 自動発動

## 調査深度の自動判断

**明示的な指定がない場合、文脈から自動判断する:**

| 深度 | 判断基準 |
|------|----------|
| `quick` | 単純な質問、既知情報の確認、1トピック |
| `medium` | 複数の観点が必要、比較検討、2-3トピック |
| `thorough` | 広範な調査、複数ライブラリ比較、実装戦略検討 |

**自動判断のヒント:**
- 「ざっくり」「簡単に」「概要」→ quick
- 「詳しく」「比較して」「どれがいいか」→ medium
- 「徹底的に」「すべて」「完全に」「網羅的に」→ thorough
- 明示的な指定がなければ → medium（デフォルト）

## エスカレーション機能

**調査中に以下の状況を検出したら、自動的に調査規模を拡大する:**

### エスカレーション条件

1. **トピックの広がり検出**
   - 調査中に関連トピックが3つ以上見つかった
   - 当初の想定より複雑な依存関係がある

2. **情報量の増大**
   - 1つのサブトピックで10件以上の重要な発見
   - 矛盾する情報が複数ソースから出現

3. **深掘りの必要性**
   - 表面的な情報では不十分と判断
   - ユーザーの質問に十分答えられない

### エスカレーション時の動作

```
【エスカレーション検出】

現在の調査で以下の状況を検出しました:
- {検出理由}

調査規模を拡大します:
- 現在: {current_depth} → 拡大後: {new_depth}
- 追加サブエージェント: {count}

続行しますか？ [Y/n]
```

**自動エスカレーションフロー:**
1. quick → medium: サブエージェント2-3体を追加起動
2. medium → thorough: サブエージェント4-6体に拡大、並列実行

### 手動エスカレーション

調査中にユーザーが以下を指示した場合もエスカレーション:
- 「もっと詳しく」
- 「深掘りして」
- 「他の観点も」
- 「徹底的に調べて」

## 引数

```
/research <topic> [options]

Options:
  --type <type>       調査タイプ: technical, codebase, comprehensive (default: comprehensive)
  --depth <level>     深さ: quick, medium, thorough (default: medium)
  --libraries <libs>  対象ライブラリ（カンマ区切り）
  --dirs <paths>      対象ディレクトリ（カンマ区切り）
```

## 実行フロー

### Phase 0: 準備

1. 引数をパース
2. メモリディレクトリを作成（`${MEMORY_DIR}/memory/YYMMDD_research-<topic>/`）
3. 05_log.mdを初期化

### Phase 1: ツール検出

利用可能なMCPとツールを検出する。

**検出対象:**
| ツール | 用途 | 検出方法 |
|--------|------|----------|
| context7 | ライブラリドキュメント | `resolve-library-id`ツールの存在確認 |
| devin | 外部リサーチ | devin関連ツールの存在確認 |
| serena | LSPベースのコード分析 | serena関連ツールの存在確認 |
| WebSearch | Web検索 | 常に利用可能 |
| Explore | コードベース探索 | 常に利用可能（Task tool経由） |

**検出ロジック:**
```
利用可能ツールを確認し、以下の優先順位で使用:
1. MCP（context7, devin, serena）があれば優先利用
2. MCPがなければWebSearch/Exploreで代替
```

### Phase 2: 調査計画

調査トピックを分析し、サブタスクに分割する。

**分割基準:**
- `--depth quick`: 分割なし、単一エージェントで実行
- `--depth medium`: 2-3サブタスクに分割
- `--depth thorough`: 4-6サブタスクに分割、並列実行

**サブエージェント種別:**
| 種別 | 役割 | 優先ツール |
|------|------|-----------|
| docs-researcher | ライブラリドキュメント調査 | context7 → WebSearch |
| code-explorer | コードベース調査 | serena → Explore |
| web-researcher | 一般的なWeb調査 | WebSearch |

### Phase 3: 調査実行

**単一エージェント（quick）:**
オーケストレーターが直接調査を実行。

**複数サブエージェント（medium/thorough）:**
Taskツールで並列実行し、結果を統合。

**CRITICAL**: サブエージェントは `subagent_type=general-purpose` を使用。

#### サブエージェント共通プロンプトテンプレート

```
あなたは「{サブエージェント種別}」の専門調査員です。

## コンテキスト
- 調査トピック: {topic}
- メモリディレクトリ: {memory_dir}
- 利用可能ツール: {available_tools}

## あなたの担当
{担当範囲の詳細}

## タスク

### 1. ツール選択
利用可能なツールから最適なものを選択:
- context7が使えるなら: resolve-library-id → query-docs
- serenaが使えるなら: LSP機能を活用
- 上記が使えないなら: WebSearch/Read/Grep/Glob

### 2. 調査実行
- 複数ソースから情報を収集
- 信頼度を評価（公式ドキュメント > ブログ記事 > フォーラム）
- 矛盾する情報があれば両方記録

### 3. 結果記録
以下の形式でJSON出力:

{
  "topic": "{担当トピック}",
  "findings": [
    {
      "content": "発見内容",
      "source": "情報源（URL/ファイルパス/MCP名）",
      "confidence": "high|medium|low"
    }
  ],
  "tools_used": ["使用したツール名"],
  "recommendations": ["推奨事項"]
}

## 注意事項
- 推測ではなく、実際に確認した情報のみ記録
- 情報源を必ず記録
- 古い情報（2年以上前）は信頼度を下げる
```

#### docs-researcher サブエージェント

```
## あなたの担当: ライブラリドキュメント調査

### 調査項目
- 公式ドキュメントからのAPI仕様
- ベストプラクティス・推奨パターン
- バージョン固有の注意事項
- サンプルコード

### ツール優先順位
1. context7（利用可能な場合）
   - resolve-library-id: ライブラリIDを取得
   - query-docs: ドキュメントをクエリ
2. WebSearch
   - 「{library} {topic} documentation 2025 2026」で検索
   - 公式サイトを優先

### 信頼度基準
- high: 公式ドキュメント、context7の結果
- medium: 信頼できるブログ（著名な開発者）
- low: Stack Overflow、古い記事
```

#### code-explorer サブエージェント

```
## あなたの担当: コードベース調査

### 調査項目
- 既存実装パターンの特定
- 関連ファイル・関数の列挙
- 依存関係の分析
- 設計パターンの把握

### ツール優先順位
1. serena（利用可能な場合）
   - LSP機能で定義ジャンプ、参照検索
2. Glob/Grep/Read
   - パターンマッチでファイル検索
   - 関連コードを読解

### 出力形式
- 発見したファイル: パス:行番号
- 実装パターン: 具体的なコード引用
- 依存関係: インポート/エクスポート関係
```

#### web-researcher サブエージェント

```
## あなたの担当: 一般的なWeb調査

### 調査項目
- 業界標準・ベストプラクティス
- 比較記事・選定ガイド
- 最新のトレンド・アップデート

### ツール
- WebSearch: 「{topic} best practices 2025 2026」
- WebFetch: 詳細ページの内容取得

### 注意
- 2024年以降の情報を優先
- 複数ソースで裏付けを取る
```

### Phase 4: 結果統合

サブエージェントの結果を統合し、構造化された出力を生成。

**統合プロセス:**
1. 各サブエージェントのJSON結果を収集
2. 重複を排除
3. 信頼度でソート
4. 矛盾する情報を整理
5. 推奨事項をまとめる

### Phase 5: 出力

**出力テンプレート:**
```yaml
# 調査結果: {topic}

## サマリー
{1-3文の要約}

## 発見事項

### {サブトピック1}
- **情報源**: {source}
- **信頼度**: {high/medium/low}
- **内容**: {content}

### {サブトピック2}
...

## 推奨事項
1. {recommendation1}
2. {recommendation2}

## 参照先
| タイプ | ソース | URL/パス |
|--------|--------|----------|
| {mcp/web/codebase} | {name} | {url/path} |

## メタデータ
- 使用ツール: {tools_used}
- サブエージェント数: {count}
- 調査深度: {depth}
```

## Taskツールによる進捗表示

```
# Phase 1
TaskCreate(subject: "ツール検出", activeForm: "利用可能ツールを検出中")

# Phase 2
TaskCreate(subject: "調査計画", activeForm: "調査計画を作成中")

# Phase 3（並列実行時）
TaskCreate(subject: "docs-researcher: {topic}", activeForm: "ドキュメント調査中")
TaskCreate(subject: "code-explorer: {topic}", activeForm: "コードベース調査中")
TaskCreate(subject: "web-researcher: {topic}", activeForm: "Web調査中")

# Phase 4
TaskCreate(subject: "結果統合", activeForm: "結果を統合中")
```

## MCP検出の詳細

### context7の検出と使用

```
# 検出
「context7」「resolve-library-id」ツールが利用可能か確認

# 使用フロー
1. resolve-library-id: ライブラリ名からIDを取得
2. query-docs: ドキュメントをクエリ

例:
resolve-library-id("react") → "react-19"
query-docs("react-19", "useEffect best practices")
```

### serenaの検出と使用

```
# 検出
「serena」関連ツールが利用可能か確認

# 使用
LSP機能（定義ジャンプ、参照検索、シンボル検索）を活用
```

### フォールバック

MCPが利用できない場合:
- ドキュメント調査 → WebSearch + WebFetch
- コードベース調査 → Glob + Grep + Read
- 外部リサーチ → WebSearch

## 注意事項

- MCPの利用可否はプロジェクトごとに異なる
- 必ずフォールバック手段を用意
- 調査結果は必ず情報源を明記
- 推測ではなく事実に基づく情報のみ記録
- 大規模調査は並列実行で効率化
- **エスカレーション時は必ずユーザーに確認を取る**（自動拡大しすぎない）
- **調査キーワードを検出しても、明らかに調査不要な場合は発動しない**
  - 例: 「調査結果を教えて」（既に調査済みの結果を聞いている）
  - 例: 「調べた内容をまとめて」（調査ではなく整理の依頼）

## スキル再読み込みによるエスカレーション

調査中に規模拡大が必要になった場合、このスキルを再度読み込んで
大規模調査モードの手順を参照する:

```
現在の調査が想定より大規模になりました。
/research スキルを再読み込みしてthorough モードで実行します。

@~/.claude/skills/research/SKILL.md を参照
```

これにより、medium→thoroughへのエスカレーション時に
完全なサブエージェント並列実行パターンを適用できる。
