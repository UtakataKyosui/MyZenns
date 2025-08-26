---
title: "Claude CodeでCommandからSubAgentを呼び出す"
emoji: "🦔"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [claudecode,claude,ai]
published: false
---

# コマンドで入力の質の担保をする

- 一回一回の入力で質の安定を保つのは困難
- プロンプトを考える思考負荷の削減
- プロンプトの再利用による再現性の実現

コマンドは、上記のような効果をもたらす。

## おすすめの運用方法

- 責務に応じてフォルダ分けする

例えば、`~/.claude/commands/`配下に
コマンドのMarkdown文書を置くと
`/[コマンド名]`になりますが、
`~/.claude/commands/frontend`配下に
コマンドのMarkdown文書を置くと
`/frontend:[コマンド名]`になります。

# SubAgentを設定をする

- 特定のタスクに特化させられる
- メインのコンテキストを圧迫しない
- 使用を許可してあること以外しない

SubAgentは、上記のような効果をもたらす。

## おすすめの運用方法

- 実装以外の細かいタスクは全てSubAgentに

# 実行例：PRレビューの修正

1. `Command`を実行する
    作成しておいた`Command`を実行する
2. `SubAgent`でコメントの内容から修正内容を抽出し、修正を行う
    `pull-request-comments-review-modify`と名付けた`SubAgent`を実行
3. `SubAgent`でテストを作成・実行
    `test-code-create-execute`と名付けた`SubAgent`を実行
4. `SubAgent`でCommitとPushを実行
    `git-commit-push`と名付けた`SubAgent`を実行
