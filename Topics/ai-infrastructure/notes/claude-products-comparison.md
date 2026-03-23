---
title: "Claudeサービス比較——Desktop・Chrome・Slack・Code・Code for Web・Cowork"
created: "2026-03-23"
topic: "ai-infrastructure"
tags:
  - topic/ai-infrastructure
---

# Claudeサービス比較——Desktop・Chrome・Slack・Code・Code for Web・Cowork

## Summary

- Anthropicは「チャット」「コーディング」「自律エージェント」の3軸で製品ラインを展開している。
- 各サービスは**単独でも使えるが、組み合わせることで真価を発揮する**設計になっている（例：Claude Code + Chrome + Slack）。
- 選ぶ基準は「誰が使うか」「どこで作業するか」「どれだけ自律性を求めるか」の3点。

---

## Notes

### 一覧マップ

```
自律性（低）─────────────────────────────────────（高）

Claude Desktop ──→ Claude for Slack ──→ Claude Code for Web ──→ Claude Code CLI ──→ Claude Cowork
     │                   │                       │                      │                  │
  デスクトップ          Slack内で               ブラウザで             ターミナルで         PC上で
  で会話・作業          チームと共有            コード操作             本格開発             ファイル操作

                                    Claude for Chrome（横断ツール）
                                    ↑ Code・Coworkと連携してブラウザを操作
```

---

### 各サービス詳細

#### 1. Claude Desktop
**→ 「ほぼWeb版と同じ。でも、ネイティブアプリが欲しい人向け」**

| 項目 | 内容 |
|---|---|
| 形態 | macOS / Windows デスクトップアプリ |
| 対象 | 全ユーザー（エンジニアでなくてもOK） |
| できること | 会話・文書作成・分析・画像生成・MCP連携 |
| できないこと | ブラウザ操作・ファイルへの自律的な書き込み |

**使うべき時：**
- Web版claude.aiのタブをずっと開いておくのが面倒なとき
- OSのショートカットでサッとClaudeを呼び出したいとき
- MCPサーバーをローカルで動かして外部ツール連携したいとき

---

#### 2. Claude for Chrome
**→ 「ブラウザ上でClaudeに手足を生やす拡張機能」**

| 項目 | 内容 |
|---|---|
| 形態 | Chrome拡張機能（ベータ） |
| 対象 | 開発者 / Webを使った作業が多いユーザー |
| できること | Webページの読み取り・フォーム入力・クリック・DOM解析・コンソールエラー検出 |
| 重要な特徴 | **既存のログイン済みセッションをそのまま使える**（他ツールは隔離ブラウザが必要） |

**使うべき時：**
- Claude Codeで書いたコードをブラウザで即テストしたいとき
- ログインが必要なWebサービスの操作を自動化したいとき
- スクレイピングや繰り返しフォーム入力を任せたいとき

> Claude Code・Coworkと組み合わせることで「コード書く→ブラウザでテスト→結果をフィードバック」が1ループで回る。

---

#### 3. Claude for Slack
**→ 「チームのSlackチャンネルにいるClaude。会話の文脈ごと渡せる」**

| 項目 | 内容 |
|---|---|
| 形態 | Slackアプリ（有料プランのみ） |
| 対象 | チームでSlackを使っているすべての人 |
| できること | @Claudeでメンション→会話・要約・翻訳・コード修正 |
| Claude Code連携 | コーディングタスクを検知し、リポジトリ選択→PR作成まで自動実行 |

**使うべき時：**
- バグ報告スレッドで「このバグ直して」→PRまで完結させたいとき
- 会議のメモをその場で要約・議事録化したいとき
- 複数人が関わる意思決定をClaudeに参加させたいとき（文脈が共有される）
- 非エンジニアとエンジニアが混在するチームでコーディングタスクを依頼するとき

---

#### 4. Claude Code（CLI）
**→ 「開発者のためのターミナル常駐型AIエージェント。最も高機能」**

| 項目 | 内容 |
|---|---|
| 形態 | CLIツール（ターミナルで `claude` コマンド） |
| 対象 | ソフトウェアエンジニア |
| できること | コード生成・編集・テスト実行・PR作成・ファイル操作・Bash実行 |
| 連携 | Chrome拡張・Slack・GitHub・MCP |
| 自律性 | 高い（複数ファイルの横断編集・コマンド実行を自分で判断） |

**使うべき時：**
- 本格的なコード開発・リファクタリング・バグ修正
- 既存のターミナルワークフローを変えたくないとき
- 複雑なマルチファイル変更を一気にやりたいとき
- CI/CDやテストの実行も含めて任せたいとき

