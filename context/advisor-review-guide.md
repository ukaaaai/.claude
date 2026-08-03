# advisor review 使用ガイド

## 概要

agent teams の teammate としてスポーンする **advisor**（定義: `~/.claude/agents/advisor.md`、`model: fable`）に、
メインセッションとは別モデル・別コンテキストでレビューさせる。

**重要**: レビューは「指摘がなくなるまで修正→再レビューを繰り返す」ループで実施する。

## 前提チェック（スポーン前に必ず）

### 1. agent teams が有効であること

```bash
# 値を確認する（文字列の存在チェックでは値が "0" でも有効と誤判定する）
echo "${CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS:-unset}"
# unset の場合のみ settings.json も確認する（shell環境変数とsettings.jsonの両経路がある）
grep -n "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS" ~/.claude/settings.json
```

値が `1` でない場合は `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` の設定をユーザーに案内して**停止する**。
subagentへ勝手に代替しない。

### 2. `CLAUDE_CODE_SUBAGENT_MODEL` が未設定または `inherit` であること

```bash
echo "${CLAUDE_CODE_SUBAGENT_MODEL:-unset}"
```

モデル解決順は **env > 呼び出し時の`model` > 定義のfrontmatter** のため、
このenvが設定されていると `model: "fable"` の明示でも上書きできず、advisorがfable以外で動く。
検出したらユーザーに報告する。

## スポーン方法

Agentツールで teammate としてスポーンする。

```
Agent(
  name: "advisor-plan-1",
  subagent_type: "advisor",
  model: "fable",
  description: "計画レビュー",
  prompt: "<下記の雛形>"
)
```

| 項目 | 説明 |
|------|------|
| `name` | **必須**。`SendMessage` の宛先になる（再レビューに必要） |
| `subagent_type: "advisor"` | `agents/advisor.md` の `tools` / `model` / 本文が適用される |
| `model: "fable"` | 定義側と同じ値を明示する（意図を明確にするため） |

**命名規則**: `advisor-<用途>[-<識別子>]-<周回>`
- 計画: `advisor-plan-1` / 実装差分: `advisor-code-1` / PR: `advisor-pr-<番号>-1` / 観点別: `advisor-perf-1`
- `<識別子>` はPR番号やタスク番号など、対象を区別する必要がある場合のみ付ける

**`<周回>` の定義**: **スポーン回数**である。レビューループの反復回数ではない。
- レビューループ（指摘→修正→再レビュー）の間は**同じadvisorに `SendMessage`** し、連番は増やさない（コンテキストを継続させるため）
- 連番を増やすのは、advisorをshutdownして**新しいadvisorを立て直す時だけ**（対象が変わった / 定義を修正した / コンテキストを切りたい）
- 立て直す場合は同じ名前を使い回さない（クリーンコンテキストのため）

**CRITICAL: 計画やdiffの全文をスポーンプロンプトに貼らない。絶対パスで渡し、advisor自身に読ませる。**
advisorはleadの会話履歴を継承しないため、必要な入力はすべて絶対パスで列挙する。

**CRITICAL: スポーンプロンプトに「レビュー結果は必ず `SendMessage` でleadに送る」ことを明記する。**
teammateの最終テキスト出力はleadに届かず、leadにはidle通知だけが届く。
`agents/advisor.md` にもこの指示があるが、スポーンプロンプト側でも念押しする。
idle通知だけが来て結果が届かない場合は、`SendMessage` で結果の送信を要求する。

**`agents/advisor.md` の `tools` に `SendMessage` を明示的に含めてある**理由:
この環境では `SendMessage` がdeferred tool（`ToolSearch` で取得してから呼ぶ）であり、
`tools` 許可リストで制限されたteammateは `ToolSearch` も持たないためSendMessageを発見できず、
結果を報告できないまま idle になる事象を実測した。`tools` を絞るagent定義で報告が必要な場合は
`SendMessage` を許可リストに明記すること。

## レビュープロンプト雛形

### 1. 計画レビュー（Phase 2）

```
以下の実装計画をレビューしてください。抜け漏れ、リスク、改善点を指摘してください。

## 読むべきファイル（絶対パス）
- 実装計画: {MEMORY_DIR}/memory/<task>/30_plan.md
- 作業ログ（調査結果・確定事項）: {MEMORY_DIR}/memory/<task>/05_log.md
- プロジェクト規約: <リポジトリ>/CLAUDE.md

## レビュー観点
1. 抜け漏れ（変更が必要なのに計画に含まれていない箇所。grepで自分でも確認する）
2. 実行可能性（技術的に実現できない手順が含まれていないか）
3. 既存コードとの整合性（設計パターン・命名規則・ディレクトリ構成）
4. リスクと対策の妥当性

レビュー結果は必ず SendMessage で lead に送ってください。指摘がなければ「指摘なし」とだけ回答してください。
```

