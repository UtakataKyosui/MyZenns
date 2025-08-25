---
title: "MCPが敷いたレールの上を歩くClaude Code"
emoji: "🛤️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [claude,mcp,claudecode,ai]
published: false
---

# 概要（記事内容はこれに尽きる）

`Claude Code`で自分がされたくない動作をされないように

- `Deny`を設定して、ツールとそのツールが使用しようとしているコマンドをリストアップしておいて、マッチしたものを棄却する
- `Hook`を設定して、ツールとそのツールが使用しようとしているコマンドの内容をパースして解析し、条件によって棄却する
- `Claude Code`を使用している間、特定のエイリアスを上書きしておいて、実行されないようにする

など、様々な手法がこのZennやQ○○a、個人ブログにたくさん上がっています。
ここに新たな方法を提案したいと思います。
それが、**Claude Code only use MCP Tools**、
**Claude Codeが使っていいのは、ユーザが導入したMCP Toolsのみ**という
大胆な方法です。

# ではどういう設定をすればいいのか

1. `Deny`設定に、`Claude Code`純正のほぼ全てのToolを設定します。

Claude Codeが勝手にパッケージをインストールしたり、予期しないコマンドを実行するリスクを防ぎます。

2. `Allow`設定に`MCP Tool`の全てのToolsを設定します。

MCP Toolsは、ユーザーが導入したものに限定することで、Claudeの行動を「レール」に乗せることができます。
これにより、Claudeは許可されたMCPサーバを通じてのみ情報を取得・操作できます。

## おすすめのMCP
```json
  "mcpServers": {
    "sequential-thinking": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-sequential-thinking"
      ],
      "env": {}
    },
    "context7": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "-y",
        "@upstash/context7-mcp"
      ],
      "env": {}
    },
    "magic": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "-y",
        "@21st-dev/magic"
      ],
      "env": {}
    },
    "serena": {
      "type": "stdio",
      "command": "uvx",
      "args": [
        "--from",
        "git+https://github.com/oraios/serena",
        "serena-mcp-server"
      ],
      "env": {}
    },
    "openmemory": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "-y",
        "openmemory"
      ],
      "env": {
        "OPENMEMORY_API_KEY": "$OPENMEMORY_API_KEY",
        "CLIENT_NAME": "openmemory"
      }
    },
    "drizzle-doc": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "-y",
        "sitemcp@latest",
        "https://orm.drizzle.team"
      ],
      "env": {}
    },
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-github"
      ],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "$GITHUB_PERSONAL_ACCESS_TOKEN"
      }
    },
    "phantom": {
      "type": "stdio",
      "command": "phantom",
      "args": [
        "mcp",
        "serve"
      ],
      "env": {}
    }
  },
```

私は上記のMCPを使っています。
説明しましょう。

### 重要度（高）

- `Serena`
    - 
- `OpenMemory`
    - 

### 重要度（中）

- `Sequential Thinking`
    - 
- `Context7`
    - 
- `Magic`
    - 

### 重要度（小）

- `SiteMCP（Drizzle Document）`
    - 
- `GitHub`
    - 
- `Phantom`
    - 

## `Deny` - `Allow`設定をする
