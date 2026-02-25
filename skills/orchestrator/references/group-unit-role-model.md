# グループ-ユニット-ロールモデル (GUR Model)

## 1. 階層構造
* **L1: PM (Main Session)** - 意思決定、進捗管理、Go/No-Go判断
* **L2: Phase EM (Team Member)** - 各フェーズの実行責任者
    * **Phase A (偵察・戦略)**: Librarian, Scout(N), Strategist(N) を管理
    * **Phase B (調査・設計)**: ドメイングループ(L3)を構築。Pre-Assessorを含む
    * **Phase C (実装・QA)**: ドメイングループ(L3)を構築。実装・検証を統括
* **L3: 動的グループ/ユニット** - 実作業単位（Phase EMがTeamCreateで生成）
    * **機能グループ**: ドメイン別（Auth, API, UI等）の作業単位
    * **技術ユニット**: (任意) Frontend, Backend等の専門分野別サブグループ

## 2. ロール定義と配置
### 常設・独立ロール
* **Pre-Assessor**: Phase Bで起動。実装前の規模・妥当性評価（ゲートキーパー）
* **Eng-Manager**: PMが直接起動。チーム構造最適化、新機能評価
* **Auditor**: PMが直接起動。最終監査
* **Librarian**: Phase Aで起動。ナレッジ管理

### 作業ロール (L3所属)
* **Researcher**: 技術調査、コード分析
* **Architect**: 設計、技術選定
* **Engineer**: 実装、リファクタ
* **QA Reviewer**: テスト、品質管理
* **Tech Writer**: ドキュメント作成
* **DevOps**: 環境構築、デプロイ

## 3. 運用ルール
* **動的生成**: Phase EMは `TeamCreate` / `TeamDelete` でグループを管理する
* **通信/管理**: ロール間は `SendMessage` (Peer通信)、タスクは `TaskCreate/Update` で管理
* **ハンドオフ**: チーム破棄前に必ず「決定事項、保存済み成果物、未解決事項」を収集・継承すること