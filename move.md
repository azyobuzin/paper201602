# 円盤をマウスで動かす

## 複雑なオブジェクトをつくる
最初の方で、`.`は変数の中の変数を取得する、といいました。このようなオブジェクトは自分でつくることができます。
たとえば車の情報についてひとつの変数にまとめてみます。

```javascript
var myCar = {
    name: "プリウス",
    generation: 4,
    grade: "Aプレミアムツーリングセレクション",
    color: "エモーショナルレッド"
};
```

これで`myCar.name`のようにアクセスできるオブジェクトをつくることができました。

これを使って今マウスで動かしている円盤の情報をまとめるオブジェクトをつくっていきます。

```javascript
{
    rodIndex: 0, // 元いたタワーの番号(0, 1, 2)
    n: 1, // 円盤の番号(1, 2, 3, 4, 5)
    x: 0, // 円盤のX座標 = カーソルのX座標
    y: 0 // 円盤のY座標 = カーソルのY座標
}
```

あとで使いますので、そのときに確認してみてください。

## 動かしている円盤の情報を保存する
ゲーム設定変数のところに動かしている円盤の情報を保存する変数を作成します。
初期状態では動かしているものはないので「何もない」ということを表す`null`を入れておきます。

```javascript
var moving = null;
```

最初に移動を中止する関数をつくっておきます。まだ動かしてすらないですが、何回も使うので先に、ね？（筆者が作ったときはあとから関数にまとめましたが）

次の関数を`init`の後に書いてください。

```javascript
function cancelMove() {
    if (moving) {
        rods[moving.rodIndex].push(moving.n);
        moving = null;
    }
}
```

`if (moving)`は`moving`が`null`でないときにブロックを実行します。

## マウスイベントを取得する
まずは円盤を動かすところまでやりましょう。
マウスの操作はイベントという形で取得することができます。
これはマウスの操作が発生したときに、この関数を呼んでね、と登録しておくと、実際にマウス操作時に関数が呼ばれるという仕組みです。

さっそく使っていきましょう。
まずは`<canvas>`の`mousedown`イベント、つまりマウスのボタンを押し始めたときに発生するイベントに関数（イベントハンドラー）を登録します。
`cancelMove`関数の後に次のプログラムを入力してください。

```javascript
canvas.addEventListener("mousedown", function(e) {
    alert("X:" + e.clientX + ", Y:" + e.clientY);
}, false);
```

`addEventListener`関数でイベントハンドラーを登録することができます。
そして驚いたかもしれませんが、引数に関数を与えています。
JavaScriptでは関数をオブジェクトとして扱うことができるので、このような書き方ができます。
`false`についてはイベントの取得方法の指定ですが、たぶん`true`でもこの場合は挙動は変わらないはずです（詳しくは調べてみてください）。

例えば次の2つの書き方はほぼ同じ意味を持ちます（厳密には違うので調べて使い分けてみてください）。

```javascript
function f(x) {
    alert(x);
}

var f = function(x) {
    alert(x);
};
```

脱線しましたが、これを実行するとクリックした場所の座標が表示されます。

それでは`alert`を消して実際の動作を書いていきましょう。
まずは一応`cancelMove()`を呼び出しておきます。
これは`<canvas>`外でマウスボタンを離してしまうなどイレギュラーも想定する必要があるためです。

次にどのタワーのところでマウスボタンを押したのかを判定して`rodIndex`変数にいれます。
`e.clientX / columnWidth`の結果の小数点以下を切り捨てると、ちょうど左から0, 1, 2となりますね。
小数点以下の切り捨てには`Math.floor`を使います。
ついでにそのタワーの円盤配列を取得しておきましょう。

```javascript
var rodIndex = Math.floor(e.clientX / columnWidth);
var disks = rods[rodIndex];
```

次にそのタワーに動かすことができる円盤があるかを判定します。
これは`配列.length`が0より大きいことを確認すれば良いです。

そして動かすことができるならば、`moving`変数に最初に紹介したオブジェクトを代入します。
`配列.pop()`は、配列の最後の要素を削除して、その削除された要素を返す関数です。これを使って`n`を取得しましょう。

```javascript
if (disks.length > 0) {
    moving = {
        rodIndex: rodIndex,
        n: disks.pop(),
        x: e.clientX,
        y: e.clientY
    };
}
```

最後に`draw()`を呼び出して表示を更新すれば完成です。

さらに`mousemove`イベントを取得して、`moving.x`と`y`を更新するようにしましょう。
難しいことはないので、これは想像で書いちゃってください（一応正解は書いておきます）。

