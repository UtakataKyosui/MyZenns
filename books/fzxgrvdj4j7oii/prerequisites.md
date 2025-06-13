---
title: "まず揃えとこ！Claude Code以外で必要なツールや拡張機能"
---

# まず揃えとこ！Claude Code以外で必要なツールや拡張機能

Claude Codeを効果的に使用するためには、いくつかのツールを導入し、適切な環境を構築することが不可欠です。これ知らんと「あれ？動かへんやん」ってなることもあるので、最初にチェックしておきましょう。

## Claude Code本体のインストールと基本設定

まず最初に、Claude Code自体をインストールしましょう。

```bash
# グローバルインストール（公式推奨）
npm install -g @anthropic-ai/claude-code

# バージョン確認
claude -v

# インストール状況の確認
/doctor
```

インストール後、何か問題があれば `/doctor` コマンドでインストール状況を確認できます。

## Claudeのためのドキュメント整備

### 1. CLAUDE.md ファイル
**Claude Codeの「ルール帳」** です。これがあるかないかで開発効率が全然違います。

- リポジトリ直下、特定フォルダ、またはホームディレクトリ（`~/.claude/CLAUDE.md`）に配置
- プロジェクト固有のコーディング規約、Git運用ルール、頻繁に使うコマンドを記載
- `/init` コマンドで作成、`/memory` コマンドで編集
- AIの指示のブレを防ぎ、トークン使用量も最適化

### 2. wiki-manual.md
**GitHub Wiki自動生成の設計書。** このファイルの品質がWikiの品質を決める。

- Claude CodeのWiki生成ルール・アウトライン指示
- GitHub WikiをGit submoduleとしてクローン
- ドキュメント自動化の要となるファイル

## 必須ツール

### 1. GitHub CLI（`gh`）
**これは絶対入れとこ！** PRレビューコメントの自動取得と返信、PR作成が超楽になります。

PRレビュー対応の自動化には、GitHub CLI (`gh`) が必須です。なぜなら：

- `gh pr view --comments` でPR全体のコメント取得
- `gh api repos/{owner}/{repo}/pulls/{pr_number}/comments` でコードの特定行に対する行コメント（見落としやすいコメント）を取得
- Claude Codeの `/pr_comments` や `/review` スラッシュコマンドも `gh` コマンドに依存
- レビューコメントのコピペ地獄から解放されます

**重要**: PR全体のコメントと行コメントの両方を取得することで、Claude Codeに全てのレビューコメントを渡すことができ、返信時には `in_reply_to` パラメータを使用することで、既存のコメントを上書きしたり、PR本体に誤ってコメントしてしまうことを防げます。

### 2. repomix ツール
**プロジェクト全体を一発でClaude Codeに理解してもらえる** 神ツール。

- XML形式でコードベースを整理
- 不要な情報（コメントや空行）を除去してトークン削減
- 大規模プロジェクトでのコンテキスト管理に必須

## Git機能の活用

### Git Worktree
**複数のタスクを同時進行したいなら必須。** まるで優秀なアシスタントが3人いるみたい！

Git Worktreeは、1つのGitリポジトリから複数のブランチをそれぞれ異なる物理フォルダとして作成するGitの標準機能です。

#### 基本的な使い方
```bash
# 新しいWorktreeを作成
git worktree add ../my-project-feature -b feature-branch develop

# Worktreeの一覧表示
git worktree list

# Worktreeの削除
git worktree remove ../my-project-feature
```

#### メリット
- 1つのリポジトリから複数ブランチを物理的に分離
- 機能開発、バグ修正、リファクタリングを独立環境で並列実行
- Claude Codeの複数インスタンスを同時稼働可能
- ブランチ切り替え時の「あ、変更内容が混ざった」を防げる

## 並列開発環境構築ツール

複数のタスクを並行してClaude Codeに実行させる「超並列開発」を実現するために、以下のツールを導入します。

### 1. Git Worktree Manager（VSCode/Cursor拡張）
Git Worktreeの操作をGUIで簡単に。コマンドライン苦手な人の救世主。

- WorktreeのGUI作成・削除・状態確認
- 並列開発環境の導入障壁を大幅に下げる
- VSCode/Cursorから直感的に操作可能

### 2. tmux
**ターミナル画面を分割して複数のClaude Codeを管理。** 

1つのターミナル画面の中で複数の仮想ターミナル（セッション、ウィンドウ、ペイン）を管理し、複数のClaude Codeセッションを並行稼働させることが可能になります。

