# グループ-ユニット-ロールモデル（Group-Unit-Role Model）

## 概要

タスクのドメインに応じて「機能グループ」を設置し、必要に応じて「技術ユニット」でサブ分割し、その下に「作業ロール」を配置する3層階層モデル。

```yaml
hierarchy:
  orchestrator: オーケストレーター（ファシリテーター）
  standing:
    - {name: 事前評価担当(Pre-Assessor), scope: 全チーム横断}
    - {name: EM, scope: 構造最適化}
  dynamic:  # タスクに応じて動的に設置
    - name: "{ドメインA}グループ"
      units:  # オプション: 技術分野が複数ある場合のみ
        - name: "Frontendユニット"
          roles: [調査担当(Researcher), 実装担当(Engineer)]
        - name: "Backendユニット"
          roles: [調査担当(Researcher), 設計担当(Architect), 実装担当(Engineer)]
      roles: [QA担当(QA Reviewer), 文書担当(Tech Writer)]  # グループ直属ロール
    - name: "{ドメインB}グループ"
      roles: [調査担当(Researcher), 実装担当(Engineer)]  # ユニットなし
```

## 3層モデル

| 層 | 名称 | 説明 | 設置条件 |
|----|------|------|---------|
| 第1層 | 機能グループ | 実装する機能単位 | タスクのドメインに応じて動的に設置 |
| 第2層 | 技術ユニット | グループ内の技術分野サブグループ | 技術分野が複数ある場合のみ（オプション） |
| 第3層 | 作業ロール | 実際の作業を行うロール | 必須。グループ直属またはユニット所属 |

## 常設ロール（横断的機能）

| ロール | 役割 |
|--------|------|
| 事前評価担当(Pre-Assessor) | 実装前ゲートキーパー、規模妥当性評価、全チーム横断 |
| EM | チーム構造の最適化、新機能リクエスト評価 |

## 動的グループ（ドメイン別）の例

認証グループ、APIグループ、UIグループ、DBグループ、インフラグループ、テストグループ等。

## ロールテンプレート

| ロール | 役割 | 参照 |
|--------|------|------|
| 調査担当(Researcher) | 技術調査、コードベース分析 | roles/researcher.md |
| 設計担当(Architect) | アーキテクチャ設計、技術選定 | roles/architect.md |
| 実装担当(Engineer) | コード実装、リファクタリング | roles/engineer.md |
| QA担当(QA Reviewer) | テスト、レビュー、品質チェック | roles/qa-reviewer.md |
| 文書担当(Tech Writer) | ドキュメント作成 | roles/tech-writer.md |
| 運用担当(DevOps) | デプロイ、環境構築 | roles/devops.md |

## チームによる動的グループ実現

動的グループはAgent Teams（TeamCreate/TeamDelete）で実現する。

| フェーズ | チーム名 | メンバー | 方式 |
|---------|---------|---------|------|
| 調査・設計 | `orch-{id}-investigate` | researcher, architect | TeamCreate |
| 実装・QA | `orch-{id}-implement` | engineer, qa-reviewer (+tech-writer, devops) | TeamCreate |

チーム内のロールはSendMessageでpeer通信し、TaskCreate/TaskUpdateでタスクを管理する。
チーム破棄前に必ずハンドオフ（decisions, arc.saved, 未解決事項の収集）を実施する。

