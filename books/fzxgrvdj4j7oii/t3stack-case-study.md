---
title: "T3Stack実践: 具体的なケーススタディ"
---

# T3Stack実践: 具体的なケーススタディ

この章では、実際のWebアプリケーション開発プロジェクトを例に、Claude Codeをどのように活用していくかを具体的に解説します。

**注意**: 本書のタイトルには「Drizzle + BetterAuth」とありますが、本来のT3StackではPrismaとNextAuth.jsが採用されています。ここでは従来のT3Stack事例を基に解説し、DrizzleやBetterAuthについても同様の原則が適用できることを示します。

## T3 Stackの採用理由と環境構築

### なぜT3 Stackを選んだか？

T3 Stack (Next.js - TypeScript - tRPC - Prisma - Tailwind CSS - NextAuth.js) を採用する理由として、以下の点が挙げられます：

- **導入が簡単で型安全な開発が可能**
- **tRPCによるフロントエンド・バックエンド間の型共有でコスト削減**
- **Vercelでのデプロイが容易**
- **豊富なエコシステムとコミュニティサポート**

### 推奨開発環境

開発環境はWindows 11 WSL2 (Ubuntu 22.04) 内でのアプリ開発が推奨されています。これにより：

- LinuxベースのツールとWindowsの利便性を両立
- Docker環境との親和性が高い
- Claude Codeの動作が安定

### プロジェクト初期設定

```bash
# T3 Stackプロジェクトの作成
npx create-t3-app@latest my-app

# 必要なオプション選択
✅ TypeScript
✅ Tailwind CSS  
✅ tRPC
✅ Prisma
✅ NextAuth.js
✅ App Router
```

## 開発準備と初期デプロイ

### データベースコンテナの起動

T3Stackではデフォルトで`start-database.sh`があり、Claude Codeに実行を依頼してDB用Dockerコンテナを立ち上げられます：

```bash
# Claude Codeへの指示例
T3Stackプロジェクトの開発環境をセットアップしてください。

1. start-database.sh を実行してDBコンテナを起動
2. 環境変数を確認して必要に応じて設定
3. Prismaのマイグレーションを実行
4. 開発サーバーを起動して動作確認
```

### 認証機能の追加

例えばGoogleアカウント認証を追加する場合：

```markdown
【Claude Codeへの指示】

NextAuth.jsを使ってGoogleアカウント認証を設定してください。

【要件】
- Google OAuth 2.0プロバイダーの設定
- セッション管理はデフォルト設定
- ログイン/ログアウトボタンコンポーネント
- 認証状態に応じたページアクセス制御

【設定情報】
- GOOGLE_CLIENT_ID: [環境変数で管理]
- GOOGLE_CLIENT_SECRET: [環境変数で管理]
- NEXTAUTH_URL: http://localhost:3000
- NEXTAUTH_SECRET: [生成してください]

【ファイル構成】
- pages/api/auth/[...nextauth].ts
- components/AuthButton.tsx  
- middleware.ts（必要に応じて）
```

### Vercelへの初回デプロイ

GitHubリポジトリをVercelに連携し、必要な環境変数を設定：

```markdown
【Claude Codeへの指示】

Vercelへのデプロイ設定を行ってください。

【手順】
1. GitHubリポジトリの作成とプッシュ
2. Vercelプロジェクトの作成
3. 本番環境用の環境変数設定
4. ビルド時のDBマイグレーション設定
5. デプロイテストの実行

【環境変数（本番用）】
- DATABASE_URL: [Supabase等のDBサービス]
- GOOGLE_CLIENT_ID: [本番用GoogleアプリのID]
- GOOGLE_CLIENT_SECRET: [本番用Googleアプリのシークレット]
- NEXTAUTH_URL: [実際のドメイン]
- NEXTAUTH_SECRET: [本番用シークレット]
```

## 機能実装フェーズ

### Issue駆動開発の実践

#### 1. Issue作成の自動化

