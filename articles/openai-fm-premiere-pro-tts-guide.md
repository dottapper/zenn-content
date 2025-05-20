---
title: "OpenAIの音声生成 OpenAI.fm（TTS/FMモデル）をPremiere Proで使うときの注意点"
emoji: "🎙️"
type: "tech"
topics: ["openai", "premierepro", "音声編集", "AIツール"]
published: false
slug: "openai-fm-premiere-pro-tts-guide"
---

## 🎧 きっかけ ──「WAV なのに無音!?」C

OpenAI の **TTS（Text-to-Speech）/FM モデル** で生成したナレーションを  
Premiere Pro にドラッグしたところ、**インポートは成功するのに波形が出ず無音**──  
そんな症状に遭遇しました。
https://www.openai.fm/

調べてみると、**ファイル仕様と Premiere Pro の「自動コンフォーム」** が
かみ合わないのが原因。この記事では **原因の理解 → 解決策 → 実践コマンド** まで
サッとまとめます。

---

## 📐 OpenAI TTS 出力 WAV の仕様

| 項目           | 値                          |
| -------------- | --------------------------- |
| コンテナ       | WAV (RIFF)                  |
| ビット深度     | **16-bit PCM**（signed LE） |
| サンプルレート | **24 kHz** 固定             |
| チャンネル     | Mono                        |
| 浮動小数点か?  | いいえ（32-float ではない） |

> **ポイント**  
> 多くの DAW／NLE は 44.1 kHz / 48 kHz を想定。  
> **24 kHz** だと "自動サンプルレート変換" がうまく働かず、無音扱いになる
> ケースがあります。

---

## 🚩 Premiere Pro で起こる 2 つのパターン

1. **波形ゼロ & 再生しても無音**
2. **読み込みエラー**（「ファイル形式がサポートされていません」等）

いずれも **24 kHz → 48 kHz** へ変換すれば再生可能になることを確認しました。  
ビット深度は 16-bit のままで OK です。

---

## 🔧 解決策：16-bit / 48 kHz PCM に変換する４つの方法

### 1. Adobe Audition

1. `openai_tts.wav` を開く
2. **File → Export → File**
3. 設定を下記に変更し `Save`
   - **Format:** WAV (Microsoft)
   - **Sample Rate:** `48000 Hz`
   - **Bit Depth:** `16-bit`
   - **Dither:** _Triangular / High-Pass Noise Shaping_ 推奨
4. 書き出したファイルを Premiere に読み込む

### 2. Adobe Media Encoder（単体でも、Premiere からでも可）

1. 変換したい WAV をキューに追加
2. **Format:** WAV
3. **Audio ▸ Sample Rate:** 48 kHz
4. **Audio ▸ Sample Size:** 16-bit
5. エンコード実行

### 3. ffmpeg（最速）

```bash
# brew install ffmpeg してある前提
ffmpeg -i openai_tts.wav -ar 48000 -sample_fmt s16 -ac 1 openai_tts_48k.wav
```

- `-ar 48000` … 48 kHz
- `-sample_fmt s16` … 16-bit PCM
- `-ac 1` … モノラル維持（ステレオ化したいなら `-ac 2`）

### 4. ChatGPT にスクリプトを生成させる

> _「ffmpeg で 24 kHz/16-bit の WAV を 48 kHz/16-bit に一括変換する
> PowerShell／Bash スクリプトを書いて」_ と依頼すれば、
> そのままローカルで実行できるスクリプトを生成してくれます。

---

## 📝 コピペ用 ffmpeg テンプレ

```bash
for f in *.wav; do
  ffmpeg -loglevel error -y -i "$f" \
         -ar 48000 -sample_fmt s16 -ac 1 \
         "${f%.wav}_48k.wav"
done
```

---

## ✅ まとめ

- **OpenAI TTS の WAV = 24 kHz / 16-bit PCM**
- Premiere Pro は 48 kHz 前提のため **自動変換に失敗すると無音** になる
- **Audition／Media Encoder／ffmpeg** で **48 kHz / 16-bit** に変換すれば解決
- 量が多いときは **ffmpeg + スクリプト** が最速
- ChatGPT は _変換コマンドを生成するアシスタント_ として使うのが安全

これで「WAV なのに音が出ない！」問題とはサヨナラ。