> 「Claude Codeを使うために、あえてブラウザを使わない」というエンジニアが多い。

---

#### 5. Claude Code for Web
**→ 「ターミナル不要のClaude Code。ブラウザとモバイルから使える」**

| 項目 | 内容 |
|---|---|
| 形態 | Webアプリ（claude.aiの「Code」タブ） + iOSアプリ |
| 対象 | エンジニアだが、ターミナルをセットアップしたくない人 / モバイルで作業したい人 |
| できること | Claude Code CLIとほぼ同等。GitHubリポジトリ連携・エージェント管理 |
| 追加機能 | 複数エージェントを並列起動・管理できるUI |
| 料金 | Pro（$20/月）・Max（$100/月）・$200/月プランで利用可 |

**使うべき時：**
- 環境構築なしにすぐClaude Codeを試したいとき
- 出先でiPhoneからコードを確認・修正したいとき
- CLIより視覚的にエージェントを管理したいとき

---

#### 6. Claude Cowork
**→ 「ナレッジワーカーのためのClaude Code。PCで自律的に仕事をこなす」**

| 項目 | 内容 |
|---|---|
| 形態 | デスクトップアプリ（内部でローカルVM上で動作） |
| 対象 | エンジニア以外のナレッジワーカー（営業・人事・財務・法務など） |
| できること | ローカルファイルの読み書き・外部サービス連携・スケジュール実行 |
| コネクター | Gmail・Google Drive・Notion・Slack・Microsoft 365（Outlook・SharePoint・OneDrive）など38種以上 |
| プラグイン | 職種別にスキル・コネクター・サブエージェントをセットにしたプリセット |
| 安全設計 | 隔離VM上で動作。アクセス許可を明示的に設定する必要あり |

**使うべき時：**
- 「毎週月曜にSlackのメッセージをまとめてNotionにレポートを書く」などの定型作業を自動化したいとき
- Outlookのメール→スプレッドシート更新→報告書作成、のような複数ツールをまたぐワークフロー
- エンジニアでなくてもClaude Codeレベルの自律タスクを使いたいとき

---

### 選び方チートシート

| 自分の状況 | 使うべきサービス |
|---|---|
| とりあえずClaudeと会話したい | **Claude Desktop** or claude.ai Web |
| Slackでチームと使いたい | **Claude for Slack** |
| コードを書いている（ターミナルOK） | **Claude Code CLI** |
| コードを書いている（ブラウザ派） | **Claude Code for Web** |
| Webサービスを操作させたい | **Claude for Chrome**（+ Code or Coworkと組み合わせ） |
| エンジニア以外。ファイルや業務ツールを自動化したい | **Claude Cowork** |

### 組み合わせ例

```
開発ワークフロー:
  Claude Code CLI  +  Claude for Chrome  +  Claude for Slack
      ↓                     ↓                      ↓
  コード書く          ブラウザでテスト         PRをSlackで通知・共有

ナレッジワークフロー:
  Claude Cowork  +  Claude for Chrome  +  Claude Desktop
      ↓                     ↓                    ↓
  ファイル自律処理     Webから情報取得         確認・会話
```

---

## Questions

- Claude CoworkとClaude Desktopは将来的に統合されるのか？
- Claude for ChromeはFirefox / Edgeには対応するのか？
- Claude Code CLIとCoworkのタスク実行能力はどこが境界線か？

---

## References

- [Anthropic：Claude for Chrome（公式）](https://claude.com/claude-for-chrome)
- [Anthropic：Cowork（公式）](https://claude.com/product/cowork)
- [Anthropic：Claude for Slack（公式）](https://claude.com/claude-for-slack)
- [TechCrunch：Anthropic brings Claude Code to the web（2025/10）](https://techcrunch.com/2025/10/20/anthropic-brings-claude-code-to-the-web/)
- [TechCrunch：Claude Code is coming to Slack（2025/12）](https://techcrunch.com/2025/12/08/claude-code-is-coming-to-slack-and-thats-a-bigger-deal-than-it-sounds/)
- [Claude Code Docs：Use Claude Code with Chrome](https://code.claude.com/docs/en/chrome)
- [DataCamp：Claude Cowork Tutorial](https://www.datacamp.com/tutorial/claude-cowork-tutorial)
- [Claude Help Center：Get started with Cowork](https://support.claude.com/en/articles/13345190-get-started-with-cowork)
