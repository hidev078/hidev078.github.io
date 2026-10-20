# CLAUDE.md — ひでぶログ（Hugo + PaperMod）

このファイルは、このリポジトリで作業する Claude Code への指示書。**作業前に必ず全体を読み、ここに書かれたルールを最優先で守ること。**

## 1. プロジェクト概要

**ひでぶログ** — Hugo + PaperMod で構築した日本語の技術ブログ。

| 項目 | 値 |
| --- | --- |
| 公開URL | https://hidev078.github.io/ |
| リポジトリ | `hidev078/hidev078.github.io`（**Public**・ユーザーサイト） |
| ブランチ | `main`（このブランチのみ。プッシュ＝即本番公開） |
| デプロイ | GitHub Actions `.github/workflows/deploy.yml` → `actions/deploy-pages` |
| Hugo | Extended **0.168.0**（ローカル・CI で統一。PaperMod は 0.158.0 以上が必須） |
| テーマ | PaperMod（`themes/PaperMod/`、git submodule） |
| 言語・地域 | `ja` / `ja-JP` / `Asia/Tokyo` / `hasCJKLanguage = true` |

## 2. 作業環境（Windows）

- OS は Windows 11。シェルは PowerShell（主）と Git Bash
- Hugo は winget でインストール済み。新しいシェルで `hugo` が見つからない場合は PATH を再読込する:
  ```powershell
  $env:Path = [Environment]::GetEnvironmentVariable('Path','User') + ';' + [Environment]::GetEnvironmentVariable('Path','Machine')
  ```
- GitHub CLI は `C:\Program Files\GitHub CLI\gh.exe`（PATH 未反映時はフルパスで呼ぶ）
- 改行コードは Git が LF/CRLF を自動変換する（`LF will be replaced by CRLF` 警告は無害）
- 一時的なビルド出力は `$env:TEMP` 配下に出し、作業後に削除する。リポジトリ内の `public/` を確認用に使わない

## 3. コマンド

```powershell
hugo server -D                                  # プレビュー http://localhost:1313（下書き含む・自動リロード）
hugo server                                     # 本番と同じ条件（下書き除外）でプレビュー
hugo new content posts/<slug>.md                # 新規記事（archetypes/default.md から生成）
hugo new content posts/<slug>/index.md          # 画像付き記事（ページバンドル）
hugo --gc --minify -d "$env:TEMP\hugo-check"    # 本番ビルド確認（WARN/ERROR が出ないこと）
hugo list drafts                                # 下書き一覧
hugo list future                                # 未来日時で非公開の記事一覧
git submodule update --init --recursive         # clone 直後にテーマが空のとき
git submodule update --remote themes/PaperMod   # テーマ更新（要ユーザー確認・下記参照）
```

- `hugo server` をバックグラウンドで起動した場合は、作業終了時に停止する
- GitHub Actions の結果確認: `gh run list -R hidev078/hidev078.github.io --limit 1` → `gh run watch <id> --exit-status`

## 4. ディレクトリ構成

| パス | 内容 |
| --- | --- |
| `hugo.toml` | サイト設定（唯一の設定ファイル。`config/` ディレクトリは作らない） |
| `content/posts/` | ブログ記事 |
| `content/about.md` | About ページ（メニュー: About） |
| `content/archives.md` / `content/search.md` | アーカイブ・検索ページ（`layout` 指定。メニュー非表示だが URL は有効） |
| `archetypes/default.md` | 記事テンプレート |
| `layouts/` | テーマの上書き（Hugo 0.146+ の新テンプレート構成: `_partials/`, `_shortcodes/`, `_markup/`） |
| `assets/` | Hugo Pipes で処理する CSS など（`assets/css/extended/*.css` は PaperMod が自動読込） |
| `static/` | そのまま配信するファイル（favicon、`robots.txt` 以外の静的ファイル） |
| `i18n/` | 翻訳文言の上書き（`i18n/ja.yaml`） |
| `themes/PaperMod/` | テーマ本体（**直接編集禁止**） |
| `.github/workflows/deploy.yml` | ビルド＆デプロイ |
| `public/`, `resources/_gen/`, `.hugo_build.lock` | 生成物（`.gitignore` 済み。コミットしない） |

## 5. 設定（hugo.toml）の要点