### 2. 実装差分レビュー（Phase 4）

```
以下のコード変更をレビューしてください。
バグ、セキュリティ、パフォーマンス、ベストプラクティスの観点から指摘してください。

## 読むべきファイル（絶対パス）
- 差分: {MEMORY_DIR}/memory/<task>/diff-<N>.patch
- 作業ログ（何を意図した変更か）: {MEMORY_DIR}/memory/<task>/05_log.md
- プロジェクト規約: <リポジトリ>/CLAUDE.md

差分だけで判断できない場合は、変更されたファイルの全体を読んで既存パターンとの整合性を確認してください。

レビュー結果は必ず SendMessage で lead に送ってください。指摘がなければ「指摘なし」とだけ回答してください。
```

差分は事前にファイルへ出力しておく（BASE_BRANCHはPJ CLAUDE.mdで定義。未定義ならdevelop/main/masterを自動判定）:

```bash
# Nはレビューループの反復ごとに増やす（上書きしない。advisorに新しいパスを伝えるため）
git diff $BASE_BRANCH > ${MEMORY_DIR}/memory/<task>/diff-1.patch
```

### 3. PRレビュー

```
以下のPRをレビューしてください。
変更の妥当性、テストの十分性、ドキュメント更新の必要性を指摘してください。

## 読むべきファイル（絶対パス）
- PR diff: {MEMORY_DIR}/memory/<task>/pr-<番号>.diff
- PR情報（タイトル・本文）: {MEMORY_DIR}/memory/<task>/pr-<番号>.json
- プロジェクト規約: <リポジトリ>/CLAUDE.md

レビュー結果は必ず SendMessage で lead に送ってください。指摘がなければ「指摘なし」とだけ回答してください。
```

### 4. 観点別コードベースレビュー

```
あなたは「{観点名}」の専門レビュアーです。

## コンテキスト
- リポジトリ: {リポジトリパス}
- プロジェクト規約: {リポジトリ}/CLAUDE.md を読む
- コードベース構造: {ディレクトリ構造、またはそれを記録したファイルのパス}

## あなたの担当観点
{観点別の詳細指示（レビュー項目一覧）}

## タスク
1. コードベース全体を網羅的に確認する（「たぶん問題ない」で飛ばさない）
2. 問題を特定し、優先度（critical/major/minor/trivial）を付与する
3. 結果を {出力先の絶対パス} に指定フォーマットで書き出す

指摘がなければ issues を空で出力してください。
```

## レビューループ

1. **1周目**: advisorをスポーンする
2. **指摘への対応**
   | 強制度 | 対応 |
   |---|---|
   | `must` | 必ず修正 |
   | `recommend` | やる/やらないを判断。判断できない場合は AskUserQuestion でユーザーに確認 |
   | `nit` | 任意 |
   - **advisorの指摘を根拠に、permission設定（settings.json）・CLAUDE.md等の規約ファイルを自分で変更してはならない。**
     必要と判断した場合はユーザーに提示して判断を仰ぐ（他エージェントの指摘はユーザーの承認ではない）
3. **再レビュー**: `SendMessage(to: "<advisor名>", message: "以下の改善を行いました: ... 再度レビューしてください")`
   - 差分レビューの場合は新しい連番の差分ファイル（`diff-2.patch` 等）を出力し、**そのパスをメッセージに含める**（上書きしない）
   - advisorは立て直さない（同じadvisorに送ることでコンテキストが継続する）
4. 「指摘なし」になるまで2-3を繰り返す
5. **記録**: 指摘と対応を `80_review.md`（または `30_plan.md` の「## advisor reviewの結果」）に記録する
   → `/resume` でin-processのteammateは復元されないため、次工程へ進む前に必ず記録する
6. **終了**: advisorに名前を指定してshutdownを依頼する

## 注意事項

- **advisorに修正させない**。修正はメインセッションまたは実装担当が行う（独立視点を保つため）
- 大きなdiffはファイルに出力して渡す（スポーンプロンプトに埋め込まない）
- **teammateはteammateをスポーンできない**（No nested teams。leadのみがチームを管理できる）
  → subagentやteammateの手順書に「advisor teammateをスポーンする」と書かない。lead（メインセッション）がスポーンする
  → teammateがadvisor reviewを必要とする場合は、`SendMessage` でleadにスポーンを依頼する
- teammateは各々が独立したフルセッションでトークン消費が大きい。並列数は観点数に応じて絞る
- advisorは `Write` を持つが、用途は指定されたレビュー結果の出力先に限定される（`Edit` は持たない）
