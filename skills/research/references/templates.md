# Research Skill テンプレート詳細

## 入力テンプレート

### 基本形式

```yaml
research_request:
  topic: string              # 調査テーマ（必須）
  type: enum                 # 調査タイプ（デフォルト: comprehensive）
    - technical              # 技術調査（ライブラリ、API、実装方法）
    - codebase               # コードベース調査（既存コード分析）
    - comprehensive          # 複合調査（技術 + コードベース）
  depth: enum                # 調査深度（デフォルト: medium）
    - quick                  # 簡易調査（分割なし、即答レベル）
    - medium                 # 標準調査（2-3サブタスク）
    - thorough               # 徹底調査（4-6サブタスク、並列実行）
  scope:                     # 調査範囲（オプション）
    libraries: [string]      # 対象ライブラリ
    directories: [string]    # 対象ディレクトリ
    keywords: [string]       # 検索キーワード
  output_format: enum        # 出力形式（デフォルト: structured）
    - summary                # サマリーのみ
    - detailed               # 詳細出力
    - structured             # 構造化出力（YAML/JSON形式）
```

### 入力例

**技術調査（quick）:**
```
/research "React Server Components" --type technical --depth quick --libraries react,next
```

**コードベース調査:**
```
/research "認証フロー" --type codebase --dirs src/auth,src/middleware
```

**複合調査（thorough）:**
```
/research "GraphQL実装パターン" --type comprehensive --depth thorough --libraries apollo,graphql --dirs src/api
```

## 出力テンプレート

### 基本形式

```yaml
research_result:
  # メタ情報
  metadata:
    topic: string            # 調査テーマ
    type: string             # 調査タイプ
    depth: string            # 調査深度
    tools_used: [string]     # 使用したツール
    subagent_count: number   # サブエージェント数
    timestamp: string        # 実行日時

  # サマリー
  summary: string            # 1-3文の要約

  # 発見事項
  findings:
    - topic: string          # サブトピック
      source:
        type: enum           # mcp | web | codebase
        name: string         # ソース名
        url_or_path: string  # URL or ファイルパス
      content: string        # 発見内容
      confidence: enum       # high | medium | low
      evidence: string       # 根拠（引用、コードスニペット等）

  # 推奨事項
  recommendations:
    - priority: enum         # high | medium | low
      content: string        # 推奨内容
      rationale: string      # 理由

  # 参照先一覧
  sources:
    - type: enum             # mcp | web | codebase
      name: string           # ソース名
      url_or_path: string    # URL or ファイルパス
      accessed_at: string    # アクセス日時
```

### 出力例（structured）

```yaml
research_result:
  metadata:
    topic: "React Server Components"
    type: technical
    depth: medium
    tools_used: ["context7", "WebSearch"]
    subagent_count: 2
    timestamp: "2026-01-27T10:30:00+09:00"

  summary: |
    React Server Componentsは、サーバーサイドでレンダリングされるコンポーネントで、
    クライアントバンドルサイズを削減しつつサーバーリソースに直接アクセスできる。
    Next.js 15ではデフォルトでサポートされている。

  findings:
    - topic: "基本概念"
      source:
        type: mcp
        name: context7
        url_or_path: "react-19/server-components"
      content: |
        Server Componentsは「use client」ディレクティブなしで定義され、
        サーバーサイドでのみ実行される。データベースアクセスやファイルシステム
        操作を直接コンポーネント内で行える。
      confidence: high
      evidence: "React 19 公式ドキュメント"

    - topic: "パフォーマンス最適化"
      source:
        type: web
        name: "Vercel Blog"
        url_or_path: "https://vercel.com/blog/rsc-optimization"
      content: |
        Streaming SSRと組み合わせることで、TTFBを大幅に改善できる。
        Suspense境界を適切に配置することが重要。
      confidence: medium
      evidence: "Vercel Engineering Blog 2025年12月"

  recommendations:
    - priority: high
      content: "データフェッチはServer Componentsで行い、インタラクティブ部分のみClient Componentsに分離"
      rationale: "バンドルサイズ削減とサーバーリソースの効率的利用"

    - priority: medium
      content: "Suspense境界を細かく設定してストリーミングを活用"
      rationale: "ユーザー体験の向上（部分的なコンテンツ表示）"

  sources:
    - type: mcp
      name: context7
      url_or_path: "react-19"
      accessed_at: "2026-01-27T10:25:00+09:00"
    - type: web
      name: "Vercel Blog"
      url_or_path: "https://vercel.com/blog/rsc-optimization"
      accessed_at: "2026-01-27T10:28:00+09:00"
```

### 出力例（summary）

```
## 調査結果: React Server Components

React Server Componentsは、サーバーサイドでレンダリングされるコンポーネントで、
クライアントバンドルサイズを削減しつつサーバーリソースに直接アクセスできる。
Next.js 15ではデフォルトでサポートされている。

**主な推奨事項:**
1. データフェッチはServer Componentsで行う
2. Suspense境界を細かく設定する

**参照:** context7 (react-19), Vercel Blog
```

## サブエージェント間通信フォーマット

### サブエージェントからオーケストレーターへの報告

```json
{
  "subagent_type": "docs-researcher",
  "status": "completed",
  "topic": "React Server Components",
  "findings": [
    {
      "content": "発見内容",
      "source": "context7:react-19",
      "confidence": "high"
    }
  ],
  "tools_used": ["context7.resolve-library-id", "context7.query-docs"],
  "errors": [],
  "recommendations": ["推奨事項"]
}
```

### エラー報告

```json
{
  "subagent_type": "docs-researcher",
  "status": "partial",
  "topic": "React Server Components",
  "findings": [...],
  "errors": [
    {
      "tool": "context7",
      "message": "Rate limit exceeded",
      "fallback_used": "WebSearch"
    }
  ]
}
```

## 信頼度評価基準

| 信頼度 | 条件 |
|--------|------|
| high | 公式ドキュメント、context7結果、著名な開発者のブログ |
| medium | 信頼できるブログ、Stack Overflow（多数の賛成票）、2年以内の記事 |
| low | 古い記事（2年以上前）、個人ブログ、未検証の情報 |

## 調査深度別のサブタスク分割

### quick（分割なし）

単一エージェントで直接調査。
- 所要時間目安: 1-2分
- 用途: 簡単な質問、既知の情報確認

### medium（2-3サブタスク）

```
トピック: "GraphQL実装パターン"
→ サブタスク1: docs-researcher - Apollo Client設定
→ サブタスク2: code-explorer - 既存GraphQL実装の分析
```

### thorough（4-6サブタスク、並列）

```
トピック: "マイクロサービス認証"
→ サブタスク1: docs-researcher - JWT仕様
→ サブタスク2: docs-researcher - OAuth 2.0フロー
→ サブタスク3: web-researcher - 業界ベストプラクティス
→ サブタスク4: code-explorer - 既存認証コード分析
→ サブタスク5: code-explorer - ミドルウェア実装パターン
```
