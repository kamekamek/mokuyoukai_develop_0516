---
marp: true
theme: default
class: invert
backgroundColor: '#111827'
color: '#f9fafb'
---

# AI個人開発してみようオフ会

2025.05.17 Sun @ Michikusa Office

---

## 本日のスケジュール

| 時間 | 内容 |
| --- | --- |
| 13:15〜13:45 | 説明 + ハンズオン |
| 13:45〜14:15 | 開発タイム・交流 |
| 14:15〜14:45 | デプロイタイム |
| 14:45〜16:00 | 開発タイム・交流 |

---

## 今日のゴール

個人開発する上で自分やユーザーを守るために最低限押さえておくべき考え方を知る

---

![w:1000](./assets/selfdev_flow.png)

---

# 個人開発する上で気をつけること

---

## GitHubは基本プライベートで作る

GitHubはデフォルトでパブリックリポジトリとして作られるので注意。
基本的にプライベートリポジトリで作る。

| 種類 | 公開範囲 |
| --- | --- |
| パブリック | 全世界に公開される |
| プライベート | 自分・指定した人のみ |

**プライベートにする理由**
- 間違えて機密情報を入れても被害を最小限にできる
- アイデアや未完成の機能を守れる

---

プライベートだからといって何を書いても安全ではない。
APIキーやパスワードはプライベートリポジトリにも置かないのが基本。

![w:700](./assets/repository.png)


---

## APIキー・シークレットキーについて

LINE・OpenAI・Slackなどの外部サービスと連携するときに発行される「サービスを使うための鍵」。

**やってはいけないこと**
- コードに直接書く（ハードコーディング）
- GitHubにアップロードする
- AIへの質問文に貼る

```js
// 悪い例
const openaiApiKey = "sk-xxxxxxxxxxxxxxxx";

// よい例
const openaiApiKey = process.env.OPENAI_API_KEY;
```

---

## APIキーは `.env` に入れる

```env
OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxx
LINE_CHANNEL_ACCESS_TOKEN=xxxxxxxxxxxxxxxx
SLACK_BOT_TOKEN=xoxb-xxxxxxxxxxxxxxxx
```

`.env` はGitHubに上げない。`.gitignore` に追加する。

```gitignore
.env
```


---


`.env.example` ：環境変数のサンプル
項目名だけ書いてgitで管理する（値は書かない）

```env
OPENAI_API_KEY=
LINE_CHANNEL_ACCESS_TOKEN=
SLACK_BOT_TOKEN=
```

