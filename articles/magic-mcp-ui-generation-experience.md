---
title: "Magic MCPでUIを自然言語から生成するタイプのMCPを試す"
emoji: "🎨"
type: "tech"
topics: ["Claude", "MagicMCP", "Cursor", "UI生成", "LLM"]
published: true
slug: magic-mcp-ui-generation-experience
---

## ✨ はじめに

ずっと調べたいとおもってた MCP。気になるけど後回ししておりまして、気になったのが、 Magic MCP。
「自然言語から UI を作れる」ってどういうこと？しかもリアルタイムで？
気になって、 Cursor で連携して実際に使ってみました。

この記事では、私が体験した **つまづき／動作感のギャップ／そして気づき** をベースに、 Magic MCP の連携方法と、実際の UI 生成までのプロセスをまとめます。ざっくり。何かのヒントになれば嬉しいです。

![21st.devのホームページ](/images/MagicMCP/screenshot-homepage.png)

---

## 🪄 Magic MCP とは？

> Cursor などのエディタ内 AI と連携し、**自然言語から UI コンポーネントを生成し、自動でコードに書き出してくれる拡張機能**です。

使える AI：

- Claude（Chat）
- Cursor（エディタ AI）
- Windsurf
- VSCode（MCP 対応）

出力されるのは、React (TypeScript) + Tailwind CSS ベースの `.tsx` ファイルなど。
フォルダ構成も `/components/ui/` に統一され、再利用しやすい形式で出力されます。

---

## 🛠 セットアップ方法（所要時間：約 5 分）

### 0. 事前準備

**Node.js が入っているか確認**

```bash
node -v   # v18 以上なら OK
```

**API キーを発行**

- [21st.dev の Magic コンソール](https://console.21st.dev)へアクセス
- 右上の「API」→「Get your Key」あるいは「Generate new key」をクリック
- 表示された Key をワンクリックでコピー（＊再表示できない場合が多いので必ずメモ帳にも控えておくと安心）
- キーは公開 NG です。ターミナルに貼る時は履歴を削除するか、シェル変数に入れましょう

### 1. もっとも簡単

```bash
npx @21st-dev/cli@latest install cursor --api-key <YOUR_KEY>
```

- `cursor`を`claude` / `windsurf` / `vscode` に替えれば各 IDE 用に自動設定
- 実行すると `~/.claude/mcp.json` などにエントリが生成され、すぐに使えます
- 数十秒で「Installed Magic MCP ✓」と出れば成功です
- 所要時間：3〜4 分（ダウンロード込み）

### 2. Cursor で手動追加したい場合

1. Cursor ▷ Settings ▷ Features ▷ MCP
2. "+ Add New MCP Server" をクリック
3. 以下の情報を入力:
   - Name: Magic
   - Type: stdio
   - Command: `npx -y @21st-dev/magic@latest`
   - Env: `API_KEY=<発行したキー>`
4. 保存 → Refresh を押すとツール一覧に `@21st-dev/magic/ui` が出現

---

## 🪪 実際にやってみたこと

### 1. Claude に `/ui search floating action button` と打つ

> ここでの「Claude」は web.claude.ai の直接インターフェースと Cursor 内の Claude の両方を指します。
> Magic MCP の UI コマンドは Claude 専用に最適化されており、GPT-4 など他のモデルでは
> 正常に動作しない場合があります。

すると、以下のような UI 提案がテキストで表示されました：

- メニュー展開型 FAB
- ソーシャルボタン型 FAB
- 縦展開型 FAB

### 2. 「pick 3」と送ってみる

Claude は「フローティングアクションボタンを HTML に直接実装しました」と返答。
中身を見たら、React ではなく `manga-meeting-minutes.html` に静的 HTML が埋め込まれていました。

![漫画企画会議のモックアップUI](/images/MagicMCP/ezgif.com-video-to-gif-converter.gif)

---

### ✅ できたこと

- Claude に自然言語で指示 → UI 候補が返る
- 気に入ったスタイルを選んで、ファイルにコードが書き出される
- 表示するにはページで import して JSX で貼り付けが必要

---

## 💡 実は「GUI で選べるモード」もある？

他のユーザーの動画では、ポップアップで 3 つのバリアントから UI を選べる画面が登場していました。

が…

```bash
npx @21st-dev/agent@latest
```

→ を試すと `npm error 404`。
どうやらその「Magic Agent」はまだ**ベータ／限定公開中**のようで、現時点で誰でも使えるわけではありません。

---

## 🎨 感想まとめ

- Cursor 内の Claude と自然言語でやりとりしながら UI が作れるのは新感覚
- コードがしっかりしていて再利用しやすい

---

## 📜 これから使う人へ：おすすめコマンド

```text
/ui search button
/ui create a modal with title, content, and confirm button
pick 1
"Make it smaller and purple (#6c5ce7)"
"Save this to /components/ui/Fab.tsx"
```

---

## 🚀 まとめ

Magic MCP は、Claude や Cursor と組み合わせると「UI プロトタイピングの武器」になります。
完全な GUI ではないけど、**自然言語からの UI 生成ツールとしては今最強クラス**。

もっと人間的で、直感的な開発の一歩を踏み出したい人には、確実におすすめできます。

---

## ✍️ おわりに

Magic MCP、試してみる価値は十分にあります。自然言語から UI を生成する未来が、もう始まっているのを実感できるはずです。
