# 並列開発ルール（git worktree + Claude Code）

## 概要

Claude Codeは複数のタスクを並列で効率的に処理するため、git worktreeを活用した並列開発を意識する。

## git worktreeとは

git worktreeは1つのリポジトリから複数の作業ディレクトリを作成する機能。各worktreeで独立したブランチを扱えるため、タスクを切り替える際のコンテキストスイッチを最小限に抑えられる。

## いつworktreeを提案すべきか

以下の状況でworktreeの使用を提案する：

1. **複数の独立したタスクがある場合**
   - 互いに依存しない複数のissue/タスクを並行して進めたい
   - 例：機能Aの開発中に、緊急のバグ修正Bが必要になった

2. **レビュー待ちの間に別作業をしたい場合**
   - PRがレビュー待ちの間、別のタスクに着手したい
   - 元のブランチに影響を与えずに新しい作業を開始できる

3. **PRレビューを行う場合**
   - 他のメンバーのPRを手元で確認したい
   - 自分の作業を中断せずにレビュー環境を用意できる

4. **実験的な変更を試したい場合**
   - 本線の作業を中断せずに、別のアプローチを試したい

## ディレクトリ構成

worktreeの配置には主に2つのパターンがある：

### パターン1: メインディレクトリと同階層に配置

```
/path/to/
├── project/                           # メインリポジトリ
├── project-issue-123-add-api/         # Issue #123 用
├── project-issue-456-fix-bug/         # Issue #456 用
└── project-pr-789/                    # PR #789 レビュー用
```

**メリット:**

- シンプルで理解しやすい
- `cd ../project-*` で簡単に移動可能

### パターン2: worktrees専用ディレクトリで管理

```
/path/to/
├── project/                           # メインリポジトリ
└── project-worktrees/                 # worktrees専用ディレクトリ
    ├── issue-123/                     # Issue #123 用
    ├── issue-456/                     # Issue #456 用
    └── pr-789/                        # PR #789 レビュー用
```

**メリット:**

- worktreeとメインリポジトリが明確に分離
- ディレクトリが整理されて見やすい
- worktreeの一括管理が容易

### Claude Codeでの推奨

- 少数のworktree（1-2個）: パターン1
- 複数のworktree（3個以上）: パターン2
- いずれの場合も、VSCodeのワークスペース機能や`git worktree list`で管理

## ブランチ命名規則

worktree作成時のブランチ名：

- `issue/<issue番号>-<簡潔な説明>` - issue対応
- `feature/<機能名>` - 新機能開発
- `fix/<修正内容>` - バグ修正
- `hotfix/<緊急修正内容>` - 緊急修正

## 基本コマンド

### worktree操作

```bash
# worktree一覧の確認
git worktree list

# パターン1: メインディレクトリと同階層に作成
# Issue用worktreeの作成（新規ブランチ）
git worktree add ../$(basename $(pwd))-issue-<番号>-<説明> -b issue/<番号>-<説明>

# 例: Issue #123 の API追加機能
git worktree add ../project-issue-123-add-api -b issue/123-add-api

# パターン2: worktrees専用ディレクトリに作成
git worktree add ../$(basename $(pwd))-worktrees/issue-<番号> -b issue/<番号>-<説明>

# 例: Issue #123 用（worktreesディレクトリ配下）
git worktree add ../project-worktrees/issue-123 -b issue/123-add-api

# PRレビュー用worktreeの作成（既存ブランチをチェックアウト）
git worktree add ../project-pr-<番号> <branch-name>

# 例: PR #789 のレビュー
git worktree add ../project-pr-789 feature/new-authentication

# worktreeの削除
git worktree remove ../project-issue-123-add-api

# 不要なworktree情報のクリーンアップ
git worktree prune
```

### worktree間の移動

```bash
# 直接移動（パターン1）
cd ../project-issue-123-add-api

# 直接移動（パターン2）
cd ../project-worktrees/issue-123

# Claude Codeを起動
claude-code

# VSCodeでworktreeを開く
code ../project-issue-123-add-api

# 現在のworktree位置を確認
pwd
git worktree list
```

## 注意点

### 同一ブランチの制約
- 1つのブランチは1つのworktreeでしか使用できない
- 別のworktreeで使用中のブランチにはチェックアウトできない

### 共有されるもの
- `.git/` ディレクトリは全worktreeで共有
- `git stash` は全worktreeで共有される
- リベースや履歴の書き換えは他のworktreeに影響を与える

### クリーンアップ

- 作業完了・PR マージ後はworktreeを削除する
- 定期的に `git worktree list` で確認し、不要なものを削除
- worktreeのディレクトリも物理的に削除（`git worktree remove`で自動削除される）

## Claude Codeとしての振る舞い

### 1. 現在のworktree状況を把握する

会話の開始時や新しいタスクに着手する前に、現在のworktree状況を確認する。

```bash
# worktreeの一覧を確認
git worktree list

# 現在のディレクトリとブランチを確認
pwd
git branch --show-current
```

### 2. 並列作業の提案

以下の状況でworktreeの使用を積極的に提案する：

- ユーザーが複数のissue/タスクに言及した場合
- PRレビュー待ちの間に別のタスクを始めたい場合
- 現在の作業を保持したまま緊急の修正が必要な場合
- 実験的な変更を試したい場合

**提案例:**

```text
複数のタスクがあるようですので、git worktreeを使った並列開発をお勧めします。

現在: issue/51-allow-deny-commands (このまま継続)
新規: issue/54-parallel-development-rules

新しいworktreeを作成しますか？
```

### 3. worktree作成時の命名規則