#### 基本的な使い方
```bash
# 新しいセッションを作成
tmux new-session -d -s development

# 水平分割
tmux split-window -h

# 各ペインでClaude Codeを起動
tmux send-keys -t 0 'cd ../myapp-feature-auth && claude-code' Enter
tmux send-keys -t 1 'cd . && claude-code' Enter
```

#### 活用例
- ペイン分割で個別のClaude Codeインスタンス起動
- 各Claude Codeからの報告をメインペインに集約する「報連相システム」構築
- 長時間のタスク実行中でも他の作業を継続

### 3. ccmanager
**Git Worktreeで並列稼働するClaude Codeセッションの一括管理ツール。**

複数のWorktreeディレクトリの管理の煩雑さや、`tmux`ワークフローとの競合、視覚情報の多さによる認知負荷といった問題を解決するために開発されたCLIツールです。

#### 主な機能
- 各WorktreeのClaude Codeセッションのステータス（Idle, Waiting, Busy）を一目で確認
- Git Worktreeの作成、マージ、削除をUIから直接実行
- 並列開発環境の管理を劇的に簡素化

### 4. Phantom
`ccmanager`と同様にGit worktreeの管理を効率化するCLIツールです。

#### 特徴
- `docker exec`のようにワークツリーの中でコマンドを実行する機能
- `tmux`や`fzf`との統合機能
- `.gitignore`されたファイルをコピーする機能

#### 使用例
```bash
# 新しいWorktreeを作成（.envファイルもコピー）
phantom create feature-auth --copy-file .env

# Worktree内でコマンド実行
phantom exec feature-auth npm install
```

## 高度な自動化ツール

### 5. Claude Desktop + MCPサーバー
**AI自律開発の最終兵器。** Issue作成からデプロイまで全自動。

Claude (Pro Plan) を利用している場合、Claude DesktopとModel Context Protocol (MCP) サーバーを連携させることで、AIエージェントによるGitHubのIssue作成から実装、PR作成、マージ、デプロイ、ドキュメント作成まで、自律的な開発フローを実現できます。

#### 主要MCPサーバー

**`github` MCPサーバー**
- GitHubリポジトリと連携
- Issue作成、Commit、Push、PR作成、PRレビューなどを可能にする
- GitHub APIを通じた包括的な操作が可能

**`claude_code` MCPサーバー**
- Claude DesktopからClaude Code（WSL内のインスタンスなど）を制御
- コードベースの理解、ファイル操作、テスト実行を実現
- フォーマッターやリンターのチェック、ビルド実行といった開発のメイン作業をPro Planの利用料金内で実行
- `npx tsc --noEmit`、`npx @biomejs/biome check`、`npx next build`などのコマンド実行結果を基準としてAIに品質を保たせることが可能

**`obsidian-mcp-tools` MCPサーバー**
- MarkdownエディタのObsidianと連携
- アプリケーションのドキュメント管理を支援
- リポジトリから開発者向けやエンドユーザー向けのドキュメントを生成・更新

#### メリット
- コード品質をコマンド実行結果という明確な基準で担保
- AIによる自動化の範囲を大幅に広げ、開発全体の労力を削減
- エラー内容をフィードバックすることで、AI側で品質を向上させる環境を構築

### 6. Obsidian
**ドキュメント管理の相棒。** 人間が怠りがちなドキュメント更新をAIに任せる。

ドキュメント管理にはObsidianが推奨されます。`obsidian-mcp-tools` MCPサーバーを介してClaude Codeと連携することで、以下が可能になります：

- 開発者向け・エンドユーザー向けドキュメント自動生成・更新
- リポジトリのコード変更に連動したドキュメント更新
- 常に最新かつ高品質なドキュメント維持
- ナレッジベースとしてのプロジェクト知見の蓄積

## 補完ツール

### 7. 通常のClaude（Normal Claude）
**Claude Codeの前に設計相談、行き詰まった時の戦略相談。**

Claude Codeは複雑な設計レベルの議論には向かない場合があるため、効果的な役割分担が重要です：

#### 効果的な役割分担
- **通常のClaude**: 設計相談、アーキテクチャ検討、問題解決戦略の策定
- **Claude Code**: 通常のClaudeから得た「明確な指示」による一発実装

#### 知見の活用
- Claude Codeで問題にぶつかった際は、そのログを通常のClaudeに相談
- より効率的な解決アプローチを提案してもらう
- 無駄な試行錯誤を削減し、トークン消費を抑制

