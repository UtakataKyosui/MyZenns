---
title: "プロジェクトの設定、これだけは押さえとこ"
---

# プロジェクトの設定、これだけは押さえとこ

Claude Codeを使う上で、プロジェクトの設定がちゃんとしてるかどうかで開発効率が大きく変わります。「最初の設定は面倒やけど、後で絶対楽になる」っていうやつですね。

## CLAUDE.mdの詳細設定

前の章でも触れましたが、CLAUDE.mdは本当に重要です。T3Stack（Drizzle + BetterAuth）での実践的な設定例を詳しく見ていきましょう。

### 基本的なCLAUDE.md構成

```markdown
# CLAUDE.md

## プロジェクト概要
T3Stack + Drizzle + BetterAuth構成のWebアプリケーション

### 技術スタック
- Next.js 14 (App Router)
- TypeScript 5.x
- tRPC v10
- Drizzle ORM + PostgreSQL
- BetterAuth
- Tailwind CSS
- React Hook Form + Zod

## 開発コマンド
```bash
# 開発環境
npm run dev          # 開発サーバー起動 (localhost:3000)
npm run dev:db       # PostgreSQL起動 (Docker)

# データベース
npm run db:generate  # Drizzleスキーマ生成
npm run db:migrate   # マイグレーション実行
npm run db:studio    # Drizzle Studio (localhost:4983)
npm run db:reset     # データベースリセット

# 品質チェック
npm run type-check   # TypeScript型チェック
npm run lint         # ESLint実行
npm run lint:fix     # ESLint自動修正
npm run format       # Prettier実行

# テスト
npm run test         # Jest実行
npm run test:watch   # Jest監視モード
npm run test:ui      # Vitest UI
```

## ディレクトリ構造とルール
```
src/
├── app/                 # Next.js App Router
│   ├── (auth)/         # 認証関連ページ
│   ├── (dashboard)/    # ダッシュボード
│   └── api/            # APIルート
├── components/         # 再利用可能コンポーネント
│   ├── ui/            # shadcn/ui系
│   └── forms/         # フォーム系
├── server/            # サーバーサイド
│   ├── api/           # tRPCルーター
│   └── db/            # データベース設定
├── lib/               # ユーティリティ
└── styles/            # グローバルスタイル
```

## コーディング規約
### 命名規則
- コンポーネント: PascalCase (UserProfile.tsx)
- ファイル名: kebab-case (user-profile.tsx)
- 関数・変数: camelCase
- 定数: UPPER_SNAKE_CASE
-型: PascalCase + 'T' suffix (UserT, PostT)

### インポート順序
1. React関連
2. Next.js関連  
3. 外部ライブラリ
4. 内部ライブラリ(@/から始まる)
5. 相対パス

### ファイル配置ルール
- tRPCルーターは機能別に分割 (users.ts, posts.ts)
- Drizzleスキーマは src/server/db/schema.ts に統合
- BetterAuth設定は src/lib/auth.ts
- 環境変数は .env.local (.env.example参照)

## 技術的制約・注意点
### Drizzle ORM
- スキーマ変更時は必ず `npm run db:generate` 実行
- リレーションは relations() で明示的に定義
- クエリビルダーよりもselect文を推奨
- any型の使用禁止、unknown使用を推奨

### BetterAuth
- セッション管理はサーバーサイドで実装
- クライアント側での直接トークン操作禁止
- ミドルウェアでの認証チェック必須
- OAuth設定は環境変数で管理

### tRPC
- プロシージャは小さく分割
- 入力検証はZodスキーマ必須
- エラーハンドリングはTRPCErrorで統一
- キャッシュ戦略を明示

### Next.js App Router
- サーバーコンポーネントをデフォルト使用
- クライアントコンポーネントは最小限に
- メタデータはmetadata exportで定義
- 動的ルートのvalidationは必須

## よく使うコード例
### tRPCプロシージャの基本形
```typescript
export const userRouter = createTRPCRouter({
  getById: publicProcedure
    .input(z.object({ id: z.string() }))
    .query(async ({ input, ctx }) => {
      const user = await ctx.db.query.users.findFirst({
        where: eq(users.id, input.id),
      });
      if (!user) throw new TRPCError({ code: 'NOT_FOUND' });
      return user;
    }),
});
```

### Drizzleスキーマの基本形
```typescript
export const users = pgTable('users', {
  id: uuid('id').defaultRandom().primaryKey(),
  name: varchar('name', { length: 255 }).notNull(),
  email: varchar('email', { length: 255 }).notNull().unique(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
});
```

## package.jsonの最適化

Claude Codeが理解しやすいように、scriptsを統一的に命名しましょう。

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "type-check": "tsc --noEmit",
    "lint": "next lint",
    "lint:fix": "next lint --fix",
    "format": "prettier --write .",
    "format:check": "prettier --check .",
    "db:generate": "drizzle-kit generate:pg",
    "db:migrate": "drizzle-kit push:pg",
    "db:studio": "drizzle-kit studio",
    "db:reset": "drizzle-kit drop && npm run db:migrate",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}
```

### 環境変数の管理

`.env.example`を用意して、Claude Codeが環境設定を理解できるようにします。

```bash
# .env.example

# Database
DATABASE_URL="postgresql://username:password@localhost:5432/myapp"

# BetterAuth
BETTER_AUTH_SECRET="your-secret-here"
BETTER_AUTH_URL="http://localhost:3000"

# OAuth Providers
GOOGLE_CLIENT_ID="your-google-client-id"
GOOGLE_CLIENT_SECRET="your-google-client-secret"
GITHUB_CLIENT_ID="your-github-client-id"
GITHUB_CLIENT_SECRET="your-github-client-secret"

# Optional
UPLOADTHING_SECRET=""
UPLOADTHING_APP_ID=""
```

## TypeScript設定の最適化

### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "es2017",
    "lib": ["dom", "dom.iterable", "es6"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": [
    "next-env.d.ts",
    "**/*.ts",
    "**/*.tsx",
    ".next/types/**/*.ts"
  ],
  "exclude": ["node_modules"]
}
```

### globals.d.ts

```typescript
// globals.d.ts

