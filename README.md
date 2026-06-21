# leoshubham.github.io

I'll customize this artifact:


flappy_bird.html
Interactive artifact 
Transform any artifact into something uniquely yours by customizing its core elements.

Change the topic - Adapt the content for a different subject
Update the style - Refresh the visuals or overall design
Make it personal - Tailor specifically for your needs
Share your vision - I'll bring it to life
Where would you like to begin?


Claude Fable 5 is currently unavailable.
Learn more(opens in new tab)




Claude is AI and can make mistakes. Please double-check responses.
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Flappy Bird</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }

  body {
    background: #1a1a2e;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    min-height: 100vh;
    font-family: 'Segoe UI', sans-serif;
    overflow: hidden;
  }

  h1 {
    color: #f7c948;
    font-size: 2rem;
    letter-spacing: 3px;
    margin-bottom: 12px;
    text-shadow: 0 0 16px #f7c94888;
  }

  #scoreBoard {
    display: flex;
    gap: 32px;
    margin-bottom: 12px;
  }

  .score-box {
    color: #fff;
    font-size: 1rem;
    text-align: center;
  }

  .score-box span {
    display: block;
    font-size: 1.6rem;
    font-weight: 700;
    color: #f7c948;
  }

  canvas {
    border-radius: 16px;
    border: 3px solid #f7c94844;
    box-shadow: 0 0 40px #f7c94822;
    display: block;
    cursor: pointer;
  }

  #message {
    margin-top: 14px;
    color: #aaa;
    font-size: 0.95rem;
    letter-spacing: 1px;
  }
</style>
</head>
<body>

<h1>🐦 FLAPPY BIRD</h1>
<div id="scoreBoard">
  <div class="score-box">Score<span id="score">0</span></div>
  <div class="score-box">Best<span id="best">0</span></div>
</div>
<canvas id="gameCanvas" width="400" height="550"></canvas>
<div id="message">Click or press SPACE to start</div>

<script>
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const scoreEl = document.getElementById('score');
const bestEl = document.getElementById('best');
const msgEl = document.getElementById('message');

// ── Game config ──────────────────────────────────────────
const W = canvas.width, H = canvas.height;
const GRAVITY    = 0.45;
const FLAP       = -8.5;
const PIPE_W     = 58;
const PIPE_GAP   = 145;
const PIPE_SPEED = 2.8;
const PIPE_FREQ  = 90; // frames between pipes

// ── State ────────────────────────────────────────────────
let bird, pipes, frame, score, best, state;
// state: 'idle' | 'playing' | 'dead'

function init() {
  bird  = { x: 90, y: H / 2, vy: 0, r: 16 };
  pipes = [];
  frame = 0;
  score = 0;
  state = 'idle';
  scoreEl.textContent = 0;
  msgEl.textContent   = 'Click or press SPACE to start';
}

// ── Input ─────────────────────────────────────────────────
function flap() {
  if (state === 'idle')    { state = 'playing'; msgEl.textContent = ''; }
  if (state === 'playing') { bird.vy = FLAP; }
  if (state === 'dead')    { init(); }
}

canvas.addEventListener('click',  flap);
document.addEventListener('keydown', e => {
  if (e.code === 'Space' || e.code === 'ArrowUp') { e.preventDefault(); flap(); }
});
canvas.addEventListener('touchstart', e => { e.preventDefault(); flap(); }, { passive: false });

// ── Pipe helpers ──────────────────────────────────────────
function spawnPipe() {
  const minY = 70, maxY = H - 70 - PIPE_GAP;
  const topH = Math.random() * (maxY - minY) + minY;
  pipes.push({ x: W, topH, scored: false });
}

function rectsOverlap(ax,ay,aw,ah, bx,by,bw,bh) {
  return ax < bx+bw && ax+aw > bx && ay < by+bh && ay+ah > by;
}

function birdHitsPipe(p) {
  const bx = bird.x - bird.r + 4, by = bird.y - bird.r + 4;
  const bs = bird.r * 2 - 8;
  // top pipe
  if (rectsOverlap(bx,by,bs,bs, p.x,0,PIPE_W,p.topH)) return true;
  // bottom pipe
  if (rectsOverlap(bx,by,bs,bs, p.x,p.topH+PIPE_GAP,PIPE_W,H)) return true;
  return false;
}

// ── Drawing ───────────────────────────────────────────────
function drawBg() {
  // Sky gradient
  const sky = ctx.createLinearGradient(0,0,0,H);
  sky.addColorStop(0, '#16213e');
  sky.addColorStop(1, '#0f3460');
  ctx.fillStyle = sky;
  ctx.fillRect(0,0,W,H);

  // Clouds (static for simplicity)
  ctx.fillStyle = 'rgba(255,255,255,0.07)';
  [[60,80,90,28],[200,50,70,20],[310,100,100,30],[130,160,60,18]].forEach(([x,y,w,h])=>{
    ctx.beginPath(); ctx.ellipse(x,y,w,h,0,0,Math.PI*2); ctx.fill();
  });

  // Ground
  const grd = ctx.createLinearGradient(0, H-50, 0, H);
  grd.addColorStop(0, '#2d6a4f');
  grd.addColorStop(1, '#1b4332');
  ctx.fillStyle = grd;
  ctx.fillRect(0, H-50, W, 50);

  // Grass line
  ctx.fillStyle = '#52b788';
  ctx.fillRect(0, H-50, W, 8);
}

