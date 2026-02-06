# テンプレート配布ガイド

このガイドでは、テンプレートリポジトリから各リポジトリへの設定配布機能の使い方を説明します。

## 概要

- **配布元**: テンプレートリポジトリ（Caller Workflow を設定したリポジトリ）
- **配布先**: Organization 配下の全リポジトリ
- **配布方法**: GitHub Actions + git-xargs で PR を自動作成

## アーキテクチャ

配布ワークフローは **Reusable Workflow** として `Spectee/.github` で一元管理されています。

```
Spectee/.github (Public)
└── .github/workflows/
    └── distribute-template.yml  ← Reusable Workflow（共通ロジック）

Spectee/claude-code-template (Private)
├── .claude/                     ← 配布対象
├── docs/                        ← 配布対象
├── .syncignore                  ← 除外パターン
└── .github/workflows/
    └── distribute.yml           ← Caller Workflow（Reusable を呼び出す）
```

**メリット**:
- 配布ロジックの更新は `.github` の1箇所のみ
- 新しい配布元の追加は Caller Workflow（約15行）を作成するだけ

## 初期セットアップ

配布機能を利用するには、GitHub App と Organization Secret の設定が必要です。

### 1. GitHub App を作成

1. GitHub にログイン
2. **Settings** → **Developer settings** → **GitHub Apps** → **New GitHub App**
3. 基本情報を設定:
   - GitHub App name: `Code Distributor`
   - Homepage URL: `https://github.com/Spectee`
4. Permissions を設定:
   - Contents: Read and write
   - Pull requests: Read and write
5. **Create GitHub App** をクリック
6. App ID をメモ
7. **Generate a private key** をクリックして秘密鍵をダウンロード
8. **Install App** → Spectee Organization にインストール

### 2. Organization Secret に登録

