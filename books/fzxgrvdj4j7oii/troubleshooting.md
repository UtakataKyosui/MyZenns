---
title: "困ったときの解決法"
---

# 困ったときの解決法

Claude Codeとの共同開発では様々な問題に直面することがあります。ここでは、よくある問題とその解決策、そして効率的なトラブルシューティングの考え方を解説します。

## Claude Codeの動作に関する問題

### インストールがうまくいかない

**症状**: Claude Codeのインストールで失敗する、正常に動作しない

**解決法**:
```bash
# インストール状況の診断
/doctor

# 権限の問題の場合
sudo npm install -g @anthropic-ai/claude-code

# キャッシュのクリア
npm cache clean --force

# 完全に再インストール
npm uninstall -g @anthropic-ai/claude-code
npm install -g @anthropic-ai/claude-code
```

**注意点**: `/doctor` コマンドを実行して、Claude Codeのインストール状態を確認します。npmパーミッションの問題やその他の問題を診断してくれます。

### セッションが途中で終了する/エラーが発生する

#### コンテキストオーバーロード

**症状**: 会話が長引くと、Claude Codeがエラーを出したり、同じアプローチを繰り返したりする

**解決法**:
```bash
# 会話履歴を完全にリセット
/clear

# コンテキストを圧縮（履歴は保持）
/compact

# セッションを復元（/clear後に使用）
claude --resume
```

**使い分け**:
- `/clear`: 完全なリセットが必要な場合
- `/compact`: 履歴を保持しつつコンテキストを圧縮したい場合

#### レート制限

**症状**: 「Rate limit exceeded」などのエラーが発生

**解決法**:
```bash
# 現在のトークン使用量を確認
/cost

# モデルをSonnetに切り替えてレート制限を緩和
/model sonnet

# 利用プランの確認（Maxプランの検討）
```

**対策**:
- 無駄な試行錯誤を避ける
- プロンプトを効率化してトークン消費を抑える
- 必要に応じて上位プランを検討

#### Claudeが同じ間違いを繰り返す/「精神論」を言う

**症状**: 同じエラーを何度も繰り返す、具体的な解決策ではなく抽象的なアドバイスを提供する

**根本原因**: ドキュメント戦略の不備

**解決法**:
1. **重要な情報をCLAUDE.mdに直接記載**
   ```markdown
   # CLAUDE.md に追加する例
   
   ## PRコメント返信の正しい方法
   ✅ 正解: gh api repos/{owner}/{repo}/pulls/{pr_number}/comments --method POST -f in_reply_to={comment_id}
   ❌ 間違い: gh pr comment {pr_number} --body="..." (PR全体へのコメントになる)
   ❌ 間違い: gh api repos/{owner}/{repo}/pulls/comments/{comment_id} --method POST (上書きしてしまう)
   ```

2. **具体的なコマンド例を示す**
   ```markdown
   ## データベースマイグレーション手順
   1. npx prisma db push
   2. npx prisma generate
   3. npm run dev で動作確認
   
   ## やってはいけないコマンド
   ❌ npx prisma migrate reset (本番データが消える)
   ```

3. **対話による継続的改善**
   - エラーが起きたら原因を分析してCLAUDE.mdに追記
   - 成功パターンも記録して再現性を高める

## GitHub連携に関する問題

### PRの行コメントを見落とす

**症状**: PRレビューで特定行のコメントに気づかない、対応漏れが発生

**原因**: `gh pr view --comments` ではPR全体のコメントしか取得できず、コードの特定行に対するコメントが取得されない

**解決法**:
```bash
# PR全体のコメントを取得
gh pr view {pr_number} --comments

# コードの特定行に対するコメントも取得（重要！）
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments
```

**CLAUDE.mdに記録すべき内容**:
```markdown
## PRレビュー対応の必須手順
1. gh pr view --comments でPR全体コメント取得
2. gh api repos/{owner}/{repo}/pulls/{pr_number}/comments で行コメント取得
3. 両方のコメントを確認して漏れなく対応
```

### PRコメントの返信がうまくいかない

#### コメントを上書きしてしまう

**症状**: 既存のコメントが意図せず変更されている

**原因**: 間違ったAPIエンドポイントの使用

