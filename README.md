Here’s a complete, self-contained Pong game. Save it as `index.html` and open it in a browser. Move the mouse over the game to control the left paddle, or use the **↑/↓** arrow keys.

```html name=index.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Pong</title>
  <style>
    * { box-sizing: border-box; }

    body {
      margin: 0;
      min-height: 100vh;
      display: grid;
      place-items: center;
      background: #10131a;
      color: #f4f4f4;
      font-family: system-ui, sans-serif;
    }

    main {
      width: min(800px, 94vw);
      text-align: center;
    }

    h1 { margin: 0 0 12px; }

    canvas {
      display: block;
      width: 100%;
      aspect-ratio: 8 / 5;
      background: #080a0f;
      border: 2px solid #404755;
      border-radius: 8px;
      cursor: none;
      touch-action: none;
    }

    p { color: #b8c0cc; }
  </style>
</head>
<body>
  <main>
    <h1>Pong</h1>
    <canvas id="game" width="800" height="500" aria-label="Pong game"></canvas>
    <p>Move your mouse over the game or use ↑ / ↓ to control the left paddle.</p>
  </main>

  <script>
    const canvas = document.getElementById("game");
    const ctx = canvas.getContext("2d");

    const WIDTH = canvas.width;
    const HEIGHT = canvas.height;
    const PADDLE_WIDTH = 12;
    const PADDLE_HEIGHT = 90;
    const BALL_RADIUS = 9;
    const PLAYER_SPEED = 420;

    const player = { x: 30, y: HEIGHT / 2 - PADDLE_HEIGHT / 2, score: 0 };
    const computer = { x: WIDTH - 30 - PADDLE_WIDTH, y: HEIGHT / 2 - PADDLE_HEIGHT / 2, score: 0 };
    const ball = { x: WIDTH / 2, y: HEIGHT / 2, vx: 300, vy: 170 };

    const keys = { up: false, down: false };

    function clampPaddle(paddle) {
      paddle.y = Math.max(0, Math.min(HEIGHT - PADDLE_HEIGHT, paddle.y));
    }

    function serve(direction) {
      ball.x = WIDTH / 2;
      ball.y = HEIGHT / 2;
      ball.vx = 300 * direction;
      ball.vy = (Math.random() * 240) - 120;
    }

    canvas.addEventListener("pointermove", (event) => {
      const bounds = canvas.getBoundingClientRect();
      const scaleY = HEIGHT / bounds.height;
      player.y = (event.clientY - bounds.top) * scaleY - PADDLE_HEIGHT / 2;
      clampPaddle(player);
    });

    document.addEventListener("keydown", (event) => {
      if (event.key === "ArrowUp" || event.key === "ArrowDown") {
        event.preventDefault();
      }
      if (event.key === "ArrowUp") keys.up = true;
      if (event.key === "ArrowDown") keys.down = true;
    });

    document.addEventListener("keyup", (event) => {
      if (event.key === "ArrowUp") keys.up = false;
      if (event.key === "ArrowDown") keys.down = false;
    });

    function update(delta) {
      if (keys.up) player.y -= PLAYER_SPEED * delta;
      if (keys.down) player.y += PLAYER_SPEED * delta;
      clampPaddle(player);

      // The computer tracks the ball, but moves more slowly than it.
      const computerCenter = computer.y + PADDLE_HEIGHT / 2;
      const target = ball.y;
      const aiSpeed = 300;
      if (Math.abs(target - computerCenter) > 8) {
        computer.y += Math.sign(target - computerCenter) * aiSpeed * delta;
      }
      clampPaddle(computer);

      ball.x += ball.vx * delta;
      ball.y += ball.vy * delta;

      // Top and bottom wall collisions.
      if (ball.y - BALL_RADIUS <= 0) {
        ball.y = BALL_RADIUS;
        ball.vy = Math.abs(ball.vy);
      } else if (ball.y + BALL_RADIUS >= HEIGHT) {
        ball.y = HEIGHT - BALL_RADIUS;
        ball.vy = -Math.abs(ball.vy);
      }

      // Paddle collisions: only bounce when the ball is moving toward that paddle.
      if (
        ball.vx < 0 &&
        ball.x - BALL_RADIUS <= player.x + PADDLE_WIDTH &&
        ball.x + BALL_RADIUS >= player.x &&
        ball.y + BALL_RADIUS >= player.y &&
        ball.y - BALL_RADIUS <= player.y + PADDLE_HEIGHT
      ) {
        ball.x = player.x + PADDLE_WIDTH + BALL_RADIUS;
        ball.vx = Math.abs(ball.vx) * 1.04;
        ball.vy += ((ball.y - (player.y + PADDLE_HEIGHT / 2)) / (PADDLE_HEIGHT / 2)) * 100;
      }

      if (
        ball.vx > 0 &&
        ball.x + BALL_RADIUS >= computer.x &&
        ball.x - BALL_RADIUS <= computer.x + PADDLE_WIDTH &&
        ball.y + BALL_RADIUS >= computer.y &&
        ball.y - BALL_RADIUS <= computer.y + PADDLE_HEIGHT
      ) {
        ball.x = computer.x - BALL_RADIUS;
        ball.vx = -Math.abs(ball.vx) * 1.04;
        ball.vy += ((ball.y - (computer.y + PADDLE_HEIGHT / 2)) / (PADDLE_HEIGHT / 2)) * 100;
      }

      // Score when the ball gets past a paddle, then serve toward the player who missed.
      if (ball.x + BALL_RADIUS < 0) {
        computer.score++;
        serve(1);
      } else if (ball.x - BALL_RADIUS > WIDTH) {
        player.score++;
        serve(-1);
      }
    }

    function draw() {
      ctx.clearRect(0, 0, WIDTH, HEIGHT);

      // Dashed center line.
      ctx.strokeStyle = "#343a46";
      ctx.setLineDash([10, 12]);
      ctx.beginPath();
      ctx.moveTo(WIDTH / 2, 0);
      ctx.lineTo(WIDTH / 2, HEIGHT);
      ctx.stroke();
      ctx.setLineDash([]);

      // Scoreboard.
      ctx.fillStyle = "#f4f4f4";
      ctx.font = "bold 42px system-ui, sans-serif";
      ctx.textAlign = "center";
      ctx.fillText(player.score, WIDTH / 2 - 60, 58);
      ctx.fillText(computer.score, WIDTH / 2 + 60, 58);

      // Paddles and ball.
      ctx.fillStyle = "#66e3c4";
      ctx.fillRect(player.x, player.y, PADDLE_WIDTH, PADDLE_HEIGHT);
      ctx.fillStyle = "#ffca6a";
      ctx.fillRect(computer.x, computer.y, PADDLE_WIDTH, PADDLE_HEIGHT);

      ctx.fillStyle = "#f4f4f4";
      ctx.beginPath();
      ctx.arc(ball.x, ball.y, BALL_RADIUS, 0, Math.PI * 2);
      ctx.fill();
    }

    let lastTime = 0;
    function frame(time) {
      const delta = Math.min((time - lastTime) / 1000 || 0, 0.03);
      lastTime = time;
      update(delta);
      draw();
      requestAnimationFrame(frame);
    }

    requestAnimationFrame(frame);
  </script>
</body>
</html>
```
