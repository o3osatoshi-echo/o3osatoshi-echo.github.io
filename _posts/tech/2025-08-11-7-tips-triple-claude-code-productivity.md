---
layout:        post
title:         "Claude Codeの開発効率を3倍に引き上げる7つの上級者向けTips"
date:          2025-08-11 00:00:00 +0900
modified_date: 2025-08-11 00:00:00 +0900
categories:    [tech, ai]
tags:          [watching]
---

![thumbnail](/assets/2025-08-11-7-tips-triple-claude-code-productivity/thumbnail.png)

{% include watching-notice.md %}

※本ブログは、YouTube動画（[Claude Codeの開発効率を3倍に引き上げる7つの上級者向けTips](https://youtu.be/4AMjoQsOLOg?si=PEnf7R06cC0YRCCr)）の視聴備忘録である

### 開発の進め方

1. 要件定義
   - はじめに `init` を実行
   - ヌケモレなく要件定義するために、**AIにヒアリングさせる**
      - 大枠の要望を記述した上で、「要件を詰めたいので、ヒアリングしてください」と追記する
   - ヒアリング結果は、**要件定義書としてCLAUDE.mdに出力**させておく
      - CLAUDE.md
         - 守ってほしいルールなどを一般的に記述する
         - 使用する技術の**ベストプラクティス**も記載しておくと良い
         - モノレポ構成でアプリごとに配置することも可能
2. チケット分割
   - 要件定義書に基づいて必要な機能を洗い出させる
   - プロンプト例
     > 要件定義書を機能毎にチケット分割してください。/docs配下にお願いします。
     > マークダウンファイルを使用し、ファイル名に連番を振ってください。
     > 各ファイルでTodo管理も行います。- []で終わったら- [x]を入れるようにCLAUDE.mdファイルに記載しておいてください。
3. チケットベースで開発を進めさせる

#### オプション Tips

- `--dangerously-skip-permissions` ... 許可を求められない
- `--resume` ... 過去の会話を遡れる
- `--continue` ... 直近の会話から再開できる


### hooks

- **PreToolUse** ... 事前処理を指定できる
- **PostToolUse** ... 事後処理を指定できる（prettierを実行するなど）
- **Stop** ... すべての処理が完了したあとの処理を指定できる（通知など）


### GitHub との連携

1. ghコマンドをインストールする
2. リポジトリを作成させる
   > ghコマンドを使って、リポジトリを作成してください
3. issueを作成させる
   > 今のアプリで、実装すべきissueを作成してください
   - リポジトリ内でチケット管理しても良いし、GitHubのissueでチケット管理しても良い
   - Claude GitHub Actionsで、issueベースに開発を進めていくことも可能
      - `/install-github-app` で、Actionsがセットアップされる


### MCP
外部サービスをAI経由で操作できる。

1. プロジェクトルートに `.mcp.json` を作成する
2. `claude --mcp-config=.mcp.json` で接続
3. playwrightでmcpのテスト
   > 手動テストをPlaywrightMCPを起動して行ってください。講座が閲覧できてるか確認したいです。

#### Tips
- [mcp.so](https://mcp.so/) で人気なmcpを調べられる
- [MCPify](https://mcpify.ai/) でMCPサーバの作成も可能


### Agents
各専門領域に特化したエージェントを作成できる。
Nextjs専門のエージェントやバックエンド専門のエージェントなど。
**エージェント毎にコンテキストが保持**されるため、分野毎のコンテキストを汚染せずに済む。

1. `Create new agent`
2. `Generate with Claude` ※推奨設定から作成する場合
   - エージェントの内容を指示　※以下は公式推奨文
     > User the code-reviewer subagent to check my recent changes
   - `.claude/agents` 配下にエージェント用mdが生成される

#### Tips
- 各分野用に調整されたmdを配布している人もいるので探してみると良い


### カスタムスラッシュコマンド

1. `.claude/commands` ディレクトリを作成
2. `xx.md` でファイルを作成し、Claude Codeへの指示文を記載する
3. スラッシュで追加したコマンド `xx` が使用できるようになる

#### Tips
- Claude Codeにカスタムコマンドを作成させることも可能