**間違ったコマンド**:
```bash
# ❌ これは既存コメントを上書きしてしまう
gh api repos/{owner}/{repo}/pulls/comments/{comment_id} --method POST
```

#### PR本体にコメントしてしまう

**症状**: 特定のコードコメントへの返信ではなく、PR全体へのコメントになる

**間違ったコマンド**:
```bash
# ❌ これはPR全体へのコメントになる
gh pr comment {pr_number} --body="..."
```

#### 正しい解決法

**正解のコマンド**:
```bash
# ✅ 特定のコメントに直接返信
gh api repos/{owner}/{repo}/pulls/{pr_number}/comments \
  --method POST \
  -f body="修正完了しました。Fixed in commit [abc123](link)" \
  -f in_reply_to={comment_id}
```

**CLAUDE.mdに記録すべき内容**:
```markdown
## PRコメント返信の正しい方法
✅ gh api repos/{owner}/{repo}/pulls/{pr_number}/comments --method POST -f in_reply_to={comment_id}

## やってはいけないコマンド  
❌ gh api repos/{owner}/{repo}/pulls/comments/{comment_id} --method POST (上書き)
❌ gh pr comment {pr_number} --body="..." (PR全体コメント)

## 返信ルール
- 全てのコメント（ニットピック含む）に返信
- 「Fixed」「Won't implement」「Acknowledged」などステータス明記
- 修正コミットIDを Fixed in commit [abc123](link) 形式で含める
```

## 並列開発における問題

### Worktree作成時の環境構築

**症状**: 新しいWorktreeを作成するたびに環境構築をやり直す必要がある

**解決法**:

1. **シェルスクリプトによる自動化**
   ```bash
   # setup.sh
   #!/bin/bash
   echo "環境構築を開始します..."
   npm install
   cp .env.example .env.local
   npm run db:generate
   npm run db:migrate
   echo "✅ 環境構築完了！"
   ```

2. **PhantomのpostCreateフック活用**
   ```json
   {
     "postCreate": [
       "npm install",
       "cp .env.example .env.local",
       "npm run db:generate"
     ]
   }
   ```

3. **環境構築のシンプル化**
   - 複雑な設定は避ける
   - Dockerコンテナの活用
   - 必要最小限の依存関係

### 「あれ、このウィンドウはどのタスクだっけ？」

**症状**: 複数のエディタウィンドウでどのタスクを実行しているか分からなくなる

**解決法**: VSCode/Cursorの拡張機能「Peacock」を使用

```json
// .vscode/settings.json
{
  "peacock.favoriteColors": [
    {
      "name": "機能開発",
      "value": "#007acc"
    },
    {
      "name": "バグ修正", 
      "value": "#f14c4c"
    },
    {
      "name": "リファクタリング",
      "value": "#38b2ac"
    }
  ]
}
```

**使い方**:
1. Ctrl+Shift+P → "Peacock: Change to a Favorite Color"
2. タスクの種類に応じて色を選択
3. ウィンドウ枠の色で視覚的に区別

### コンフリクトの発生

**症状**: 並列開発によりマージ時にコンフリクトが発生

**予防策**:

1. **タスクを小さく分割する**
   ```markdown
   ## 同時実行OK
   ✅ 新しいページ追加 + 既存ページバグ修正
   ✅ フロントエンド機能 + バックエンドAPI
   ✅ テストコード追加 + ドキュメント更新
   
   ## 同時実行NG  
   ❌ 同じファイルを変更する機能
   ❌ 同じテーブルスキーマを変更する機能
   ❌ 同じコンポーネントを修正する機能
   ```

2. **こまめなmainの取り込み**
   ```bash
   # 1つのタスクが完了してmainにマージされたら
   # 他の作業中ブランチに即座に取り込み
   
   git fetch origin
   git rebase origin/main
   # または
   git merge origin/main
   ```

3. **小さな変更でのコンフリクト解消**
   - 大きなコンフリクトになる前に解消
   - 定期的なマージで衝突範囲を限定

### tmuxの操作

**症状**: tmuxの基本操作が分からない、セッション管理が混乱する

