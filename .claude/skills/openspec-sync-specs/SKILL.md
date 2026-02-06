---
name: openspec-sync-specs
description: 変更からメインspecsにデルタspecsを同期。ユーザーが変更をアーカイブせずにデルタspecの変更でメインspecsを更新したい場合に使用。
license: MIT
compatibility: openspec CLIが必要。
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.0.2"
---

変更からメインspecsにデルタspecsを同期します。

これは**エージェント駆動**の操作です - デルタspecsを読み、直接メインspecsを編集して変更を適用します。これによりインテリジェントなマージが可能になります（例：要件全体をコピーせずにシナリオを追加）。

**入力**: オプションで変更名を指定。省略した場合は会話のコンテキストから推測を試みます。曖昧または不明確な場合は、利用可能な変更を表示して選択を促す必要があります。

**手順**

1. **変更名が指定されていない場合、選択を促す**

   `openspec list --json`を実行して利用可能な変更を取得。**AskUserQuestionツール**を使用してユーザーに選択させる。

   デルタspecsがある変更を表示（`specs/`ディレクトリ下）。

   **重要**: 変更を推測または自動選択しない。常にユーザーに選択させる。

2. **デルタspecsを見つける**

   `openspec/changes/<name>/specs/*/spec.md`でデルタspecファイルを探す。

   各デルタspecファイルには以下のようなセクションが含まれる：
   - `## ADDED Requirements` - 追加する新しい要件
   - `## MODIFIED Requirements` - 既存の要件への変更
   - `## REMOVED Requirements` - 削除する要件
   - `## RENAMED Requirements` - 名前を変更する要件（FROM:/TO:フォーマット）

   デルタspecsが見つからない場合、ユーザーに通知して停止。

3. **各デルタspecについて、メインspecsに変更を適用**

   `openspec/changes/<name>/specs/<capability>/spec.md`にデルタspecがある各capabilityについて：

   a. **デルタspecを読む** - 意図された変更を理解

   b. **メインspecを読む** - `openspec/specs/<capability>/spec.md`（まだ存在しない可能性あり）

   c. **インテリジェントに変更を適用**：

      **ADDED Requirements:**
      - 要件がメインspecに存在しない場合 → 追加
      - 要件が既に存在する場合 → 一致するように更新（暗黙のMODIFIED扱い）

      **MODIFIED Requirements:**
      - メインspecで要件を見つける
      - 変更を適用 - 以下のことができる：
        - 新しいシナリオを追加（既存のものをコピーする必要なし）
        - 既存のシナリオを変更
        - 要件の説明を変更
      - デルタで言及されていないシナリオ/コンテンツを保持

      **REMOVED Requirements:**
      - 要件ブロック全体をメインspecから削除

      **RENAMED Requirements:**
      - FROM要件を見つけ、TOに名前変更

   d. **capabilityがまだ存在しない場合、新しいメインspecを作成**：
      - `openspec/specs/<capability>/spec.md`を作成
      - Purposeセクションを追加（簡潔でよい、TBDとマーク）
      - ADDED要件を含むRequirementsセクションを追加

4. **サマリーを表示**

   すべての変更を適用した後、要約：
   - どのcapabilitiesが更新されたか
   - どのような変更が行われたか（要件の追加/変更/削除/名前変更）

**デルタSpecフォーマットリファレンス**

```markdown
## ADDED Requirements

### Requirement: New Feature
システムは新しいことをするべき。

#### Scenario: Basic case
- **WHEN** ユーザーがXを行う
- **THEN** システムがYを行う

## MODIFIED Requirements

### Requirement: Existing Feature
#### Scenario: New scenario to add
- **WHEN** ユーザーがAを行う
- **THEN** システムがBを行う

## REMOVED Requirements

### Requirement: Deprecated Feature

## RENAMED Requirements

- FROM: `### Requirement: Old Name`
- TO: `### Requirement: New Name`
```

**キープリンシプル: インテリジェントマージ**

プログラム的なマージとは異なり、**部分的な更新**を適用できます：
- シナリオを追加するには、MODIFIEDの下にそのシナリオだけを含める - 既存のシナリオをコピーしない
- デルタは*意図*を表し、全面的な置き換えではない
- 変更を合理的にマージするために判断を使用

**成功時の出力**

```
## Specs同期: <change-name>

メインspecsを更新:

**<capability-1>**:
- 要件を追加: "New Feature"
- 要件を変更: "Existing Feature"（1シナリオ追加）

**<capability-2>**:
- 新しいspecファイルを作成
- 要件を追加: "Another Feature"

メインspecsが更新されました。変更はアクティブなまま - 実装完了後にアーカイブしてください。
```

**ガードレール**
- 変更を行う前にデルタspecとメインspecの両方を読む
- デルタで言及されていない既存コンテンツを保持
- 何かが不明確な場合は、明確化を求める
- 変更中に何を変更しているか表示
- 操作は冪等であるべき - 2回実行しても同じ結果になる
