---
title: "知ってたら差がつく上級テクニック"
---

# 知ってたら差がつく上級テクニック

Claude Codeを「超並列開発」の相棒として活用することで、開発効率を劇的に向上させることができます。ここでは、複数のタスクを同時に進め、面倒な作業をAIに任せるための上級テクニックを解説します。

## Git Worktreeと複数Claude Codeセッションによる並列開発

Git Worktreeを使用すると、1つのリポジトリから複数のブランチをそれぞれ異なる物理フォルダとして作成できます。これにより、複数のClaude Codeインスタンスを並行して稼働させ、異なるタスクを同時に進めることが可能になります。

### 実践的な並列開発の進め方

#### 1. 基本的なWorktree作成と管理

```bash
# メイン機能開発用
git worktree add ../myapp-feature-auth feature/better-auth

# バグ修正用
git worktree add ../myapp-bugfix-login bugfix/login-validation

# リファクタリング用
git worktree add ../myapp-refactor-api refactor/api-structure

# Worktreeの一覧確認
git worktree list
```

#### 2. 各Worktreeでの独立したClaude Code起動

それぞれのWorktreeディレクトリで `claude` コマンドを実行し、独立したClaude Codeセッションを開始します：

```bash
# ターミナル1: 機能開発
cd ../myapp-feature-auth
claude

# ターミナル2: バグ修正
cd ../myapp-bugfix-login
claude

# ターミナル3: リファクタリング
cd ../myapp-refactor-api
claude
```

### tmuxによる効率的なセッション管理

`tmux`を使って1つのターミナル画面内に複数のペインを分割し、それぞれのペインで個別のClaude Codeインスタンスを起動することで、異なるタスクを並列で処理させることができます。

#### tmuxセッションの構築例

```bash
# 開発用セッションを作成
tmux new-session -d -s development

# ペインを水平分割（左右に分ける）
tmux split-window -h

# さらに右ペインを垂直分割（上下に分ける）
tmux split-window -v

# 各ペインでClaude Codeを起動
tmux send-keys -t 0 'cd ../myapp-feature-auth && claude' Enter
tmux send-keys -t 1 'cd ../myapp-bugfix-login && claude' Enter  
tmux send-keys -t 2 'cd ../myapp-refactor-api && claude' Enter

# セッションにアタッチ
tmux attach-session -t development
```

#### 報連相システムの構築

各Claude Codeからの報告をメインペインに集約する「報連相システム」も構築できます：

```bash
# メインの報告用ペインを作成
tmux new-window -n 'reports'

# CLAUDE.mdに以下のルールを追記
echo "# 完了通知ルール
全てのタスクが完了したら、必ず最後に以下を実行してください：
echo '✅ [$(date)] $(pwd | basename): タスク完了'" >> CLAUDE.md
```

### ccmanagerとPhantomによる高度な管理

#### ccmanagerの活用

`ccmanager`を使用することで、複数のWorktreeと Claude Codeセッションを一括管理できます：

```bash
# 各WorktreeのClaude Codeセッション状態を確認
ccmanager status

# 新しいWorktreeを作成してClaude Codeを自動起動
ccmanager create feature/new-dashboard

# 完了したタスクをマージして削除
ccmanager merge feature/better-auth
ccmanager cleanup feature/better-auth
```

#### Phantomの活用

`Phantom`では、より柔軟なWorktree管理とコマンド実行が可能です：

```bash
# 設定ファイルをコピーしてWorktree作成
phantom create feature/analytics --copy-file .env --copy-file .env.local

# Worktree内でコマンドを実行
phantom exec feature/analytics npm install
phantom exec feature/analytics npm run db:migrate

# 複数Worktreeで同じコマンドを一括実行
phantom exec-all npm run type-check
```

## PRレビュー対応の自動化

CodeRabbitのようなAIレビューサービスからの指摘やGitHubのPRコメントへの対応をClaude Codeに自動化させることで、「伝言係」からの脱却を図ります。

### 完全なPRレビュー対応ワークフロー

#### 1. PRコメントの完全取得

重要なのは、**全て**のコメントを取得することです：

```bash
# PR概要コメントを取得
gh pr view {pr_number} --comments

# コードの特定行に対するコメント（見落としやすい）を取得
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments
```

#### 2. Claude Codeへの指示

取得したコメントをClaude Codeに渡し、包括的な対応を指示します：

```markdown
以下のPRレビューコメントに対応してください：

【PR概要コメント】
[gh pr viewの結果をペースト]

【行コメント】
[gh apiの結果をペースト]

【対応ルール】
- 全てのコメント（ニットピック含む）に返信する
- 「Fixed」「Won't implement」「Acknowledged」などステータスを明記
- 修正したコミットIDを `Fixed in commit [abc123](link)` の形式で含める
- `in_reply_to={comment_id}` パラメータで特定コメントに直接返信
```

#### 3. CLAUDE.mdでのルール化

これらのPRコメント対応ルールを`CLAUDE.md`に記載し、Claude Codeが確実に守るようにします：

```markdown
# PRコメント対応ルール

## 必須手順
1. `gh pr view --comments` でPR全体コメント取得
2. `gh api repos/{owner}/{repo}/pulls/{pr_number}/comments` で行コメント取得
3. 全コメントに返信（ニットピック含む）
4. 返信時は `in_reply_to={comment_id}` 必須

## やってはいけないコマンド
❌ `gh api repos/{owner}/{repo}/pulls/comments/{comment_id} --method POST` 
   → 既存コメントを上書きしてしまう
❌ `gh pr comment {pr_number} --body="..."` 
   → PR全体へのコメントになってしまう

## 正しいコマンド
✅ `gh api repos/{owner}/{repo}/pulls/{pr_number}/comments --method POST -f body="..." -f in_reply_to={comment_id}`
```

