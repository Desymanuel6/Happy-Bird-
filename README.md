<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>Flappy Clone</title>
<style>
    * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }
  
  body {
    background: linear-gradient(to bottom, #70c5ce 0%, #70c5ce 70%, #ded895 70%);
    font-family: 'Arial', sans-serif;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    overflow: hidden;
    touch-action: manipulation;
  }

  canvas {
    border: 3px solid #000;
    background: #70c5ce;
    max-width: 100%;
    max-height: 100%;
  }

  #score {
    position: absolute;
    top: 20px;
    font-size: 48px;
    color: white;
    text-shadow: 3px 3px 0 #000;
    font-weight: bold;
  }

  #startScreen, #gameOverScreen {
    position: absolute;
    text-align: center;
    color: white;
    text-shadow: 2px 2px 0 #000;
  }

  #gameOverScreen {
    display: none;
  }

  button {
    margin-top: 20px;
    padding: 15px 30px;
    font-size: 20px;
    background: #f8c537;
    border: 3px solid #000;
    border-radius: 10px;
    cursor: pointer;
    font-weight: bold;
  }

  button:active {
    transform: scale(0.95);
  }

  h1 {
    font-size: 36px;
    margin-bottom: 10px;
  }
</style>
</head>
<body>
<div id="score">0</div>
<div id="startScreen">
  <h1>FLAPPY CLONE</h1>
  <p>Tap or Space to flap</p>
  <button onclick="startGame()">START</button>
</div>
<div id="gameOverScreen">
  <h1>GAME OVER</h1>
  <p id="finalScore">Score: 0</p>
  <button onclick="restartGame()">RESTART</button>
</div>
<canvas id="game" width="320" height="480"></canvas>

<script>
const canvas = document.getElementById('game');
const ctx = canvas.getContext('2d');
const scoreEl = document.getElementById('score');
const startScreen = document.getElementById('startScreen');
const gameOverScreen = document.getElementById('gameOverScreen');
const finalScoreEl = document.getElementById('finalScore');

let bird, pipes, score, gameRunning, gravity, velocity;

function initGame() {
  bird = { x: 50, y: 150, width: 30, height: 30 };
  pipes = [];
  score = 0;
  gameRunning = false;
  gravity = 0.5;
  velocity = 0;
  
  scoreEl.textContent = '0';
  generatePipe();
}

function generatePipe() {
  const gap = 120;
  const minHeight = 50;
  const maxHeight = canvas.height - gap - minHeight - 80;
  const pipeHeight = Math.floor(Math.random() * (maxHeight - minHeight)) + minHeight;
  
  pipes.push({
    x: canvas.width,
    topHeight: pipeHeight,
    bottomY: pipeHeight + gap,
    bottomHeight: canvas.height - (pipeHeight + gap) - 80,
    scored: false
  });
}

function startGame() {
  startScreen.style.display = 'none';
  gameRunning = true;
  gameLoop();
}

function restartGame() {
  gameOverScreen.style.display = 'none';
  initGame();
  startGame();
}

function gameLoop() {
  if (!gameRunning) return;

  update();
  draw();
  
  requestAnimationFrame(gameLoop);
}

function update() {
  // Update bird
  velocity += gravity;
  bird.y += velocity;

  // Update pipes
  pipes.forEach(pipe => {
    pipe.x -= 2;
    
    // Score when pipe passes bird
    if (!pipe.scored && pipe.x + 52 < bird.x) {
      score++;
      scoreEl.textContent = score;
      pipe.scored = true;
    }
  });

  // Remove off-screen pipes
  pipes = pipes.filter(pipe => pipe.x + 52 > 0);

  // Add new pipe
  if (pipes.length === 0 || pipes[pipes.length - 1].x < canvas.width - 200) {
    generatePipe();
  }

  // Check collisions
  if (bird.y + bird.height > canvas.height - 80 || bird.y < 0) {
    endGame();
  }

  pipes.forEach(pipe => {
    if (bird.x + bird.width > pipe.x && bird.x < pipe.x + 52) {
      if (bird.y < pipe.topHeight || bird.y + bird.height > pipe.bottomY) {
        endGame();
      }
    }
  });
}

function draw() {
  // Clear
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  
  // Draw pipes
  ctx.fillStyle = '#5a9a4a';
  ctx.strokeStyle = '#000';
  ctx.lineWidth = 3;
  
  pipes.forEach(pipe => {
    // Top pipe
    ctx.fillRect(pipe.x, 0, 52, pipe.topHeight);
    ctx.strokeRect(pipe.x, 0, 52, pipe.topHeight);
    
    // Bottom pipe
    ctx.fillRect(pipe.x, pipe.bottomY, 52, pipe.bottomHeight);
    ctx.strokeRect(pipe.x, pipe.bottomY, 52, pipe.bottomHeight);
  });

  // Draw ground
  ctx.fillStyle = '#ded895';
  ctx.fillRect(0, canvas.height - 80, canvas.width, 80);
  ctx.strokeStyle = '#000';
  ctx.lineWidth = 3;
  ctx.beginPath();
  ctx.moveTo(0, canvas.height - 80);
  ctx.lineTo(canvas.width, canvas.height - 80);
  ctx.stroke();

  // Draw bird
  ctx.fillStyle = '#f8c537';
  ctx.fillRect(bird.x, bird.y, bird.width, bird.height);
  ctx.strokeStyle = '#000';
  ctx.strokeRect(bird.x, bird.y, bird.width, bird.height);
  
  // Bird eye
  ctx.fillStyle = '#000';
  ctx.beginPath();
  ctx.arc(bird.x + 20, bird.y + 10, 3, 0, Math.PI * 2);
  ctx.fill();
}

function flap() {
  if (!gameRunning) return;
  velocity = -8;
}

function endGame() {
  gameRunning = false;
  finalScoreEl.textContent = `Score: ${score}`;
  gameOverScreen.style.display = 'block';
}

// Controls
document.addEventListener('keydown', e => {
  if (e.code === 'Space') {
    e.preventDefault();
    if (!gameRunning && startScreen.style.display !== 'none') {
      startGame();
    } else {
      flap();
    }
  }
});

canvas.addEventListener('click', flap);
canvas.addEventListener('touchstart', e => {
  e.preventDefault();
  flap();
});

// Init
initGame();
</script>
</body>
</html>
