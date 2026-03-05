---
title: "Open Duck Mini カラーシミュレーターを作った"
date: 2026-03-04T00:00:00+09:00
slug: duck-color-simulator
draft: true
description: "Open Duck Miniというオープンソース二足歩行ロボットの3Dプリントパーツの色をブラウザ上でシミュレーションするWebアプリを作りました。"
tags: ["3Dプリント", "ロボット", "React", "Three.js", "WebApp"]
categories: ["開発"]
---

## きっかけ

[Open Duck Mini](https://github.com/apirrone/Open_Duck_Mini) というオープンソースの小型二足歩行ロボットを自作しようとしている。このロボットは全身のパーツをFDM方式の3Dプリンターで印刷して組み立てるのだが、印刷前に「どの色の組み合わせにするか」を確認したくなった。

実際に印刷してみてから「なんか思ってたのと違う」となるのは避けたい。

## 作ったもの

**Open Duck Mini カラーシミュレーター**

ブラウザ上でOpen Duck Mini v2の全パーツ（45種類）を3D表示し、グループごとに色を変えて配色を確認できるWebアプリ。

{{< figure src="screenshot.png" alt="カラーシミュレーターのスクリーンショット" >}}

- **ボディ・頭部・脚部・足部・アンテナ**など部位ごとに色を設定できる
- カラープリセット（ホワイト / ブラック / イエロー×ブラック など）
- ドラッグで回転、ホイールでズーム
- 電子部品（サーボ・基板など）も表示/非表示の切り替え可能

## 技術スタック

| 技術 | 用途 |
|------|------|
| React + Vite | フロントエンド |
| React Three Fiber | 3Dレンダリング（Three.js のReactラッパー） |
| @react-three/drei | STLLoader・OrbitControls |
| Tailwind CSS | スタイリング |

## 工夫した点

### URDFからアセンブリ座標を取得

45個のパーツをバラバラに並べるだけでは意味がないので、Open Duck MiniのURDF（ロボット記述ファイル）をPythonで解析して、各パーツのワールド座標を自動計算した。

```python
# キネマティクスチェーンを再帰的にたどって世界座標を計算
def get_world_transform(link):
    if link not in joint_map:
        return np.eye(4)
    parent, T = joint_map[link]
    parent_T = get_world_transform(parent)
    return parent_T @ T
```

これでデフォルトポーズ（全関節0°）での組み立て状態を正確に再現できた。

### 座標系の変換

URDFはZ-up座標系（Z軸が上）、Three.jsはY-up座標系（Y軸が上）。ここでハマった。

個別にXYZ座標を入れ替えようとしたらパーツがバラバラになったので、全パーツを1つのGroupで包んでX軸-90°回転を当てる方法に切り替えた。

```jsx
<group rotation={[-Math.PI / 2, 0, 0]}>
  {PARTS.map(part => <Part key={part.id} {...part} />)}
</group>
```

シンプルな解決策だが、これが一番確実だった。

## 今後

- 配色をURLパラメーターに保存して共有できるようにしたい
- GitHub Pagesで公開予定
