---
title: "AIニュース自動収集・要約・Slack投稿ワークフロー"
created: "2026-03-13"
status: active
tags:
  - topic/ai-news-workflow
  - status/active
---

# AIニュース自動収集・要約・Slack投稿ワークフロー

各種まとめサイトのAIニュースを自動収集し、Claude APIで要約してSlackに投稿するワークフローの設計。

## Findings
- [[workflow-approaches]] — アプローチ比較・推奨スタック・ニュースソース候補
- [[n8n-step-by-step]] — n8nでの構築手順（Step by Step）

## Key Questions
- n8nとMakeどちらが使いやすいか？
- スクレイピングの法的・技術的制約は？
- 要約の品質をどう担保するか？
