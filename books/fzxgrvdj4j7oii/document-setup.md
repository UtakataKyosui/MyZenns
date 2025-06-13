---
title: "Claude Codeのためのドキュメント整備"
---

# Claude Codeのためのドキュメント整備

Claude Codeを使い倒すために、まず最初にやっとかなアカンのが「ドキュメント整備」です。これをサボると後々めっちゃ後悔することになるので、最初にちゃんとやっときましょう。

## なんでドキュメント整備が必要なん？

Claude Codeは賢いですが、あなたのプロジェクトの「お作法」までは分かりません。
- 「このプロジェクトではどんなコーディング規約使ってるん？」
- 「テストはどうやって実行するん？」
- 「デプロイする時はどんなコマンド叩くん？」

こういうことを毎回説明するの、めっちゃめんどくさいですよね。だからこそ、最初に「ルール帳」を作っておくんです。

## T3Stack（Drizzle + BetterAuth）でのドキュメント整備事例

今回の改良版T3Stack構成での実際の事例を見ながら、どんな感じでドキュメントを整備していくかを紹介します。

### 1. CLAUDE.md の作成

#### 基本的な配置場所
```
your-project/
├── CLAUDE.md          # プロジェクト全体のルール
├── src/
│   ├── CLAUDE.md      # src配下のルール
│   └── components/
│       └── CLAUDE.md  # コンポーネント固有のルール
└── ~/.claude/
    └── CLAUDE.md      # 個人の全プロジェクト共通ルール
```

#### T3Stack用のCLAUDE.md例

```markdown
# CLAUDE.md

## プロジェクト概要
改良版T3Stack構成のWebアプリケーション
- Next.js 14 (App Router)
- tRPC v10
- Tailwind CSS
- TypeScript
- Drizzle ORM (PostgreSQL)
- BetterAuth

## 開発コマンド
```bash
# 開発サーバー起動
npm run dev

# データベース操作
npm run db:generate  # スキーマ生成
npm run db:migrate   # マイグレーション実行
npm run db:studio    # Drizzle Studio起動

# 型チェック・リント
npm run type-check
npm run lint
npm run format

# テスト
npm run test
npm run test:watch
```

## ディレクトリ構造
```
src/
├── app/              # Next.js App Router
├── components/       # 再利用可能コンポーネント
├── server/          # tRPCサーバー
│   ├── api/         # APIルーター
│   └── db/          # データベース設定
├── lib/             # ユーティリティ
└── styles/          # スタイル
```

## コーディング規約
- コンポーネント名はPascalCase
- ファイル名はkebab-case
- tRPCルーターは機能別に分割
- Drizzleスキーマは`src/server/db/schema.ts`に配置
- BetterAuth設定は`src/lib/auth.ts`に配置

## 技術的な注意点
### Drizzle ORM
- スキーマ変更時は必ず`npm run db:generate`実行
- リレーションは`relations()`で明示的に定義
- 型安全性を重視、`any`は使用禁止

### BetterAuth
- セッション管理はサーバーサイドで実装
- ミドルウェアでの認証チェック必須
- クライアントサイドでのトークン直接操作禁止

### tRPC
- プロシージャは小さく分割
- 入力検証はZodスキーマ必須
- エラーハンドリングは統一形式で

## 外部サービス
- データベース: PostgreSQL (開発環境はDocker)
- 認証プロバイダー: Google, GitHub
- デプロイ: Vercel

### 2. プロジェクト固有の設定ファイル

#### package.jsonのscripts最適化
Claude Codeが理解しやすいように、scripts名を統一しましょう。

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "type-check": "tsc --noEmit",
    "format": "prettier --write .",
    "db:generate": "drizzle-kit generate:pg",
    "db:migrate": "drizzle-kit push:pg",
    "db:studio": "drizzle-kit studio",
    "test": "jest",
    "test:watch": "jest --watch"
  }
}
```

#### 環境変数の管理
`.env.example`を用意して、Claude Codeが環境設定を理解できるようにします。

```bash
# .env.example

# Database
DATABASE_URL="postgresql://username:password@localhost:5432/myapp"

# BetterAuth
NEXTAUTH_SECRET="your-secret-here"
NEXTAUTH_URL="http://localhost:3000"

# OAuth Providers
GOOGLE_CLIENT_ID="your-google-client-id"
GOOGLE_CLIENT_SECRET="your-google-client-secret"
GITHUB_CLIENT_ID="your-github-client-id"
GITHUB_CLIENT_SECRET="your-github-client-secret"
```

### 3. 技術ドキュメントの整備

#### API仕様書の自動生成設定
tRPCの型情報から自動でAPI仕様書を生成する設定を用意します。

#### データベーススキーマドキュメント
Drizzleのスキーマから自動でER図やテーブル定義書を生成する仕組みを作ります。

## 実際にやってみよう

### Step 1: 基本のCLAUDE.mdを作る
```bash
/init
```
Claude Codeの`/init`コマンドで基本的なCLAUDE.mdを生成し、上記の例を参考にカスタマイズしてください。

### Step 2: プロジェクト情報を整理
- 使用している技術スタック
- よく使うコマンド
- ディレクトリ構造
- コーディング規約

これらを箇条書きでCLAUDE.mdに記載します。

### Step 3: 実際に使ってみて改善
Claude Codeを使いながら、「あ、これも書いとけばよかった」というものをどんどん追加していきます。

## よくある失敗例と対策

| 失敗例 | 問題点 | 対策 |
|-------|-------|------|
| **情報が多すぎる** | CLAUDE.mdに何でもかんでも書き込むと、かえって重要な情報が埋もれてしまう | 本当に必要な情報だけを厳選して記載。詳細は別ファイルに分けて参照する形にする |
| **更新を忘れる** | プロジェクトが進化してもCLAUDE.mdを更新しないと、古い情報でClaude Codeが混乱する | 定期的（週1回など）にCLAUDE.mdの見直しを行う。大きな変更があった時は即座に更新 |
| **抽象的すぎる記述** | 「いい感じに実装してください」みたいな曖昧な指示だと、Claude Codeも困ってしまう | 具体的なファイル名、関数名、期待する動作を明記する |
| **外部ファイル参照が多い** | 「詳細は○○を参照」が多いと、Claude Codeがセッション間で見落とす | 重要な情報はCLAUDE.mdに直接記載。外部参照は最小限に |
| **ルールの重複** | 同じルールが複数箇所に書かれていると、どれが正しいか分からなくなる | ルール帳の定期的な棚卸しで重複を排除 |

## まとめ

ドキュメント整備は面倒に思えるかもしれませんが、一度やっておけば後の開発がめっちゃ楽になります。特にT3Stackみたいに複数の技術が組み合わさった構成では、Claude Codeに「このプロジェクトの流儀」を理解してもらうことが重要です。

次の章では、実際にClaude Codeの基本的な使い方を見ていきましょう。