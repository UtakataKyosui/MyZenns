---
title: "卒論をLateXで書くあなたへ"
emoji: "📄"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: [latex,githubactions,gpt4omini,textlint]
published: true
---

# LateXを急に書くことになったあなたへ。
卒論をLateXで書かなければならなくなった人、そして卒論の誤字脱字とか内容チェックを効率化したい人向けに
本記事を作成しました。

## 使用するツールについて

| 使用ツール名 | 役割 |
| --- | --- |
| Textlint | GPTを通すまでもないような文字の簡潔なチェック |
| GPT-4o-mini | LLMからの文章チェックにより得られる内容の改善提案 | 
| GitHub Actions | 別ブランチからのPull Request作成時に上記2つの自動起動 | 

## GitHub Actionsを定義

:::details GitHub Actionsがわからない人向けのざっくりした説明
GitHubに対するイベント（Push・PRなど）が発生したら
ymlファイルで定義した処理が自動的に実行されるもの。
:::

GitHubに自動で以下の要件のプログラムを動かしてもらいましょう。

1. Textlintで簡単な文字の誤りのチェック

2. GPTに文章のチェックをさせる
:::details どういうことをチェックさせるといいか
- 現在の内容における不足点
- より理解がしやすくなる方法の提案
- 追加すべき図表や視覚的要素の提案
- 誤字脱字の指摘
:::

3. （Optional）GPTにどのような追加をすればより良くなるかアドバイスをもらう
:::details どういうアドバイスをもらえばいい？
- 事例やデータの引用の提案
- 足りないデータを補う参考URLや参考書籍などの紹介
- 全体の構成をより論理的にするための改善の提案
:::

### Actionsを書いていくぞ～！
```yml:llm-review.yml
name: GPT Checker

# Texファイルに変化があるPRのみ動作
on: 
    pull_request:
        paths:
            - "*.tex" 

jobs: 
    # まずは軽い誤字脱字のチェック
    linter:
        name: linter
        runs-on: ubuntu-latest
        steps:
            - uses: actions/checkout@v4
              with: 
                  submodules: true # SubModules管理しているものがあっても動くように
            - uses: tsuyoshicho/action-textlint@v3
              with:
                github_token: ${{ secrets.GITHUB_TOKEN }}
                reporter: github-pr-revirew
                level: warning
                textlint_flags: "*.tex"
                
    # もうちょいレベル高めの修正アドバイスをもらう
    gpt-review:
        needs: linter
        runs-on: ubuntu-latest
        steps: 
            - uses: anc95/ChatGPT-CodeReview@main
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
          # Optional
          LANGUAGE: Japanese
          MODEL: gpt-4o-mini
          PROMPT: "LateX 文書をレビューし、どのように修正すればいいか指摘してください。\n\n$(cat check_list.txt)"
          top_p: 1
          temperature: 1
          INCLUDE_PATTERNS: "*.tex" 

    # より内容を充実させるためのアドバイスをもらう
    gpt-advice:
        needs: gpt-review
        runs-on: ubuntu-latest
        steps: 
            - uses: anc95/ChatGPT-CodeReview@main
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
          # Optional
          LANGUAGE: Japanese
          MODEL: gpt-4o-mini
          PROMPT: "LateX 文書をレビューし、以下の項目\n\n$(cat upgrade_list.txt)\n\n についてアドバイスをください。"
          top_p: 1
          temperature: 1
          INCLUDE_PATTERNS: "*.tex" 
```

### 必要なファイル

| ファイル名 | 設置場所 | 役割 |
| --- | --- | --- |
|`llm-review.yml`|リポジトリ上の<br />`.github/workflows/`<br />ディレクトリ|PR作成時の<br />Actions定義|
|`check_list.txt`|リポジトリの<br />ルートディレクトリ| GPTに伝える<br />チェック内容を書いておく |
|`upgrade_list.txt`|リポジトリの<br />ルートディレクトリ| GPTに伝える<br />内容を充実させる<br />アドバイスの項目を<br />書いておく |
|`.textlintrc`|リポジトリの<br />ルートディレクトリ| Textlintが実行される時の<br />設定ファイル |
|`prh.yml`|リポジトリの<br />ルートディレクトリ| ルールベースの文字チェックと<br />置替え対象のデータの定義ファイル |

#### 表の下2ファイルの設定例
```json
{
    "plugins": [
        "latex2e"
    ],
    "rules": {
        "prh": {
            "rulePaths": [
                "./prh.yml"
            ]
        },
        "preset-ja-technical-writing": {
            "sentence-length": {
                "max": 100
            },
            "max-kanji-continuous-len": {
                "max": 6,
                "allow": [
                    "株式会社丸丸丸丸社",
                    "丸丸県立丸丸大学"
                ]
            },
            "no-mix-dearu-desumasu": {
                "preferInHeader": "である",
                "preferInBody": "である",
                "preferInList": "である",
                "strict": true
            },
            "ja-no-mixed-period": {
                "periodMark": "．",
                "allowPeriodMarks": [],
                "allowEmojiAtEnd": false,
                "forceAppendPeriod": true
            },
            "no-double-negative-ja": true,
            "no-dropping-the-ra": true,
            "no-doubled-conjunctive-particle-ga": true,
            "no-doubled-conjunction": true,
            "no-exclamation-question-mark": true,
            "ja-no-weak-phrase": true,
            "ja-no-successive-word": true,
            "ja-no-abusage": true,
            "ja-no-redundant-expression": true,
            "no-unmatched-pair": true
        },
        "preset-japanese": true,
        "ja-space-between-half-and-full-width": {
            "space": "always"
        },
        "no-mixed-zenkaku-and-hankaku-alphabet": true,
        "max-comma": {
            "max": 4
        },
        "@textlint-rule/no-invalid-control-character": true,
        "textlint-rule-spellcheck-tech-word": false
    }
}
```
:::message alert
一部、自身の個人情報を守るため、実際に使用した
ものとは異なる設定が書かれています。
:::

```yml:prh.yml
version: 1

rules:
  - expected: "，"
    pattern: 
      - "、"
      - ","
    allow:
      - "[0-9],[0-9]"
  - expected: したがって
    pattern: なので
  - expected: ゆえに
    pattern: なので
  # こっちを採用
  - expected: 多数
    pattern: たくさん
  # - expected: 複数
  #  pattern: たくさん
  - expected: である
    pattern: です
  - expected: した
    pattern: ました
  - expected: する
    pattern: します
```

# 終わりに
これをより良いものにしたい場合は、ご自身で自由に手を加えてください。
Texで色々しないといけないのは初心者には高いハードルで、まずそれに慣れるまで苦労するかもしれませんが、
文章チェック位はこれで自動化できます。
問題があるとすれば、この方法にはGPTのAPIを使う以上少しお金がかかりますが、4o-miniであればそれも比較的安く済むので、「LLMのAPIって使うの高そうで怖いな」という方も、これを気に使ってみてはどうでしょうか