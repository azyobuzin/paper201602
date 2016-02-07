# さぁ仕上げだ！

## そこに置けるかを判定する
次は`mouseup`イベントでマウスボタンを離したときの処理を書いていきましょう。

```javascript
canvas.addEventListener("mouseup", function(e) {
    if (!moving) { return; } // 移動中でないなら何もしない
    
    // mousedown と同じ
    var rodIndex = Math.floor(e.clientX / columnWidth);
    var disks = rods[rodIndex];
    
}, false);
```

`if`の行に見慣れないものがありますが、`!`は否定を意味し、「`moving`が`null`ならば」という条件になっています。
`return`はこの関数から抜け出すという意味があります。
つまり「`moving`が`null`ならば、この先の処理は実行せずイベントハンドラをおしまいにする」というプログラムです。

さて、次に進む前に`if`文の使い方にもう一歩踏み込んでおきましょう。`if`には条件を満たさないときに実行される`else`というブロックも指定することができます。

```javascript
if (条件) {
    alert("条件を満たしました。");
} else {
    alert("条件を満たしませんでした。");
}
```

このように書くことで条件によってどちらか片方のブロックだけを実行するプログラムを書くことができます。

ところで、後で説明すると言った`true`と`false`ですが、これは条件を満たすかどうかをチェックした結果です。
数Ⅰでやった（まだの人もいるかもしれませんが）、真(`true`)と偽(`false`)です。
たとえば`1 == 1`（`==`は「等しい」を表します）は`true`ですし、`1 > 2`は`false`です。
`if`文や`for`文は、条件部分のプログラムを実行して`true`ならこうする、というような動き方をします。

（`moving`に関しては特別な事例です。あれは暗黙的に`moving != null`（`!=`は「等しくない」を表します）として処理されています。）

つまり次のプログラムは必ず「真」と表示されます。

```javascript
if (true) {
    alert("真");
} else {
    alert("偽");
}
```

次にこれらを踏まえて「かつ」と「または」を指示する方法です。

「かつ」は`&&`を条件の間に入れます。
```javascript
条件1 && 条件2
```
これは次のような順番で実行されます。

1. 条件1を実行して`true`ならば2へ、`false`ならば`false`を返して終了
2. 条件2を実行して結果を返して終了

「または」は`||`を条件の間に入れます。
```javascript
条件1 || 条件2
```
これは次のような順番で実行されます。

1. 条件1を実行して`true`ならば`true`を返して終了、`false`ならば2へ
2. 条件2を実行して結果を返して終了

つまり条件1が`false`だった時点で条件2はチェックされません。この特性は後で役に立ってきます。

それでは、取得した`rodIndex`に円盤が置けるかを判断する条件を考えていきましょう。

1. 元々いたタワー(`moving.rodIndex`)と同じタワーに置こうとした場合は、`cancelMove`した結果と同じになるので、置けないと判断することにします（あとで手数を数える関係もあるし）。
2. 置こうとしているタワーに円盤が存在していなければ、どんな円盤でも置くことができます。
3. 移動中の円盤が置こうとしているタワーの一番上にある円盤より小さければ置くことができます。

これらをまとめると、このような条件のプログラムができあがります。

```javascript
rodIndex != moving.rodIndex && (disks.length == 0 || moving.n < disks[disks.length - 1])
```

まずは「かつ」と「または」に注目して読み解いていきましょう。
まとまりとして解釈させるために括弧を使ってしまったので難しくなってしまいましたが、
```
条件1 = rodIndex != moving.rodIndex
条件2 = disks.length == 0 || moving.n < disks[disks.length - 1]
条件 = 条件1 && 条件2
```
というふうに分解すればわかりやすいのではないでしょうか。

次にプログラムの実行順を考えていきましょう。
最初に`rodIndex != moving.rodIndex`をチェックしています。
ここでもし同じタワーに置こうとしているならば`false`が確定しますのでここで条件のチェックは終了です。
そうでなければ次に`disks.length == 0`がチェックされます。
ここで`disks`の要素数が0ならば`true`が確定するのでチェックは終了です。
最後に`moving.n < disks[disks.length - 1]`で円盤の大きさをチェックしています。
ここで重要なのは「`disks`の要素数が0ならばチェックが終了する」ことです。
もし`disks[disks.length - 1]`つまり`disks[-1]`を実行してしまったらエラーとなりプログラムがストップしてしまうからです。

というわけで少し複雑な条件でしたが理解できましたか？理解できなくても「かつ」と「または」で条件を組み合わせているんだなというところまでわかれば、進んでOKです。

それではこの条件を`if`文にぶち込みましょう。条件を満たさなかった場合は`cancelMove`を呼び出して元の状態に戻すことにします。

```javascript
if (rodIndex != moving.rodIndex && (disks.length == 0 || moving.n < disks[disks.length - 1])) {
    
} else {
    cancelMove();
}
```

条件が`true`のときには、移動先のタワーに円盤を追加して、`moving`を`null`にすれば良いですね。

```javascript
disks.push(moving.n);
moving = null;
```


`if`文の後に`draw();`を呼び出せば、これでマウスで自由に円盤を動かし置けるようになりました。

## ゲームの終了を判定する
このままではいつまで経ってもゲームオーバーがないので、すべての円盤を右側に持って行くことができたら終了としましょう。
条件は`rods[2].length == diskCount`で表せますね。早速さっきの`if`の`true`のブロックに追加しましょう。

```javascript
if (rods[2].length == diskCount) {
    draw(); // 移動が完了した円盤を描画しておく
    alert("クリア！");
    init();
}
```

これで「クリア！」と表示されたあと、最初の状態に戻るようになりました。おお、ちゃんとゲームになった。

## 手数を数える
ついでなので何手でクリアできたか数えるようにしましょう。ゲーム設定変数に`moveCount`を追加します。
```javascript
var moveCount = 0;
```

さらに`init`にも`moveCount = 0`を加えておきましょう。

そして終了判定の`if`の前で`moveCount`を1増やし(`moveCount++;`)て、`"クリア！"`を`moveCount + "手でクリア！"`に書き換えれば完成です。

お疲れさまでした！！！！！！！！

疲れましたね。僕も書いていて疲れました。ええ。

## 完成した hanoi.html
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
        var moveCount = 0;

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

            moveCount = 0;

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

        canvas.addEventListener("mouseup", function(e) {
            if (!moving) { return; } // 移動中でないなら何もしない
            
            // mousedown と同じ
            var rodIndex = Math.floor(e.clientX / columnWidth);
            var disks = rods[rodIndex];
            
            if (rodIndex != moving.rodIndex && (disks.length == 0 || moving.n < disks[disks.length - 1])) {
                disks.push(moving.n);
                moving = null;
                moveCount++;
                if (rods[2].length == diskCount) {
                    draw(); // 移動が完了した円盤を描画しておく
                    alert(moveCount + "手でクリア！");
                    init();
                }
            } else {
                cancelMove();
            }
            
            draw();
        }, false);

        init();
    </script>
</body>
</html>
```
