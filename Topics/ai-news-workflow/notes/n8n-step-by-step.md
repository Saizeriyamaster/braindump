---
title: "n8nでAIニュース自動収集・要約・Slack投稿ワークフロー：Step by Step"
created: "2026-03-13"
topic: ai-news-workflow
tags:
  - topic/ai-news-workflow
---

# n8nでAIニュース自動収集・要約・Slack投稿ワークフロー：Step by Step

→ [[workflow-approaches]] のアプローチAの詳細手順

## Summary
- n8nはDockerで1コマンド起動。ノード同士をつなぐだけでワークフローが作れる
- RSS Feed → Aggregate → HTTP Request（Claude API）→ Slack の4ステップが基本構成
- Credentialsに各APIキーを登録しておくと各ノードで使い回せる

## Notes

---

### STEP 0：n8nを起動する

**Dockerで起動（最速）：**
```bash
docker run -it --rm \
  -p 5678:5678 \
  -v ~/.n8n:/home/node/.n8n \
  n8nio/n8n
```
ブラウザで `http://localhost:5678` を開く。
`-v ~/.n8n:/home/node/.n8n` でデータを永続化（これがないと再起動でワークフローが消える）。

**n8n.cloud（クラウド版）でも可：**
→ アカウント作成するだけ。Dockerなし。無料プランあり。

---

### STEP 1：Credentialsを登録する

左サイドバー → **Credentials** → **Add Credential**

#### 1-a. Claude API（Anthropic）
- Type: `HTTP Header Auth`
- Name: `Anthropic API`
- Header Name: `x-api-key`
- Header Value: `sk-ant-xxxxxxxx`（Anthropic Consoleで取得）

#### 1-b. Slack
- Type: `Slack OAuth2 API` または `Slack API`
- Bot Token（`xoxb-...`）を入力
- Slack App設定で `chat:write` スコープを付与しておく

> Slack Appの作り方：https://api.slack.com/apps → Create New App → Bot Token Scopes に `chat:write` 追加 → ワークスペースにインストール

---

### STEP 2：新しいワークフローを作成

左サイドバー → **Workflows** → **New Workflow**

---

### STEP 3：Cron（スケジュール）ノードを追加

「+」ボタン → `Schedule Trigger` を検索して追加

設定：
- Trigger Interval: `Cron`
- Expression: `0 8 * * *`（毎朝8時に実行）
  - 分 時 日 月 曜日 の順。`0 23 * * *` にするとUTC 23:00 = JST 8:00

---

### STEP 4：RSS Feedノードを追加（ニュース取得）

Schedule Triggerの右の「+」→ `RSS Feed Read` を検索して追加

設定例（TechCrunch AI）：
- URL: `https://techcrunch.com/category/artificial-intelligence/feed/`

**複数サイト取得する場合：**
RSSノードを複数並べて同じSchedule Triggerにつなぐ。

例：
```
Schedule Trigger
  ├→ RSS Feed（TechCrunch AI）
  ├→ RSS Feed（VentureBeat AI）
  ├→ RSS Feed（The Verge AI）
  └→ RSS Feed（AINOW）
```

各RSSノードは出力として `title`, `link`, `pubDate`, `content` などのフィールドを持つ。

---

### STEP 5：Aggregateノードで記事をまとめる

複数のRSSノードの後ろに `Merge` または `Aggregate` ノードを追加。

- ノード種別：`Merge` → Mode: `Append`（全ノードの出力を1つにまとめる）

これで複数サイトの記事が1つのリストになる。

---

### STEP 6：Codeノードで記事テキストを整形する

`Code` ノードを追加（JavaScriptで自由に処理できる）

```javascript
// 直近24時間の記事だけに絞り込み、テキストを整形する
const items = $input.all();
const oneDayAgo = new Date(Date.now() - 24 * 60 * 60 * 1000);

const filtered = items
  .filter(item => new Date(item.json.pubDate) > oneDayAgo)
  .slice(0, 20); // 最大20件

const articleText = filtered
  .map((item, i) => `${i + 1}. [${item.json.title}](${item.json.link})\n${item.json.contentSnippet || ''}`)
  .join('\n\n');

return [{ json: { articleText, count: filtered.length } }];
```

---

### STEP 7：HTTP Requestノードで Claude API を呼ぶ

`HTTP Request` ノードを追加

設定：
- Method: `POST`
- URL: `https://api.anthropic.com/v1/messages`
- Authentication: Credential で `Anthropic API`（STEP 1で登録したもの）を選択
- Headers:
  - `anthropic-version`: `2023-06-01`
  - `content-type`: `application/json`
- Body（JSON）:

```json
{
  "model": "claude-sonnet-4-6",
  "max_tokens": 1500,
  "messages": [
    {
      "role": "user",
      "content": "以下のAIニュース記事リストを日本語で要約してください。\n・重要度の高いものを上位5件選んで\n・各記事を2〜3行で要約\n・最後に全体の傾向を1段落でまとめる\n\n{{ $json.articleText }}"
    }
  ]
}
```

> `{{ $json.articleText }}` はn8nの式言語。前のCodeノードの出力を参照する。

レスポンスから要約テキストを取り出すには次のCodeノードで：
```javascript
const response = $input.first().json;
const summary = response.content[0].text;
return [{ json: { summary } }];
```

---

### STEP 8：Slackノードで投稿

`Slack` ノードを追加

設定：
- Resource: `Message`
- Operation: `Post`
- Channel: `#ai-news`（チャンネル名 or ID）
- Text:

```
*今日のAIニュースまとめ 📰*

{{ $json.summary }}
```

- Credential: STEP 1で登録した Slack Credentialを選択

---

### STEP 9：テスト実行

ワークフロー上部の **「Test workflow」** ボタンをクリック。
各ノードの出力が右側パネルに表示されるので、データが正しく流れているか確認する。

問題なければ **「Activate」** トグルをONにして本番稼働。

---

### 完成したフロー全体図

```
[Schedule Trigger（毎朝8時）]
  ├→ [RSS Feed: TechCrunch AI]
  ├→ [RSS Feed: VentureBeat]
  ├→ [RSS Feed: The Verge AI]
  └→ [RSS Feed: AINOW]
        ↓
  [Merge（Append）]
        ↓
  [Code（フィルタ・整形）]
        ↓
  [HTTP Request（Claude API）]
        ↓
  [Code（レスポンス取り出し）]
        ↓
  [Slack（#ai-newsに投稿）]
```

---

### よくあるトラブル

| 症状 | 原因・対処 |
|------|-----------|
| RSSノードがエラー | URLが間違っている or サイトがRSSを廃止している |
| Claude APIが401エラー | APIキーが間違い or Credentialの設定ミス |
| Slackに投稿されない | Bot TokenのスコープにchatWriteがない or チャンネルにBotを招待していない |
| 記事が空 | 直近24時間のフィルタが厳しすぎる → 期間を広げる |

---

### コスト感（参考）

- Claude Sonnet 4.6: 入力$3/MTok・出力$15/MTok
- 記事20件×平均500文字 = 約10,000トークン（入力） → 約$0.03/回
- 毎日実行で月 ≈ **$1以下**

## Questions
- Slack Block Kitを使ってリッチな投稿フォーマットにするには？
- エラー時にメール通知する仕組みは？
- 重複記事を除外するにはCodeノードでどう処理する？

## References
- 一般知識ベースで作成（2026-03-13）
- [[workflow-approaches]] — アプローチ全体比較
