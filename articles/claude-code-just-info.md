---
title: "Claude Codeに正しい情報を収集してもらうには"
emoji: "✨"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["claude", "claudecode", "mcp"]
published: true
---

# `Claude Code`が間違った情報を持ってくるのがストレス

`Claude Code`を使っているとき、Web フロントエンドの文脈の開発を任させたときに
`TailwindCSS`の v4 を導入しているのに、なぜか v3 の設定ファイルを生成してくるときがあります。

これ、いまだによくあります。
`Claude Code`を使う上で、なるべく少ないトークン消費で収めるには
正確な情報を一回で持ってきてもらえる方が助かります。

# そのためにどうするか

## ライブラリ・パッケージに関する情報

`Context7`MCP を使用しましょう。以下の流れになります。

1. 特定のライブラリのサイトを`SiteMCP`で MCP 化して、ライブラリの情報を正しく得られるようにする
2. それでも情報が足りなければ、`Context7`MCP で情報の調査をさせる

### 流れをどう伝えておくか

そこで、この記事の内容を参考にします。

https://zenn.dev/izumin_0401/articles/claude-codemd

```md:CLAUDE.md
・・・

## 情報参照フロー
1. 指定されたライブラリのドキュメントサイトのMCPが設定されているかを、`.mcp.json`で調べる
    1. されていたら、そのMCPを通して情報を調査して、3に移る
    2. されていなければ、2へ移る
2. `Context7`MCP で情報の調査を行う
3. 調査結果を、`任意のMarkdownファイルのパス`を記録する
4. 開発（実装）に戻る

・・・
```

### どうやって MCP を設定するか（例）

```json:.mcp.json
{
	"mcpServers": {
		"file-system": {
			"command": "npx",
			"args": ["-y", "@modelcontextprotocol/server-filesystem", "/workspaces"]
		},
		"Context7": {
			"command": "npx",
			"args": ["-y", "@upstash/context7-mcp"]
		},
		"loco-cite": {
			"command": "npx",
			"args": [
				"-y",
				"sitemcp@latest",
				"https://loco.rs",
				"-m",
				"/**"
			]
		},
		"Shuttle": {
			"command": "shuttle",
			"args": ["mcp", "start"]
		},
		"tailwindcss": {
			"command": "npx",
			"args": [
				"-y",
				"sitemcp@latest",
				"https://tailwindcss.com",
				"-m",
				"/**"
			]
		},
		"shadcn-ui": {
			"command": "npx",
			"args": [
				"-y",
				"sitemcp@latest",
				"https://ui.shadcn.com/docs",
				"-m",
				"/**"
			]
		}
	}
}
```

## 上記以外

公式が提供している MCP を使うのが正直ベストなのですが、
なかった場合は、`SiteMCP`を利用し、特定のライブラリのドキュメントページを MCP 化した上で、
ドキュメントページが MCP 化されていないライブラリの情報は
`Firecrawl`MCP で情報の調査をさせるという方法をとるようにすることにしています。

つまり、

1. 特定のライブラリのサイトを`SiteMCP`で MCP 化して、ライブラリの情報を正しく得られるようにする
2. それでも情報が足りなければ、`Firecrawl`MCP で情報の調査をさせる

という流れを`Claude Code`に伝えておけば、正しい情報を取得し、
活用してもらえるはずです。

### 流れをどう伝えておくか

```md:CLAUDE.md
・・・

## 情報参照フロー
1. 指定されたライブラリのドキュメントサイトのMCPが設定されているかを、`.mcp.json`で調べる
    1. されていたら、そのMCPを通して情報を調査して、3に移る
    2. されていなければ、2へ移る
2. `Firecrawl`MCP で情報の調査を行う
3. 調査結果を、`任意のMarkdownファイルのパス`に記録する
4. 開発（実装）に戻る

・・・
```

### どうやって MCP を設定するか（例）

```json:.mcp.json
{
	"mcpServers": {
		"file-system": {
			"command": "npx",
			"args": ["-y", "@modelcontextprotocol/server-filesystem", "/workspaces"]
		},
		"firecrawl-mcp": {
			"command": "npx",
			"args": ["-y", "firecrawl-mcp"],
			"env": {
				"FIRECRAWL_API_KEY": "$FIRECRAWL_API_KEY",
				"FIRECRAWL_RETRY_MAX_ATTEMPTS": "5",
				"FIRECRAWL_RETRY_INITIAL_DELAY": "2000",
				"FIRECRAWL_RETRY_MAX_DELAY": "30000",
				"FIRECRAWL_RETRY_BACKOFF_FACTOR": "3",
				"FIRECRAWL_CREDIT_WARNING_THRESHOLD": "2000",
				"FIRECRAWL_CREDIT_CRITICAL_THRESHOLD": "500"
			}
		},
		"loco-cite": {
			"command": "npx",
			"args": [
				"-y",
				"sitemcp@latest",
				"https://loco.rs",
				"-m",
				"/**"
			]
		},
		"Shuttle": {
			"command": "shuttle",
			"args": ["mcp", "start"]
		},
		"tailwindcss": {
			"command": "npx",
			"args": [
				"-y",
				"sitemcp@latest",
				"https://tailwindcss.com",
				"-m",
				"/**"
			]
		},
		"shadcn-ui": {
			"command": "npx",
			"args": [
				"-y",
				"sitemcp@latest",
				"https://ui.shadcn.com/docs",
				"-m",
				"/**"
			]
		}
	}
}
```

# まとめ

今回は、２つの方法を書きましたが、

- ライブラリの最新情報が欲しいなら`Context7`MCP
- ライブラリ以外の情報をガンガン集めたい時に`Firecrawl`MCP

使うというのが良いと思います。
