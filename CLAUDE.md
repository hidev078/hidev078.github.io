# CLAUDE.md

## プロジェクト概要

**My Tech Notes** — Hugo + PaperMod で構築した日本語の技術ブログ。

- 公開URL: https://hidev078.github.io/
- リポジトリ: `hidev078/hidev078.github.io`（**Public**）
- `main` へのプッシュで GitHub Actions（`.github/workflows/deploy.yml`）がビルドし、`actions/deploy-pages` で GitHub Pages に公開される

## 構成

| パス | 内容 |
| --- | --- |
| `hugo.toml` | サイト設定（日本語・Asia/Tokyo・CJK対応、メニュー、PaperMod パラメータ） |
| `content/posts/` | ブログ記事（Markdown、TOML フロントマター `+++`） |
| `content/about.md` | About ページ |
| `content/archives.md`, `content/search.md` | アーカイブ・検索ページ（メニュー非表示） |
| `archetypes/default.md` | `hugo new` で使う記事テンプレート |
| `themes/PaperMod/` | テーマ（git submodule。**直接編集しない**） |
| `layouts/`, `assets/`, `static/` | テーマの上書き・独自ファイルはここに置く |

## よく使うコマンド

```powershell
hugo server -D                         # ローカルプレビュー（下書き含む） http://localhost:1313
hugo new content posts/<slug>.md       # 新規記事
hugo --gc --minify                     # 本番ビルドの確認（出力は public/、コミットしない）
git submodule update --remote themes/PaperMod   # テーマ更新
```

- Hugo は Extended 0.168.0（PaperMod は 0.158.0 以上が必要）。ワークフローの `HUGO_VERSION` とローカルのバージョンを揃える
- `buildFuture = false` のため、未来日時の記事は公開されない
- 記事の `date` はタイムゾーン付き（例: `2026-10-20T09:00:00+09:00`）で書く

## 記事の書き方

- ファイル名（slug）は半角英小文字とハイフン（例: `hugo-github-pages.md`）
- フロントマターには `title`, `date`, `draft`, `tags`, `categories` を含める
- 公開前は `draft = true` で作成し、ユーザーの確認後に `false` にする

## セキュリティ指示（必ず守る）

このリポジトリは**公開リポジトリ**であり、プッシュした内容はそのまま全世界に公開・キャッシュされる。削除しても履歴やキャッシュに残る前提で扱うこと。

### 秘密情報・個人情報

- APIキー、トークン、パスワード、秘密鍵、`.env` などの認証情報を**ファイル・記事・コミットメッセージに一切書かない**
- 記事中のコード例・ログ・スクリーンショットに含まれるトークン、メールアドレス、社内URL、IPアドレス、ローカルの絶対パス（`C:\Users\...`）は伏せ字かダミー値に置き換える
- 本名・住所・勤務先など、ユーザーが明示していない個人情報を書かない
- 秘密情報を誤ってコミットした場合は、プッシュせずに直ちにユーザーへ報告する。プッシュ済みなら、履歴の書き換えより先に**該当キーの無効化・再発行**をユーザーに促す

### コミット・プッシュ

- `git add -A` / `git add .` を使う前に `git status` で対象を確認し、意図しないファイル（空ファイル、一時ファイル、バックアップ、ログ）を含めない
- `main` へのプッシュは即公開になるため、**ユーザーの明示的な指示があるときだけ**行う
- `git push --force`、履歴の書き換え、ブランチ・リポジトリの削除はユーザーの確認なしに行わない
- `public/`, `resources/_gen/`, `.hugo_build.lock` はコミットしない（`.gitignore` 済み）

### GitHub Actions

- ワークフローの `permissions` は最小限（`contents: read`, `pages: write`, `id-token: write`）を維持し、広げない
- 使うアクションは公式（`actions/*`）か信頼できる提供元に限る。サードパーティ製アクションを追加する場合はコミットSHAで固定する
- `pull_request_target` トリガーや、外部入力（Issue タイトル、PR 本文など）を `run:` に直接埋め込む書き方をしない
- シークレットを `echo` などでログに出力しない

### コンテンツ・テーマ

- `hugo.toml` の `markup.goldmark.renderer.unsafe = true` により記事内の生 HTML がそのまま出力される。外部から貰った HTML/JS を記事に貼らない。`<script>` や外部 `<iframe>` を追加する場合はユーザーに確認する
- 外部スクリプト・トラッキングコード（アクセス解析、広告、コメント欄など）を追加する場合はユーザーに確認する
- 外部サイトへのリンクを `target="_blank"` で開く場合は `rel="noopener noreferrer"` を付ける
- テーマ（submodule）の更新は、変更内容を確認してから行う