**基本操作**:
```bash
# セッション作成
tmux new-session -s development

# セッション一覧
tmux list-sessions

# セッションにアタッチ
tmux attach-session -t development

# デタッチ（セッションを維持したまま離脱）
Ctrl+b d
```

**よく使うコマンド**:
```bash
# プレフィックスキー: Ctrl+b

# ペイン操作
Ctrl+b %    # 垂直分割
Ctrl+b "    # 水平分割  
Ctrl+b o    # ペイン移動
Ctrl+b x    # ペイン削除

# ウィンドウ操作
Ctrl+b c    # 新しいウィンドウ
Ctrl+b n    # 次のウィンドウ
Ctrl+b p    # 前のウィンドウ

# セッション操作
Ctrl+b s    # セッション一覧
Ctrl+b $    # セッション名変更
```

**推奨設定**:
```bash
# ~/.tmux.conf
# マウス操作を有効化
set -g mouse on

# ペイン番号を表示
bind-key q display-panes

# ステータスバーの設定
set -g status-bg blue
set -g status-fg white
```

## 品質に関する問題

### リンター/フォーマッター/ビルドエラー

**症状**: AIが生成したコードにTypeScriptエラー、リンター警告、フォーマッターの不一致などが多発

**解決法**:

1. **段階的なチェック**
   ```bash
   # 1. TypeScript型チェック
   npx tsc --noEmit
   
   # 2. リンターチェック  
   npx @biomejs/biome check
   
   # 3. フォーマッター
   npx @biomejs/biome format --write .
   
   # 4. Next.jsビルド
   npx next build
   ```

2. **Claude Codeへの指示**
   ```markdown
   以下のコマンドを順番に実行し、全てエラーなしになるまで修正を繰り返してください：
   
   1. npx tsc --noEmit
   2. npx @biomejs/biome check  
   3. npx next build
   
   エラーが出た場合は、エラー内容を分析して修正方針を立ててから修正してください。
   ```

3. **CLAUDE.mdに品質基準を明記**
   ```markdown
   ## コード品質基準
   - TypeScript strict mode準拠
   - ESLint/Biome設定に準拠
   - Prettierフォーマット適用
   - Next.jsビルドエラーゼロ
   - テストカバレッジ80%以上
   ```

### E2Eテストが全滅

**症状**: E2Eテストの結果が思わしくない、大部分のテストが失敗する

**解決法**:

1. **段階的なデバッグ**
   ```markdown
   E2Eテストが失敗しています。以下の手順で対応してください：
   
   1. まず失敗したテストのエラー内容を詳細に出力
   2. 失敗原因を分析して修正方針を立案
   3. 修正方針について説明してください
   4. 承認後、修正を実施
   5. 再度テスト実行
   ```

2. **環境の確認**
   ```bash
   # 開発サーバーが起動しているか確認
   npm run dev
   
   # データベースの状態確認
   npm run db:studio
   
   # テスト用データの準備
   npm run db:seed
   ```

3. **テストの分割実行**
   ```bash
   # 特定のテストファイルのみ実行
   npx playwright test auth.spec.ts
   
   # ヘッドレスモードを無効にしてデバッグ
   npx playwright test --headed
   
   # スクリーンショット付きで実行
   npx playwright test --screenshot=only-on-failure
   ```

## まとめ

トラブルシューティングで最も重要なのは、**問題を再現可能な形で記録し、解決策をCLAUDE.mdに蓄積していく**ことです。

### 効果的なトラブルシューティングの進め方

1. **問題の明確化**
   - 症状を具体的に記録
   - 再現手順を明確にする
   - エラーメッセージを正確にコピー

2. **段階的な解決**
   - 一度に全てを解決しようとしない
   - 小さな問題から順番に解決
   - 解決策を検証してから次に進む

3. **知見の蓄積**
   - 解決した問題をCLAUDE.mdに記録
   - 同じ問題の再発を防ぐ
   - チーム内での知見共有

4. **予防策の実装**
   - 問題が起きにくい環境構築
   - 自動化による人的ミスの削減
   - 定期的な環境メンテナンス

困ったときこそ、Claude Codeと協力して解決策を見つけ、そのプロセス自体を資産として蓄積していきましょう。次の章では、本書の総まとめとして、これからの開発スタイルについて考えていきます。