// BetterAuth
declare global {
  namespace NodeJS {
    interface ProcessEnv {
      BETTER_AUTH_SECRET: string;
      BETTER_AUTH_URL: string;
      GOOGLE_CLIENT_ID: string;
      GOOGLE_CLIENT_SECRET: string;
      DATABASE_URL: string;
    }
  }
}

export {};
```

## VS Code設定

プロジェクトルートに`.vscode/settings.json`を作成：

```json
{
  "typescript.preferences.importModuleSpecifier": "relative",
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  },
  "tailwindCSS.experimental.classRegex": [
    ["cva\\(([^)]*)\\)", "[\"'`]([^\"'`]*).*?[\"'`]"],
    ["cx\\(([^)]*)\\)", "(?:'|\"|`)([^']*)(?:'|\"|`)"]
  ]
}
```

## 開発効率を上げる設定

### 1. エイリアス設定

パスエイリアスを活用して、インポートを簡潔に：

```typescript
// Before
import { UserProfile } from '../../../components/user/UserProfile'

// After
import { UserProfile } from '@/components/user/UserProfile'
```

### 2. 型定義ファイルの整理

```typescript
// src/lib/types.ts
export type UserT = typeof users.$inferSelect;
export type CreateUserT = typeof users.$inferInsert;
export type UpdateUserT = Partial<CreateUserT>;

export type PostT = typeof posts.$inferSelect;
export type CreatePostT = typeof posts.$inferInsert;
```

### 3. ユーティリティ関数の準備

```typescript
// src/lib/utils.ts
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}

export function formatDate(date: Date | string) {
  return new Intl.DateTimeFormat('ja-JP').format(new Date(date));
}
```

## Claude Codeに最適化したプロジェクト構成

### 1. README.mdの充実

```markdown
# My T3Stack App

## 開発環境セットアップ

1. 依存関係インストール: `npm install`
2. 環境変数設定: `.env.example` を `.env.local` にコピー
3. データベース起動: `npm run dev:db`
4. マイグレーション: `npm run db:migrate`
5. 開発サーバー起動: `npm run dev`

## よく使うコマンド

- `npm run dev`: 開発サーバー起動
- `npm run db:studio`: データベース管理画面
- `npm run type-check`: 型チェック
- `npm run lint`: コード品質チェック

## プロジェクト構成

- [CLAUDE.md](./CLAUDE.md): Claude Code用の設定
- [docs/](./docs/): 詳細ドキュメント
```

### 2. docs/ ディレクトリの活用

プロジェクトの詳細情報をdocs/に整理：

```
docs/
├── api.md          # API仕様書
├── database.md     # DB設計書
├── deployment.md   # デプロイ手順
└── troubleshooting.md  # よくある問題と解決法
```

## まとめ

プロジェクト設定のチェックリスト：

- ✅ CLAUDE.md の詳細設定
- ✅ package.json scripts の統一
- ✅ 環境変数の整理（.env.example）
- ✅ TypeScript設定の最適化
- ✅ VS Code設定の追加
- ✅ README.md の充実
- ✅ docs/ ディレクトリの整備

これらを最初にきちんと設定しておけば、Claude Codeとの開発がめちゃくちゃスムーズになります。次の章では、さらに高度なテクニックを学んでいきましょう！