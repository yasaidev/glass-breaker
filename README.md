# 爆音グラス破壊サイト

<https://hakai.yasai.dev/>

![demo](https://user-images.githubusercontent.com/3955027/104024067-e4b03780-5205-11eb-89d2-7899b78626d0.gif)

## [これ](https://hakai.yasai.dev/)は何？

グラスを音で破壊する[サイト](https://hakai.yasai.dev/)です，

具体的には，グラスの固有周波数を再生して激しく共振させることで破壊します．

## 使い方
<https://hakai.yasai.dev/usage>

### 準備

- スマホもしくはパソコン
- 割れやすそうなワイングラス
- いい感じの長さのストロー
- できれば爆音スピーカー
(最悪スマホスピーカーでもOK)

### 破壊

- Analysisボタンを押す
  (マイクの確認ダイアログがでたら許可)
- もう一度Analysisボタンを押す
- カウントダウン後グラスをはじいて音を出す
- サイトが音を自動分析
- Playボタンは破壊音を再生！

## 破壊の仕組み

はじめにマイクからグラスを弾いた音をFFTして，周波数のピークをとり固有周波数として求めます．

次に，その固有周波数のサイン波を生成し，再生します．

結果，グラスは共振し激しく揺れて限界を超えた際破壊されてしまいます．

## Tech

- JS framework: Vue.js
- Design Framework : Vuetify
- オーディオ/描画周り: p5.js

## 構築

### セットアップ

```
npm install
```

#### 開発用ローカルサーバ

```
npm run serve
```

#### 本番用ビルド

```
npm run build
```

#### Lint

```
npm run lint
```

#### Customize configuration

See [Configuration Reference](https://cli.vuejs.org/config/).
