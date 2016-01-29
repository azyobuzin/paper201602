# 円盤を描画する

## ひとつの円盤を描画する
難しいことは小さくして考えることは大事です。というわけで最初は1つだけからやっていきましょう。

まず円盤はこんな感じにします。

![円盤のサンプル](images/disk.png)

これは楕円と四角形の組み合わせでつくることができます。

![円盤の作り方](images/drawingdisk.gif)

ここでパスの「追加」と言ってきたのが効いてきます。
底面の楕円と四角形を合せたパスに対して塗りつぶしを行うことで、うまいこと必要な部分だけを塗りつぶせていますね。

それでは、円盤を描画する関数を作っていきましょう。と、その前に円盤についての設定をゲーム設定変数のところに追加しておきましょう。

```javascript
var diskWidth = 135; // 1番大きい円盤の幅
var diskHeight = 20; // 円盤の四角形の高さ
var diskPers = 19; // 円盤の楕円の高さ
var diskColorLight = "#FFFF00"; // 円盤の側面の色
var diskColorDark = "#FFD700"; // 円盤の上面の色
var diskStrokeColor = "#444444" // 円盤の線の色
```

土台よりひと回り小さくしました。それでは`drawBase`関数を作っていきましょう。次のコードを`draw`関数の後に追加します。

```javascript
// n: 円盤番号（大きさ）
// x, y: 中心座標
function drawDisk(n, x, y) {
    var w = diskWidth;
    var ry = diskPers / 2;
    
    // ここから先を説明していきます
}
```

最初は一番大きい円盤だけを描画するので、今は`n`については気にしないでください。
`w`と`ry`はそれぞれ四角形の幅と楕円の縦方向の半径です。これは後で`n`に対する処理を加えるときに変更するので、便宜上変数にしておきました。

まずは線の色を指定しておきましょう。線の色は`ctx.strokeStyle`で指定できます。使い方は`fillStyle`と同じです。
```javascript
ctx.strokeStyle = diskStrokeColor;
```

ではさっきのアニメーションで示した通り、底面・側面・上面の順番で描いていきましょう。まず底面の楕円のパスを追加します。
中心の座標は四角形の下の辺の中心になるので、X座標が`x`、Y座標が`y - (diskHeight / 2)`になりますね（`+`,`-`より`*`,`/`の方が優先されるので括弧はなくても良い）。

```javascript
ctx.beginPath();
ellipse(x, y + (diskHeight / 2), w / 2, ry);
```

ここにさらに側面の四角形のパスを加えます。`rect`の引数に与える座標は四角形の左上の座標なので、
X座標は`x - (w / 2)`、Y座標は`y - (diskHeight / 2)`です。

```javascript
ctx.rect(x - (w / 2), y - (diskHeight / 2), w, diskHeight);
```

そして線を引いて色を塗れば底面と側面のできあがりです。なお、色を塗ってから線を引くと、見えてほしくない線が描かれてしまって見栄えが悪くなるので順番に気をつけましょう。

```javascript
ctx.stroke();
ctx.fillStyle = diskColorDark;
ctx.fill();
```

次に上面を描きます。今度は中心を四角形の上の辺の中心にすれば良いですね。

```javascript
ctx.beginPath();
ellipse(x, y - (diskHeight / 2), w / 2, ry);
ctx.stroke();
ctx.fillStyle = diskColorLight;
ctx.fill();
```

これで`drawDisk`は完成です。`draw`関数から呼び出してみましょう。
前回のループの後に`drawDisk`の呼び出しを追加します。
`n`は一番大きい円盤 = 5番目に大きい円盤ということで`5`にしておきましょう。座標は適当に。

```javascript
drawDisk(5, 100, 100);
```

このようになればOKです。

![空飛ぶ円盤](images/drawdiskie.png)

## 円盤を管理する