- TOML は**テーブル（`[xxx]`）の後に書いたキーはそのテーブルに属する**。トップレベルのキーは必ず最初のテーブルより前に書く
- 非推奨キーを使わない（例: `languageName` → `label`、`paginate` → `[pagination] pagerSize`）。追加・変更後は `hugo` を実行して `WARN deprecated` が出ないことを確認する
- `baseURL = "https://hidev078.github.io/"`。CI では `--baseURL` で上書きされる
- `DateFormat = "2006年1月2日"`（Go の日付レイアウト。`2006-01-02` の数字は固定値）
- `buildFuture = false` / `buildDrafts = false`: 未来日時・下書きは公開されない
- `enableGitInfo = false`: 有効化すると `lastmod` を Git から取得する（CI は `fetch-depth: 0` 済み）
- `[outputs] home = ["HTML", "RSS", "JSON"]`: JSON は検索ページに必須。削除しない
- `markup.goldmark.renderer.unsafe = true`: 記事内の生 HTML がそのまま出力される（セキュリティ節を参照）
- メニューは `[[menu.main]]`（Home / Posts / Tags / About、`weight` 10 刻み）

## 6. 記事の書き方

### ファイル

- ファイル名（slug）は **半角英小文字・数字・ハイフン** のみ（例: `hugo-github-pages.md`）。日本語ファイル名は使わない
- 画像を使う記事はページバンドル（`content/posts/<slug>/index.md` と同じフォルダに画像）にする
- 新規記事は `draft = true` で作成し、ユーザーの確認後に `false` にする

### フロントマター（TOML `+++` で統一）

```toml
+++
title = '記事タイトル'
date = 2026-10-20T09:00:00+09:00   # タイムゾーン必須。現在時刻より未来だと公開されない
draft = true
tags = ['Hugo', 'GitHub']          # 既存タグの表記に揃える（大文字小文字・全角半角）
categories = ['技術メモ']
summary = '一覧・OGP に出る要約（120字程度）'
# --- PaperMod の任意パラメータ ---
# ShowToc = true / TocOpen = false
# hideMeta = false / hideSummary = false
# searchHidden = false
# [cover]
#   image = 'cover.png'   # ページバンドル内の画像
#   alt = '画像の説明'
#   relative = true
+++
```

- `date` の時刻を今日の日付にする場合は、**現在時刻より前**にする（未来だと 404 になる）
- 既存のタグ・カテゴリは `hugo list all` や `content/` を grep して確認し、表記ゆれを作らない

### 本文