## 自律的な開発フロー（IssueからPR/マージまで）

Claude DesktopとMCPサーバーを連携させることで、AIエージェントに開発プロセス全体を自律的に進めさせることが可能になります。

### Issue駆動開発の自動化

#### 1. Issue作成の自動化

```markdown
Claude Codeへの指示例：

開発中に思いついた以下の機能について、GitHubのIssueを作成してください：

【機能概要】
ユーザーが投稿にいいねできる機能

【要求事項】
- コードベースとコミットログから背景情報を調査
- エピックやラベルを適切に設定
- 実装に必要なドキュメント作成タスクも含める
- 関連するテストの起票も忘れずに
```

#### 2. 実装フェーズの品質保証

```markdown
【品質保証ルール】
実装完了前に以下のコマンドを必ず実行し、エラーがなくなるまで修正を繰り返してください：

1. TypeScript型チェック: `npx tsc --noEmit`
2. リンター: `npx @biomejs/biome check`
3. フォーマッター: `npx @biomejs/biome format --write`
4. ビルド確認: `npx next build`
5. テスト実行: `npm run test`

エラーが発生した場合は、エラー内容を分析して修正方針を立ててから修正してください。
```

#### 3. Draft PR作成とレビューの自動化

```markdown
作業が完了したら、以下の手順でPR作成を行ってください：

1. `good. create a draft pr` でDraft PR作成
2. 変更内容の要約とテスト結果を含めたPR説明を生成
3. パフォーマンス、エラーハンドリング、アクセシビリティの観点から自己レビュー
4. 改善点があれば追加で修正
```

## その他の効率化テクニック

### ポートの分散による同時デバッグ

Next.jsなどの開発サーバーを起動する際、Worktreeごとにポート番号を分けることで、複数の開発サーバーを同時に起動し、動作確認を行えます：

```bash
# 各Worktreeで異なるポートを使用
# feature-auth: ポート3000
cd ../myapp-feature-auth
npm run dev

# bugfix-login: ポート3001  
cd ../myapp-bugfix-login
PORT=3001 npm run dev

# refactor-api: ポート3002
cd ../myapp-refactor-api
PORT=3002 npm run dev
```

### 完了通知システム

長時間タスクの完了を確実に把握するために、CLAUDE.mdに以下のルールを追記：

```markdown
# 完了通知ルール
全てのタスクが完了したら、必ず最後に以下を実行してください：
echo -e "\a" && echo "🎉 [$(date)] $(pwd | basename): 全タスク完了！"
```

これにより、ターミナルのベル音で通知を受け取ることができます。

### コンフリクトのリスク軽減

並列開発における最大の課題はコンフリクトです。以下の戦略で最小化します：

#### タスクの分割戦略
```markdown
【コンフリクト回避のためのタスク分割】

同時実行OK:
- 新しいページの追加 + 既存ページのバグ修正
- フロントエンド機能追加 + バックエンドAPI追加  
- テストコード追加 + ドキュメント更新

同時実行NG:
- 同じファイルを変更する機能
- 同じテーブルスキーマを変更する機能
- 同じコンポーネントを修正する機能
```

#### こまめなマージ戦略
```bash
# 1つのタスクが完了してmainにマージされたら
# 他の作業中ブランチに即座に取り込み

# 作業中のブランチで
git fetch origin
git rebase origin/main
# または
git merge origin/main
```

### 指示書（ISSUE.md）の活用

Git Worktreeで複数のタスクを並列で進める際、Claude Codeへの指示を毎回チャットに打ち込むのは大変です。以下のようなタスク指示用のMarkdownファイルを用意します：

```markdown
# ISSUE.md - タスク指示書

## 概要
BetterAuth認証システムの実装

## 要件定義
1. **認証プロバイダー設定**
   - Google OAuth 2.0
   - GitHub OAuth
   - セッション管理はサーバーサイド

2. **実装ファイル**
   - `src/lib/auth.ts`: BetterAuth設定
   - `src/app/api/auth/[...auth]/route.ts`: APIルート
   - `src/components/AuthButton.tsx`: ログインボタン
   - `src/middleware.ts`: 認証チェック

3. **品質要件**
   - TypeScript strict mode準拠
   - テストカバレージ90%以上
   - セキュリティベストプラクティス準拠

## 完了条件
- [ ] 全ファイルが実装済み
- [ ] テストが全て通過
- [ ] ビルドエラーゼロ
- [ ] セキュリティチェック完了
```

Claude Codeに「`ISSUE.md`にある要件を理解して、完璧に実装してください」と指示することで、Claude Codeは指示書を忠実に実行してくれます。

**この指示書（要件定義）の質が、AI開発の成果を大きく左右します**。

## まとめ

上級テクニックの核心は、**Claude Codeを単体で使うのではなく、ツールとプロセスを組み合わせたシステムとして活用する**ことです。

### 重要なポイント
- ✅ Git Worktree + tmux + ccmanager/Phantomで超並列開発を実現
- ✅ PRレビュー対応の完全自動化で「伝言係」から脱却
- ✅ Issue→実装→PR→マージまでの自律的な開発フロー
- ✅ 指示書（ISSUE.md）による確実なタスク実行
- ✅ 品質保証の自動化でコード品質を担保

これらのテクニックを習得することで、まさに「優秀なアシスタントが複数人いるかのような開発体験」を実現できます。

次の章では、これらのテクニックをT3Stack実際のプロジェクトでどう活用するかを詳しく見ていきましょう。