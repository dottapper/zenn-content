---
title: "SuperSplat Viewer調整メモ ─ LPヒーローへの組み込みとトラブルシュート"
emoji: "🔧"
type: "tech"
topics: ["supersplat", "3dgs", "threejs", "webgl"]
published: false
---

## ✨ この記事でわかること

- SuperSplat Viewer を LP のヒーローセクションに置いたとき "勝手に回転する／やたら寄っている" を止める方法
- **settings.json** と iframe の URL パラメータをいじるだけで解決できるポイント
- もっと大きく（または小さく）見せたいときの調整法

![調整前（大きすぎる、画面に寄り過ぎている）](/images/supersplat-viewer/0513-05.png)
![調整前（上によって、勝手に回転している状態）](/images/supersplat-viewer/0513-03.png)
![調整後の状態](/images/supersplat-viewer/0513-04.png)

---

## 🐛 発生していた問題

1. **不要な自動回転** – Hero 部分に埋め込むとモデルが勝手にグルグル
2. **カメラの寄り／引き** – ブラウザで確認すると位置がズレる
3. **どこを触ればいいか不明** – Viewer が生成した HTML/JS が多く、設定箇所を探すのが大変

---

## 🔑 核心は _settings.json_

試行錯誤の末、_settings.json_ を正しく書き直すだけでほぼ解決できると判明しました。以下が最小構成のサンプルです。

```json
{
  "camera": {
    "fov": 57,
    "position": [0.06, 2.84, 15.86],
    "target": [0.0, 0.5, 0.0],
    "startAnim": "none",
    "animTrack": null,
    "autoRotate": false
  },
  "controls": {
    "autoRotate": false,
    "enableRotate": true,
    "enablePan": true,
    "enableZoom": true
  },
  "background": { "color": [0, 0, 0] },
  "animTracks": []
}
```

✅ JSON にコメントは書けません
ドキュメント用の説明はファイル外に置きましょう。

---

## 🛠 iframe の書き方

```html
<iframe
  id="splatViewer"
  src="scene.compressed-viewer/index.html?settings=../settings.json&noanim&startAnim=none&autoRotate=false"
  class="viewer-frame"
  allow="autoplay; fullscreen; accelerometer; gyroscope"
  style="pointer-events:auto; will-change:transform; transform:translateZ(0);"
  loading="eager"
></iframe>
```

noanim と autoRotate=false でアニメ無効

pointer-events:auto がないとマウス操作を拾えないので注意

## 🔍 表示サイズを変えたいとき

settings.json で カメラ Z と FOV を変えるだけ。

```json
{
  "camera": {
    "fov": 45,
    "position": [0.06, 2.84, 12.0]
  }
}
```

Z を小さく / FOV を小さく → 大きく 見える

Z を大きく / FOV を大きく → 小さく 見える

---

## 🎛 代替テク

| 方法                                      | 何が起きるか         | 使い所                                      |
| ----------------------------------------- | -------------------- | ------------------------------------------- |
| `<pc-entity scale="1.2 1.2 1.2">`         | モデル自体をスケール | PlayCanvas 等で細かく制御したい             |
| `.viewer-frame { transform:scale(1.1); }` | iframe ごと拡大      | "画像的" に少しだけ大きく見せたいだけなら楽 |

---

## ✅ チェックリスト

☑️- startAnim: "none"
☑️- camera.autoRotate, controls.autoRotate → どちらも false
☑️- iframe URL に noanim&autoRotate=false
☑️- pointer-events:auto を忘れない
☑️- JSON は コメント禁止

---

## 📝 まとめ

・カメラ設定と URL パラメータを揃えると 不要な自動回転が完全停止。

・カメラ Z と FOV の 2 数値だけで "大きさ" の第一印象はほぼ決まる。

・Viewer 側コードを触らずに済むので、バージョンアップにも強い。
