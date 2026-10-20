+++
title = 'テスト記事: Markdown の表示確認'
date = 2026-10-19T10:00:00+09:00
draft = false
tags = ['Hugo', 'テスト']
categories = ['技術メモ']
summary = 'Markdown の各要素がテーマでどう表示されるかを確認するためのテスト記事です。'
+++

この記事は **ひでぶログ** の表示確認用テスト記事です。

## 見出しとテキスト

通常の段落、*斜体*、**太字**、`インラインコード`、[リンク](https://gohugo.io/) の表示を確認します。

### リスト

- 箇条書き 1
- 箇条書き 2
  - ネストした項目

1. 番号付き 1
2. 番号付き 2

## コードブロック

```python
def greet(name: str) -> str:
    return f"こんにちは、{name}さん"

print(greet("Hugo"))
```

```powershell
hugo server -D
```

## 引用と表

> Hugo は世界最速クラスの静的サイトジェネレーターです。

| 項目 | 値 |
| --- | --- |
| ジェネレーター | Hugo |
| テーマ | PaperMod |
