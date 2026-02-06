---
name: "OPSX: 同期"
description: 変更からメインspecsにデルタspecsを同期
category: Workflow
tags: [workflow, specs, experimental]
---

変更からメインspecsにデルタspecsを同期します。

これは**エージェント駆動**の操作です - デルタspecsを読んでメインspecsを直接編集して変更を適用します。これによりインテリジェントなマージが可能になります（例：要件全体をコピーせずにシナリオを追加）。

**入力**: `/opsx:sync`の後にオプションで変更名を指定（例：`/opsx:sync add-auth`）。省略した場合は会話のコンテキストから推測を試みます。曖昧または不明確な場合は、利用可能な変更を表示して選択を促す必要があります。

**手順**

1. **変更名が指定されていない場合、選択を促す**

   `openspec list --json`を実行して利用可能な変更を取得。**AskUserQuestionツール**を使用してユーザーに選択させます。

   デルタspecsがある変更（`specs/`ディレクトリ配下）を表示。

   **重要**: 推測や自動選択をしない。必ずユーザーに選択させる。

2. **デルタspecsを見つける**

   `openspec/changes/<name>/specs/*/spec.md`でデルタspecファイルを探す。

   各デルタspecファイルには以下のようなセクションが含まれる：
   - `## ADDED Requirements` - 追加する新しい要件
   - `## MODIFIED Requirements` - 既存要件への変更
   - `## REMOVED Requirements` - 削除する要件
   - `## RENAMED Requirements` - 名前変更する要件（FROM:/TO:形式）

   デルタspecsが見つからない場合、ユーザーに通知して停止。

3. **各デルタspecに対して、メインspecsに変更を適用**

   `openspec/changes/<name>/specs/<capability>/spec.md`にデルタspecがある各機能に対して：

   a. **デルタspecを読んで**意図した変更を理解

   b. **メインspecを読む**（`openspec/specs/<capability>/spec.md`、まだ存在しないかもしれない）

   c. **インテリジェントに変更を適用**：

      **ADDED Requirements:**
      - メインspecに要件が存在しない場合 → 追加
      - 要件が既に存在する場合 → 一致するように更新（暗黙のMODIFIEDとして扱う）

      **MODIFIED Requirements:**
      - メインspecで要件を見つける
      - 変更を適用 - これは以下の可能性：
        - 新しいシナリオの追加（既存のものをコピーする必要なし）
        - 既存シナリオの変更
        - 要件説明の変更
      - デルタで言及されていないシナリオ/内容を保持

      **REMOVED Requirements:**
      - メインspecから要件ブロック全体を削除

      **RENAMED Requirements:**
      - FROM要件を見つけ、TOに名前変更

   d. **新しいメインspecを作成**（機能がまだ存在しない場合）：
      - `openspec/specs/<capability>/spec.md`を作成
      - Purposeセクションを追加（簡潔でよい、TBDとマーク）
      - ADDED要件でRequirementsセクションを追加

4. **サマリーを表示**

   すべての変更適用後、まとめ：
   - 更新された機能
   - 行われた変更（追加/変更/削除/名前変更された要件）

**デルタspecフォーマットリファレンス**

```markdown
## ADDED Requirements

### Requirement: 新機能

システムは新しいことをするものとする。

#### Scenario: 基本ケース

- **WHEN** ユーザーがXをする
- **THEN** システムがYをする

## MODIFIED Requirements

### Requirement: 既存機能
#### Scenario: 追加する新シナリオ

- **WHEN** ユーザーがAをする
- **THEN** システムがBをする

## REMOVED Requirements

### Requirement: 非推奨機能

## RENAMED Requirements

- FROM: `### Requirement: 旧名`
- TO: `### Requirement: 新名`
```

**重要な原則: インテリジェントマージ**

プログラム的なマージとは異なり、**部分的な更新**を適用できます：
- シナリオを追加するには、MODIFIEDにそのシナリオだけを含める - 既存シナリオをコピーしない
- デルタは*意図*を表し、全面的な置き換えではない
- 変更を賢くマージするために判断を使用

**成功時の出力**

```
## Specs同期完了: <change-name>

メインspecsを更新:

**<capability-1>**:
- 要件を追加: 「新機能」
- 要件を変更: 「既存機能」（1シナリオ追加）

**<capability-2>**:
- 新しいspecファイルを作成
- 要件を追加: 「別の機能」

メインspecsが更新されました。変更はアクティブのまま - 実装完了後にアーカイブしてください。
```

**ガードレール**
- 変更を加える前にデルタとメイン両方のspecsを読む
- デルタで言及されていない既存内容を保持
- 不明確な場合は明確化を求める
- 変更内容を進行中に表示
- 操作は冪等であるべき - 2回実行しても同じ結果
