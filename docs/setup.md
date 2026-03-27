# セットアップガイド

## 前提条件

- Go 1.23以上がインストール済み
- Slackワークスペースの管理者権限（またはApp作成権限）

## 1. Slack Appの作成

1. [Slack API: Applications](https://api.slack.com/apps) にアクセス
2. **Create New App** → **From scratch** を選択
3. App名（例: `slack-crawler`）を入力し、対象ワークスペースを選択
4. **Create App** をクリック

## 2. Bot Token Scopesの設定

左メニューから **OAuth & Permissions** を開き、**Bot Token Scopes** に以下を追加する。

| スコープ | 用途 |
|---------|------|
| `channels:history` | パブリックチャンネルのメッセージ取得 |
| `channels:read` | パブリックチャンネルの情報取得 |
| `groups:history` | プライベートチャンネルのメッセージ取得 |
| `groups:read` | プライベートチャンネルの情報取得 |
| `users:read` | ユーザー情報取得 |

> **Note**: プライベートチャンネルのクロールが不要であれば `groups:history` と `groups:read` は省略可。

## 3. ワークスペースへのインストール

1. **OAuth & Permissions** ページ上部の **Install to Workspace** をクリック
2. 権限の確認画面で **許可する** を選択
3. **Bot User OAuth Token**（`xoxb-` で始まる文字列）が表示されるのでコピー

## 4. 環境変数の設定

```bash
cp .env.example .env
```

`.env` を編集して以下を設定する。

```bash
# コピーしたBot Token
SLACK_BOT_TOKEN=xoxb-xxxx-xxxx-xxxx

# クロール対象チャンネルID（カンマ区切りで複数指定可）
SLACK_CHANNEL_IDS=C0XXXXXXX,C0YYYYYYY
```

> **チャンネルIDの確認方法**: Slackでチャンネルを開き、チャンネル名をクリック → モーダル最下部にチャンネルIDが表示される。

## 5. Botをチャンネルに招待

クロール対象の各チャンネルで以下のコマンドを実行する。

```
/invite @slack-crawler
```

Botがチャンネルに参加していないと `not_in_channel` エラーになるため、この手順は必須。

## 6. ビルドと実行

```bash
# ビルド
go build -o slack-crawler ./cmd/slack-crawler

# チャンネル一覧の確認
./slack-crawler channels

# クロール実行
./slack-crawler crawl --channel C0XXXXXXX
```

## トラブルシューティング

### `invalid_auth` エラー

- `SLACK_BOT_TOKEN` が正しくコピーされているか確認
- トークンが `xoxb-` で始まっているか確認（User Tokenの `xoxp-` ではない）

### `not_in_channel` エラー

- 対象チャンネルでBotを `/invite` しているか確認
- プライベートチャンネルの場合、`groups:history` スコープが付与されているか確認

### `missing_scope` エラー

- Slack App設定で必要なスコープが全て追加されているか確認
- スコープ追加後はAppの再インストールが必要（OAuth & Permissions → Reinstall to Workspace）
