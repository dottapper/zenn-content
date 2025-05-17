---

title: "Magic MCPでUIを自然言語から生成してみた：勘違いと気づきから始まるリアル体験記"
emoji: "🎨"
type: "tech"
topics: \["Claude", "MagicMCP", "Cursor", "UI生成", "LLM"]
published: false
----------------

## ✨ はじめに

ずっと調べたいとおもってたMCP。気になるけど後回ししておりまして、気になったのが、 Magic MCP。
「自然言語からUIを作れる」ってどういうこと？しかもリアルタイムで？
気になって、 Cursor で連携して実際に使ってみました。

結論から言うと……

> 勝手に動くと思ってたけど、これは"きてる"技術でした。

この記事では、私が体験した **つまづき／動作感のギャップ／そして気づき** をベースに、 Magic MCP の連携方法と、実際のUI生成までのプロセスをまとめます。

---

## 🌟 想定読者

- Cursor を使っているけど、MCP がよくわからない人
- Magic MCP って何ができるの？と思っている人
- 自然言語で UI 生成って実際どの程度便利なの？と疑問な人

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

### 1. もっとも簡単：CLI ワンライナー

```bash
npx @21st-dev/cli@latest install claude --api-key <YOUR_KEY>
```

- `claude` を `cursor` / `windsurf` / `vscode` に替えれば各 IDE 用に自動設定
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

![漫画企画会議のモックGIF](/images/ezgif.com-video-to-gif-converter.gif)

---

## 🌀 動きを見てわかった「勝手な予期」

### ❌ 「リアルタイムに UI を操作しながら生成できる」と思っていた

→ **WYSIWYG【Framer みたいな】UI エディタではなかった！**

### ✅ 正しくは：

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

### 良かった点

- Claude と自然言語でやりとりしながら UI が作れるのは新感覚
- コードがしっかりしていて再利用しやすい
- ダークテーマやアクセントカラーも指示できる

### 惜しかった点

- GUI でリアルタイムに調整できると思っていたが違った
- "どのプロジェクトで使えるのか"がわかりにくかった（環境変数とセットアップが鍵）

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

私も最初は「うまく動かない」「あれ？これだけ？」と感じていました。
でも、使っていくうちに「あ、これは"新しい UI の作り方"なんだ」と納得できるようになりました。

Magic MCP、試してみる価値は十分にあります。自然言語から UI を生成する未来が、もう始まっているのを実感できるはずです。