```markdown
【Claude Codeへの指示】

以下の新機能について、適切なGitHubのIssueを作成してください：

【機能概要】
ユーザーが投稿にいいねできる機能

【調査要求】
- 現在のコードベースとコミットログから関連する背景情報を調査
- データベーススキーマの現状を確認
- 既存の認証システムとの連携方法を検討

【Issue作成要件】
- 適切なエピックとラベルを設定
- 実装タスクの詳細な分解
- 必要なテストタスクの特定
- ドキュメント更新タスクの起票
- 受け入れ条件の明確化
```

Claude Codeは、コードベースやコミットログから背景情報を調査し、エピックやラベル、ドキュメント作成の起票まで含めて、意図通りのIssueを作成してくれます。

#### 2. 段階的な実装依頼

Issueに沿って、段階的にClaude Codeに実装を進めてもらいます：

```markdown
【フェーズ1: データベーススキーマ】
Prismaスキーマに「いいね」機能用のテーブルを追加してください。

要件:
- Likeテーブル（id, userId, postId, createdAt）
- User-Like間のリレーション
- Post-Like間のリレーション  
- 一意制約（同じユーザーが同じ投稿に複数いいねできない）

【フェーズ2: tRPC API】
いいね機能のtRPCルーターを実装してください。

要件:
- like.toggle (いいねの追加/削除)
- like.getByPostId (投稿のいいね数取得)
- like.getByUserId (ユーザーのいいね一覧)
- 適切な認証チェック
- Zodによる入力検証

【フェーズ3: フロントエンド】
いいねボタンコンポーネントを実装してください。

要件:
- リアルタイムのいいね数表示
- ボタンの状態管理（いいね済み/未済み）
- Optimistic Updates
- エラーハンドリング
```

### 品質保証の徹底

実装後にTypeScript、リンター、フォーマッター関連のエラーが多発する場合があるため、以下のコマンドを実行させ、エラーがなくなるまでAIに修正を繰り返させます：

```markdown
【品質保証チェックリスト】

実装完了前に以下を必ず実行し、全てエラーなしになるまで修正してください：

1. **TypeScript型チェック**
   ```bash
   npx tsc --noEmit
   ```

2. **ESLintチェック** 
   ```bash
   npx eslint . --ext .ts,.tsx
   ```

3. **Prettierフォーマット**
   ```bash
   npx prettier --check .
   ```

4. **Next.jsビルド**
   ```bash
   npx next build
   ```

5. **Prismaスキーマ検証**
   ```bash
   npx prisma validate
   ```

エラーが発生した場合は、エラー内容を出力して修正方針を考えてから修正してください。
```

### テストの実施と修正

PlaywrightのようなE2Eテストを実施し、問題があればエラー内容をフィードバックしながら修正を試みます：

```markdown
【テスト実行指示】

以下のテストを実行し、全て通過するまで修正してください：

1. **ユニットテスト**
   ```bash
   npm run test
   ```

2. **E2Eテスト**
   ```bash
   npm run test:e2e
   ```

テストが失敗した場合：
1. エラー内容を詳細に出力
2. 失敗原因を分析して修正方針を立案
3. 修正を実施
4. 再度テスト実行

この際、「エラー内容を出力して、修正方針を考えて」→「この方針で修正して」という段階的な指示がスムーズな改善につながります。
```

## プッシュ・マージとドキュメンテーション

### PR作成とレビューの自動化

実装が完了したら、Claude CodeにPR作成からレビューまで依頼します：

```markdown
【PR作成指示】

実装が完了しました。以下の手順でPR作成を行ってください：

1. **Draft PR作成**
   ```bash
   good. create a draft pr
   ```

2. **PR説明の生成**
   - 変更内容の要約
   - 実装した機能の詳細
   - テスト結果の報告
   - スクリーンショット（可能であれば）

3. **自己レビューの実施**
   以下の観点からコードをレビューし、問題があれば修正：
   - パフォーマンス
   - エラーハンドリング
   - アクセシビリティ
   - セキュリティ
   - コードの可読性

4. **Ready for reviewに変更**
   自己レビューが完了したらDraftを外してください。
```

Claude Codeは修正内容を汲み取ったPRを作成し、様々な観点から自己添削のようなレビューを提供してくれます。

### ドキュメント生成の自動化

最後に、Obsidianと連携したClaude Codeにドキュメントをまとめてもらいます：

