# nibasl
小恐龍遊戲
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <title>Professional Dino Mod - Dev Edition</title>
    <style>
        body { margin: 0; overflow: hidden; background: #f7f7f7; font-family: 'Courier New', Courier, monospace; }
        canvas { display: block; background: #fff; margin: 20px auto; border-bottom: 2px solid #535353; }
        .info { text-align: center; color: #535353; }
    </style>
</head>
<body>
    <div class="info">
        <h1>Dino Mod: Engineer Version</h1>
        <p>500pts: GOLD | 1000pts: FLAME | Speed: RANDOMIZED</p>
        <h2 id="scoreDisplay">Score: 0</h2>
    </div>
    <canvas id="gameCanvas"></canvas>

<script>
/** @type {HTMLCanvasElement} */
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const scoreEl = document.getElementById('scoreDisplay');

canvas.width = 800;
canvas.height = 200;

// 遊戲參數
let score = 0;
let gameSpeed = 6;
let isGameOver = false;
let frameCount = 0;

// 隨機速度控制：每 120 幀變更一次目標速度
let targetSpeed = 6;
const lerp = (start, end, amt) => (1 - amt) * start + amt * end;

// 恐龍物件
const dino = {
    x: 50,
    y: 150,
    width: 40,
    height: 40,
    dy: 0,
    jumpForce: 12,
    gravity: 0.6,
    grounded: false,
    color: '#535353',
    state: 'NORMAL',
    
    update() {
        // 重力邏輯
        if (!this.grounded) {
            this.dy += this.gravity;
            this.y += this.dy;
        }

        if (this.y + this.height >= 190) {
            this.y = 190 - this.height;
            this.dy = 0;
            this.grounded = true;
        }

        // 狀態機：根據分數改變型態
        if (score >= 1000) {
            this.state = 'FLAME';
            this.color = `rgb(255, ${Math.random() * 100}, 0)`; // 閃爍火焰色
        } else if (score >= 500) {
            this.state = 'GOLDEN';
            this.color = '#FFD700'; // 金色
        } else {
            this.state = 'NORMAL';
            this.color = '#535353';
        }
    },

    draw() {
        ctx.fillStyle = this.color;
        // 如果是火焰狀態，增加發光特效
        if (this.state === 'FLAME') {
            ctx.shadowBlur = 15;
            ctx.shadowColor = 'red';
        } else if (this.state === 'GOLDEN') {
            ctx.shadowBlur = 10;
            ctx.shadowColor = 'orange';
        } else {
            ctx.shadowBlur = 0;
        }
        ctx.fillRect(this.x, this.y, this.width, this.height);
        ctx.shadowBlur = 0; // 重設，避免影響其他元件
    }
};

// 障礙物
const obstacles = [];
function spawnObstacle() {
    obstacles.push({
        x: canvas.width,
        y: 160,
        width: 20 + Math.random() * 30,
        height: 30
    });
}

// 核心循環
function animate() {
    if (isGameOver) return;
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    frameCount++;
    score += 0.1;
    scoreEl.innerText = `Score: ${Math.floor(score)}`;

    // --- 隨機速度邏輯 ---
    if (frameCount % 120 === 0) {
        targetSpeed = 5 + Math.random() * 12; // 隨機目標速度 5~17
    }
    gameSpeed = lerp(gameSpeed, targetSpeed, 0.05); // 平滑過渡

    // 恐龍更新
    dino.update();
    dino.draw();

    // 障礙物邏輯
    if (frameCount % Math.floor(1000 / gameSpeed) === 0) spawnObstacle();

    for (let i = obstacles.length - 1; i >= 0; i--) {
        let o = obstacles[i];
        o.x -= gameSpeed;

        ctx.fillStyle = '#ff4d4d';
        ctx.fillRect(o.x, o.y, o.width, o.height);

        // 碰撞檢測 (AABB)
        if (dino.x < o.x + o.width &&
            dino.x + dino.width > o.x &&
            dino.y < o.y + o.height &&
            dino.y + dino.height > o.y) {
            isGameOver = true;
            alert(`Game Over! Score: ${Math.floor(score)}`);
            location.reload();
        }

        if (o.x + o.width < 0) obstacles.splice(i, 1);
    }

    requestAnimationFrame(animate);
}

// 控制
window.addEventListener('keydown', (e) => {
    if (e.code === 'Space' && dino.grounded) {
        dino.dy = -dino.jumpForce;
        dino.grounded = false;
    }
});

animate();
</script>
</body>
</html>
