# Marp CLI 使い方メモ

`slides/` フォルダ内で実行する。

> ローカルの画像を使っている場合は `--allow-local-files` が必要。

```bash
# HTMLで出力（ブラウザで開けるスライド）
marp slide-deck.md --allow-local-files

# PDFで出力
marp slide-deck.md --pdf --allow-local-files

# PPTXで出力
marp slide-deck.md --pptx --allow-local-files

# ブラウザでリアルタイムプレビュー（編集しながら確認）
marp --server .
```

プレビューは http://localhost:8080 で開く。

## インストール

```bash
npm install -g @marp-team/marp-cli
```

## PDF出力で日本語が文字化けする場合

Chrome/Chromiumが必要。インストール済みであれば自動で使われる。

```bash
CHROME_PATH="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" marp slide-deck.md --pdf
```
