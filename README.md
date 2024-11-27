<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>二人対戦ゲーム</title>
    <style>
        canvas {
            background: white;
            display: block;
            margin: auto;
            border: 1px solid black;
        }
    </style>
</head>
<body>
<canvas id="gameCanvas" width="800" height="600"></canvas>

<script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");

    // プレイヤーの設定
    const players = [
        { 
            x: 100, 
            y: 300, 
            width: 40, 
            height: 40, 
            color: "blue", 
            speed: 5, 
            bullets: [], 
            health: 100, 
            keys: { up: "w", down: "s", left: "a", right: "d", shoot: "e" }, 
            direction: "right", 
            bulletSpeed: 7,   // プレイヤー1の弾の速度
            shootInterval: 500,  // プレイヤー1の発射間隔（ミリ秒）
            lastShot: 0 
        },
        { 
            x: 700, 
            y: 300, 
            width: 40, 
            height: 40, 
            color: "red", 
            speed: 5, 
            bullets: [], 
            health: 100, 
            keys: { up: "ArrowUp", down: "ArrowDown", left: "ArrowLeft", right: "ArrowRight", shoot: "l" }, 
            direction: "left", 
            bulletSpeed: 7,   // プレイヤー2の弾の速度
            shootInterval: 500,  // プレイヤー2の発射間隔（ミリ秒）
            lastShot: 0 
        }
    ];

    const bulletSize = 10;
    const bulletDamage = 10;
    const keys = {};
    let winner = null; // 勝者を記録

    // キーイベントのリスナー
    document.addEventListener("keydown", (e) => { keys[e.key] = true; });
    document.addEventListener("keyup", (e) => { keys[e.key] = false; });

    // プレイヤーを描画
    function drawPlayers() {
        players.forEach(player => {
            ctx.fillStyle = player.color;
            ctx.fillRect(player.x, player.y, player.width, player.height);
        });
    }

    // 弾を描画
    function drawBullets() {
        players.forEach(player => {
            ctx.fillStyle = player.color;
            player.bullets.forEach((bullet, index) => {
                // 弾を移動
                bullet.x += bullet.dx * player.bulletSpeed; // 各プレイヤーごとの弾の速度
                bullet.y += bullet.dy * player.bulletSpeed;

                // 弾が画面外に出たら削除
                if (bullet.x < 0 || bullet.x > canvas.width || bullet.y < 0 || bullet.y > canvas.height) {
                    player.bullets.splice(index, 1);
                }

                // 弾の描画
                ctx.fillRect(bullet.x, bullet.y, bulletSize, bulletSize);

                // 他のプレイヤーへの衝突判定
                players.forEach(target => {
                    if (target !== player &&
                        bullet.x < target.x + target.width &&
                        bullet.x + bulletSize > target.x &&
                        bullet.y < target.y + target.height &&
                        bullet.y + bulletSize > target.y) {
                        
                        // 弾のダメージ処理
                        target.health -= bulletDamage;
                        player.bullets.splice(index, 1);

                        // 勝敗判定
                        if (target.health <= 0 && winner === null) {
                            winner = player.color; // 勝者を設定
                        }
                    }
                });
            });
        });
    }

    // プレイヤーの動き
    function movePlayers() {
        players.forEach(player => {
            if (keys[player.keys.up] && player.y > 0) player.y -= player.speed;
            if (keys[player.keys.down] && player.y < canvas.height - player.height) player.y += player.speed;
            if (keys[player.keys.left] && player.x > 0) player.x -= player.speed;
            if (keys[player.keys.right] && player.x < canvas.width - player.width) player.x += player.speed;

            // プレイヤーの向き変更
            if (keys[player.keys.left]) {
                player.direction = "left";  // 左に動くとき
            } else if (keys[player.keys.right]) {
                player.direction = "right"; // 右に動くとき
            }

            // 弾の発射
            if (keys[player.keys.shoot]) {
                const currentTime = Date.now();
                if (currentTime - player.lastShot > player.shootInterval) { // プレイヤーごとの発射間隔
                    // 発射方向が左なら、左方向に、右なら右方向に発射
                    const dx = player.direction === "left" ? -1 : 1; 
                    const dy = 0;  // 縦方向は動かさない
                    player.bullets.push({
                        x: player.x + player.width / 2 - bulletSize / 2, // プレイヤーの中央から発射
                        y: player.y + player.height / 2 - bulletSize / 2, // プレイヤーの中央から発射
                        dx: dx,
                        dy: dy
                    });
                    player.lastShot = currentTime;
                }
            }
        });
    }

    // スコアと体力の描画
    function drawHUD() {
        ctx.fillStyle = "black";
        ctx.font = "20px Arial";
        ctx.fillText(`Player 1 Health: ${players[0].health}`, 20, 30);
        ctx.fillText(`Player 2 Health: ${players[1].health}`, canvas.width - 200, 30);

        if (winner) {
            // 勝者が決まったとき、背景に勝者を表示
            ctx.fillStyle = "green";
            ctx.font = "30px Arial";
            ctx.fillText(`${winner}の勝利！`, canvas.width / 2 - 100, canvas.height / 2);
        }
    }

    // ゲーム進行
    function gameLoop() {
        if (winner) {
            return; // 勝者が決まったらゲームを停止
        }
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        movePlayers();
        drawPlayers();
        drawBullets();
        drawHUD();
        requestAnimationFrame(gameLoop);
    }

    gameLoop();
</script>
</body>
</html>