1. [Spectee Organization の Settings](https://github.com/organizations/Spectee/settings/secrets/actions) を開く
2. 以下の Organization Secret を追加（対象リポジトリ: All repositories または配布元リポジトリを指定）:
   - `CODE_DISTRIBUTOR_APP_ID`: GitHub App の App ID
   - `CODE_DISTRIBUTOR_APP_SECRET`: 秘密鍵ファイルの内容（PEM形式）

## 自動配布

`.claude/` または `docs/` に変更を push すると、自動的に全リポジトリに PR が作成されます。

```
main に push → distribute ワークフロー起動 → 各リポジトリに PR 作成
```

### PR とコミットメッセージ

- **ブランチ名**: `chore/sync-template`
- **コミットメッセージ**:
  ```
  chore: sync from Spectee/claude-code-template (2026-02-04 15:30 JST)

  Source: https://github.com/Spectee/claude-code-template/commit/abc1234
  ```

コミットメッセージには以下が含まれます:
- **実行日時（JST）**: テンプレートのリリースノートと照合可能
- **ソースリンク**: 変更元のコミットを確認してフィードバック可能

マージ前に複数回配布した場合、同じ PR に追加コミットされます（新しい PR は作成されません）。各コミットにそれぞれのソースリンクが付くため、どの変更がどのソースに対応するか確認できます。

## 手動配布

Actions タブから手動実行することで、柔軟な配布が可能です。

### 使用例

#### 1. 特定のリポジトリにのみ配布

```yaml
repos: my-app, another-app
```

`my-app` と `another-app` にのみ配布されます。

#### 2. 特定のリポジトリを除外

```yaml
exclude_repos: legacy-app, archived-project
```

`legacy-app` と `archived-project` を除く全リポジトリに配布されます。

#### 3. 特定のディレクトリのみ配布

```yaml
sync_dirs: .claude
```

`.claude/` のみ配布され、`docs/` は配布されません。

#### 4. 特定のファイルを配布

```yaml
sync_dirs: .claude/settings.json, docs/README.md
```

指定したファイルのみ配布されます。

#### 5. ドライラン（事前確認）

```yaml
dry_run: true
```

実際には配布せず、何が配布されるかのみ表示します。

#### 6. 組み合わせ例

```yaml
repos: test-repo
sync_dirs: .claude
dry_run: true
```

`test-repo` に `.claude/` のみをドライラン配布（確認のみ）。

## 配布対象の設定

### sync_dirs（ホワイトリスト）

配布対象は各配布元の Caller Workflow（`distribute.yml`）で定義されています。

```yaml
jobs:
  distribute:
    uses: Spectee/.github/.github/workflows/distribute-template.yml@main
    with:
      sync_dirs: '.claude,docs'  # 配布対象
```

- ディレクトリまたはファイルを指定可能
- カンマ区切りで複数指定

### .syncignore（ブラックリスト）

配布対象内で除外したいファイルは `.syncignore` に記載します。

```
# ローカル設定ファイル
*.local.*

# 環境変数ファイル
.env
.env.*

# GitHub設定（各リポジトリ固有）
.github/*
```

**注意**: `.syncignore` はディレクトリ配布時もファイル配布時も適用されます。

## 受け取り側（ターゲットリポジトリ）の対応

1. PR が作成されたら内容を確認
2. 問題なければマージ
3. コンフリクトがある場合は手動で解決

### コンフリクトが発生した場合

配布元とターゲットで同じファイルが異なる編集を受けている場合、コンフリクトが発生します。

```bash
# ターゲットリポジトリで
git checkout chore/sync-template
git merge main
# コンフリクトを解決
git add .
git commit
git push
```

## セキュリティ

### git-xargs のバージョン固定とチェックサム検証

Reusable Workflow（`Spectee/.github`）では以下のセキュリティ対策を実施しています:

- **バージョン固定**: `latest` ではなく特定バージョン（例: `v0.1.16`）を使用
- **チェックサム検証**: ダウンロード後に SHA256 で改ざんを検知

バージョンを更新する場合は `Spectee/.github` の `distribute-template.yml` を更新してください。

## トラブルシューティング

### PR が作成されない

- ターゲットリポジトリに変更がない（既に最新）
- ターゲットリポジトリがアーカイブされている
- `repos` で指定したリポジトリ名が間違っている

### ワークフローが実行されない

- `README.md` など配布対象外のファイルのみ変更した場合は実行されません
- Caller Workflow の `on.push.paths` に含まれるパスの変更でのみトリガーされます

### 特定のファイルが配布されない

- `.syncignore` のパターンにマッチしている可能性があります
- `.syncignore` を確認してください

## 他のリポジトリを配布元として設定する

Reusable Workflow 構成により、新しい配布元の追加は簡単です。

### 前提条件

- Organization Secret に `CODE_DISTRIBUTOR_APP_ID` と `CODE_DISTRIBUTOR_APP_SECRET` が設定済み
- GitHub App「Code Distributor」が Organization にインストール済み

### 手順

#### 1. テンプレートをコピー

`claude-code-template` リポジトリから Caller Workflow テンプレートをコピー:

```bash
# 配布元にしたいリポジトリで実行
curl -o .github/workflows/distribute.yml \
  https://raw.githubusercontent.com/Spectee/claude-code-template/main/.github/workflows/distribute.template.yml
```

または、[distribute.template.yml](https://github.com/Spectee/claude-code-template/blob/main/.github/workflows/distribute.template.yml) を手動でコピーして `.github/workflows/distribute.yml` として保存。

#### 2. 配布対象を設定

コピーしたファイル内の `TODO` コメントを参考に編集:

1. **`on.push.paths`**: 配布対象のパスを指定
2. **`sync_dirs` のデフォルト値**: 配布対象ディレクトリを指定（`on.push.paths` と同期させること）

**例: `.github/workflows` と `config` を配布する場合**

```yaml
on:
  push:
    branches: [main]
    paths:
      - '.github/workflows/**'
      - 'config/**'

jobs:
  distribute:
    uses: Spectee/.github/.github/workflows/distribute-template.yml@main
    with:
      sync_dirs: ${{ inputs.sync_dirs || '.github/workflows,config' }}
```

**自動配布を無効にする場合**: `on.push` セクションを削除し、`workflow_dispatch` のみにする。

#### 3. .syncignore を作成（任意）

配布対象から除外したいファイルパターンを記載:

```
# 例: ローカル設定
*.local.*
.env

# 例: テスト用ファイル
*.test.*
```

#### 4. 動作確認

1. **ドライラン**で対象を確認:
   - Actions タブ → Distribute Template → Run workflow
   - `dry_run: true` で実行

2. **特定リポジトリでテスト**:
   - `repos: test-repo` で限定配布

3. **全体配布**:
   - 配布対象ファイルを変更して main に push

### 複数の配布元を運用する場合

| 配布元リポジトリ | sync_dirs | 用途 |
|------------------|-----------|------|
| claude-code-template | `.claude,docs` | Claude Code 設定 |
| infra-template | `.github/workflows` | CI/CD 設定 |
| security-baseline | `CODEOWNERS` | セキュリティ設定 |

### 注意事項

- **ブランチ名の競合**: 複数の配布元から同じリポジトリに配布する場合、ブランチ名 `chore/sync-template` が競合する可能性があります。
- **Organization Secret**: `CODE_DISTRIBUTOR_APP_ID` と `CODE_DISTRIBUTOR_APP_SECRET` は Organization Secret として設定されているため、同じ Organization 内の任意のリポジトリから利用できます。
