+++
title = 'ひでぶログができるまで：Hugo + PaperMod + GitHub Pages でブログを公開した全記録'
date = 2026-10-20T20:00:00+09:00
draft = false
tags = ['Hugo', 'PaperMod', 'GitHub Pages', 'GitHub Actions', 'Claude Code']
categories = ['技術メモ']
summary = '空っぽのフォルダから、Hugo と PaperMod でブログを作り、GitHub Actions で GitHub Pages に自動デプロイするまでの手順と、途中でハマったポイントをまとめました。'
ShowToc = true
TocOpen = true
+++

このブログ「ひでぶログ」は、**空っぽのフォルダ**から始めて、1日ほどで公開までたどり着きました。
作業は AI コーディングアシスタントの **Claude Code** と対話しながら進めています。

この記事では、その道のりを順番に振り返ります。途中でつまずいたポイントも正直に書いたので、これから Hugo でブログを作る人の参考になればうれしいです。

## 完成した構成

最初に、できあがったものの全体像です。

| 項目 | 内容 |
| --- | --- |
| サイトジェネレーター | [Hugo](https://gohugo.io/)（Extended 版） |
| テーマ | [PaperMod](https://github.com/adityatelange/hugo-PaperMod)（git submodule で導入） |
| ホスティング | GitHub Pages（`ユーザー名.github.io` リポジトリ） |
| デプロイ | GitHub Actions + `actions/deploy-pages` |
| 作業環境 | Windows 11 / PowerShell / Git Bash |

記事を書いて `main` ブランチにプッシュするだけで、自動でビルドされて公開されます。

```text
記事を書く → git push → GitHub Actions がビルド → GitHub Pages に公開
```

## ステップ 1：サイトの骨組みと PaperMod の導入

まずは Git リポジトリを作り、テーマを **git submodule** として追加しました。
submodule にしておくと、テーマ本体を自分のリポジトリに丸ごとコピーせずに済み、あとから更新するのも簡単です。

```powershell
git init -b main
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

ディレクトリ構成は `hugo new site` で作られるものと同じです（`archetypes/`、`content/`、`layouts/`、`static/` など）。

## ステップ 2：日本語向けの hugo.toml

日本語ブログとして気持ちよく使えるように、言語と日付まわりを設定しました。

```toml
title = "ひでぶログ"
theme = "PaperMod"

locale = "ja-JP"
defaultContentLanguage = "ja"
timeZone = "Asia/Tokyo"
hasCJKLanguage = true   # 日本語の文字数・読了時間を正しく数える

[params]
  DateFormat = "2006年1月2日"   # → 2026年10月20日 のように表示
  ShowReadingTime = true
  ShowWordCount = true
  ShowCodeCopyButtons = true
  ShowToc = true
```

ポイントは 2 つです。

- **`hasCJKLanguage = true`**：これがないと、日本語の文章が「1 単語」として数えられてしまい、読了時間や要約が不自然になります。
- **`DateFormat`**：Go の日付書式は `2006年1月2日` のように「**決まった日時を例として書く**」方式です。`YYYY` のような記号ではない点に注意しましょう。

メニューは Home / Posts / Tags / About の 4 つにしました。

```toml
[[menu.main]]
  name = "Home"
  url = "/"
  weight = 10
[[menu.main]]
  name = "Posts"
  url = "/posts/"
  weight = 20
# Tags, About も同様に追加
```

## ステップ 3：Hugo のインストールとローカルプレビュー

Windows では winget で一発です。PaperMod は Hugo **0.158.0 以上**が必要なので、最新の Extended 版を入れました。

```powershell
winget install Hugo.Hugo.Extended
hugo server -D
```

ブラウザで `http://localhost:1313` を開けば、プレビューが表示されます。ファイルを保存するたびに自動でリロードされるので快適です。

## ステップ 4：GitHub Actions で自動デプロイ

Hugo の公式ドキュメントで推奨されている、`actions/deploy-pages` を使う構成にしました。流れはシンプルです。

1. Hugo をインストール（バージョンはローカルと揃えて固定）
2. リポジトリを取得（`submodules: recursive` でテーマも一緒に取得）
3. `actions/configure-pages` で公開 URL を取得
4. `hugo --gc --minify --baseURL <公開URL>` でビルド
5. `actions/upload-pages-artifact` → `actions/deploy-pages` で公開

```yaml
permissions:
  contents: read
  pages: write
  id-token: write
```

権限は必要最小限だけにしています。公開リポジトリなので、ここは特に意識しました。

## ステップ 5：リポジトリ作成と公開

リポジトリ名は `ユーザー名.github.io` にしました。この名前にすると、**サブパスなしの `https://ユーザー名.github.io/`** で公開されます。

GitHub CLI（`gh`）を使うと、リポジトリの作成から Pages の設定までターミナルで完結します。

```powershell
winget install GitHub.cli
gh auth login --web
gh repo create <ユーザー名>.github.io --public
gh api -X POST repos/<ユーザー名>/<ユーザー名>.github.io/pages -f build_type=workflow
git push -u origin main
```

最後の `gh api` は、Settings → Pages の「Source」を **GitHub Actions** にするのと同じ操作です。これを忘れるとデプロイが失敗します。

## ハマったポイント集

スムーズに見えて、実はいくつかつまずいています。同じところで悩む人のためにまとめておきます。

### 1. TOML のキーが別のテーブルに入っていた

```text
WARN  Unknown kind "enablegitinfo" in outputs configuration.
```

TOML では、`[outputs]` のようなテーブルの**後ろに書いたキーは、すべてそのテーブルの中身**として扱われます。
トップレベルの設定（`enableGitInfo` など）は、**最初のテーブルより前**に書く必要があります。

### 2. 書いたはずの記事が表示されない

サンプル記事の日時を `20:00` にしていたところ、作業していた時刻がそれより前だったため「**未来の記事**」と判定され、ページが生成されませんでした。

Hugo はデフォルトで未来日時の記事を公開しません（`buildFuture = false`）。記事が出ないときは、`draft` と `date` を真っ先に確認しましょう。

```powershell
hugo list drafts   # 下書きの一覧
hugo list future   # 未来日時で非公開になっている記事の一覧
```

### 3. コミットが無いと enableGitInfo がエラーになる

```text
failed to load Git data: fatal: your current branch 'main' does not have any commits yet
```

`enableGitInfo = true` にすると、Git の履歴から更新日を取得します。そのため、**まだ 1 つもコミットが無い状態ではビルドが止まります**。いったん `false` にしておき、必要になったら有効にする方針にしました。

### 4. ワークフローファイルのプッシュが拒否された

```text
refusing to allow an OAuth App to create or update workflow
`.github/workflows/deploy.yml` without `workflow` scope
```

`.github/workflows/` 配下のファイルをプッシュするには、トークンに **`workflow` 権限**が必要です。次のコマンドで権限を追加し、ブラウザで承認したら解決しました。

```powershell
gh auth refresh -h github.com -s workflow
```

### 5. インストール直後に hugo コマンドが見つからない

winget でインストールした直後は、**開いているターミナルに PATH の変更が反映されていません**。ターミナルを開き直せば解決します。

### 6. 非推奨の設定キー

Hugo はバージョンアップのたびに設定キーが整理されます。今回は `languageName` が非推奨になっており、`label` に置き換えました。ビルド時に出る `WARN deprecated` は、放置せずにその都度直しておくと安心です。

## 公開リポジトリならではの気配り

GitHub Pages を無料で使うには、リポジトリを **Public** にする必要があります。つまり、設定ファイルもコミット履歴も**すべて世界中から見える**ということです。

そこで、次のことを意識しました。

- トークンやパスワード、ローカルのパス（`C:\Users\...`）などを、ファイルや記事に書かない
- `git add .` でまとめて追加せず、`git status` で中身を確認してからコミットする
  - 実際に、作った覚えのない空ファイルがコミットされそうになったことがありました
- GitHub Actions の権限は最小限にとどめる
- プロジェクトのルールを `CLAUDE.md` にまとめ、AI アシスタントにも同じルールを守ってもらう

## AI アシスタントと作ってみた感想

今回は、ほぼすべての作業を Claude Code との対話で進めました。

- 「日本語向けに設定して」「GitHub Pages にデプロイできるようにして」と**やりたいことを伝えるだけ**で、設定ファイルやワークフローができあがる
- 書いた設定を実際にビルドして、警告やエラーまで確認してくれる
- 一方で、ブラウザでの認証やリポジトリの名前など、**自分で決めるべきこと・自分にしかできないこと**ははっきり分かれている

ハマったポイントも、原因と一緒に説明してもらえたので、ただ動くだけでなく「なぜそうなるのか」まで理解できたのが大きな収穫でした。

## まとめ

| ステップ | やったこと |
| --- | --- |
| 1 | Git リポジトリを作り、PaperMod を submodule で導入 |
| 2 | 日本語向けの `hugo.toml` を作成（言語・タイムゾーン・日付書式・CJK 対応） |
| 3 | Hugo をインストールしてローカルでプレビュー |
| 4 | GitHub Actions で自動デプロイするワークフローを作成 |
| 5 | `ユーザー名.github.io` リポジトリを作成して公開 |

これで「書いてプッシュするだけ」の環境が整いました。これからこのブログに、日々の技術的な学びを少しずつ書きためていきます。