ユーザーの既存のworktree構成を確認し、それに合わせた命名を提案する。

**パターン1の場合:**
```bash
git worktree add ../project-issue-<番号>-<説明> -b issue/<番号>-<説明>
```

**パターン2の場合:**
```bash
git worktree add ../project-worktrees/issue-<番号> -b issue/<番号>-<説明>
```

### 4. worktree間の作業切り替え

別のworktreeで作業が必要な場合は、明確にユーザーに伝える。

**伝え方の例:**

```text
このタスクはissue/54-parallel-development-rulesブランチで作業する必要があります。
以下のworktreeに移動してください：

cd /path/to/project-worktrees/issue-54

または、新しいターミナル/Claude Codeセッションを開いて作業することもできます。
```

### 5. Claude Codeセッションの並列実行

各worktreeで独立したClaude Codeセッションを実行できることをユーザーに伝える。

**メリット:**

- 各タスクのコンテキストが独立して保持される
- タスクごとに異なるメモリやTODOリストを管理できる
- 作業の切り替えが容易

### 6. クリーンアップの提案

以下のタイミングでworktreeの削除を提案する：

- PRがマージされた後
- タスクが完了し、そのブランチが不要になった後
- `git worktree list` で多数の古いworktreeが残っている場合

**提案例:**

```bash
issue/54-parallel-development-rulesのPRがマージされました。
worktreeをクリーンアップしますか？

git worktree remove /path/to/project-worktrees/issue-54
```

### 7. 注意すべきエラー

worktree使用時に発生しうるエラーとその対処法：

#### エラー: fatal: 'branch-name' is already checked out at '...'

- 原因: そのブランチは既に別のworktreeで使用中
- 対処: `git worktree list` で確認し、既存のworktreeに移動するか、別のブランチ名を使用

#### エラー: fatal: invalid reference: branch-name

- 原因: 指定したブランチが存在しない
- 対処: `-b` オプションで新規ブランチとして作成する

### 8. IssueとPRの更新を意識する

worktreeでの作業中や完了時に、IssueとPRの情報を適切に更新する。

#### PRのタイトルと説明の更新

コード変更がPRの元の目的から大きく変わった場合、PRのタイトルと説明を更新する。

**更新すべきタイミング:**

- 実装内容が当初の計画から大幅に変更された場合
- 新しい機能や修正を追加した場合
- アプローチを変更した場合（例: ghq依存からシンプルな構成に変更）

**更新コマンド:**

```bash
# PRのタイトルを更新
gh pr edit <PR番号> --title "新しいタイトル"

# PRの説明を更新
gh pr edit <PR番号> --body "$(cat <<'EOF'
## Summary
...
EOF
)"

# タイトルと説明を同時に更新
gh pr edit <PR番号> --title "新しいタイトル" --body "新しい説明"
```

#### Issueへの進捗報告

関連するIssueに進捗状況をコメントで報告する。

**報告すべきタイミング:**

- 作業を開始したとき
- 大きなマイルストーンを達成したとき
- ブロッカーや課題が発生したとき
- 作業が完了し、レビュー準備ができたとき

**コメント例:**

```bash
gh issue comment <Issue番号> --body "作業を開始しました。worktree: \`project-worktrees/issue-<番号>\`"

gh issue comment <Issue番号> --body "PR #<番号> を作成しました。レビューをお願いします。"
```

#### 関連IssueとPRの紐付け

PRとIssueを適切に紐付けて、トレーサビリティを確保する。

**紐付け方法:**

- PR説明に `Closes #<Issue番号>` または `Fixes #<Issue番号>` を記載
- Issue番号への参照により、PR一覧からIssueを追跡可能に
- マージ時に自動的にIssueがクローズされる

**例:**

```markdown
## 関連Issue

Closes #54
Related to #53
```

#### 複数worktreeでの作業状況の可視化

複数のworktreeで並行作業している場合、各IssueやPRに現在の状況を明記する。

**提案例:**

```text
現在、以下のworktreeで並行作業中です：
- issue-51: コマンド許可・禁止機能（レビュー待ち）
- issue-54: 並列開発ルール（実装中）
- issue-42: スキル作成機能（保留中）

次にどの作業を優先しますか？
```

### 9. CLAUDE.local.mdでの常時有効化の案内

ユーザーが初めてworktreeを使用した際、今後も常にworktreeを活用するかを確認し、希望する場合は`CLAUDE.local.md`への設定を案内する。

**案内のタイミング:**

- ユーザーが初めてworktreeを作成したとき
- worktreeを使った作業が完了したとき

**案内例:**

```text
git worktreeでの並列開発、いかがでしたか？

今後も常にworktreeを活用したい場合は、CLAUDE.local.mdに設定を追加できます。
設定しますか？
```

**CLAUDE.local.mdに追記する内容:**

```markdown
## 並列開発（git worktree）

常にgit worktreeを使用した並列開発を行う。

- 新しいタスクやissueに着手する際は、worktreeを作成する
- 会話開始時に `git worktree list` で現在の状況を確認する
- PRレビュー時は、レビュー用worktreeを作成する
- 複数タスクは並列でworktreeを使って処理する
```

**設定手順:**

1. プロジェクトルートに `CLAUDE.local.md` を作成（なければ）
2. 上記の内容を追記
3. `.gitignore` に `CLAUDE.local.md` を追加（共有リポジトリの場合）

**注意点:**

- `CLAUDE.local.md` はユーザー固有の設定ファイルなので、通常gitにコミットしない
- チーム全体で使う場合は `CLAUDE.md` に記載する
- 既に `CLAUDE.local.md` がある場合は、既存の内容に追記する形で案内する
