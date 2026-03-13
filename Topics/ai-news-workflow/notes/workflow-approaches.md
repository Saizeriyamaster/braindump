---
title: "AIニュース自動収集ワークフロー：アプローチ比較"
created: "2026-03-13"
topic: ai-news-workflow
tags:
  - topic/ai-news-workflow
---

# AIニュース自動収集ワークフロー：アプローチ比較

## Summary
- n8n（ノーコード）が最速で試せる。Python+GitHub Actionsが本格運用向け
- ニュース取得はRSSが最も安定。スクレイピングはサイトごとに対応が必要
- Claude APIで要約し、Slack WebhookまたはSDKで投稿するのが基本構成

## Notes

### システム構成の全体像

```
[スケジューラー] → [ニュース取得] → [LLM要約] → [Slack投稿]
```

---

### アプローチA：n8n（ノーコード・推奨）

**フロー例：**
```
Cron trigger (毎朝8時)
  → RSS Feed node × N個（各ニュースサイト）
  → Merge node（記事一覧をまとめる）
  → Claude API node（要約プロンプトを送る）
  → Slack node（チャンネルに投稿）
```

**特徴：**
- オープンソース、Docker or n8n.cloudで動かせる
- 視覚的なフローエディタ
- Claude API・Slack・HTTP Request・RSS読み込みノードが標準装備
- セルフホストなら無料

**セットアップ手順：**
1. `docker run -it --rm -p 5678:5678 n8nio/n8n` で起動
2. RSSノードで各ニュースサイトのフィードURLを設定
3. HTTP Requestノード + JSONパースでスクレイピングも可能
4. Claude APIノードに要約プロンプトを設定
5. Slackノードに Webhook URL or Bot Token を設定

---

### アプローチB：Python + GitHub Actions（本格運用向け）

**フロー例：**
```
GitHub Actions (cron: '0 8 * * *')
  → fetch_news.py（feedparser + requests）
  → summarize.py（anthropic SDK）
  → post_slack.py（slack_sdk）
```

**主要ライブラリ：**
```python
feedparser          # RSSフィード取得
requests            # HTTPリクエスト
beautifulsoup4      # HTML解析・スクレイピング
anthropic           # Claude API
slack_sdk           # Slack投稿
```

**GitHub Actions設定例：**
```yaml
on:
  schedule:
    - cron: '0 23 * * *'  # JST 8:00 = UTC 23:00
jobs:
  news:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pip install -r requirements.txt
      - run: python main.py
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
```

**Claude API要約プロンプト例：**
```python
client = anthropic.Anthropic()
message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": f"""以下のAIニュース記事を日本語で3行に要約してください。
重要度の高いものから順に並べてください。

{articles_text}"""
    }]
)
```

---

### アプローチC：エージェント型（Strands SDK）

[[agent-strands]] で調査したAWS Strands SDKを使う方法。

```python
from strands import Agent
from strands.tools import web_fetch, slack_post

agent = Agent(
    model="claude-sonnet-4-6",
    tools=[web_fetch, slack_post]
)
agent.run("今日のAIニュースを主要サイトから収集し、要約してSlackの#ai-newsに投稿して")
```

- LLMが自律的にどのニュースが重要かを判断できる
- ただし実行コストが高く、再現性が低い
- プロトタイプ段階には向かない

---

### ニュースソース候補

| サイト | 言語 | RSSあり | URL |
|--------|------|---------|-----|
| TechCrunch AI | 英語 | ✅ | techcrunch.com/category/artificial-intelligence/feed/ |
| VentureBeat AI | 英語 | ✅ | venturebeat.com/category/ai/feed/ |
| The Verge AI | 英語 | ✅ | theverge.com/rss/ai/index.xml |
| Hacker News | 英語 | ✅ (API) | news.ycombinator.com/rss |
| AINOW | 日本語 | ✅ | ainow.ai/feed/ |
| Zenn（AIタグ） | 日本語 | ✅ | zenn.dev/topics/ai/feed |
| note（AI） | 日本語 | ✅ | note.com/hashtag/AI のRSS |

---

### 推奨構成まとめ

| 目的 | 推奨 |
|------|------|
| とりあえず動かしたい | n8n (Docker) |
| 細かくカスタマイズしたい | Python + GitHub Actions |
| エージェント的に使いたい | Strands + Claude |
| チームで使いたい | n8n.cloud or Make |

---

### 注意点
- スクレイピングはサイトのToS確認が必要。RSSが使えるサイトはRSSを優先する
- Claude APIのコストに注意（多数記事を毎日要約すると積み上がる）
- Slack Bot Tokenは `chat:write` スコープが必要

## Questions
- まとめサイトでRSSがないサイトをどうスクレイピングするか？
- 重複記事のフィルタリングをどうするか？
- Slack Blockキットを使ったリッチな投稿フォーマットは？

## References
- 一般知識・[[agent-strands]] ベースで作成（2026-03-13）
