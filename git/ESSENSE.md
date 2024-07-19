# Gitの本質

例えば、長方形を描くような次のコードがあったとします。

```js
function drawRect(ctx, l, t, w, h, r, g, b) {
    ctx.fillStyle = `rgb(${r}, ${g}, ${b})`
    ctx.fillRect(l, t, w, h)
}
function drawBlock(ctx) {
    drawRect(ctx, 100, 120, 320, 240, 255, 128, 0)
}
```

ここで、二人の開発者AさんとBさんが非同期的に次の変更を考えたとします。

- Aさん「`drawRect`の座標と色は[0, 1]で与えたいなあ」
- Bさん「ブロックちょっとずらしたいなあ」

この変更を、Aさんから順に適応してみましょう。
Aさんの変更を適応した後、コードは次のようになります。

```js
function drawRect(ctx, l, t, w, h, r, g, b) {
    ctx.fillStyle = `rgb(${r*255}, ${g*255}, ${b*255})`
    ctx.fillRect(l*WIDTH, t*HEIGHT, w*WIDTH, h*HEIGHT)
}
function drawBlock(ctx) {
    drawRect(ctx, 0.16, 0.25, 0.5, 0.5, 1.0, 0.5, 0.0)
}
```

さらに、Bさんの変更を適応した後、コードは次のようになります。

```js
function drawRect(ctx, l, t, w, h, r, g, b) {
    ctx.fillStyle = `rgb(${r*255}, ${g*255}, ${b*255})`
    ctx.fillRect(l*WIDTH, t*HEIGHT, w*WIDTH, h*HEIGHT)
}
function drawBlock(ctx) {
    drawRect(ctx, 120, 140, 320, 240, 255, 128, 0)
}
```

結果のコードを実行すると、長方形が遥か彼方に吹っ飛んでしまいました。

なぜこのようになってしまったかと言うと、**コンフリクト**を無視してしまったからです。
コンフリクトとは、変更箇所が干渉することです。
今回の例では、6行目でコンフリクトが起こりました。
二人共が6行目を変更していましたが、Aさんの変更をBさんの変更で上書きしてしまったのです。

このような事態を避けるために開発されたツールが**バージョン管理ソフト**であり、そのうちなんやかんやあってLinus Torvaldsが作ったものがGitです。
言い換えると、Gitの本質とは、**誰が・いつ・どこを変更したのかを記録することで変更を適切に統合するためのもの**なのです。
