<!DOCTYPE html<
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Knife Hit Game with Rotating Texture</title>
  <style>
    body {
      margin: 0;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      background: linear-gradient(to bottom, #1e1e2f, #0f0f1f);
      font-family: Arial, sans-serif;
    }

    canvas {
      display: block;
      background: radial-gradient(circle, #2d2d44, #1e1e2f);
      box-shadow: 0 0 10px #000;
    }

    .game-over {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      text-align: center;
      display: none;
      color: white;
    }

    .game-over h1 {
      font-size: 3em;
      margin-bottom: 20px;
    }

    .game-over button {
      padding: 10px 20px;
      font-size: 1.5em;
      background: #ff4757;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
    }

    .game-over button:hover {
      background: #ff6b81;
    }
  </style>
</head>
<body>
  <canvas id="gameCanvas" width="400" height="600"></canvas>

  <div class="game-over" id="gameOverScreen">
    <h1>Game Over</h1>
    <h2>High Score: <span id="highScore">0</span></h2>
    <button onclick="restartGame()">Restart</button>
  </div>

  <script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const gameOverScreen = document.getElementById('gameOverScreen');
    const highScoreDisplay = document.getElementById('highScore');

    const target = {
      x: canvas.width / 2,
      y: 200,
      radius: 50,
      angle: 0,
      speed: 0.05,
      knives: []
    };

    const player = {
      knives: [],
      score: 0,
      highScore: 0,
      stage: 1
    };

    let isGameOver = false;

    function drawTarget() {
      // วาดวงกลมเป้าหมาย
      ctx.save();
      ctx.translate(target.x, target.y);
      ctx.rotate(target.angle);

      // พื้นหลังเป้าหมาย
      ctx.fillStyle = '#f39c12';
      ctx.beginPath();
      ctx.arc(0, 0, target.radius, 0, Math.PI * 2);
      ctx.fill();

      // วาดลวดลายหมุน
      ctx.strokeStyle = '#e67e22';
      ctx.lineWidth = 5;
      for (let i = 0; i < 8; i++) {
        ctx.beginPath();
        ctx.moveTo(0, 0);
        ctx.lineTo(target.radius * Math.cos((Math.PI / 4) * i), target.radius * Math.sin((Math.PI / 4) * i));
        ctx.stroke();
      }

      ctx.restore();

      // วาดมีดที่ติดกับเป้าหมาย
      target.knives.forEach(knife => {
        ctx.save();
        ctx.translate(target.x, target.y);
        ctx.rotate(knife.angle);
        ctx.fillStyle = '#ffffff';
        ctx.fillRect(0, -target.radius - 20, 4, 20);
        ctx.restore();
      });
    }

    function drawKnives() {
      player.knives.forEach(knife => {
        ctx.fillStyle = '#ffffff';
        ctx.fillRect(knife.x - 2, knife.y, 4, 20);
      });
    }

    function drawScore() {
      ctx.fillStyle = '#fff';
      ctx.font = '20px Arial';
      ctx.fillText(`Score: ${player.score}`, 10, 30);
      ctx.fillText(`Stage: ${player.stage}`, canvas.width - 100, 30);
    }

    function update() {
      // หมุนเป้าหมาย
      target.angle += target.speed;

      // อัปเดตมีดของผู้เล่น
      player.knives.forEach(knife => {
        knife.y -= 5;

        // ตรวจสอบว่ามีดชนเป้าหมาย
        if (knife.y <= target.y - target.radius - 20) {
          const relativeAngle = (Math.atan2(target.y - knife.y, target.x - knife.x) - target.angle) % (Math.PI * 2);

          // ตรวจสอบการชนกับมีดอื่น
          const collision = target.knives.some(existingKnife => 
            Math.abs(relativeAngle - existingKnife.angle) < 0.2
          );

          if (collision) {
            endGame();
            return;
          }

          // ติดตั้งมีดบนเป้าหมาย
          target.knives.push({ angle: relativeAngle });
          player.knives = player.knives.filter(k => k !== knife);
          player.score++;
          checkNextStage();
        }
      });

      // ลบมีดที่หลุดออกนอกจอ
      player.knives = player.knives.filter(knife => knife.y > -20);
    }

    function checkNextStage() {
      if (target.knives.length === player.stage + 2) {
        player.stage++;
        target.knives = [];
        target.speed += 0.02; // เพิ่มความเร็วในการหมุน
        target.radius = Math.max(30, target.radius - 5); // ลดขนาดเป้าหมาย
      }
    }

    function endGame() {
      isGameOver = true;
      gameOverScreen.style.display = 'block';
      player.highScore = Math.max(player.highScore, player.score);
      highScoreDisplay.textContent = player.highScore;
    }

    function restartGame() {
      isGameOver = false;
      player.score = 0;
      player.stage = 1;
      target.knives = [];
      target.speed = 0.05;
      target.radius = 50;
      gameOverScreen.style.display = 'none';
      gameLoop();
    }

    canvas.addEventListener('click', () => {
      if (isGameOver) return;
      player.knives.push({ x: target.x, y: canvas.height - 50 });
    });

    function gameLoop() {
      if (isGameOver) return;

      ctx.clearRect(0, 0, canvas.width, canvas.height);
      drawTarget();
      drawKnives();
      drawScore();
      update();

      requestAnimationFrame(gameLoop);
    }

    gameLoop();
  </script>
</body>
</html>