🐰Vercelでデプロイする場合は「[Environment Variables](https://vercel.com/docs/environment-variables)」で設定する必要がある


※APIキー等がコミットされてないか確認する方法として、`gitleaks`というツールがある（詳細は参考資料）

---

## API keyの定期的な監視とローテーション

- API keyの利用上限を設定しておく
- 定期的に使用状況の確認（通知設定できると良い。）
- できれば、API Keyをローテーション（定期的な再発行）する



---

## OpenAIのダッシュボード

![w:900](./assets/openai_dashboard.png)

https://platform.openai.com/usage




---

## 上限の設定(OpenAI)
![w:900](./assets/limit_dashborad.png)
https://platform.openai.com/settings/organization/limits

---

## キーをPushしてしまった場合

消しても履歴に残る可能性がある。**「作り直す」が基本。**

1. そのキーを無効化する
2. 新しいキーを発行する
3. `.env` やVercelの値を差し替える
5. 不審な利用や課金がないか確認する

---

## AIに危険なコマンドを実行させない

| コマンド | 理由 |
| --- | --- |
| `rm` | ファイルを完全に削除する |
| `rmdir` | ディレクトリを削除する |
| `chmod` | ファイル権限を変更する |
| `sudo` | 管理者権限で実行する |
| `mv` | ファイル移動（上書きリスクあり） |
| `cp` | ファイルコピー（上書きリスクあり） |

Macでは `.zshrc` に以下を追加するとゴミ箱に送れる。

```shell
alias rm =  command trash "$@" 
```


---

## Claude Code の設定で制限する

`.claude/settings.json` のDenyリストに追加する。
完璧ではないが、防げるところは防いでおく。

```json
{
  "permissions": {
    "deny": [
      "Bash(rm:*)",
      "Bash(rm -rf ~*)",
      "Bash(rm -rf .*)",
      "Bash(sudo:*)",
      "Bash(rmdir:*)"
    ]
  }
}
```

---

# ハンズオン

---

## Gitで戻れる状態を作る

Gitは変更履歴を残すための道具。
AIにコードを書かせると、一度にたくさんのファイルが変わることがある。

作業前にコミットしておくと、問題が起きたときに戻しやすくなる。

```shell
git add .
git commit -m "作業前の状態を保存"

# 作業後の変更を確認する。
git log --oneline 
```

logの確認には
VSCodeの拡張機能「**Git Graph**」がおすすめ。

---

## コミットの取り消し

Gitの変更は3段階を経る。`git reset` はどの段階まで巻き戻すかを指定するコマンド。

| オプション | コミット | ステージ | 作業ディレクトリ |
| --- | --- | --- | --- |
| `--soft` | 取り消す | 残る | 残る |
| なし（`--mixed`） | 取り消す | 消える | 残る |
| `--hard` | 取り消す | 消える | **消える** |

基本は `--soft` を使う。変更内容が消えないのでやり直しがきく。

---

## git reset の使い方

`HEAD` は「今いるコミット」を指す。`HEAD~1` は「1つ前のコミット」。

```shell
# 直前のコミットを取り消す
git reset --soft HEAD~1

# 特定のコミットまで戻す
git reset --soft abc1234
```

コミットIDは `git log --oneline` で確認できる。

---

## 現在地

| 時間 | 内容 | |
| --- | --- | --- |
| 13:00〜13:30 | 講義 + ハンズオン | ✅ |
| **13:30〜14:30** | **開発タイム・交流** | 👈 |
| 14:30〜15:00 | Vercel解説・作ったもの共有・クロージング | |

---

# 開発タイム・交流

AIを使って何か作ってみよう。
詰まったら隣の人に聞いてみよう。

---

## 現在地

| 時間 | 内容 | |
| --- | --- | --- |
| 13:00〜13:30 | 講義 + ハンズオン | ✅ |
| 13:30〜14:30 | 開発タイム・交流 | ✅ |
| **14:30〜15:00** | **Vercel解説・作ったもの共有・クロージング** | 👈 |

---

## Vercelでデプロイする

- Pushすると自動でデプロイされる
- 環境変数をVercel側（[Environment Variables](https://vercel.com/docs/environment-variables)）で設定する
- Claude CodeのVercelプラグインを使うと「デプロイして」だけで設定が完了する

---

## ブランチ戦略

最初は `main` だけでよい。公開アプリになってきたら2つに分ける。

| ブランチ | 役割 |
| --- | --- |
| `main` | 本番用。常に動く状態を保つ |
| `develop` | 開発用。AIと一緒に作業する場所 |

**なぜ分けるのか**
- 本番環境に影響を与えず、機能開発を進められる
- developブランチで一度デプロイして動作確認することで本番環境での予期せぬ不具合を事前に防ぐことができる

---

## **開発の流れ**
1. `develop` ブランチを作成して開発する
2. `main` に対してPR(Pull Request)を作成する
3. PRの中で発行されたVercelのプレビューURLで確認する
4. 問題なければ `main` にマージする

---

新しいブランチの作り方

```bash
git checkout -b develop
```

checkout: ブランチの移動
-b：新しいブランチを作ってそのブランチに移動する

---

## VercelのプレビューURLを使う

VercelをGitHubと連携すると、ブランチ・PRごとにプレビューURLが作られる。

**確認できること**
- 画面が崩れていないか
- スマホでも見られるか
- AIが変えた内容が正しいか
- 本番に出してよい状態か

いきなり本番に出すのではなく、プレビューURLで確認してから本番に入れる。

---

# 作ったもの共有

作ったもの・開発中のもの・気づき、なんでもOK

---

# まとめ

- GitHubはプライベートで。APIキーはコードに書かない
- `.env` に書いて `.gitignore` で除外、Vercelの環境変数に設定する
- 従量課金のAPIを使うときは利用上限を設定し、定期的に使用状況を確認する
- API. Keyをushしてしまったら即無効化・再発行する
- AIに `rm` / `sudo` などの危険なコマンドを実行させない
- 作業前にコミットして「戻れる状態」を作る

---

# 補足資料

---

## gitleaks でスキャンする

コミット履歴にAPIキー等が含まれていないかをスキャンするツール。

```shell
# スキャン
gitleaks detect

# 詳細表示
gitleaks detect --verbose
```

**インストール（Mac）**
```shell
brew install gitleaks
```

**Homebrewのインストール（Mac / WSL共通）**
```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

---

## gitleaks（Windows）

1. https://github.com/gitleaks/gitleaks/releases にアクセス
2. 最新版の `gitleaks_x.x.x_windows_x64.exe` をダウンロード
3. 任意の場所（例：`C:\tools\gitleaks`）に保存
4. そのフォルダを環境変数 PATH に追加
5. Git Bash を開いて疎通確認

```shell
gitleaks version
```

---

## gitleaks の出力例

```
Finding:     OPENAI_API_KEY=sk-proj-aB3xK9mNpQ2...
Secret:      sk-proj-aB3xK9mNpQ2...
RuleID:      openai-api-key
File:        .env
Line:        1
Commit:      a1b2c3d4e5f6...
Author:      your-name
Date:        2025-05-03T17:05:00Z

leaks found: 1
```

見つかったら該当のAPIキーをすぐ無効化して新しいキーを発行する。

---

## 要件定義の進め方

- AIにヒアリングしてもらいながら要件を固める
- UIをバイブコーディングで先に作り、後から要件を詰める方法もあり

**ポイント**
- 機能を盛り込みすぎない
- AIの回答（文章）をちゃんと読む
- 理解できないところは深掘りして、自分の言葉で残す

```
今から〇〇というアプリを作りたい。
一問一答でヒアリングして要件を固めてください。
```

---

## AIの出力はMarkdownよりHTMLが強い

Markdownは読みやすいが、**100行を超えると読まれにくい。**
HTMLならAIが色・図・表・インタラクションを自由に表現できる。

---

## Markdownでは難しいこと

| Markdown | HTML |
| --- | --- |
| 文字・箇条書きのみ | 色・デザイン（CSS） |
| ASCII図が限界 | 図・フローチャート（SVG） |
| 静的なテキスト | インタラクション（JavaScript） |

---

## HTMLで作れるもの

- 仕様書・実装計画
- PRのコードレビュー資料
- ビジュアルレポート・週次報告

```
「〇〇について調べてHTMLファイルにまとめて」
```

---

## 参考

- https://zenn.dev/motowo/articles/ai-agent-html-output-design
- https://thariqs.github.io/html-effectiveness/
- https://x.com/trq212/status/2052809885763747935

---

##