function drawPipe(p) {
  const radius = 6;

  // Neon green pipes
  const pipeGrad = ctx.createLinearGradient(p.x, 0, p.x + PIPE_W, 0);
  pipeGrad.addColorStop(0,   '#43a047');
  pipeGrad.addColorStop(0.4, '#66bb6a');
  pipeGrad.addColorStop(1,   '#1b5e20');

  ctx.fillStyle = pipeGrad;

  // Top pipe body
  ctx.beginPath();
  ctx.roundRect(p.x, 0, PIPE_W, p.topH - 12, [0, 0, radius, radius]);
  ctx.fill();

  // Top pipe cap
  ctx.beginPath();
  ctx.roundRect(p.x - 5, p.topH - 22, PIPE_W + 10, 22, [0, 0, radius, radius]);
  ctx.fill();

  // Bottom pipe body
  const btmY = p.topH + PIPE_GAP;
  ctx.beginPath();
  ctx.roundRect(p.x, btmY + 12, PIPE_W, H - btmY - 12, [radius, radius, 0, 0]);
  ctx.fill();

  // Bottom pipe cap
  ctx.beginPath();
  ctx.roundRect(p.x - 5, btmY, PIPE_W + 10, 22, [radius, radius, 0, 0]);
  ctx.fill();

  // Shine
  ctx.fillStyle = 'rgba(255,255,255,0.15)';
  ctx.fillRect(p.x + 6, 0,         8, p.topH - 12);
  ctx.fillRect(p.x + 6, btmY + 12, 8, H);
}

function drawBird() {
  const x = bird.x, y = bird.y, r = bird.r;
  const angle = Math.min(Math.max(bird.vy * 3.5, -30), 90) * Math.PI / 180;

  ctx.save();
  ctx.translate(x, y);
  ctx.rotate(angle);

  // Body
  const bodyGrad = ctx.createRadialGradient(-3,-3,2,0,0,r);
  bodyGrad.addColorStop(0, '#ffe066');
  bodyGrad.addColorStop(1, '#f7c948');
  ctx.fillStyle = bodyGrad;
  ctx.beginPath(); ctx.ellipse(0, 0, r, r*0.88, 0, 0, Math.PI*2); ctx.fill();

  // Wing
  ctx.fillStyle = '#f9a825';
  ctx.beginPath(); ctx.ellipse(-4, 4, 9, 5, -0.3, 0, Math.PI*2); ctx.fill();

  // Eye white
  ctx.fillStyle = '#fff';
  ctx.beginPath(); ctx.ellipse(7, -5, 6, 6, 0, 0, Math.PI*2); ctx.fill();

  // Pupil
  ctx.fillStyle = '#222';
  ctx.beginPath(); ctx.ellipse(9, -5, 3, 3, 0, 0, Math.PI*2); ctx.fill();

  // Beak
  ctx.fillStyle = '#e65100';
  ctx.beginPath();
  ctx.moveTo(13, -1); ctx.lineTo(20, 1); ctx.lineTo(13, 4);
  ctx.closePath(); ctx.fill();

  ctx.restore();
}

function drawOverlay() {
  if (state === 'dead') {
    ctx.fillStyle = 'rgba(0,0,0,0.45)';
    ctx.fillRect(0,0,W,H);
    ctx.fillStyle = '#f7c948';
    ctx.font = 'bold 40px Segoe UI';
    ctx.textAlign = 'center';
    ctx.fillText('Game Over!', W/2, H/2 - 30);
    ctx.fillStyle = '#fff';
    ctx.font = '20px Segoe UI';
    ctx.fillText(`Score: ${score}`, W/2, H/2 + 10);
    ctx.fillStyle = '#aaa';
    ctx.font = '15px Segoe UI';
    ctx.fillText('Click or SPACE to restart', W/2, H/2 + 44);
    ctx.textAlign = 'left';
  }
  if (state === 'idle') {
    ctx.fillStyle = 'rgba(0,0,0,0.35)';
    ctx.fillRect(0,0,W,H);
    ctx.fillStyle = '#f7c948';
    ctx.font = 'bold 36px Segoe UI';
    ctx.textAlign = 'center';
    ctx.fillText('Flappy Bird', W/2, H/2 - 20);
    ctx.fillStyle = '#ddd';
    ctx.font = '17px Segoe UI';
    ctx.fillText('Click or SPACE to flap', W/2, H/2 + 20);
    ctx.textAlign = 'left';
  }
}

// ── Game loop ─────────────────────────────────────────────
function update() {
  if (state !== 'playing') return;

  frame++;

  // Spawn
  if (frame % PIPE_FREQ === 0) spawnPipe();

  // Bird physics
  bird.vy += GRAVITY;
  bird.y  += bird.vy;

  // Floor / ceiling
  if (bird.y + bird.r >= H - 50) { bird.y = H - 50 - bird.r; die(); }
  if (bird.y - bird.r <= 0)      { bird.y = bird.r; bird.vy = 0; }

  // Pipes
  pipes.forEach(p => {
    p.x -= PIPE_SPEED;
    if (birdHitsPipe(p)) die();
    if (!p.scored && p.x + PIPE_W < bird.x) {
      p.scored = true;
      score++;
      scoreEl.textContent = score;
      if (score > (best || 0)) { best = score; bestEl.textContent = best; }
    }
  });
  pipes = pipes.filter(p => p.x + PIPE_W > 0);
}

function die() {
  state = 'dead';
  bird.vy = FLAP * 0.6;
  msgEl.textContent = '';
}

function draw() {
  drawBg();
  pipes.forEach(drawPipe);
  drawBird();
  drawOverlay();
}

function loop() {
  update();
  draw();
  requestAnimationFrame(loop);
}

// ── Start ─────────────────────────────────────────────────
init();
loop();
</script>
</body>
</html>