## VSCode/Cursor拡張機能

### 8. Peacock
**並列作業時の混乱防止。** ウィンドウ枠に色付けでタスク別に視覚的区別。

複数のVSCode/Cursorウィンドウで並列作業を行う際に、「あれ、このウィンドウはどのタスクだっけ？」という混乱を防ぎます：

- 複数VSCode/Cursorウィンドウの色分け管理
- タスクの種類ごとに色を割り当て（例：機能開発=青、バグ修正=赤）
- 認知負荷軽減、作業効率向上
- 並列開発環境での必須アイテム

## 環境構築のシンプル化と自動化

Git Worktreeで新しいフォルダを作成するたびに環境構築が必要になるため、普段から複雑な環境構築は避け、できるだけシンプルな構成を心がけるのがおすすめです。

### シェルスクリプトの活用
```bash
# setup.sh
#!/bin/bash
npm install
cp .env.example .env.local
npm run db:generate
echo "セットアップ完了！"
```

`bash setup.sh` のように一発で環境構築が終わるようなシェルスクリプトを用意しておくと、自分もAIも楽になります。

### Phantomのセットアップコマンド
`Phantom`ツールでは、`phantom.config.json`にセットアップコマンドを定義しておくことで、Worktree作成時に自動で依存関係のインストールなどを実行させることができます。

## T3Stack（Drizzle + BetterAuth）環境でのツール活用のポイント

### 開発効率を最大化する組み合わせ
1. **CLAUDE.md** でT3Stack固有の設定を記録
2. **repomix** でプロジェクト全体を効率的に共有
3. **Git Worktree** で機能開発・バグ修正・テストを並列実行
4. **GitHub CLI** でPR作成からマージまでの一連の流れを自動化
5. **tmux** で複数のClaude Codeセッションを効率的に管理

### 実際の開発フローでの活用例

#### Phantomを使った超並列開発フロー

```bash
# 1. 複数のWorktreeを効率的に作成
phantom create feature-awesome      # 新機能開発用
phantom create bugfix-login        # バグ修正用
phantom create refactor-api         # リファクタリング用

# 2. 各Worktreeで自動セットアップ (phantom.config.json設定済みの場合)
# 自動で以下が実行される:
# - .env ファイルのコピー
# - pnpm install
# - 初期環境構築

# 3. tmuxで複数Claude Codeインスタンスを起動
tmux new-session -d -s development

# 4. 各ペインでClaude Codeを起動
phantom open feature-awesome     # VSCodeで開く
phantom open bugfix-login       # 別のウィンドウで開く

# 5. repomixでプロジェクト構造をClaude Codeに共有
repomix --output project-structure.xml --style xml --compress
```

#### Phantomの設定例（phantom.config.json）

```json
{
  "postCreate": {
    "copyFiles": [".env", ".env.local", "tsconfig.json"],
    "commands": [
      "pnpm install",
      "pnpm db:generate",
      "echo '✅ Worktree setup completed!'"
    ]
  }
}
```

#### 並列開発の実践フロー

```bash
# 3-5個のClaude Codeインスタンスを同時稼働
# 各インスタンスが異なるWorktreeで作業

# タスク例:
# Worktree1: 新機能開発
# Worktree2: バグ修正
# Worktree3: リファクタリング
# Worktree4: ドキュメント更新
# Worktree5: テスト追加

# 完了したタスクから順次PRを作成
phantom exec feature-awesome "gh pr create --draft"
phantom exec bugfix-login "gh pr create"
```

### 推奨環境
開発環境はWindows 11 WSL2 (Ubuntu 22.04) 内でのアプリ開発が推奨されています。これにより、LinuxベースのツールとWindowsの利便性を両立できます。

## まとめ

これらのツールを揃えることで、Claude Codeの真の力を引き出せます。特に重要なのは：

✅ **基本ツール**: GitHub CLI、repomix、CLAUDE.md
✅ **並列開発**: Git Worktree、tmux、ccmanager/Phantom
✅ **高度な自動化**: Claude Desktop + MCPサーバー
✅ **開発支援**: Obsidian、Peacock、通常のClaude

最初の設定は少し手間かもしれませんが、一度環境を構築すれば開発効率が劇的に向上し、**複数のタスクをAIに並行して実行させる「超並列開発」**が実現できます。

次の章では、これらのツールを使ってどうドキュメントを整備していくかを、T3Stack実例で詳しく見ていきましょう。