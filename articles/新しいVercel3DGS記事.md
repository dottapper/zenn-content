---
title: "Vercel で 3D Gaussian Splatting (.ply) が表示されない?! Git LFS ポインタ問題を 15 分で解決したメモ"
emoji: "🚀"
type: "tech"
topics: ["three.js", "vercel", "git-lfs", "webgl", "3dgs"]
published: false
---

## TL;DR

- **Network タブで .ply が 0 kB** → ほぼ 100 % Git LFS ポインタしか配信されていない。
- **GitHub で `Stored with Git LFS`** と出ていたらポインタ確定。
- 対応は 2 択 👇

  1. **Vercel 側で Git LFS を有効にする**（ビルドが走るプロジェクトなら _Git → Enable LFS_ を ON）。
  2. **ポインタを外し実体 32 MB を直接コミット**（`git lfs untrack` → `git rm --cached` → `git add`）。

- hash 付き _Preview URL_ はログイン必須。公開共有は **Production URL** を使う。

## 背景

花の 3D Gaussian Splatting (.ply) を **ヒーローセクション**に置いた LP を Vercel にデプロイしたら、ローカルで見えている 3D が本番では真っ白＆赤いローディングバーだけ…。

DevTools には `Invalid ply header`。サイズを見たら **0 kB** で配信されていた。

![Network 0kB screenshot](./images/network-0kb.png)

![3DGSが表示されない状態](/images/vercel-3dgs/0514-02.png)

## 原因を絞り込むまでの 5 分

| チェック                  | 結果                       |
| ------------------------- | -------------------------- |
| **Network Size**          | 0 kB / 304 → ファイルが空  |
| **GitHub で .ply を開く** | _Stored with Git LFS_ 表示 |
| **`git lfs ls-files`**    | 対象 .ply がリストに居る   |

→ **Git LFS ポインタだけがリポジトリにあり、Vercel ビルドで実体が取れていない** と確定。

## 解決策 A — Vercel で Git LFS を有効化

1. **Project → Settings → Git → Git Large File Storage** を **ON**。
2. `Redeploy`。
3. ビルドログに

   ```bash
   > git lfs fetch --all
   > git lfs checkout
   ```

   が出れば OK。

> 注意: **static only** プロジェクトだと `vercel build` が 0.06 秒で終わり、LFS コマンドが走らない。<br>package.json に `"vercel-build": "git lfs pull"` を置くなど「ビルドステップ」を作る必要がある。

## 解決策 B — ポインタ解除＋実体コミット (今回はこれで突破)

```bash
# 1. LFS 追跡を外す
git lfs untrack "scene.compressed-viewer/scene.compressed.ply"

# 2. ポインタをインデックスから外す
git rm --cached scene.compressed-viewer/scene.compressed.ply

# 3. 実体が 32 MB あるか確認
ls -lh scene.compressed-viewer/scene.compressed.ply

# 4. 実体を再追加
git add scene.compressed-viewer/scene.compressed.ply

# 5. コミット & プッシュ
git commit -m "Embed PLY directly (remove LFS)"
git push origin main
```

GitHub でファイルを開くと _Binary file cannot be displayed_ とだけ出て、**Stored with Git LFS** が消えれば成功。Vercel が自動デプロイ → Network に **32 MB** が載れば 3D 復活！

![3DGSが表示された成功状態](/images/vercel-3dgs/0514-03.png)

## 「ログインしないと見られない?」を解決

- `https://<hash>-project.vercel.app` は _Preview URL_ → `Protect Preview Deployments` が ON だと要ログイン。
- **Production** エイリアス `https://project.vercel.app` を共有すれば誰でも閲覧可能。
- _Protect Preview Deployments_ は _Settings → General_ で OFF にできる。

## ついでにやったこと

- 不要になった `sample/` と `.DS_Store` を `git rm -r`。
- `.gitignore` に `.DS_Store` 追加。
- progress bar が気になったので CSS で非表示。

```css
#progressBar {
  display: none !important;
}
```

## 今後のチューニングメモ

- **Draco + glTF** or **meshopt** でさらに軽く。（※これは 3DGS ではなく、従来の 3D メッシュモデル用の圧縮技術です）
- ロード後に `OrbitControls.enableZoom = false` で UX 改善。（3DGS も glTF も共通して使える）
- `IntersectionObserver` で遅延ロードして LCP を稼ぐ。（複数の 3D モデルがあるページで効果的）

### 3DGS 特有の最適化メモ

- ply ファイルの品質設定（圧縮率）を調整して容量とクオリティのバランスを探る
- モバイル向けに低解像度版を別途用意することも検討

---

### 参考リンク

- Vercel Docs — [Git LFS](https://vercel.com/docs/git-integrations/git-lfs)
- Three.js Docs — [PLYLoader](https://threejs.org/docs/index.html?q=ply#examples/en/loaders/PLYLoader)

> 以上、泣きながら 3D ガウシアンスプラットを LP に載せた日の備忘録でした。
> ただ、まだこれでいいのかと不安なところもあるので、（外部サーバーに３ D をおいた方がいいのか？など）。いろいろ試してみます。
