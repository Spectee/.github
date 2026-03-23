# .github

Spectee Organization 共通の GitHub 設定を管理するリポジトリです。

## このリポジトリの用途

| 格納物 | パス | 説明 |
| --- | --- | --- |
| Issue テンプレート | `.github/ISSUE_TEMPLATE/` | Epic・Task など、Organization 共通の Issue テンプレート |
| Reusable Workflow | `.github/workflows/` | テンプレート配布などの共通ワークフロー |
| Organization プロフィール | `profile/README.md` | GitHub 上の Organization トップページに表示される紹介文 |

各リポジトリに個別のテンプレートやファイルが存在しない場合、ここに置かれたデフォルトファイルが自動的に適用されます。
個別リポジトリに同名のファイルがある場合は、そちらが優先されます。

## なぜこのリポジトリはパブリックなのか

GitHub 公式ドキュメントに以下の記載があります。

> **Note:** The `.github` repository must be public for most default community health files to be applied organization-wide. Private `.github` repositories are not supported.
>
> — [Creating a default community health file - GitHub Docs](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file)

つまり、`.github` リポジトリを **パブリック** にしておかないと、Issue テンプレートや CONTRIBUTING ガイドラインなどの **デフォルトコミュニティヘルスファイル** が Organization 内の他リポジトリへ自動適用されません。
この制約のため、本リポジトリは意図的にパブリックとして公開しています。

> [!IMPORTANT]
> パブリックリポジトリであるため、機密情報（シークレット、内部 URL、認証情報など）は絶対にコミットしないでください。

## 参考リンク

- [Creating a default community health file - GitHub Docs](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file)