```markdown
【ドキュメント生成指示】

実装完了に伴い、以下のドキュメントを更新/作成してください：

1. **README.md更新**
   - 新機能の説明追加
   - セットアップ手順の確認
   - 環境変数の一覧更新

2. **API仕様書作成**
   - tRPCルーターの仕様を整理
   - Mermaidを使ったER図の更新
   - エンドポイント一覧の生成

3. **開発者向けドキュメント**
   - いいね機能の実装詳細
   - データベーススキーマの変更履歴
   - テストケースの説明

4. **エンドユーザー向け説明**
   - 新機能の使い方
   - スクリーンショット付きの操作手順
```

READMEをベースに開発者向けやエンドユーザー向けのドキュメントを作成させ、API仕様書ではMermaidを使ってDBのER図を整理させることも可能です。

## 並列開発の実践例

### 複数機能の同時開発

Git Worktreeを活用して、複数の機能を同時に開発する実例：

```bash
# メインブランチから3つのWorktreeを作成
git worktree add ../myapp-likes feature/likes-system
git worktree add ../myapp-comments feature/comments-system  
git worktree add ../myapp-notifications feature/notification-system

# 各Worktreeで異なるポートを使用
# likes: ポート3000
cd ../myapp-likes
npm run dev

# comments: ポート3001
cd ../myapp-comments
PORT=3001 npm run dev

# notifications: ポート3002
cd ../myapp-notifications  
PORT=3002 npm run dev
```

### タスクの適切な分割

```markdown
【同時実行OK】
✅ いいね機能 + コメント機能 (異なるテーブル)
✅ 通知機能 + プロフィール機能 (独立したページ)
✅ フロントエンド改善 + バックエンドAPI追加

【同時実行NG】
❌ 同じPostテーブルを変更する機能
❌ 同じ認証システムを変更する機能
❌ 同じコンポーネントを修正する機能
```

## ケーススタディからの学び

### 自動化のメリット

- **開発時間の短縮**: Issue作成からPR・マージまでの作業が自動化
- **労力の削減**: 手作業のコピペやルーチンワークを排除
- **品質の向上**: 一貫した品質チェックプロセス
- **ドキュメントの最新性**: AIによる継続的なドキュメント管理

### プロンプトの重要性

```markdown
【効果的なプロンプトの例】

❌ 悪い例:
「いいね機能を作って」

✅ 良い例:  
「T3StackのPrisma + tRPC + NextAuth構成で、以下の仕様でいいね機能を実装してください：

【データベース】
- Likeテーブル（id, userId, postId, createdAt）
- 一意制約: (userId, postId)

【tRPC API】  
- like.toggle: いいねの追加/削除
- like.getCount: 投稿のいいね数取得
- 認証必須、Zod検証付き

【フロントエンド】
- OptimisticUpdatesでUX向上
- Tailwindでスタイリング
- エラーハンドリング実装

【品質要件】
- TypeScript strict mode
- ESLint/Prettier準拠
- E2Eテスト含む
```

### 限界と改善点

**現在の課題:**
- Claude Pro Planの利用制限
- 長いスレッドでのパフォーマンス低下
- スレッド上限による処理中断
- 切り替えコストの高さ

**改善策:**
- **Maxプランの検討**: より高い利用制限
- **プロンプティングの工夫**: より効率的な指示方法
- **タスクの細分化**: 長時間のスレッドを避ける
- **定期的なコンテキストリセット**: `/compact`や`/clear`の活用

## まとめ

T3Stack実践を通じて分かったことは、Claude Codeは**単なるコード生成ツールではなく、開発プロセス全体を変革するパートナー**だということです。

### 成功の要因

1. **適切なツール組み合わせ**: Git Worktree + tmux + Claude Code
2. **明確な指示**: 具体的で段階的なプロンプト
3. **品質保証の自動化**: 一貫したチェックプロセス
4. **ドキュメント戦略**: CLAUDE.mdとObsidianの活用

### 今後の展望

Claude Codeの進化と共に、さらに効率的な開発手法が生まれることでしょう。重要なのは、**ツールに振り回されるのではなく、自分の開発スタイルに合わせてカスタマイズし続ける**ことです。

次の章では、開発中に遭遇する様々な問題とその解決法について詳しく見ていきましょう。