```javascript
canvas.addEventListener("mousemove", function(e) {
    if (moving) {
        moving.x = e.clientX;
        moving.y = e.clientY;
        draw();
    }
}, false);
```

この状態で動かしても、クリックした瞬間円盤が消えるだけなので、`draw`関数に`moving`の情報を基に円盤を描画する処理を追加しましょう。
`draw`の最後に`moving`が`null`でないなら円盤を描画という処理追加します。

```javascript
if (moving) {
    drawDisk(moving.n, moving.x, moving.y);
}
```

これで円盤を自由自在に動かせるようになりました。マウスボタンから手を離してもずっとついてきますが。

## ここまでの hanoi.html
```html
<!DOCTYPE html>
<html>
<head>
    <title>ハノイの塔</title>
</head>
<body>
    <canvas id="screen" width="600" height="200"></canvas>

    <script>
        var canvas = document.getElementById("screen");
        var width = canvas.width;
        var height = canvas.height;
        var ctx = canvas.getContext("2d");
        
        var columnWidth = width / 3; // 1列あたりの幅 = 200
        var baseWidth = 150; // 土台の幅
        var baseHeight = 20; // 土台の高さ
        var baseColor = "#CC6633" // 土台の色
        var diskWidth = 135; // 1番大きい円盤の幅
        var diskHeight = 20; // 円盤の四角形の高さ
        var diskPers = 19; // 円盤の楕円の高さ
        var diskColorLight = "#FFFF00"; // 円盤の側面の色
        var diskColorDark = "#FFD700"; // 円盤の上面の色
        var diskStrokeColor = "#444444" // 円盤の線の色
        var diskCount = 5;
        var rods = [];
        var deltaDiskWidth = 15; // 円盤の幅の減少量
        var moving = null;

        // cx, cy: 中心座標
        // rx: X軸方向の半径, ry: Y軸方向の半径
        function ellipse(cx, cy, rx, ry)
        {
            ctx.save();
            ctx.scale(1, ry / rx);
            ctx.arc(cx, cy * rx / ry, rx, 0, 2 * Math.PI, false);
            ctx.restore();
        }

        function draw() {
            ctx.clearRect(0, 0, width, height);

            for (var i = 0; i < 3; i++) {
                ctx.beginPath();
                var x = columnWidth * i + columnWidth / 2;
                var y = height - baseHeight / 2;
                ellipse(x, y, baseWidth / 2, baseHeight / 2);
                ctx.fillStyle = baseColor;
                ctx.fill();
            }

            for (var i = 0; i < 3; i++) {
                var disks = rods[i];
                var x = columnWidth * i + columnWidth / 2;

                for (var j = 0; j < disks.length; j++) {
                    var n = disks[j];
                    var y = height - 23 - (j * diskHeight);
                    drawDisk(n, x, y);
                }
            }

            if (moving) {
                drawDisk(moving.n, moving.x, moving.y);
            }
        }

        // n: 円盤番号（大きさ）
        // x, y: 中心座標
        function drawDisk(n, x, y) {
            var w = diskWidth - ((diskCount - n) * deltaDiskWidth);
            var ry = diskPers / 2 * (w / diskWidth);
            ctx.strokeStyle = diskStrokeColor;

            ctx.beginPath();
            ellipse(x, y + (diskHeight / 2), w / 2, ry);
            ctx.rect(x - (w / 2), y - (diskHeight / 2), w, diskHeight);
            ctx.stroke();
            ctx.fillStyle = diskColorDark;
            ctx.fill();

            ctx.beginPath();
            ellipse(x, y - (diskHeight / 2), w / 2, ry);
            ctx.stroke();
            ctx.fillStyle = diskColorLight;
            ctx.fill();
        }

        function init() {
            rods = [ [], [], [] ];

            for (var i = diskCount; i >= 1; i--) {
                rods[0].push(i);
            }

            draw();
        }

        function cancelMove() {
            if (moving) {
                rods[moving.rodIndex].push(moving.n);
                moving = null;
            }
        }

        canvas.addEventListener("mousedown", function(e) {
            cancelMove();

            var rodIndex = Math.floor(e.clientX / columnWidth);
            var disks = rods[rodIndex];

            if (disks.length > 0) {
                moving = {
                    rodIndex: rodIndex,
                    n: disks.pop(),
                    x: e.clientX,
                    y: e.clientY
                };
            }

            draw();
        });

        canvas.addEventListener("mousemove", function(e) {
            if (moving) {
                moving.x = e.clientX;
                moving.y = e.clientY;
                draw();
            }
        }, false);

        init();
    </script>
</body>
</html>
```