- 見出しは `##` から始める（`#` は記事タイトルに使われる）
- コードブロックには必ず言語名を付ける（```` ```powershell ```` など）。コピーボタンとハイライトが有効
- 画像は Markdown 記法 `![代替テキスト](image.png)` を使い、代替テキストを必ず書く
- テーマ付属ショートコード: `figure`, `collapse`, `video`, `audio`, `rawhtml`, `inTextImg`
- 日本語と英数字の間のスペースは記事内で統一する（既存記事に合わせる）

## 7. テーマのカスタマイズ

- **`themes/PaperMod/` 内のファイルは絶対に編集しない**（submodule の更新で消える・差分がコミットされない）
- 上書きは同じパスで `layouts/` に置く（例: `layouts/_partials/footer.html`）。丸ごとコピーより、まず以下の拡張ポイントを使う:
  - `layouts/_partials/extend_head.html` — `<head>` への追加（メタタグ、CSS など）
  - `layouts/_partials/extend_footer.html` — フッター/本文末尾への追加（スクリプトなど）
  - `layouts/_partials/comments.html` — コメント欄（有効化は `comments = true`）
  - `assets/css/extended/custom.css` — 独自 CSS（自動で結合・fingerprint される）
- 翻訳文言の変更は `i18n/ja.yaml` に必要なキーだけ書く（テーマの `themes/PaperMod/i18n/ja.yaml` を参照）
- テーマ更新（`git submodule update --remote`）は、更新前後のコミットと変更点（`theme.toml` の `min_version` を含む）を確認し、ユーザーの了承を得てから行う。`min_version` が上がった場合は `deploy.yml` の `HUGO_VERSION` も確認する

## 8. 作業後の確認チェックリスト

変更をコミットする前に、該当するものを確認する:

1. `hugo --gc --minify -d "$env:TEMP\hugo-check"` で **WARN / ERROR が 0 件**
2. 追加・変更したページが出力されている（未来日時・`draft = true` で消えていないか）
3. `<title>` やメニューなど、変更点が出力 HTML に反映されている
4. 確認用の出力ディレクトリを削除した
5. `git status` で意図したファイルだけが変更されている

プッシュ後は GitHub Actions の結果を確認し、公開 URL が HTTP 200 を返すことを確かめてから完了を報告する。失敗した場合はログ（`gh run view <id> --log-failed`）を確認して原因を報告する。

## 9. Git 運用

- コミット・プッシュは**ユーザーの明示的な指示があるときだけ**行う（`main` へのプッシュ＝即公開）
- `git add -A` / `git add .` は使わず、変更したファイルを明示して add する。add 前に `git status` を確認する
- 見覚えのないファイル・変更（空ファイル、一時ファイル、ユーザーが手で加えた変更）があれば、コミット前にユーザーへ報告する
- コミットメッセージは英語の命令形で 1 行目を簡潔に（例: `Add post about Hugo shortcodes`）
- `git push --force`、`git reset --hard`、履歴の書き換え、ブランチ・リポジトリ・Pages 設定の削除や変更は、ユーザーの確認なしに行わない
- submodule のポインタ（`themes/PaperMod`）は、テーマ更新を指示されたときだけ変更する

## 10. セキュリティ（必ず守る）

このリポジトリは**公開リポジトリ**であり、プッシュした内容はコミット履歴を含めて全世界に公開・キャッシュされる。削除しても残る前提で扱うこと。

### 秘密情報・個人情報

- APIキー、トークン（`ghp_` / `gho_` など）、パスワード、秘密鍵、`.env`、クッキー、認証付き URL を**ファイル・記事・コミットメッセージに一切書かない**
- 記事中のコード例・ログ・スクリーンショットに含まれる認証情報、メールアドレス、社内・ローカルの URL、IP アドレス、ローカルの絶対パス（`C:\Users\...`）、ユーザー名は伏せ字かダミー値に置き換える
- 画像を追加する場合、写り込み（デスクトップ、通知、ブラウザのタブ、住所など）に注意し、気になる点はユーザーに確認する
- 本名・住所・勤務先・顔写真など、ユーザーが公開を明示していない個人情報を書かない
- 秘密情報を誤ってコミットした場合は、**プッシュせず**直ちにユーザーへ報告する。プッシュ済みなら、履歴の書き換えより先に**該当キーの無効化・再発行**をユーザーに促す

### GitHub Actions

- `permissions` は最小限（`contents: read`, `pages: write`, `id-token: write`）を維持し、広げない
- アクションは公式（`actions/*`）を優先する。サードパーティ製を追加する場合はユーザーに確認し、**コミット SHA で固定**する
- Hugo は公式リリース（`github.com/gohugoio/hugo/releases`）からのみ取得する
- `pull_request_target` トリガーや、外部入力（Issue タイトル、PR 本文、ブランチ名など）を `run:` に直接埋め込む書き方をしない
- シークレットを `echo` などでログに出力しない

### コンテンツ・テーマ

- `unsafe = true` のため、記事内の HTML はそのまま出力される。**外部から取得した HTML / JavaScript を記事に貼らない**
- `<script>`、外部 `<iframe>`、アクセス解析・広告・コメント欄・トラッキングコードを追加する場合はユーザーに確認する
- 外部リソース（CDN の JS/CSS）は極力使わない。使う場合は信頼できる提供元に限り、可能なら SRI（`integrity` 属性）を付ける
- 外部リンクを `target="_blank"` で開く場合は `rel="noopener noreferrer"` を付ける
- テーマ更新時は、変更内容に不審な外部通信やスクリプト追加がないか確認する

## 11. トラブルシューティング

| 症状 | 原因と対処 |
| --- | --- |
| 記事が表示されない（404） | `draft = true` / `date` が未来 / ファイル名・`url` の誤り。`hugo list drafts` `hugo list future` で確認 |
| テーマが空・`layout not found` | submodule 未取得。`git submodule update --init --recursive` |
| `failed to load Git data` | `enableGitInfo = true` かつコミット無し。初回コミット後に有効化する |
| `WARN ... deprecated` | 非推奨の設定キー。警告文の指示どおり新しいキーに置き換える |
| `Unknown kind "xxx" in outputs` | TOML のキーが `[outputs]` などのテーブル内に入っている。テーブルより前に移動 |
| プッシュが `workflow` scope で拒否 | `gh auth refresh -h github.com -s workflow` をユーザーにブラウザで承認してもらう |
| CI でビルド失敗 | `gh run view <id> --log-failed`。ローカルと `HUGO_VERSION` が一致しているか確認 |
| 公開サイトに反映されない | Actions の成功を確認後、数十秒〜数分待ってキャッシュを無視して再読込 |
| 検索ページが動かない | `[outputs] home` に `JSON` があるか、`content/search.md` の `layout = 'search'` を確認 |
